---
layout: post
title:  챗봇 Hands-on Lab (6) - Insight(분석) 기능 사용하기
categories: chatbot
tags: [chatbot, 오라클 챗봇, Hands-on-Lab, Insight]
---

이 Lab에서는 지금까지 만든 챗봇에 분석 기능을 추가해 볼 것 입니다.


# Insights (분석 기능 활성화)
오라클 챗봇에는 내장된 분석 기능이 있습니다. 이 기능을 사용하려면 **Enable Insight**를 선택하여 명시적으로 활성화 해주어야 합니다.

설정 메뉴를 선택하여 아래와 같이 **Enable Insights**를 활성화 해 줍니다.

![](/assets/images/chatbot_lecture/insights/01_enable_insight.png)

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

**Retainer** 탭으로 이동합니다. 이 메뉴에서는 Intent Resolution에 대한 결과를 조건절을 이용해서 검색해 볼 수 있습니다. **unresolvedIntent**가 있나 확인합니다. 해당 조건에 맞는 내용이 없다면 채팅창을 이용하여 대화를 더 시도해 봅니다.

![](/assets/images/chatbot_lecture/insights/07_insights.png)

위 예시에서는 **안녕**이라는 Utterance가 unresolvedIntent에 해당된 것을 볼 수 있습니다. 이 Utterance를 Intent의 Sample Data로 사용하고 싶다면 바로 적용할 Intent를 선택한 후 **Add Example**을 수행하면 됩니다.

![](/assets/images/chatbot_lecture/insights/08_insights.png)

채팅창에서 다시 **안녕** 이라고 대화를 해보면 이제 메뉴가 보여지게 될 것입니다.

다음과 같이 **Win Margin**으로도 검색해 봅니다. Win Margin이 60% 이하 인것을 조회합니다.
해당 검색 결과에서 Intent에 새롭게 추가하고자 하는 Utterance가 있다면 위의 방법 처럼 **Add Example**을 하게 되면 바로 적용이 됩니다.

![](/assets/images/chatbot_lecture/insights/09_insights.png)

# Intent Quality Report

챗봇의 Intent를 학습 시키다 보면 비슷한 Sample Data들이 서로 다른 Intent들에 포함되어 Intent간의 구분/구별을 명확하게 하지 못하게 되는 경우가 있습니다. 이 때문에 챗봇의 정확도가 감소하게 될 수 있는데, 이 Intent의 품질을 검사해 주는 기능을 활용하면 챗봇의 정확도를 개선하는데 도움이 됩니다.


비슷 간의 명확한 구별/구분 (distinct)

과정을 완료하였습니다.

# Chatbot-Workshop Lab 
* Lab 개요 : [챗봇 Hands-on Lab (1) - Lab 개요](/chatbot/2019/챗봇-Hands-on-Lab_1/)
* Lab 100 : [챗봇 Hands-on Lab (2) - 금융봇을 이용하여 챗봇 기본 기능 익히기](/chatbot/2019/챗봇-Hands-on-Lab_2/)
* Lab 200 : [챗봇 Hands-on Lab (3) - 피자봇 만들기 ](/chatbot/2019/챗봇-Hands-on-Lab_3/)
* Lab 300 : [챗봇 Hands-on Lab (4) - [채널 연결] Web Chat 연결하기](/chatbot/2019/챗봇-Hands-on-Lab_4/)