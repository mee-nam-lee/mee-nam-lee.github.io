---
layout: post
title:  Digital Assistant(챗봇)와 Content and Experience(CECS) 연동하기
categories: chatbot
tags: [chatbot, 오라클 챗봇, CECS]
---

이 포스트에서는 오라클 챗봇인 **Digital Assistant**와 **Content and Experience Cloud (이후 CECS)**라는 Content 관리 솔루션과 연계하는 방법에 대해서 다룰 것입니다. 

챗봇을 통해서 CECS에 저장되어 있는 컨텐츠를 검색하고 검색된 결과 문서나 폴더의 **Public Link**를 통해서 바로 해당 컨텐츠를 확인할 수 있는 예제를 설명할 것입니다.

이를 위해서는 Chatbot의 커스텀 컴포넌트 작성이 필요한데, 커스텀 컴포넌트 작성과 관련하여서는 이전 기고에서 몇 차례 다루었기 때문에 여기에서는 CECS 연계 코드 부분만 설명하도록 하겠습니다.

 * [챗봇 커스텀 컨포넌트 작성 및 Compute CS에서 커스텀 컴포넌트 구동하기](/chatbot/2019/chatbot_adw/)
 * [Embedded Container에 커스텀 컴포넌트 올리기](/chatbot/2019/챗봇-Embedded-Container/)

## 사전 준비 사항 
아래 서비스가 미리 생성되어 있어야 합니다. 

 * Oracle Digital Assistant (ODA)
 * Oracle Content and Experience Cloud (CECS)

## Custom Component 
CECS 연동을 위해서는 CECS에서 제공하는 REST API를 사용할 것 입니다. 

* [CECS REST API 참고하기](https://docs.oracle.com/en/cloud/paas/content-cloud/rest-api-documents/rest-endpoints.html)

이 포스트에서는 컨텐츠 검색과 컨텐츠 접근을 위해서 Public Link를 얻어오는 기능을 사용하기 때문에 다음 두개의 REST API를 사용합니다.

* [Search Folders or Files](https://docs.oracle.com/en/cloud/paas/content-cloud/rest-api-documents/op-documents-api-1.2-folders-search-items-get.html)
* [Publiclinks](https://docs.oracle.com/en/cloud/paas/content-cloud/rest-api-documents/api-publiclinks.html)

예제는 챗봇으로 부터 컨텐츠 검색을 위한 **keyword**를 입력받고 keyword에 해당하는 컨텐츠를 검색한 후, 해당 컨텐츠의 publiclink 정보를 가저와서 챗봇에 보내준 후 챗봇의 url 버튼 기능을 이용하여 해당 컨텐츠로 이동하는 예제 입니다.

작성된 커스텀 컴포넌트 Code Snippet은 다음과 같습니다.

### searchcecs.js
cecs의 REST API를 호출하고 챗봇에 보여줄 메시지를 생성하는 Custom Component 로직입니다.
이 코드에서 **publiclink**에 해당하는 부분을 각자 환경에 맞게 수정합니다.

```js
// 생략
const publink = "https://[CECS DOMAIN]/documents/link/"
const searchapi = "/folders/search/items";
const publicfilelinkapi ="/publiclinks/file/";
const publicfolderlinkapi ="/publiclinks/folder/";
// 생략
```

* [전체 searchcecs.js 코드](https://github.com/mee-nam-lee/chatbot/blob/master/bot-start/components/searchcecs.js)

### util.js
REST API를 실제로 호출하는 코드 입니다. **baseForAPI**와 **auth** 부분의 각자 환경에 맞게 수정합니다.

```js
// 생략
var baseForAPI = "https://[CECS CLOUD DOMAIN]/documents/api/1.2";
var auth = 'Basic ' + Buffer.from('[USERID]:[PASSWORD]').toString('base64');
// 생략
```

* [전체 util.js 코드](https://github.com/mee-nam-lee/chatbot/blob/master/bot-start/components/utils/util.js)

## Bot에서 Custom 컴포넌트 연결
Custom 컴포넌트 연계 방법은 이전 포스트를 참고하시기 바랍니다. 컴포넌트가 연결되고 나면 아래와 같이 서비스가 보이게 됩니다.

![Alt text](/assets/images/chatbot_cecs/cecs_service.png)

## Bot Dialogue Flow
Flow는 다음과 같이 구성합니다. **keyword**를 입력 받아서 custom component의 **properties**로 사용합니다.

```yaml
states:
  askKeyword:
    component: "System.Text"
    properties:
      prompt: "검색하실 키워드를 입력해 주세요"
      variable: "keyword"
    transitions: {}
    
  searchCECS:
    component: "searchcecs"
    properties: 
      keyword: "${keyword}"
    transitions:
      return: "done"   
```

## Test 
Web Chat 채널을 통해 테스트 하게 되면 다음처럼 보이게 됩니다. 검색된 결과의 **링크 열기**를 클릭하면 해당 문서 보기로 이동하게 됩니다.

![Alt text](/assets/images/chatbot_cecs/chatbot_cecs_result.png)

챗봇의 내장 Test UI에서는 다음처럼 보여집니다. 
![Alt text](/assets/images/chatbot_cecs/testui_result.png)


모두 완료되었습니다. 

## 참고 자료 
- [CECS REST API 참고하기](https://docs.oracle.com/en/cloud/paas/content-cloud/rest-api-documents/rest-endpoints.html)

Oracle 챗봇 컴포넌트 작성을 위한 자세한 SDK 가이드는 다음을 참고하세요

- [Oracle Bots Node.js SDK](https://github.com/oracle/bots-node-sdk/)




