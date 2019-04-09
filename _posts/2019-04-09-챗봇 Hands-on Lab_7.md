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

10_createjob.png)

활성화 후 바로 Insights 메뉴로 이동하면 분석 내용을 볼 수 있습니다. Insights가 활성화 되고 나서 수행된 대화가 없기 때문에 아직은 수집된 내용이 없을 것입니다.

이전 Lab에서 연결한 채널을 (Web / Mobile) 이용하여 챗봇과 수차례 대화를 진행합니다.

Insights 내용을 Refresh 하면 분석된 내용이 보이기 시작합니다.

![](/assets/images/chatbot_lecture/insights/02_insights.png)

![](/assets/images/chatbot_lecture/insights/03_insights.png)

**Intents** 탭으로 이동하면 어떤 Intent가 어떤 경로로 수행되었는지 볼 수 있습니다.
Intent List에 구현된 Intent들이 다 나오지 않는다면, **해당 Intent에 해당하는 Utterance를 이용**하여 대화를 더 시도해 봅니다. 

![](/assets/images/chatbot_lecture/insights/04_insights.png)

**Conversation** 탭에서 그동안 수행되었던 대화 내용을 확인 할 수 있습니다. 테스트한 대화 내용이 보여지는지 확인 합니다.

![](/assets/images/chatbot_lecture/insights/05_insights.png)

**View Conversation**을 클릭하여 대화 내용을 채팅창에서 확인할 수도 있습니다. 
> 이 창에서는 실제 채팅창에서 보여지는 Layout으로 보여지는 것은 아니기 때문에 진행된 대화 흐름을 이해하는 용도라고 이해하면 됩니다.

![](/assets/images/chatbot_lecture/insights/06_insights.png)

**Retrainer** 탭으로 이동합니다. 이 메뉴에서는 Intent Resolution에 대한 결과를 조건절을 이용해서 검색해 볼 수 있습니다. **unresolvedIntent**가 있나 확인합니다. 해당 조건에 맞는 내용이 없다면 채팅창을 이용하여 대화를 더 시도해 봅니다.

![](/assets/images/chatbot_lecture/insights/07_insights.png)

위 예시에서는 **안녕**이라는 Utterance가 unresolvedIntent에 해당된 것을 볼 수 있습니다. 이 Utterance를 Intent의 Sample Data로 사용하고 싶다면 바로 적용할 Intent를 선택한 후 **Add Example**을 수행하면 됩니다.

![](/assets/images/chatbot_lecture/insights/08_insights.png)

채팅창에서 다시 **안녕** 이라고 대화를 해보면 이제 메뉴가 보여지게 될 것입니다.

다음과 같이 **Win Margin**으로도 검색해 봅니다. Win Margin이 60% 이하 인것을 조회합니다.
해당 검색 결과에서 Intent에 새롭게 추가하고자 하는 Utterance가 있다면 위의 방법 처럼 **Add Example**을 하게 되면 바로 적용이 됩니다.

![](/assets/images/chatbot_lecture/insights/09_insights.png)

# Intent Quality Report

챗봇의 Intent를 학습 시키다 보면 비슷한 Sample Data들이 서로 다른 Intent들에 포함되어 Intent간의 구분/구별을 명확하게 하지 못하게 되는 경우가 있습니다. 이 때문에 챗봇의 정확도가 감소하게 될 수 있는데, 이 Intent의 품질을 검사해 주는 기능을 활용하면 챗봇의 정확도를 개선하는데 도움이 됩니다.


**Quality** 메뉴를 클릭하여 Report 생성 화면으로 이동합니다. **Run Report** 버튼을 클릭하여 Report 생성 작업을 시작 시킵니다. 

![](/assets/images/chatbot_lecture/insights/10_runreport.png)

잠시 후 Report가 나타나면 다음과 같이 Conflict이 있을 수 있는 Intent들을 보여줍니다. 빨간색 경고에 해당하는 Intent를 살펴봐야 합니다. 아래 결과는 **"주문"** / **"주문 취소"**가 유사한 Utterance이지만 서로 다른 Intent에 속해 있어 Accuracy가 떨어질 수 있음을 보여주고 있습니다.

![](/assets/images/chatbot_lecture/insights/11_reportresult.png)

이상으로 과정을 완료하였습니다.

# Chatbot-Workshop Lab 
* Lab 개요 : [챗봇 Hands-on Lab (1) - Lab 개요](/chatbot/2019/챗봇-Hands-on-Lab_1/)
* Lab 100 : [챗봇 Hands-on Lab (2) - 금융봇을 이용하여 챗봇 기본 기능 익히기](/chatbot/2019/챗봇-Hands-on-Lab_2/)
* Lab 200 : [챗봇 Hands-on Lab (3) - 피자봇 만들기 ](/chatbot/2019/챗봇-Hands-on-Lab_3/)
* Lab 300 : [챗봇 Hands-on Lab (4) - [채널 연결] Web Chat 연결하기](/chatbot/2019/챗봇-Hands-on-Lab_4/)