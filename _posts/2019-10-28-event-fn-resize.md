---
layout: post
title:  Events와 Functions을 사용한 Thumb Nail Image 생성하기
categories: Cloud
tags: [Oracle Cloud, OCI, Event, Functions, fn project, Node.js, Sharp]
---

이 문서에서는 Object Storage에 저장되는 이미지를 OCI의 **Events Service**와 Serverless 서비스인 **Functions**을 사용하여 자동으로 Thumb Nail Image를 생성하는 방법에 대해서 설명할 것이다.
사용되는 서비스들과 서비스들 간의 호출 관계는 다음과 같다.

![](/assets/images/fn_resize/00_arch.png)

# Prerequistes
이 예제에서 사용하는 서비스들은 다음과 같다.

- OCI Events Service
- OCI Notification Service
- OCI Functions Service
- OCI Object Storage Service

# Fn App 생성

제일 먼저 Function을 배포할 애플리케이션을 생성한다. 
**Functions** 메뉴는 다음에서 찾으면 된다.

![](/assets/images/fn_resize/01_fn_menu.png)

**Create Application** 버튼을 클릭하여 Fn를 배포할 애플리케이션을 생성한다.

![](/assets/images/fn_resize/02_create_fn_app.png)

애플리케이션 이름과 애플리케이션을 생성할 **Virtual Cloud Network (VCN)**과  **Subnet**을 선택하고, Fn의 Log 저장 옵션으로 **Object Storage**를 선택한다.

![](/assets/images/fn_resize/02_create_fn_app2.png)

다음과 같이 애플리케이션이 생성되어 있는 것을 볼 수 있다. 

![](/assets/images/fn_resize/03_fn_app_list.png)

# Resize Function Deploy

생성한 애플리케이션에 **fn**을 배포해 볼 것이다. 로컬 환경에 Fn 구축을 위한 절차와, OCI Functions에 접속하는 절차는 다음을 참고한다.

- [Configuring Your Tenancy for Function Development](https://docs.cloud.oracle.com/iaas/Content/Functions/Tasks/functionsconfiguringtenancies.htm)

**Fn Project** CLI를 Client에 설치하고 OCI Functions 서비스와의 연결을 설정하고 나면 로컬 환경에서 **fn** 커맨드를 통해서 다음과 같이 연결된 **context**를 확인할 수 있다
Current Context가 OCI Functions를 사용하는 것으로 설정되어있는지 확인한다. ***** 표시가 되어있는 Context가 Current Context이다

![](/assets/images/fn_resize/04_fn_contexts.png)

여기에서 사용할 예제 fn은 Object Storage Bucket으로부터 image를 읽어와서 **Sharp** 모듈로 이미지 사이즈를 조정한 후에 다시 Object Storage에 넣는 예제이다. 

- [Sharp](https://sharp.pixelplumbing.com/en/stable/)

다음에서 필요한 소스를 받아온다.
Function Code는 func.js를 참고하면 된다.

```
> git clone https://github.com/mee-nam-lee/NodeJS_Work.git
> cd myapp/resizefn
```

resizefn 디렉토리로 이동하여 fn을 배포한다.
아래 커맨드를 수행하면 resizefn이 myapp 애플리케이션에 배포되게 된다.

```
> fn -v deploy --app myapp
```

![](/assets/images/fn_resize/05_fn_deploy.png)

OCI 콘솔의 Functions Application으로 들어가면 방금 Deploy한 Function이 아래와 같이 보이는 것을 확인할 수 있을 것이다.

![](/assets/images/fn_resize/06_fn_list.png)

# Object Bucket 생성

이제 이미지를 저장할 Object Storage Bucket을 다음과 같이 생성한다.
**org_image**는 Original 이미지가 저장된 Bucket이고 **small_image**는 Thumb Nail로 사이즈 조정된 이미지가 저장될 Bucket이다.

![](/assets/images/fn_resize/07_object.png)

# Notification Service 생성 

Bucket에 Object가 생성되면 Object Create 이벤트가 발생하고 이 이벤트에 의해서 이메일 Notification을 해주기 위해서 Email 통지를 받는 Notification을 먼저 생성해 준다. 

![](/assets/images/fn_resize/08_create_topic.png)

이메일 Notification을 받기 위하여 **PROTOCOL** 항목을 **Email**로 선택하고 메일 주소를 입력한다.

![](/assets/images/fn_resize/09_subscription.png)

# Event Serivce 생성 

이제 Bucket에 Object가 생성되면 resize fn를 호출하고 Email Notification을 호출할 Event 서비스를 만들어 보도록 하겠다.

먼저 **org_image**에 Original Image가 생성되면 resizefn을 호출하는 Event Rule를 생성해 준다.
다음과 같이 Event Condition과 Action 항목을 입력해 준다.

![](/assets/images/fn_resize/10_event_rule1.png)

Thumb Nail Image가 만들어지고 **small_image** Bucket에 이미지가 저장되면 Email 통지를 해주기 위한 Event Rule도 생성한다.
이 Event Condition에서 사용하는 BucketName 조건을 입력하고 Action 항목에 생성해 둔 Notification 서비스를 선택해 준다.

![](/assets/images/fn_resize/11_event_rule2.png)

다음과 같이 두개의 Rule이 생성된 것을 볼 수 있다.

![](/assets/images/fn_resize/12_event_list.png)


# Test

이제 준비는 다 되었다. Object Storage Console로 이동하여 원본 크기의 이미지를 **org_image** Bucket에 Upload 해보자

![](/assets/images/fn_resize/13_upload.png)

이미지가 업로드 되었다.

![](/assets/images/fn_resize/14_uploaded.png)

Object가 생성되고 나서 Event가 정상적으로 발생하였는지 확인하기 위해서 Event의 Metrics 페이지를 확인해 본다. Event가 발생하였고 성공적으로 Function을 호출했음을 알 수 있다.

![](/assets/images/fn_resize/15_event_metric.png)

Functions Metrics에서도 해당 fn이 성공적으로 수행되었음을 볼 수 있다.

![](/assets/images/fn_resize/16_fn_metric.png)

**small_image** Bucket에 가 보면 Thumb Nail이 생성되어 있는 것을 확인할 수 있다.

![](/assets/images/fn_resize/17_small_image.png)

이미지가 잘 변환되었는지 다운 받아서 확인해 본다.

![](/assets/images/fn_resize/18_download.png)

![](/assets/images/fn_resize/19_thumb.png)

이벤트 규칙에 의해서 Thumb Nail 이미지 생성 Event에 의해서 Email 통지 또한 수행 되었다.

![](/assets/images/fn_resize/20_email.png)

# 참고 자료

- [Introduction to Fn with Node.js](https://fnproject.io/tutorials/node/intro/)
- [Configuring Your Tenancy for Function Development](https://docs.cloud.oracle.com/iaas/Content/Functions/Tasks/functionsconfiguringtenancies.htm)