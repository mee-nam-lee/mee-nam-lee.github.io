---
layout: post
title:  Oracle IaaS와 Java Cloud Service를 사용하여 멀티 AD에 3 Tier 아키텍쳐 구축하기 (Regional Subnet 사용)
categories: Cloud
tags: [Oracle Cloud, Virtual Cloud Network,IaaS, 3 Tier Architecture, HA, Java Cloud Service] 
---

이전 문서 [Oracle IaaS와 Java Cloud Service를 사용하여 멀티 AD(Availability Domain)에 3 Tier 아키텍쳐 구축하기](/cloud/2019/basic-3tier-arch/)에서는 Availablity Domain Specific Subnet을 이용하여 3 Tier 아키텍처를 구축하는 내용을 설명하였는데, Oracle IaaS에서 **Regional Subnet**이 지원이 추가되었기 때문에, 이전에 설명되었던 내용이 Regional Subnet 기반으로 구축되려면 어떤 부분이 변경되어야 하는지를 이 문서에서 설명하려고 합니다. 
이전 문서에서 설명하였던 과정은 이 문서에서는 모두 생략하고 변경되는 부분만 다루로록 하겠습니다. 

이전의 Availablity Domain Specific Subnet을 이용한 구축 아키텍처는 다음 그림과 같았습니다.

![](/assets/images/3tier/image1.png)

Regional Subnet을 이용하게 되면 아키텍처가 다음과 같이 변경될 수 있습니다.

![](/assets/images/3tier/regional/architecture.png)


Network 생성 / 구성 
====================

Regional Subnet을 사용하기 때문에, AD별로 별도록 Subnet을 생성할 필요가 없고 따라서 생성해야 할 Subnet 수가 줄어들게 됩니다.
일반적으로 Public Load Balancer와 Web 서버는 별도의 Subnet에 구축하고, Public Load Balancer를 구성할 경우에는 Web Server를 Private Subnet에 구성하도록 권고 되지만, 여기에서는 하나의 Public Subnet을 생성하여 Load Balancer와 Web Server를 같이 두는 형태로 구성하겠습니다.

 | **구분**    | **이름**                 |   **Route Table**  | **Security List** |
 | ------------ --------------------------- ----------------- -------------------|
 |**VCN**    | VCN\_3Tier               |    Default         |  Default |
 |**Subnet** | VCN\_3Tier\_Sub\_WEB     |   PublicRT           | PublicSL |
 |           | VCN\_3Tier\_Sub\_WAS     |PrivateRT           |PrivateSL |
 |           | VCN\_3Tier\_Sub\_DB      |PrivateRT           | DBSL |

VCN 생성
--------
동일

Service Gateway 생성
--------------------
동일

Internet Gateway 생성
---------------------
동일

Security List 생성 / 구성
-------------------------
동일

Route Table 생성 / 구성
-----------------------
동일

Subnet 생성 / 구성
------------------

**Regional** Type으로 선택하는 것 외에 나머지 사항들은 이전과 동일합니다.

![](/assets/images/3tier/regional/create_subnet.png)


나머지 Subnet들도 다음 표를 참고하여 생성합니다.

  | **Name**         |  **Subnet Type**     |   **CIDR**   |   **Route Table** |  **Subnet Access**  | **Security List** |
  |-------------------|---------------------| ---------| ---------- |----------------- |------------------- |-------------------|
 | **VCN\_3Tier\_Sub\_WEB**  | REGIONAL  |  10.0.1.0/24  | PublicRT        |  Public          |    PublicSL |
 | **VCN\_3Tier\_Sub\_WAS**  | REGIONAL  |  10.0.2.0/24  | PrivateRT       |  Private         |    PrivateSL |
 | **VCN\_3Tier\_Sub\_DB**   | REGIONAL  |  10.0.3.0/24  | PrivateRT       |  Private         |    DBSL |

BUCKET 생성
===========
동일

DB 생성
=======
동일

JCS 생성 
=========

JCS 생성을 위한 사전 준비사항과 Java Cloud Service Console을 통해 서비스 생성을 위한 JSON 파일을 얻는 과정은 동일합니다.
두 개의 JCS가 **같은 Regional Subnet**에 속하지만 **서로 다른 Availabilty Domain**에 속하도록 생성해야 하기 때문에 
JSON에서 같은 **Private Subnet의 OCID**를 사용하도록 수정하고 서로 다른 **availabilityDomain**를 사용하도록 변경해 줍니다.

```json
{
    "subnet": "VCN_3Tier_Sub_WAS private subnet의 OCID",

    // 생략

    "availabilityDomain": "JCS 별 다른 AD" ,
     
    // 생략

}
```

JSON으로 JCS 서비스를 생성하는 과정은 동일합니다.

Web 서버 생성 및 설정
=====================
동일

Coherence 설치 및 구성
======================
동일

JCS에서 Coherence 설정
======================
동일

애플리케이션 배포
=================
동일

Load Balancer 생성
==================

Regional Subnet을 사용하는 Public Load Balancer를 생성해 줍니다. Regional Subnet을 선택하게 되면 그 안에서 AD간의 HA는 자동으로 구성되도록 Load Balancer가 생성됩니다.

![](/assets/images/3tier/regional/create_lb.png)


Test
====
동일

# 이전 문서 참고

- [Oracle IaaS와 Java Cloud Service를 사용하여 멀티 AD(Availability Domain)에 3 Tier 아키텍쳐 구축하기](/cloud/2019/basic-3tier-arch/)
