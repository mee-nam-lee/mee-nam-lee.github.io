---
layout: post
title:  챗봇 Hands-on Lab (7) - Custom Component 구현하기
categories: chatbot
tags: [chatbot, 오라클 챗봇, Hands-on-Lab,Custom Component]
---

커스텀 비즈니스 로직을 챗봇에 추가하기 위해서는 Node.js 기반의 챗봇 SDK를 사용하여 Custom 로직을 구현하여야 합니다. 이 Lab에서는 지금까지 만든 챗봇에 Custom Component를 구현하여 연결해 볼 것입니다. 

# Prerequisite
기본적으로 커스텀 코드 작성을 위해서 각자의 로컬 환경에 개발툴과 Node.js, 챗봇 SDK를 설치하여 구현하여야 하지만 이 실습 과정에서는 개별 로컬 개발 환경 설정 과정을 생략하기 위해여 Cloud 기반의 소스 관리 및 빌드 기능을 이용하여 코드를 수정해 보는 과정으로 진행하도록 하겠습니다.
이를 위해서 **Oracle Developer Cloud**를 이용할 것이고, Developer Cloud 서비스 계정과 접속 정보가 제공될 것입니다.

# Developer Cloud 접속 / Project 생성

제공된 접속 정보와 계정 정보로 Developer Cloud에 접속합니다.

![](/assets/images/chatbot_lecture/component/01_devcs_login.png)

**Create** 버튼을 클릭하여 커스텀 코드를 관리하고 빌드할 **Project**를 각자 하나씩 생성할 것입니다.

![](/assets/images/chatbot_lecture/component/02_org.png)

각자에게 부여된 **Project 명**을 이용하여 Project를 생성합니다.

- Project Name : chatbot_{SEQ}

**Next**를 클릭합니다.

![](/assets/images/chatbot_lecture/component/03_project1.png)

**Initial Repository** 옵션을 선택하고 **Next**를 클릭합니다.

![](/assets/images/chatbot_lecture/component/04_project2.png)

**Import existing repository** 옵션을 선택하고 text box에 다음 repository url을 입력합니다.

- repository url : https://github.com/mee-nam-lee/chatbot_component.git

입력이 완료되었으면 **Finish**를 클릭합니다.

![](/assets/images/chatbot_lecture/component/05_project3.png)

다음과 같이 Project에 필요한 컴포넌트들이 Provision 되는 것이 보일 것입니다.

![](/assets/images/chatbot_lecture/component/06_project4.png)

Project 생성이 완료되면 자동으로 Project Home 화면으로 이동됩니다.

![](/assets/images/chatbot_lecture/component/07_project5.png)

# Custom Code 이해하기

이 Project는 챗봇의 Custom Code를 외부 Repository로부터 가져왔기 때문에 **Git** 메뉴로 이동하면 해당 코드들을 볼 수 있습니다.

![](/assets/images/chatbot_lecture/component/08_git.png)

Custom Component의 Main Component 코드는 다음 경로에 있습니다. 아래 경로로 이동해 봅니다.

![](/assets/images/chatbot_lecture/component/09_src.png)

**ListStores.js**와 **StoreService.js**가 custom component에서 사용하는 코드 입니다.

## ListStore.js
검색 조건으로 입력된 **location** 파라미터를 store 정보의 address와 비교하여 검색 조건을 주소에 포함하고 있는 store 정보만을 Card Layout 형태로 만들어서 Reply 해주는 컴포넌트 입니다.

## StoreService.js
stores.json에 미리 저장되어 있는 store 정보들 중에서 검색 조건 (location) 기반으로 필터하여 해당 조건에 맞는 store 정보만 리턴하는 컴포넌트 입니다.

# Build 구성하기
**Builds** 메뉴로 들어가서 **Create Job**을 클릭하여 Build Job을 하나 생성합니다.

![](/assets/images/chatbot_lecture/component/10_createjob.png)

다음과 같이 입력하고 **Software Template** 항목에서 미리 구성되어있는 **node.js** 탬플릿을 선택한 후 **Create Job**을 클릭합니다.

