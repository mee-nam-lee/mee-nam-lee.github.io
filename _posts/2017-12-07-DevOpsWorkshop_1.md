---
layout: post
title:  DevOps Workshop (CI/CD 실습) - (1)
categories: DevOps
tags: [Developer Cloud, DevOps, CI/CD, Hands-on-lab]
---

이 Workshop에서는 Microserivces 환경을 준비하고 Oracle Application Container Cloud Service를 사용하여 Mircoserivces를 개발하는 방법을 보여줍니다. 

또한 Developer Cloud Service를 사용하여 빌드 자동화 및 자동 배포 (CI / CD) 과정을 실습합니다.

### 소개

마이크로 서비스 아키텍처에 대한 표준은 없지만 업계에서는 주로 아래에 언급된 마이크로 서비스 아키텍처가 따라야 할 몇 가지 디자인 요소와 특성에 부합해야 합니다. 

- 여러 서비스로 구성되며, 기본적으로 작고 독립적인 서비스
- 장애가 날 수 있음을 고려하여 설계 되어야 함
- 확장성을 위한 설계가 필요함
- 메시지를 통해 호출하여 기능을 분리하도록 하여야 함
- 상태를 저장하지 않도록 함
- DevOps를 통한 자동화된 배포

마이크로 서비스에 대한 일반적인 요구 사항은 다음과 같습니다. 

- 확장적이고, 탄력적이며, 다양한 언어를 지원(Polyglot) 
- 자동적인 개발 라이프사이클(DevOps) 
- 애플리케이션 성능 모니터링 및 진단을 위한 툴 
- 가볍고 배포가 용이하고 확장성을 지닌 컨테이너 기반
- API로 서비스 노출 

**마이크로 서비스를 통해 기업은 신속하고 민첩한 방식으로 애플리케이션의 일부를 개발, 배포 및 업데이트 할 수 있으므로 보다 신속하고 유연한 방식으로 새로운 시장 요구 사항 및 경쟁에 대응할 수 있습니다.**

마이크로 서비스 아키텍처 (**MSA**)는 민첩한 개발 및 서비스 배포를 보장합니다. 그러나 조직이 MSA를 채택 할 때 여러 가지 과제가 있습니다. 다음은 그러한 과제 / 요구 사항 중 일부를 나열합니다. 

![](/assets/images/devops/000.challenges.png)


## Oracle Application Container 클라우드 서비스


[Oracle Application Container Cloud Service](https://cloud.oracle.com/en_US/application-container-cloud)는 Oracle AppDev Portfolio의 클라우드 서비스 중 하나로 Java SE, Node.js, PHP, Python 및 Ruby 응용 프로그램을 Oracle Cloud에 배포 할 수 있습니다. 

![](/assets/images/devops/000.architecture.png)


Oracle Application Container Cloud Service에는 다음과 같은 주요 기능이 있습니다. 

- Java SE, Node.js, PHP, Python 및 Ruby로 작성된 응용 프로그램을 구동할 수 있는 사전 구성된 환경. 
- Java Flight Recorder, Java Mission Control, 고급 메모리 관리 및 지속적이고 적절한 보안 업데이트의 알림 같은 Java SE 고급 기능. 
- Spring, Play, Tomcat 및 Jersey와 같은 모든 Java 프레임 워크 및 컨테이너를 지원하는 개방형 플랫폼. 
- JRuby와 같은 Java Virtual Machine (JVM) 기반 언어 지원. 이 서비스에서 Java Virtual Machine을 사용하는 언어를 실행할 수 있습니다. 
- 오라클의 엔터프라이즈 급 지원. 
- 웹 기반 사용자 인터페이스 및 REST API 

![](/assets/images/devops/000.accs.png)


또한 다른 Oracle Cloud 서비스와의 통합을 선택할 수도 있습니다. 로컬 시스템에서 애플리케이션을 개발하거나 Oracle Developer Cloud Service를 사용할 수 있습니다. 

## Oracle Developer Cloud Service

[Oracle Developer Cloud Service](https://cloud.oracle.com/en_US/application-container-cloud)은 민첩한 개발 방법론 및 DevOps 자동화를 가능하게하는 클라우드 기반 개발 플랫폼입니다.이 서비스는 팀 개발 및 배포를 위한  프로젝트 계획, 이슈 관리, 문제 추적, 코드 버전 관리, 위키, 애자일 개발 도구, 지속적인 통합 및 배포 자동화(CI/CD)를 포함합니다. 

![](/assets/images/devops/000.devcs.png)

# DevOps Workshop Lab 
이 Hands-on 웍샵에서는 Developer Cloud Service를 사용하여 약 1 시간 동안 Application Container Cloud에 1개의 마이크로 서비스를 만들고 배포합니다.

- 기존 3rd party Git Repository 활용 
- 개발자 클라우드 서비스에서 빌드 및 배포 작업 만들기 
- 개발자 클라우드 서비스와 IDE 사용  
- 클라우드 서비스에 배포 

- [DevOps Workshop 개요](./DevOpsWorkshop_1.html)
- [Lab 01로 이동](./DevOpsWorkshop_2.html)
- [Lab 02로 이동](./DevOpsWorkshop_3.html)
