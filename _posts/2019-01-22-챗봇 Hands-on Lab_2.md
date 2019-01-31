---
layout: default
title:  챗봇 Hands-on Lab (2) - 금융봇을 이용하여 챗봇 기본 기능 익히기
date:   2019-01-22 17:50:00
categories: chatbot
---

# 챗봇 Hands-on Lab (2) 
# 금융봇을 이용하여 챗봇 기본 기능 익히기

## 학습 내용 

* 챗봇 둘러보기
* Intent / Dialog 테스트

이 랩에서는 MasterBot이라는 이름의 금융 업무를 수행하는 챗봇을 살펴볼 것입니다. 

**Bot console로 들어가기**
=======
제공되는 Oracle Digital Assistant의 접속 정보를 확인하여 콘솔에 접속합니다.
로그인 후에 다음과 같은 페이지가 나오면 왼편 상탄의 아이콘을 클릭하여 메뉴바를 펼칩니다.

![](/assets/images/chatbot_lecture/00_after_login.png)

아래와 같이 왼편 메뉴에서 **Bots** 메뉴를 클릭합니다.

![](/assets/images/chatbot_lecture/01_bot_first_page.png)

**MasterBot 복제**
=======
제공되는 테스트용 MasterBot을 찾아 사용자 별로 복제한 후 테스트 할 것입니다.
다음과 같이 MasterBot이 보이는지 확인하세요. 

![](/assets/images/chatbot_lecture/02_Masterbot.png)

본인에게 할당된 SEQ 번호를 참고하여 복제한 MasterBot의 이름을 다음 처럼 바꿔줍니다.
> 이름이 중복될 경우 복제 시에 오류가 날수 있으니 주의하세요!

![](/assets/images/chatbot_lecture/03_masterbot_clone.png)

다음과 같이 입력하고 Clone 합니다.
* Name :  MasterBot_[제공된 SEQ]
* Description : 본인 이름 

![](/assets/images/chatbot_lecture/04_masterbot_clone_window.png)

복제가 성공되면 다음과 같이 새로운 BOT이 생성된 걸 볼 수 있습니다.

 ![](/assets/images/chatbot_lecture/05_after_clone_master.png)

**복제된 MasterBot에서 Intent (사용자 의도) 테스트 하기**
=======

 방금 생성한 본인의 MasterBot의 이름을 클릭하거나 아래와 같이 **Edit** 메뉴를 클릭하여 Bot의 편집 화면으로 이동합니다.

 ![](/assets/images/chatbot_lecture/06_Edit_MasterBot.png)

이 봇은 이미 플로우가 작성되어 있는 봇입니다. 이봇을 테스트 허기 위해서 이미 등록되어 있는 각 Intent의 샘플 Utterance를 Train 시켜야 합니다. 상단의 Train 버튼을 클릭하여 등록된 샘플 Utterance를 학습 시킵니다. 

 ![](/assets/images/chatbot_lecture/07_Train.png)

**Trainer Ht** 옵션이 선택되었음을 확인하고 **Submit**을 클릭합니다. 

 ![](/assets/images/chatbot_lecture/08_Train_window.png)

Train이 완료되었으면 화면에 알림이 뜨고 다시 한번 상단의 **Train** 버튼을 클릭해 보면 **Trainer Ht**의 체크 아이콘이 **초록색**으로 변경되어 있음을 볼 수 있습니다. 이는 학습이 완료되었다는 뜻 입니다.
Train 팝업창 상단의 **X** 버튼을 클릭하여 Train 윈도우를 닫습니다.

![](/assets/images/chatbot_lecture/09_after_train.png)

이제 Intent 테스트를 위한 Test 창을 열기 위해 상단의 **▶** 버튼을 클릭합니다.

![](/assets/images/chatbot_lecture/10_play_button.png)


챗봇을 테스트 해볼 수 있는 테스트 창이 나옵니다.
Intent를 먼저 테스트 해볼 것이기 때문에 Test 창에서 **Intent** 탭을 선택합니다. 
![](/assets/images/chatbot_lecture/11_test_ui_intent.png)

테스트 창 왼편의 Examples로 등록되어 있는 문장들 몇 개를 **Message** Text Box에 입력해 봅니다. 입력한 Utterance가 **Balances**라는 **Intent**에 매칭된 것을 확인할 수 있습니다.

![](/assets/images/chatbot_lecture/12_intent_test_1.png)

몇 개 샘플 Utterance를 더 테스트 해 봅니다. 

![](/assets/images/chatbot_lecture/12_intent_test_2.png)