![](/assets/images/chatbot_lecture/component/11_createjob.png)

Job이 생성되고 나면 Job Configuration 화면으로 이동됩니다. **Source Control** 탭에서 빌드 대상의 Source가 들어 었는 Repository를 지정하기 위해서 우측의 **Add Source Control**을 클릭하여 List되는 **Git**을 선택해 줍니다.

![](/assets/images/chatbot_lecture/component/12_addsource.png)

**Repository** 항목에서 이미 구성 되어있는 chatbot{SEQ}.git repository를 선택합니다.

![](/assets/images/chatbot_lecture/component/13_selectrepo.png)

**Steps** 탭으로 이동하여 Build Step을 추가합니다. node.js로 개발된 소스에 필요한 package를을 설치하고 archiving하기 위해서 **Unix Shell** Build Executor를 선택하도록 하겠습니다.

![](/assets/images/chatbot_lecture/component/14_addstep.png)

**Script** 입력 부분에 다음 3줄을 입력합니다. 
```
cd src
npm install
npm pack
```
![](/assets/images/chatbot_lecture/component/15_npm.png)

빌드된 Artifact를 저장하기 위한 post build 작업이 필요하기 때문에 **Post Build** 탭으로 이동합니다.
**Add Post Build Action** 메뉴에서 **Artifact Archiver**를 선택합니다.

![](/assets/images/chatbot_lecture/component/16_postbuild.png)

Archive할 파일을 다음과 같이 입력합니다.

