---
layout: post
title:  챗봇 Embedded Container에 Custom Component 올리기
categories: Chatbot
tags: [chatbot, 오라클 챗봇, Custom Component, Container]
---

챗봇을 구현하다 보면 Dialogue Flow 만을 이용해서는 다양한 비즈니스 로직을 구현하기에 한계가 있을 수 있습니다. 이를 위해 Oracle Digital Assistant에서는 Custom Component를 개발하여 연동할 수 있게 되어있는데, 이 개발된 Custom Component는 다음 3가지 형태로 서비스될 수 있습니다.

* Embedded Container
* Oracle Mobile Cloud
* External

Oracle Mobile Cloud와 External 옵션은 구축된 Custom Component가 Oracle Digital Assistant 외부에서 서비스되는 것을 연동하는 방식이고 Embedded Container는 Digital Assistant 내에 Container를 구동하게 하여 Custom Component 구현체가 서비스 될 수 있도록 해주는 옵션입니다. 

Custom Component의 작성 방법과 SDK 사용법은 이전 글들에서 소개가 되었으므로, 이 글에서는 이미 만들어진 Custom Component를 Embedded Container에 올리기 위해 Packaging 하는 방법만 다룰 것 입니다.

## Packaging 
Custom Component 디렉토리로 이동하여 **npm pack** 명령어를 수행합니다. my-custom-component-1.0.0.tgz 파일이 생성될 것입니다.

![](/assets/images/chatbot_embedded/npm_pack.png)

## Digital Assistant에서 Service 등록하기
Digital Assistant 콘솔로 이동하여 Service를 등록합니다.  
서비스 생성 옵션에서 **Embedded Container** 옵션을 선택하고 이전 단계에서 생성한 .tgz 파일을 업로드 합니다.

![](/assets/images/chatbot_embedded/create_service.png)

업로드가 되고 나면 다음 처럼 보이게 됩니다.

![](/assets/images/chatbot_embedded/upload_package.png)

서비스 생성을 클릭하고 나면 새로운 서비스가 등록되어있을 것입니다.

![](/assets/images/chatbot_embedded/service_created.png)

이 서비스를 사용하는 방법은 이전 글에서 설명한 방법과 동일 합니다.


# 참고 자료

Oracle 챗봇 컴포넌트 작성을 위한 자세한 SDK 가이드는 다음을 참고하세요

- [Oracle Bots Node.js SDK](https://github.com/oracle/bots-node-sdk/)