이제 Intent를 변경하여 다시 테스트 해보겠습니다. **Send Money** Intent를 선택하여 위와 같이 테스트 해봅니다. 

![](/assets/images/chatbot_lecture/12_intent_test_3.png)

**MasterBot과 대화해 보기**
=======
이제 Bot과 실제로 위에서 테스트한 Intent에 반응하는 대화를 해 볼 것입니다. 이 봇은 이미 다음 두 가지의 Intent를 위한 Flow가 작성되어 있습니다.

* Balances : 잔고 조회 
* Send Money : 계좌 이체 

위 두 Intent에 매칭이 될 수 있도록 **Utterance**를 입력하여 대화를 시도해 봅니다. 
대화 테스트는 Test UI의 **Bot** 탭에서 할 수 있습니다.

![](/assets/images/chatbot_lecture/13_test_ui_dial.png)

### Balances (잔고조회) Intent
**Message** Test Box에 다음과 같이 입력합니다.

![](/assets/images/chatbot_lecture/14_Bal_1.png)

어떤 계좌의 잔고를 보고 싶은지 **Bot**이 질문을 합니다. 어떤 것이나 선택해도 됩니다. 제공된 리스트 중 하나를 클릭하세요. 선택된 계좌의 잔고를 보여주고 대화가 종료됩니다.

![](/assets/images/chatbot_lecture/14_Bal_2.png)

Test UI의 **Reset** 버튼을 클릭하여 창을 clear 합니다. 

![](/assets/images/chatbot_lecture/15_reset.png)

다음은 게좌 유형이 **Utterance**에 포함된 대화를 테스트 해보겠습니다. 다음과 같이 질문을 하게 되면 Bot이 계좌 유형이 이미 제공되었기 때문에 게좌 유형을 묻는 질문을 하지 않고 바로 해당 계좌의 잔고를 보여주게 됩니다.

* 내 저축 계좌의 잔고가 얼마야?
* 예금 계좌에 잔고가 얼마나 남아 있지?

![](/assets/images/chatbot_lecture/14_Bal_3.png)

### Send Money (계좌 이체) Intent 테스트 하기
위 잔고 조회 예제와 같이 **Send Money** 예제도 테스트 해봅니다. 송금을 수행하기 위한 샘플 Utterance를 입력하여 송금 업무가 수행되게 합니다. 
다음과 같이 대화가 진행되는 것을 볼 수 있습니다. 송금을 진행하기 위해서는 다음 3개의 Entity(정보가) 필요하고 이를 Bot이 계속 질문하게 됩니다.

* 출금 계좌 
* 송금 계좌 
* 금액

![](/assets/images/chatbot_lecture/Send%20Money%20Test.png)


송금을 위한 3개의 Entity를 첫 사용자 Utterance에 포함하여 대화를 수행할 수 있습니다. 첫 문장에서 송금을 위한 모든 정보 (Entity)가 추출된다면 Bot은 추가적인 질문을 하지 않고 바로 송금을 시행합니다. 아래와 같이 입력하여 테스트 해
보세요

* 카드에서 출금해서 팀장님께 $50 보내줘 
* 내 예금 계좌에서 $200불 엄마한테 보내줘

 ![](/assets/images/chatbot_lecture/Send%20Money2.png)

## 사용환경 정리 
생성한 MasterBot_{SEQ}를 다음과 같이 삭제 합니다.

 ![](/assets/images/chatbot_lecture/99_delete_bot.png)

과정을 완료하였습니다.

# Chatbot-Workshop Lab 
* Lab 개요 : [챗봇 Hands-on Lab (1) - Lab 개요](/chatbot/2019/챗봇-Hands-on-Lab_1/)
* Lab 100 : [챗봇 Hands-on Lab (2) - 금융봇을 이용하여 챗봇 기본 기능 익히기](/chatbot/2019/챗봇-Hands-on-Lab_2/)
* Lab 200 : [챗봇 Hands-on Lab (3) - 피자봇 만들기 ](/chatbot/2019/챗봇-Hands-on-Lab_3/)
* Lab 300 : [챗봇 Hands-on Lab (4) - [채널 연결] Web Chat 연결하기](/chatbot/2019/챗봇-Hands-on-Lab_4/)

## 참고 자료 

금융봇(MasterBot)이 등록되어 있지 않다면 아래 파일을 다운 받아 Import 하면 됩니다.

- [MasterBot Import 파일](https://github.com/mee-nam-lee/chatbot_lecture/blob/master/labfiles/MasterBot_Korean/MasterBot_kor_wo_comp.zip)