- Files to archive : src/*.tgz

**Save**를 클릭하여 구성된 내용을 저장합니다.

![](/assets/images/chatbot_lecture/component/17_tgz.png)

상단의 **Build Now**를 클릭하면 구성된 내용으로 Build 작업이 시작됩니다.

![](/assets/images/chatbot_lecture/component/18_buildnow.png)

Build가 큐잉되고 실제 Build가 수행될 Runtime VM 환경이 준비되는 것을 기다리고 있습니다. VM이 할당되면 Build가 시작 될 것입니다.

![](/assets/images/chatbot_lecture/component/19_queue.png)

빌드가 완료되면 각 빌드 번호별 상태를 아이콘으로 표시해 줍니다. 초록색 아이콘이 보이면 빌드가 정상적으로 완료된 것입니다.

![](/assets/images/chatbot_lecture/component/20_ builddone.png)

**Actions**의 아이콘을 클릭하여 빌드 상세 로그를 아리처럼 볼 수 있습니다. 빌드 로그에서도 빌드 과정이 잘 수행되었음을 확인할 수 있습니다.

![](/assets/images/chatbot_lecture/component/21_buildlog.png)

빌드된 Artifact는 **Artifact** 메뉴클 클릭하여 확인할 수 있습니다. 아이콘을 클릭하여 들어갑니다.

![](/assets/images/chatbot_lecture/component/22_artifact.png)

**src** 폴더 모양의 아이콘을 클릭하면 component 압축 파일이 보일 것입니다. 이 파일은 Chatbot에 등록하여 서비스될 파일이기 때문에 파일명을 클릭하여 다운로드 받습니다.

![](/assets/images/chatbot_lecture/component/23_artifact_tgz.png)

다음처럼 다운된 파일을 잘 보관해 둡니다.

![](/assets/images/chatbot_lecture/component/24_download.png)

# 챗봇에서 Custom Component 등록하기

이제 개발된 Custom Component를 챗봇에서 사용하기 위하여 Chatbot 화면으로 이동해 옵니다.

각자의 **PizzaBot_{SEQ}**을 엽니다. 좌측 메뉴의 Function 메뉴를 선택하고 **+Service**를 클릭합니다.

![](/assets/images/chatbot_lecture/component/25_addservice.png)

서비스명을 **store**로 입력하고 **Embedded Container** 옵션을 선택합니다.
Package File 부분에 이전 단계에서 다운로드 받아둔 Archive를 업로드 하면 됩니다.

![](/assets/images/chatbot_lecture/component/26_createservice.png)

Upload가 되면 다음과 같이 보입니다. **Create**를 클릭하여 서비스 등록을 완료합니다.

![](/assets/images/chatbot_lecture/component/27_upload.png)

서비스 생성이 되고 나면 서비스 상태가 **Ready**가 되며 등록된 Component가 좌측에 보이게 됩니다. **ListStores**라는 custom component를 챗봇 flow에서 호출하여 사용하면 됩니다.

![](/assets/images/chatbot_lecture/component/28_servicecreated.png)

# 챗봇에 영업점 조회 추가하기 

영업점을 조회하는 **ListStores**라는 custom component를 챗봇에 추가해 보도록 하겠습니다.

## Intent 추가

현재는 영업점 조회를 위한 intent가 등록되어 있지 않습니다. 

Intent Import를 위해 다음 파일을 다운 받습니다.

- [ListStore Intent 다운받기](https://raw.githubusercontent.com/mee-nam-lee/chatbot_lecture/master/labfiles/intent/liststore-Intent.csv)

피자봇으로 이동하여 **Intent** 탭을 선택한 후 **Import Intent**를 선택합니다.

![](/assets/images/chatbot_lecture/liststore/01_importintent.png)

위 단계에서 다운로드 받은 **liststore-Intent.csv** 파일을 Import 합니다.

Intent가 Import 되고 나면 **listStore** intent가 아래처럼 보이게 될 것입니다.

![](/assets/images/chatbot_lecture/liststore/02_imported.png)

우측 상단의 **Train** 메뉴를 클릭하여 새롭게 추가된 Intent를 학습시킵니다.

![](/assets/images/chatbot_lecture/liststore/03_train.png)

**Try It Out**을 클릭하여 새롭게 추가된 Intent가 잘 동작하는 테스트 해 봅니다. 

![](/assets/images/chatbot_lecture/liststore/04_tryitout.png)

위와 같이 Intent Resolution이 잘 테스트 된다면 이제 Flow 수정 단계로 넘어갑니다.

## Flow 수정

Flow Icon을 클릭하여 Flow 수정 화면으로 이동합니다.

![](/assets/images/chatbot_lecture/liststore/05_flow.png)

Intent State에 다음을 추가합니다.

```yml
  Intent:
    component: "System.Intent"
    properties:
      variable: "iresult"
    transitions:
      actions:
        unresolvedIntent: "Unresolved"
        OrderPasta: "OrderPasta"
        ShowMenu: "ShowMenu"
        OrderPizza: "OrderPizza"
        CancelPizza: "CancelPizza"
        listStore: "listStore" # 추가해 주세요
```
![](/assets/images/chatbot_lecture/liststore/06_intentstate.png)

**Confirmation** State 아래에 다음 state를 추가해 줍니다.
```yml
  listStore:
    component: "System.Text"
    properties:
      prompt: "검색하고자하는 영업점의 위치를 입력하세요 예:삼성, 잠실"
      variable: "location"
    transitions: {}
    
  listStoreResult:
    component: "ListStores"
    properties:
      location: "${location}"
    transitions: 
      return: "done"
```
![](/assets/images/chatbot_lecture/liststore/07_liststorestate.png)

우측 상단의 **Validate**를 클릭하여 Flow에 오류가 없나 확인 합니다.

## Test
이제 작성된 로직을 테스트 해 보도록 하겠습니다.
**Skill Tester** 아이콘을 클릭하여 Test 창을 엽니다.
아래와 같이 **listStore** intent에 해당하는 utterance를 입력합니다. 
찾고자 하는 위치 정보를 입력해 주고, 원하는 결과가 나오는지 확인 합니다.

![](/assets/images/chatbot_lecture/liststore/08_Test.png)

# Custom Component 수정 하기

이 단계에서는 Custom Component 로직의 수정이 필요할 경우를 가정하여 로직 변경 후 반영하는 방법에 대해서 실습해 보겠습니다.

영업정 조회 결과로 나오는 링크 버튼의 label을 수정해야 하는 상황이라고 가정합니다.

![](/assets/images/chatbot_lecture/liststore/09_label.png)

## Developer Cloud에서 소스 수정 후 빌드

소스 코드 수정을 위해 Source Repository로 사용하고 있는 Developer Cloud로 이동합니다.
각자 생성한 Project로 이동하여 **Git** 메뉴에서 ListStores.js 파일을 찾아 갑니다.

![](/assets/images/chatbot_lecture/liststore/10_dev_liststore.png)

우측 상단의 **Edit** 아이콘을 클릭하여 Edit Mode로 변경합니다. 아래 Label 내용이 들어있는 부분을 다른 Label로 변경해 봅니다.

![](/assets/images/chatbot_lecture/liststore/11_edit.png)

아래와 같이 코드를 수정하고 우측 상단의 **Commit**을 클릭합니다.

![](/assets/images/chatbot_lecture/liststore/12_codeupdate.png)

Confirmation 화면에서 **Commit**을 클릭합니다.

![](/assets/images/chatbot_lecture/liststore/13_commit_confirm.png)

소스 코드 변경 시 자동으로 Build Job을 수행 시키는 Trigger 설정을 해 두지 않았기 때문에 명시적으로 빌드를 수행시켜줘야 합니다.
**Build** 메뉴로 이동하여 우측의 삼각형 모양의 Build 아이콘을 클릭해 주면 Build Job이 시작됩니다.

![](/assets/images/chatbot_lecture/liststore/14_build.png)

Build Job이 큐잉 되었습니다. 빌드 작업이 완료되기를 기다린 후 위 단계에서 했던 것처럼 빌드 산출물(Artifact)를 다운 받습니다.

![](/assets/images/chatbot_lecture/liststore/15_buildqueue.png)

**Artifacts**로 이동하여 최종 빌드 산출물을 다운 받습니다.

![](/assets/images/chatbot_lecture/liststore/16_artifact.png)

산출물 링크를 클릭하여 다운 받습니다.

![](/assets/images/chatbot_lecture/liststore/17_download.png)

## 챗봇에서 Custom Component 파일 Update

다운 받은 산출물을 반영하기 위하여 챗봇의 Custom Service 등록 화면으로 이동 합니다.

등록되어 있는 **Store** 서비스의 **Package File**을 Update하기 위해 **Change** 링크를 클릭해 줍니다.

![](/assets/images/chatbot_lecture/liststore/18_change.png)

다음처럼 변경된 Package가 반영되고 있는 것을 볼 수 있습니다.

![](/assets/images/chatbot_lecture/liststore/19_package.png)

서비스의 **Status**가 **Ready**가 될 때까지 기다린 후 다시 챗봇을 Test 해 봅니다.
다음처럼 변경된 Label이 보이는지 확인합니다.

![](/assets/images/chatbot_lecture/liststore/20_afterupdate.png)

이상으로 Custom Component 구현 과정을 완료하였습니다.

# Chatbot-Workshop Lab 
* [챗봇 Hands-on Lab (1) - Lab 개요](/chatbot/2019/챗봇-Hands-on-Lab_1/)
* [챗봇 Hands-on Lab (2) - 금융봇을 이용하여 챗봇 기본 기능 익히기](/chatbot/2019/챗봇-Hands-on-Lab_2/)
* [챗봇 Hands-on Lab (3) - 피자봇 만들기 ](/chatbot/2019/챗봇-Hands-on-Lab_3/)
* [챗봇 Hands-on Lab (4) - [채널 연결] Web Chat 연결하기](/chatbot/2019/챗봇-Hands-on-Lab_4/)
* [챗봇 Hands-on Lab (5) - [채널 연결] Mobile 앱 연결하기](/chatbot/2019/챗봇-Hands-on-Lab_5/)
* [챗봇 Hands-on Lab (6) - Insights(분석) 기능 사용하기](/chatbot/2019/챗봇-Hands-on-Lab_6/)
* [챗봇 Hands-on Lab (7) - Custom Component 구현하기](/chatbot/2019/챗봇-Hands-on-Lab_7/)
* [챗봇 Hands-on Lab (8) - Instant App 구현 및 챗봇 연계](/chatbot/2019/챗봇-Hands-on-Lab_8/)