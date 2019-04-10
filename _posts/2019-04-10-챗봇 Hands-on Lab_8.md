---
layout: post
title:  챗봇 Hands-on Lab (9) - Instant App 구현 및 챗봇 연계
categories: chatbot
tags: [chatbot, 오라클 챗봇, Hands-on-Lab,Instant App]
---

챗봇에서 text 기반으로 대화를 주고 받다가, 다수의 정보의 입력이 필요하거나 다양한 포맷의 응답을 제공해 줄 필요가 있을때는 웹 애플리케이션에서 익숙하게 사용하는 Form 형태의 인터페이스가 필요할 경우가 있습니다. 이렇게 필요시에만 앱의 형태 처럼 접근하게 해주고 다시 대화형 인터페이스로 돌아갈 수 있게 해주는 형태를 **Instant App**이란 용어로 부르고 있는데, 이번 Lab에서는 Instant App을 만들고 챗봇에서 연결하는 방법에 대해서 실습할 것입니다. 

# Instant App 생성

## 미리 만들어둔 Instant App 파일 다운 받기

챗봇의 **Instant App Builder**에서 새롭게 Instant App을 처음부터 생성할 수 있으나, 이 Lab에서는 기존에 생성된 Instant App을 Import 받아서 항목을 수정하는 형태로 진행할 예정입니다.

다음에서 설문을 수행하는 Instant App인 Survey 앱 (survey.json) 파일을 다운 받습니다.

