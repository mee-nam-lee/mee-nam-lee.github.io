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


이상으로 과정을 완료하였습니다.

# Chatbot-Workshop Lab 
* Lab 개요 : [챗봇 Hands-on Lab (1) - Lab 개요](/chatbot/2019/챗봇-Hands-on-Lab_1/)
* Lab 100 : [챗봇 Hands-on Lab (2) - 금융봇을 이용하여 챗봇 기본 기능 익히기](/chatbot/2019/챗봇-Hands-on-Lab_2/)
* Lab 200 : [챗봇 Hands-on Lab (3) - 피자봇 만들기 ](/chatbot/2019/챗봇-Hands-on-Lab_3/)
* Lab 300 : [챗봇 Hands-on Lab (4) - [채널 연결] Web Chat 연결하기](/chatbot/2019/챗봇-Hands-on-Lab_4/)