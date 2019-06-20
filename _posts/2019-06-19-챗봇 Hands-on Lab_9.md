---
layout: post
title:  챗봇 Hands-on Lab (9) - WebView 구현 및 챗봇 연계
categories: Chatbot
tags: [chatbot, 오라클 챗봇, Hands-on-Lab, WebView, 웹 시스템 연계]
---

챗봇에서 text 기반으로 대화를 주고 받다가, 다수의 정보의 입력이 필요하거나 다양한 포맷의 응답을 제공해 줄 필요가 있을때는 웹 애플리케이션에서 익숙하게 사용하는 Form 형태의 인터페이스가 필요할 경우가 있고, 이를 위해서 이전 Lab에서 **Instant App**에 대해서 살펴보았습니다. 이번 Lab에서는 비슷한 역할을 수행하기도 하면서, **기존 웹 시스템과의 연계** 기능으로도 사용할 수 있는 **WebView** 컴포넌트 기능에 대해 실습해 보겠습니다. 

# WebView 연계 아키텍쳐
WebView는 Chatbot에서 Web Url을 호출하는 것입니다. 단순 호출이라면 URL Link와 다를 것이 없겠지만, **Chatbot과 Input, Output을 주고 받을 수 있다는 것**이 차이라고 할 수 있겠습니다.

호출 관계는 다음과 같습니다.

![](/assets/images/chatbot_lecture/webview/00_call_flow.png)

(1)번 POST 요청을 받은 서비스 (Intermediary Service)는 실제로 브라우저에서 서비스될 서비스의 URL을 생성하여 아래와 같은 포맷으로 리턴합니다.
Response를 받은 Bot이 제공된 **webview.url**로 요청을 GET으로 보내게 됩니다.

## POST 요청의 Response 예시

```json
{
    "webview.url": "http://<webview-url>/webview/booking?callbackUrl=http://<bot-url>/connectors/v1/callback?state=cb5443 . .. 2c&param1=samsung"
}
```


# 호출될 웹 애플리케이션 준비 - 옵션 (1) 직접 빌드

챗봇을 통해서 호출될 웹 애플리케이션이 필요합니다. 여기에서는 Laptop을 검색하여 리스트를 보여주는 간단한 애플리케이션을 가지고 연습합니다.

웹 애플리케이션은 챗봇으로 부터 검색조건(제조사)을 전달받고 해당 조건에 맞는 리스트를 조회하여 보여줍니다. 
해당 리스트로 부터 Laptop 항목을 선택하면 선택 내용이 챗봇으로 전달되어 채팅창에 보여지는 간단한 애플리케이션 입니다.

WebView 애플리케이션 소스를 다음에서 다운 받습니다.

```
git clone https://github.com/mee-nam-lee/chatbot.git
cd bot_webview 
node index.js
```
기본 포트는 3000으로 동작합니다.

- 테스트 URL : http://localhost:3000/webview/booking?callbackUrl=a

아래와 같은 페이지가 보여질 것입니다.

![](/assets/images/chatbot_lecture/webview/04_webview_test.png)

>
> Laptop 정보는 danawa 사이트의 정보를 사용한 것으로 이미지와 상세 보기의 링크는 danawa 사이트를 참조하게 되어있습니다.
>

> 챗봇에서 Local PC 환경에서 동작하는 웹 애플리케이션을 호출하기 위해서는 ngrok 설정이 필요합니다.
> ngrok 설정 후 챗봇 flow에서 WebView를 호출하기 위한 URL을 ngrok url로 변경해 줘야 합니다.
> 여기에서는 이 과정은 설명하지 않습니다. 

# 호출될 웹 애플리케이션 준비 - 옵션 (2) 미리 준비된 웹 애플리케이션에 연결하기

실습을 위해서 위에서 제공한 웹 애플리케이션 소스가 서비스되고 있는 환경의 URL을 제공할 것입니다. 해당 URL이 동작하는지 테스트 후에 **챗봇 flow에서 WebView를 호출하기 위한 URL만 수정**해 주면 됩니다.

- 테스트 URL : http://[제공된 서비스 IP]/webview/booking?callbackUrl=a

