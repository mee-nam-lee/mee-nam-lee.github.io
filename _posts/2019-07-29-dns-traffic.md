---
layout: post
title:  Oracle Cloud Infrastructure - DNS - Traffic Management Steering Policy
categories: Cloud
tags: [Oracle Cloud, OCI, Traffic Management, DNS, Steering Policy]
---

**Oracle Cloud Infrasture(OCI)**의 **DNS**와 **Traffic Management** 기능을 이용하여 Cloud Region 간 또는 Cloud와 On-Premise 간의 로드발란싱을 구성할 수가 있다.
Traffic Management는 여러가지 Steering Policy를 제공하는데, 이 문서에서는 **LOAD BALANCER** Steering Policy에 대해서 알아 볼 것이다.

사용자가 웹사이트에 접근하기 위해 DNS Lookup을 수행하면 DNS에 적용된 Traffic Management의 Steering Policy를 통해서 제공되는 Origin Server IP를 발란싱해서 전달해주고 사용자 요청이 여러 Region 간에 분배될 수 있게 해준다.

![](/assets/images/traffic/lb/dns_lb.png)   

# Prerequiste
- 도메인 
    - 설정에 사용할 도메인이 있어야 한다. 이 예제에서는 무료 도메인 발급 사이트에서 발급받은 **mnlee.cf**를 사용했다.
    - 도메인 등록 사이트에서 Name Server를 **OCI DNS**의 Name Server로 연결되게 구성해 주어야 한다.

![](/assets/images/waf/00_nameserver.png)   

# Traffic Management Steering Policy 생성

OCI 콘솔에 접속해서 **Traffic Management Steering Policy**를 생성해 준다.
여러 Policy Type 중에서 **LOAD BALANCER** 타입을 선택한다.
이름과 TTL을 적당한 값으로 입력하고 로드발란싱을 적용할 DNS Record를 추가해 준다.
이 예제에서 **pho.mnlee.cf**는 **Phoenix** 리전에서 서비스하고, **ash.mnlee.cf**는 **Ashburn** 리전에서 서비스 한다.

![](/assets/images/traffic/lb/create_traffic_1.png)   

Region에서 서비스되고 있는 인스턴스나 서비스의 **Health**를 체크하여 **Healthy**한 서비스가 리턴 되도록 **Health Check** 기능을 연결해도 되지만 이 예제에서는 간단한 테스를 위하여 이 기능은 Attach하지 않았다.
Policy를 Attach할 도메인을 선택하고 **Create Policy**를 클릭한다.

![](/assets/images/traffic/lb/create_traffic_2.png)   

Traffic Management Policy가 만들어지고 난 후의 화면이다. 

![](/assets/images/traffic/lb/after_traffic_1.png)   

![](/assets/images/traffic/lb/after_traffic_2.png)   


# DNS 설정

DNS Zone 관리 화면에서 로드발란싱을 수행할 두개의 CNAME Record를 다음과 같이 추가해 준다.
추가 후에는 **Publish Changes**를 눌러줘야 변경사항이 반영된다.

![](/assets/images/traffic/lb/dns_record.png)   

# Test

DNS Lookup 로드발란싱이 잘 수행되는지 **dig** 커맨드를 통해서 테스트해 본다
첫번째 dig 명령어에서는 **ash.mnlee.cf**가 리턴 되었다.
TTL을 60초로 설정해 두었기 때문에 TTL 내에서는 동일한 Answer가 보이게 될 것이다.

TTL 경과 후 다시 DNS Lookup을 수행하면 **pho.mnlee.cf**가 리턴되는 것을 볼 수 있다.

![](/assets/images/traffic/lb/dig_ash_pho.png)   


브라우저에서 테스트 해보자.
두 Region 중 하나에서 응답을 받을 것이다.

![](/assets/images/traffic/lb/browser_ash.png) 

브라우저에서는 DNS lookup 후 리턴되는 서버 IP를 캐싱하여 가지고 있기 때문에 TTL 후에도 계속 같은 Region에서 응답을 받게 될 것이다.

사용하는 브라우저 별로 DNS 캐시 Clear하는 방법을 참고하여 Cache를 지운 후에 다시 테스트 해보자.
Chrome 브라우저에서는 다음과 같이 하면 된다.

![](/assets/images/traffic/lb/chrome_dns_clear.png) 

다시 테스트 해보면 다른 Region에서 응답을 받게 될 것이다.

![](/assets/images/traffic/lb/browser_pho.png) 


# 참고 자료
- [Overview of the Traffic Management Steering Policies Service](https://docs.cloud.oracle.com/iaas/Content/TrafficManagement/Concepts/overview.htm)