- [Survey Instant App 다운로드](https://raw.githubusercontent.com/mee-nam-lee/chatbot_lecture/master/labfiles/instantapp/survey.json)


## Instant App Builder에서 Import 하기

챗봇 콘솔로 이동합니다. 상단 메뉴의 **Instant Apps**를 클릭하여 Instant App 빌더 화명으로 이동합니다.

![](/assets/images/chatbot_lecture/instantapp/01_instant.png)

**Add Instant App**을 클릭하여 다운받아둔 survey.json을 import 합니다.

![](/assets/images/chatbot_lecture/instantapp/02_import.png)

**Schema File** 항목에 survey.json을 끌어다 놓으면 Import가 됩니다.

![](/assets/images/chatbot_lecture/instantapp/03_upload.png)

Import 후에는 해당 Instant App의 Edit 화면으로 바로 이동합니다.
좌측 메뉴에서 **INSTANT APP INFO**를 클릭하여 **API ID**를 확인 합니다. 이 API ID는 챗봇에서 Instant App을 호출할 때 사용됩니다.
**Save** 버튼을 클릭하여 저장합니다.

![](/assets/images/chatbot_lecture/instantapp/04_survey.png)

이 앱이 어떤 기능이 있는지 챗봇과 연계 전에 **Instant App Builder**에서 바로 테스트 해 볼 수 있습니다.
우측 화면의 **Test** 탭을 클릭하고 모바일 화면 하단의 **Start as Recipient**를 클릭합니다.

![](/assets/images/chatbot_lecture/instantapp/05_test.png)

팝업으로 나타나는 **survey: Parameters** 부분에서는 **OK**를 눌러주면 됩니다.
서비스 만족도의 별점에 점수에 따라 다른 질문들이 나타납니다. 별점의 점수를 바꿔보세요.

![](/assets/images/chatbot_lecture/instantapp/06_test1.png)

화면이 어떻게 바뀌는지 확인한 후 화면에 보이는 입력값을을 넣어주고 **설문 완료** 버튼을 클릭합니다.

![](/assets/images/chatbot_lecture/instantapp/07_test2.png)

설문이 완료되고 Confirmation이 보여집니다. Instant App의 Test를 종료하기 위해 **Stop** 버튼을 클릭합니다.

![](/assets/images/chatbot_lecture/instantapp/08_test3.png)

어떤 Component로 이루어진 Instant App인지 좌측의 Layout을 하나씩 살펴보세요.

![](/assets/images/chatbot_lecture/instantapp/09_layout.png)

**PANE_2**의 **Add Element**를 클릭하여 새로운 Element 하나를 더 추가해 보겠습니다.
Content의 **Social Buttons**를 클릭해서 추가합니다.

![](/assets/images/chatbot_lecture/instantapp/10_edit.png)

**PREVIEW**에서 추가된 Element를 바로 확인할 수 있습니다. **Save**를 클릭하여 변경사항을 저장합니다.

![](/assets/images/chatbot_lecture/instantapp/11_social.png)

# 챗봇에 연결하기

이제 Survey Instant App을 챗봇에서 호출해 보도록 하겠습니다.
Flow에서 Intant App을 호출아는 Built-in Component를 이용해야 합니다. 

이 설문 앱을 피자 주문이 완료된 후에 만족도 평가를 위해서 호출하는 형태로 챗봇 Flow를 변경해 보도록 하겠습니다.

Flow Edit 화면으로 들어가서 다음 부분을 수정, 추가해 줍니다.

```yaml
 Confirmation:
    component: "System.CommonResponse"
    properties:
      keepTurn: true ## 추가
      metadata:
        responseItems:
        - text: "주문해 주셔서 감사합니다. 주문하신 ${pizzaSize} ${orderedPizza} 피자가 30분 내로 배달될 예정입니다."
          type: "text"
          name: "conf"
          separateBubbles: true
        - type: "attachment"
          attachmentType: "image"
          name: "image"
          attachmentUrl: "${orderedPizzaImage}"
      processUserMessage: false
    transitions: {} ## 변경

## 아래는 추가
  askSurvey:
    component: "System.List"
    properties: 
      prompt: "서비스 만족도 진행 중입니다. 참여하시겠습니까?"
      options: 
      - label: "네"
        value: "yes" 
      - label: "아니요"
        value: "no" 
    transitions:
      actions:
        yes: "doSurvey"
        no: "thankyou"      

  thankyou:
    component: "System.Output"
    properties:
      text: "감사합니다."
    transitions:
      return: "done"

  doSurvey:
    component: "System.Interactive"
    properties:
      sourceVariableList:
      variable: "dummy"
      id: "survey2"    ## Instant App의 API ID를 사용합니다.
      title: "설문을 진행합니다."
      linkLabel: "설문으로 바로가기"
      textOnlyResponse: true
    transitions: {}
   
  surveyOutput:
    component: "System.Output"
    properties:
      text: "${dummy.value.result}"
    transitions: 
      return: "done"
```
Flow 수정 후 상단의 **Validate**를 클릭하여 Flow에 오류가 없는지 확인합니다.

![](/assets/images/chatbot_lecture/instantapp/12_flow.png)

# Test
Flow가 수정되었으면 **Skill Tester**에서 테스트를 수행해 봅니다.
피자 주문 프로세스를 완료한 후 설문 진행 여부를 묻는 Flow가 나오게 구성되어 있습니다.
**설문으로 바로가기** 링크를 클릭하면 Instant App으로 이동합니다.

![](/assets/images/chatbot_lecture/instantapp/13_callsurvey.png)


Instant App에서 설문을 완료하고 챗봇으로 돌아옵니다.

브라우저 기반의 챗봇에서 Instant App을 호출하면 별도의 Tab으로 이동하여 Instant App이 실행된 후 다시 챗봇이 수행 중인 Tab으로 돌아와야 합니다.
모바일 앱에서 실행 될 때에는 어떻게 동작하는 모바일 앱에서도 확인 해 보세요.

![](/assets/images/chatbot_lecture/instantapp/20_mobile2.jpeg)

설문 완료 후 앱의 돌아가기를 클릭하면 모바일 챗봇으로 돌아갑니다.
![](/assets/images/chatbot_lecture/instantapp/20_mobile1.jpeg)

**Script** 입력 부분에 다음 3줄을 입력합니다. 
```
cd src
npm install
npm pack
```

이상으로 Custom Component 구현 과정을 완료하였습니다.

# Chatbot-Workshop Lab 
* Lab 개요 : [챗봇 Hands-on Lab (1) - Lab 개요](/chatbot/2019/챗봇-Hands-on-Lab_1/)
* Lab 100 : [챗봇 Hands-on Lab (2) - 금융봇을 이용하여 챗봇 기본 기능 익히기](/chatbot/2019/챗봇-Hands-on-Lab_2/)
* Lab 200 : [챗봇 Hands-on Lab (3) - 피자봇 만들기 ](/chatbot/2019/챗봇-Hands-on-Lab_3/)
* Lab 300 : [챗봇 Hands-on Lab (4) - [채널 연결] Web Chat 연결하기](/chatbot/2019/챗봇-Hands-on-Lab_4/)