# 챗봇 FLOW

WebView를 호출하는 Flow는 다음과 같습니다.

```yaml
metadata:
  platformVersion: "1.0"
main: true
name: webview
context:
  variables:
    param1: "string"
    selectedlaptops: "string"

states: 
  text:
    component: "System.Text"
    properties:
      prompt: "검색하고 싶은 브랜드를 입력하세요 (예 : LG, 삼성, HP, ASUS etc)"
      variable: "param1"
      maxPrompts: 3
    transitions: {}           
      
  callWebview:
    component: "System.Webview"
    properties:
      sourceVariableList: "param1"
      variable: "selectedlaptops"
      prompt: "검색 결과 확인을 위해 '검색 결과창 열기'를 클릭하세요."
      ### 웹 애플리케이션이 서비스되고 있는 실제 URL로 변경해 주세요 ###
      webAppUrl: "http://ea36cf28.ngrok.io/webview" 
      linkLabel: "검색 결과창 열기"
      cancelLabel: "Cancel"
    transitions: {}  
    
  showMore: 
    component: "System.CommonResponse"
    properties:
      metadata:
        responseItems:
        - type: "text"
          text: "선택된 결과 입니다."
          separateBubbles: true
          name: "Selected Results"
        - type: "cards"
          cardLayout: "horizontal"
          name: "Laptops"
          cards:
          - title: "${selectedlaptops.value.maker}"
            description: "${selectedlaptops.value.model} ${selectedlaptops.value.spec} ${selectedlaptops.value.weight} ${selectedlaptops.value.price}"
            imageUrl: "${selectedlaptops.value.image}"
            name: "LaptopsCard"
            actions:
            - label: "상세 보기 페이지 이동"
              type: "url"
              payload:
                variables:
                  url: "${selectedlaptops.value.link}"                
      processUserMessage: true
    transitions:
      return: "done"  
```

# 테스트

챗봇 Test UI를 열어 다음과 깉이 테스트 합니다. 
**검색 결과창 열기**를 클릭하면 새로운 브라우저 창에서 WebView 애플리케이션이 열리게 됩니다.
항목을 하나 선택하고 다시 챗봇 창으로 돌아옵니다.

![](/assets/images/chatbot_lecture/webview/01_bot_testui.png)

**검색 결과창 열기**를 클릭했을 때 보이는 웹 애플리케이션은 다음처럼 보이게 됩니다.

![](/assets/images/chatbot_lecture/webview/02_webview.png)

연결 관계는 다음과 같습니다.

![](/assets/images/chatbot_lecture/webview/03_test.png)

이상으로 WebView 구현 과정을 완료하였습니다.

# Chatbot-Workshop Lab 
* [챗봇 Hands-on Lab (1) - Lab 개요](/chatbot/2019/챗봇-Hands-on-Lab_1/)
* [챗봇 Hands-on Lab (2) - 금융봇을 이용하여 챗봇 기본 기능 익히기](/chatbot/2019/챗봇-Hands-on-Lab_2/)
* [챗봇 Hands-on Lab (3) - 피자봇 만들기 ](/chatbot/2019/챗봇-Hands-on-Lab_3/)
* [챗봇 Hands-on Lab (4) - [채널 연결] Web Chat 연결하기](/chatbot/2019/챗봇-Hands-on-Lab_4/)
* [챗봇 Hands-on Lab (5) - [채널 연결] Mobile 앱 연결하기](/chatbot/2019/챗봇-Hands-on-Lab_5/)
* [챗봇 Hands-on Lab (6) - Insights(분석) 기능 사용하기](/chatbot/2019/챗봇-Hands-on-Lab_6/)
* [챗봇 Hands-on Lab (7) - Custom Component 구현하기](/chatbot/2019/챗봇-Hands-on-Lab_7/)
* [챗봇 Hands-on Lab (8) - Instant App 구현 및 챗봇 연계](/chatbot/2019/챗봇-Hands-on-Lab_8/)
* [챗봇 Hands-on Lab (9) - WebView 구현 및 챗봇 연계](/chatbot/2019/챗봇-Hands-on-Lab_9/)