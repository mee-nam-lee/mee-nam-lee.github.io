---
layout: post
title:  챗봇 Hands-on Lab (5) - [채널 연결] Mobile 앱 연결하기
categories: chatbot
tags: [chatbot, 오라클 챗봇, Hands-on-Lab, Channel, mobile]
---

이 Lab에서는 지금까지 만든 챗봇을 모바일 앱을 통해 연결하는 실습을 해 볼 것 입니다.


# 모바일(Android) 채널 생성
좌측 메인 메뉴의 **Channels** 메뉴를 선택합니다.
![](/assets/images/chatbot_lecture/channel/01_channel_settings.png)

 **+ Channel** 버튼을 클릭하여 새로운 채널을 생성하도록 합니다. 

다음과 같이 입력하고 **Create** 버튼을 클릭합니다.

* Name : PizzaBot_{SEQ}.Android 
> 본인이 생성한 PizzaBot과 동일 명으로 합니다 
* Descrption : 학생이름 = [본인이름]
* Channel Type : **Android**

![](/assets/images/chatbot_lecture/channel/07_android.png)

다음과 같이 채널이 만들어 진 것을 확인 합니다. 아직 채널이 활성화 된 상태는 아닙니다.

![](/assets/images/chatbot_lecture/channel/08_android_after_create.png)

만든 채널을 각자의 PizzaBot과 연결하기 위해 **Route To**를 클릭하여 본인의 Bot과 연결되게 합니다.

아래와 같이 연결되었는지를 확인하고 **Channel Enabled**를 활성화 시킵니다.

![](/assets/images/chatbot_lecture/channel/09_android_route.png)

채널 생성은 완료되었습니다.

## 모바일 앱 다운 받기
Oracle Digital Assistant는 Android 앱에서 채팅 기능을 사용할 수 있도록 SDK와 샘플 애플리케이션을 제공하고 있습니다.
이 제공되는 샘플 SDK를 다운 받아 앱을 수정하여 빌드하고 배포하여서 테스트 하도록 되어있지만, 이 Lab에서는 미리 미리 빌드된 apk를 제공하여 설치 후 바로 테스트 할 수 있도록 하였습니다. 

### 참고 : Android 용 Sample 다운로드
- [Oracle Digital Assistant (ODA) and Oracle Mobile Cloud (OMC) Downloads](https://www.oracle.com/technetwork/topics/cloud/downloads/amce-downloads-4478270.html) :  **bots-client-sdk-android-samples-19.1.3.0.zip**을 다운로드 합니다.
- [Android 앱 설정](https://docs.oracle.com/en/cloud/paas/digital-assistant/use-chatbot/channels-topic.html#GUID-4B97C781-6972-44B9-A7D3-9F2F57CE09B9)


### 미리 빌드된 샘플 SDK APK 설치 
위 샘플을 빌드한 APK를 다음에서 다운받아 안드로드이 폰에 설치 합니다. 

- [ChatSample SDK APK 다운로드](https://github.com/mee-nam-lee/chatbot_lecture/raw/master/labfiles/mobilechat/app-release.apk)

설치 후 앱을 실행 시킵니다. 다음과 같은 화면이 보일 것 입니다.

![](/assets/images/chatbot_lecture/channel/15_android_01.jpeg)

방금 전에 만든 채널의 **App ID**를 아래와 같이 Text Box에 입력한 후 **계속**을 클릭합니다.

![](/assets/images/chatbot_lecture/channel/16_android_appid.png)

다음 화면에서 챗봇과 대화하기를 클릭합니다.

![](/assets/images/chatbot_lecture/channel/17_android_before_chat.jpeg)

다음과 같이 대화가 바로 이루어지는 것을 볼 수 있습니다.

![](/assets/images/chatbot_lecture/channel/18_android_chat.jpeg)

# 모바일(iOS) 채널 생성

iOS용 모바일 앱을 위한 채널도 하나 생성해  봅니다.

* Name : PizzaBot_{SEQ}.iOS 
> 본인이 생성한 PizzaBot과 동일 명으로 합니다 
* Descrption : 학생이름 = [본인이름]
* Channel Type : **iOS**

![](/assets/images/chatbot_lecture/channel/10_ios_create.png)

채널이 생성되었으면 Route To와 Enabled를 설정해 줍니다.

![](/assets/images/chatbot_lecture/channel/11_ios_created.png)

### 참고 : iOS 용 Sample 다운로드
- [Oracle Digital Assistant (ODA) and Oracle Mobile Cloud (OMC) Downloads](https://www.oracle.com/technetwork/topics/cloud/downloads/amce-downloads-4478270.html) : **bots-client-sdk-ios-samples-19.1.3.0.zip**을 다운로드 합니다.
- [Android 앱 설정](https://docs.oracle.com/en/cloud/paas/digital-assistant/use-chatbot/channels-topic.html#GUID-4B97C781-6972-44B9-A7D3-9F2F57CE09B9)

iOS용 배포 파일은 제공되지 않습니다. 위 샘플 코드를 받아서 iOS XCode Simulator를 통해 시연해 보면 다음과 같이 보여지게 됩니다.

iOS 앱에서 실행된 화면은 다음과 같습니다. 동일한 방법으로 App ID를 입력해 줍니다.

![](/assets/images/chatbot_lecture/channel/12_ios_appid.png)

**Chat with your Bot**을 클릭하여 대화를 시작합니다.

![](/assets/images/chatbot_lecture/channel/13_ios_start_chat.png)

피자 주문을 해 보면 다음과 같이 보여집니다.

![](/assets/images/chatbot_lecture/channel/14_ios_chat_pizza.png)

과정을 완료하였습니다.

# Chatbot-Workshop Lab 
* Lab 개요 : [챗봇 Hands-on Lab (1) - Lab 개요](/chatbot/2019/챗봇-Hands-on-Lab_1/)
* Lab 100 : [챗봇 Hands-on Lab (2) - 금융봇을 이용하여 챗봇 기본 기능 익히기](/chatbot/2019/챗봇-Hands-on-Lab_2/)
* Lab 200 : [챗봇 Hands-on Lab (3) - 피자봇 만들기 ](/chatbot/2019/챗봇-Hands-on-Lab_3/)
* Lab 300 : [챗봇 Hands-on Lab (4) - [채널 연결] Web Chat 연결하기](/chatbot/2019/챗봇-Hands-on-Lab_4/)