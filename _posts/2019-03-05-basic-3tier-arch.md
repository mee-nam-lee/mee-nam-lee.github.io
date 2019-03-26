---
layout: post
title:  Oracle IaaS와 Java Cloud Service를 사용하여 멀티 AD(Availability Domain)에 3 Tier 아키텍쳐 구축하기
categories: cloud
tags: [Oracle Cloud, Virtual Cloud Network,IaaS, 3 Tier Architecture, HA, Java Cloud Service] 
---

이 문서에서는 다음과 같은 가장 일반적인 3-Tier 아키텍처 기반의 웹 시스템을 **Oracle Cloud Infrastructure(IaaS)**와 **Java Cloud Service(PaaS)**를 이용하여 구축하는 방법에 대해서 기술합니다.

구축 아키텍처는 다음 그림에서 보여지는 것과 같습니다.

![](/assets/images/3tier/image1.png)

## 필요 서비스 및 소프트웨어
 - Oracle Cloud Infrastructure
   * Compute Service
   * Network Service
   * Object Storage Service
   * Database System
- Java Cloud Service 
- 필요 소프트웨어
   * Apache (Open Source)
   * Oracle Coherence

아키텍처 특징 
--------------

-   **Virtual Cloud Network(VCN)**을 이용하여 가상 네트워크를 구성
-   두 개의 **Availability Domain(AD)**를 이용하여 **HA**가 되도록 구성
-   Java Cloud Service(JCS)와 Database는 **Private Subnet**에 구성
-   서로 다른 AD에 각기 구성된 JCS 도메인의 **세션 공유는 Coherence Cluster**를 이용하여 구성
-   웹서버에서는 해당 AD 내의 JCS만 로드 발란싱이 이루어 지도록 구성

비고
----

-   여기서 설명하는 기본 아키텍처에서는 Database 이중화 부분은 고려하지 않음
-   DB 이중화를 위해서는 추가 작업이 필요함

Network 생성 / 구성 
====================

위 아키텍처 구성을 위해 다음과 같은 Virtual Cloud Network(VCN) 구성이 필요합니다.

 | **구분**    | **이름**                 |   **Route Table**  | **Security List** |
 | ------------ --------------------------- ----------------- -------------------|
 |**VCN**    |  VCN\_3Tier              |    Default         |  Default |
 |**Subnet** | VCN\_3Tier\_Sub\_Web\_AD1  | PublicRT         | PublicSL |
 |           | VCN\_3Tier\_Sub\_Web\_AD2   |PublicRT          |PublicSL |
 |           |VCN\_3Tier\_Sub\_WAS\_AD1   |PrivateRT         |PrivateSL |
 |          |VCN\_3Tier\_Sub\_WAS\_AD2   |PrivateRT         |PrivateSL |
 |         |VCN\_3Tier\_Sub\_DB\_AD1    |PrivateRT         |DBSL |

VCN 생성
--------

VCN을 다음과 같이 생성합니다. Subnet은 두개만 필요하기 때문에 "**Create Virtual Cloud Network Only**" 옵션을 선택하고 Subnet을 나중에 추가하도록 합니다.

![](/assets/images/3tier/image2.png)

Service Gateway 생성
--------------------
Private Subnet에서 Public에 존재하는 Oracle Service에 Internet 망을 거치지 않고 Oracle Cloud 내부 네트워크를 통해 접속하게 하기 위해서는 Service Gateway가 필요합니다. 이 Service Gateway를 통해 Object Storage에 연결할 것입니다.

![](/assets/images/3tier/image3.png)

![](/assets/images/3tier/image4.png)

Internet Gateway 생성
---------------------

Public Subnet에서 인터넷 접속을 위해 사용할 Internet Gateway를 생성합니다.

![](/assets/images/3tier/image5.png)

![](/assets/images/3tier/image6.png)

Security List 생성 / 구성
-------------------------

Subnet을 개별적으로 구성하고 각 Public Subnet / Private Subnet 별로 Route Table과 Security List를 달리 구성해야 하기 때문에 
Subnet 생성 전에 Security List와 Route Table을 미리 구성해 두고 Subnet 생성 시에 미리 만들어진 것들을 이용해야 합니다.

생성된 VCN의 Security Lists 메뉴로 이동하여 필요한 Security List를 생성합니다.

![](/assets/images/3tier/image7.png)

아래와 같이 Security List를 생성하고 Ingress/Egress Rule을 추가해 줍니다.

### Public Security List : PublicSL

![](/assets/images/3tier/image8.png)

-   Ingress Rules

![](/assets/images/3tier/image9.png)

-   Egress Rules

![](/assets/images/3tier/image10.png)

### Private Security List : PrivateSL

-   Ingress Rules

  > Public Subnet (10.0.1.0/24, 10.0.2.0/24)으로 부터 들어오는 포트는 22, 80, 8001, 7001이 사용될 것이므로 이 포트를 각각 지정해 줘도 되지만 여기서는 두 Subnet에서부터 들어오는 port는 모두 허용하는 것으로 설정하였습니다.

![](/assets/images/3tier/image11.png)

-   Egress Rules

![](/assets/images/3tier/image12.png)

### Private Security List : DBSL

-   Ingress Rules

![](/assets/images/3tier/image13.png)

-   Egress Rules

![](/assets/images/3tier/image14.png)

Route Table 생성 / 구성
-----------------------

Route Table 메뉴에서 Route Table을 생성합니다.

![](/assets/images/3tier/image15.png)

### Public Route Table : PublicRT
Public Route Table에서는 Internet Gateway로 가는 Route를 설정해 줍니다.

![](/assets/images/3tier/image16.png)


### Private Route Table : PrivateRT
Private Route Table에서는 Object Storage로 가기 위한 Service Gateway로의 Route를 추가해 줍니다.

![](/assets/images/3tier/image18.png)

Subnet 생성 / 구성
------------------

만들어진 VCN에 들어가서 **Create Subnet**을 클릭합니다.

![](/assets/images/3tier/image19.png)

### Public Subnet 1 : VCN\_3Tier\_Sub\_Web\_AD1

Public Subnet을 다음과 같이 생성합니다.

![](/assets/images/3tier/image20.png)

나머지 Subnet들도 다음 표를 참고하여 생성합니다.

  | **Name**                      |  **Subnet Type**              |  **AD**   **CIDR**   |   **Route Table** |  **Subnet Access**  | **Security List** |
  |------------------------------- |------------------------------| --------| ------------- |----------------- |------------------- |-------------------|
 | **VCN\_3Tier\_Sub\_Web\_AD1**  | AVAILABILITY DOMAIN-SPECIFIC  | AD1     | 10.0.1.0/24  | PublicRT        |  Public          |    PublicSL |
 | **VCN\_3Tier\_Sub\_Web\_AD2**  | AVAILABILITY DOMAIN-SPECIFIC  | AD2     | 10.0.2.0/24  | PublicRT        |  Public          |    PublicSL |
 | **VCN\_3Tier\_Sub\_WAS\_AD1**  | AVAILABILITY DOMAIN-SPECIFIC  | AD1     | 10.0.3.0/24  | PrivateRT       |  Private         |    PrivateSL |
 | **VCN\_3Tier\_Sub\_WAS\_AD2**  | AVAILABILITY DOMAIN-SPECIFIC  | AD2     | 10.0.4.0/24  | PrivateRT       |  Private         |    PrivateSL |
 | **VCN\_3Tier\_Sub\_DB\_AD1**   | AVAILABILITY DOMAIN-SPECIFIC  | AD1     | 10.0.5.0/24  | PrivateRT       |  Private         |    DBSL |

BUCKET 생성
===========

Java Cloud Service(JCS)와 Database System 생성 시에 백업을 위한 Object Storage Bucket이 필요하기 때문에 다른 준비에 앞서서 미리 만들어 두도록 합니다. 특히 JCS에서의 Bucket 사용 관련하여서는 다음과 같은 전제 조건이 있기 때문에 Bucket 생성 시에는 OCI의 IAM 계정을 등록하고 이 계정을 이용하여 Bucket을 생성하도록 합니다.

>
> Bucket 생성은 IDCS Federated User가 아닌 IAM에 등록된 User로 만들어야 하고, 이 계정 정보를 JCS 생성시 Backup Storage Container Credential로 사용한다.
>                                                                                                                                                          
> The user creating the buckets must be a user in Oracle Cloud Infrastructure Identity and Access Management (IAM), not a federated user. 
>

JCS에서 사용할 Bucket을 하나 생성합니다.

![](/assets/images/3tier/image21.png)

Database에서 사용할 Bucket을 하나 생성 합니다.

![](/assets/images/3tier/image22.png)

DB 생성
=======

OCI 콘솔의 **Bare Metal, VM, and Exadata** 메뉴로 들어가서 해당하는 Compartment를 선택 후 **Launch DB System**을 클릭하여 DB를 생성 합니다.

![](/assets/images/3tier/image23.png)

Virtual Machine 옵션을 선택 합니다.

![](/assets/images/3tier/image24.png)

미리 생성해 둔 DB용 Subnet을 선택 합니다.

![](/assets/images/3tier/image25.png)

백업 선택은 옵션이니 사용할 경우에는 위에서 생성해 둔 BUCKET 정보를 이용합니다.

![](/assets/images/3tier/image26.png)

![](/assets/images/3tier/image27.png)

DB가 잘 생성되었는지 확인합니다.

![](/assets/images/3tier/image28.png)

JCS 생성 
=========

이 단계에서는 WebLogic 기반의 PaaS 서비스인 Java Cloud Service를 생성하도록 하겠습니다.

### 사전 준비 사항

-   JCS 용 버킷 생성 : 이전 단계에서 생성함
-   PSM(PaaS Service Manager)가 OCI의 리소스를 사용하기 위한 Policy들이 사전 정의 되어야 함 : 아래 Policy를 참고하여 속해 있는 그룹의 Policy에 추가해 줍니다.

> ![](/assets/images/3tier/image29.png)

Private Subnet을 사용하는 Java Cloud Service(JCS)는JCS 서비스 콘솔을 통해서 생성할 수가 없습니다. 현재까지는 JCS 콘솔에서는 Public Subnet 만 선택할 수 있게 되어 있습니다.

  >
  >   ***참고 문서 :*** [Create an Oracle Java Cloud Service Instance Attached to a Private Subnet on Oracle Cloud Infrastructure](https://docs.oracle.com/en/cloud/paas/java-cloud/jscug/create-instance-attached-private-subnet-oci.html#GUID-36EB6099-C792-4017-A4C7-06F796926AF6)
  >

따라서 Private Subnet을 이용하는 JCS를 생성하고자 할 경우에는 제공되는 REST API를 이용하거나 PSM(PaaS Service Manager) CLI를 이용해야 하는데. 이 가이드에서는 REST API를 이용하는 방법으로 설명합니다.

또한 참고문서(매뉴얼)에서 설명하는 방법으로 생성하게 되면 Database System on Oracle Cloud Infrastucture가 아닌 Database Cloud Service (DBCS on OCI-C)가 생성되기 때문에 **Database System on OCI를 이용한 JCS를 생성하고자 한다면 아래 설명을 따라 수행해야 합니다.**

JCS 서비스를 생성하는 REST API는 다음과 같습니다.

```
curl -X POST https://jaas.oraclecloud.com/paas/api/v1.1/instancemgmt/[IdentityServiceID]/services/jaas/instances \
-u [User Name]:[Password] \
-H 'X-ID-TENANT-NAME: [IdentityServiceID]' \
-H 'Content-Type: application/json' \
-d @requestBodyFile.json
```

여기서 입력 값으로 사용하는 **requestBodyFile.json**이라는 json의 내용이 복잡하고 많은데, 이는 JCS의 콘솔을 통해서 얻을 수 있습니다.

JSON을 얻기 위해 먼저 JCS 콘솔로 이동하여 **Create Instance**를 선택하여 Provision을 위한 정보 들을 입력합니다. Subnet List Box에서는 위에서 언급한 것처럼 Public Subnet만 보일 것이기 때문에 보이는 것 중 어느 것이나 선택해도 무방합니다. **향후 JSON을 얻고 나서 Private Subnet을 사용하는 것으로 수정해 줄 것 입니다.**

![](/assets/images/3tier/image30.png)

**Advanced** 옵션을 선택하고 다음과 같이 선택해 줍니다.

-   Enable Access to Administration Consoles: true
-   Load Balancer: None
-   Oracle Cloud Infrastructure Database : check
-   Compartment Name : 해당 Compartment
-   Database Instance Name : 앞 단계에서 생성해 둔 DB 선택
-   Object Storage Container : JCS 용으로 생성해 둔 Bucket 정보를 입력합니다.
    -   https://swiftobjectstorage.us-ashburn-1.oraclecloud.com/v1/[TENANT-ID]/[BUCKET-명]
    -   [Region 별 REST API Endpoint 참고](https://docs.cloud.oracle.com/iaas/Content/API/Concepts/apiref.htm)
    -   예: https://swiftobjectstorage.us-ashburn-1.oraclecloud.com/v1/apackrsct01/jcs_bucket
-   Object Storage Password : IAM 계정의 **Auth Token**을 입력해 줍니다.

**Next**를 클릭합니다.

![](/assets/images/3tier/image31.png)

Confirm 화면에서 화살표로 표시된 Input Parameter들이 담긴 **JSON 다운받기**를 클릭합니다. 이 서비스를 콘솔에서 생성할 것이 아니므로 여기서는 **Cancel**을 클릭합니다.

![](/assets/images/3tier/image32.png)

**service_payload_jcs서비스명.json** 파일이 생성되었을 것 입니다. 이 파일을 앞서 설명한 REST API의 Input Json으로 사용할 것입니다. 이 파일을 열어서 다음에 설명하는 항목들을 변경해 줍니다.

빨간 박스로 체크해 둔 항목을 수정해 줍니다. Subnet 부분 변경이 핵심이므로 JCS에서 사용할 **Private Subnet의 OCID**로 변경해 줍니다.

![](/assets/images/3tier/image33.png)

파일 수정이 완료되었으면 위에서 설명한 Create Instance REST API를 수행해 줍니다.

스크립트로 만들어서 다음과 같이 수행해도 됩니다. API가 수행되면 JOB ID가 다음과 같이 리턴되고, 리턴된 JOB ID로 JOB의 상태를 확인해 볼 수 있습니다.

![](/assets/images/3tier/image34.png)

- checkJCSJob.sh 스크립트 내용  

```
curl https://jaas.oraclecloud.com/paas/api/v1.1/activitylog/[identityServiceID]/job/[JOBID] \                                            
-u [usename]:[password] \                                                                            
-H 'X-ID-TENANT-NAME: [identityServiceID]'                                           
```

JOB이 실행되고 있는 사항은 JCS 콘솔에서도 확인 할 수 있습니다.

![](/assets/images/3tier/image35.png)

위와 같은 방법으로 AD1, AD2를 사용하는 WAS 용 Subnet에 각각 한 개씩 JCS 서비스를 생성 합니다.

다 완성된 모습은 다음과 같습니다.

![](/assets/images/3tier/image36.png)

Web 서버 생성 및 설정
=====================

JCS 서버의 서비스 앞 단에 WEB 서버 인스턴스를 두고 로드 발란싱을 하도록 구성할 것입니다. 웹서비는 어떤 것을 사용하여도 무방합니다.
여기에서는 Apache를 사용하는 것으로 설명합니다.

Compute 인스턴스 생성
---------------------

웹서버는 IaaS Compute 인스턴스에 구성할 것입니다. 따라서 먼저 Compute Instance를 생성합니다.

AD1과 AD2 에 각각 한개 씩 구성할 예정이므로 먼저 AD1 의 Public Subnet에 인스턴스를 다음과 같이 생성합니다.

![](/assets/images/3tier/image37.png)

![](/assets/images/3tier/image38.png)

![](/assets/images/3tier/image39.png)

생성되었습니다. SSH로 접속할 Public IP를 기억해 두세요.

![](/assets/images/3tier/image40.png)

두 번째 인스턴스는 첫번 째 인스턴스 구성을 다 완료한 후에 Custom Compute Image로 만들어서 생성할 것입니다.

그러므로 먼저 첫번째 인스턴스 구성을 먼저 완료하도록 하겠습니다.

생성된 인스턴스의 Public IP를 이용하여 SSH로 접속합니다.

Apache 설치
-----------

웹서버로 사용할 Apache를 설치하여야 합니다. Yum으로 설치하는 httpd는 shared module이 enabled 된 버전이 아니기 때문에 Apache 소스를 받아서 컴파일하여 설치하는 방법으로 Apache를 설치 해 줍니다.

-   [Apache 설치 참고 문서](https://httpd.apache.org/docs/2.4/install.html)

참고로 저는 다음 옵션을 사용하여 컴파일 하였습니다.

```
sudo ./configure --prefix=/usr/local/apache --enable-module=so --with-pcre=/usr/local/pcre/bin/pcre-config -enable-rewrite --with-mpm=worker --enable-ssl  
```

WebLogic Plugin 설치
--------------------

Plugin 소프트웨어를 다음에서 다운 받습니다.

-   [웹로직 플러그인 다운 받기](https://www.oracle.com/technetwork/middleware/webtier/downloads/index-jsp-156711.html)

![](/assets/images/3tier/image41.png)

다운 받은 zip 파일을 안에는 여러 웹서버 및 OS용 Plugin들이 모두 들어 있습니다. 이 중에서 Linux용 Apache 플러그인 만 웹서버용 Compute 인스턴스로 복사해 줍니다.

```
scp -i privateKey ~/Downloads/fmw_12.2.1.3.0_wlsplugins_Disk1_1of1/WLSPlugins12c-12.2.1.3.0/WLSPlugin12.2.1.3.0-Apache2.2-Apache2.4-Linux_x86_64-12.2.1.3.0.zip opc@[웹서버 Public IP]:/home/opc/wlplugin/
```

복사된 플러그인 zip 파일의 압축을 풉니다. 아래 라이브러리들 중 mod_wl.so 파일을 사용할 것입니다.

![](/assets/images/3tier/image42.png)

Apache 구성
-----------

WebLogic Plugin 구성 상세는 다음을 참고하세요

-   [Configuring the Plug-In for Apache HTTP Server](https://docs.oracle.com/middleware/12213/webtier/develop-plugin/apache.htm#PLGWL395)

이 가이드에서는 **/usr/local/apache** 경로에 Apache가 설치되어 있습니다. conf 디렉토리로 이동하여 httpd.conf 파일을 수정해 줍니다.

-   **WebLogicHost**는 JCS 서비스의 Private IP를 참고하여 변경합니다.

-   애플리케이션은 아직 배포되지 않았으나 미리 설정해 둡니다.

-   JCS가 Private Subnet에 구성되었기 때문에 현재로서는 웹로직 콘솔을 Internet을 통해서 접속할 수가 없습니다. 따라서 웹서버에서 포워딩하는 형태로 웹로직 콘솔에 접속하기 위하여 **/console 설정도 해 줍니다. 여기서의 WebLogicHost는 웹로직 Admin Server의 Host**여야 합니다.

```yaml
 ... 생략                                                          
                                                                   
# Weblogic Module 추가                                           
LoadModule weblogic_module /home/opc/wlplugin/lib/mod_wl_24.so 

... 생략  

<Location /console>
 WLSRequest On   
 WebLogicHost 10.0.3.2  
 WebLogicPort 7001 
</Location> 

<Location /cohweb>   
 WLSRequest On   
 WebLogicHost 10.0.3.2 
 WebLogicPort 8001  
</Location> 

<IfModule mod_weblogic.c> 
 WebLogicHost 10.0.3.2   
 WebLogicPort 8001  
 MatchExpression *.jsp   
 DebugConfigInfo ON    
</IfModule>  

... 생략  
```

Apache를 구동 시킵니다. Apache가 default로 80 포트로 Listen하고 있는데 해당 Port가 Firewall을 통과할 수 있도록 다음 커맨드를 통해서 등록해 줍니다.

![](/assets/images/3tier/image43.png)

Apache 구성이 완료되었습니다.

커스텀 이미지 생성
------------------

AD2에서 사용할 Web 서버 인스턴스도 생성해야 합니다. 구성이 완료된 WEB 인스턴스를 Custom Image로 만들어서 Compute Instance를 생성하겠습니다.

![](/assets/images/3tier/image44.png)

![](/assets/images/3tier/image45.png)

이미지가 생성되고 나면 이 이미지를 사용하여 Compute Instance를 생성합니다.

![](/assets/images/3tier/image46.png)

![](/assets/images/3tier/image47.png)

AD2의 Subnet을 사용합니다.

![](/assets/images/3tier/image48.png)

Compute Instance가 다 생성되고 나면, SSH로 접속하여 Apache 설정만 변경해 주면 됩니다.

AD2의 Apache는 AD2 내의 JCS 서비스를 바라보도록 httpd.conf의 WeblogicHost 정보만 바꿔주고 Apache를 기동시켜 줍니다.

Coherence 설치 및 구성
======================

이제는 Coherence 구성을 진행하도록 하겠습니다. 
두 AD 간의 두개의 JCS 서비스 간 HTTP Session 공유를 위하여 Coherence를 사용할 예정입니다. JCS 내에도 Managed Coherence 서버를 구성할 수 있지만 서로 다른 WebLogic Domain간의 세션 공유를 위해서 별도의 Coherence Cluster를 구성하도록 하겠습니다.

Coherence 구성을 위해서 두개의 Compute Instance가 필요합니다.

Coherence용 Compute Instance를 AD1에 먼저 구성한 후 구성이 완료된 후에 WEB의 경우와 마찬가지로 Custom Image를 생성하여 AD2에도 구성하도록 하겠습니다. Coherence는 Private Subnet에 만들어져야 합니다.

Compute Instance를 만드는 과정을 생략하도록 하겠습니다. 생성되고 난 후의 모습은 다음과 같습니다.

![](/assets/images/3tier/image49.png)

인스턴스가 생성된 후 SSH로 Coherence 노드에 접속합니다. Private Subnet에 속해 있기 때문에 WEB 인스턴스를 통해서 Coherence 인스턴스에 접속합니다.

![](/assets/images/3tier/image50.png)

Coherence 설치
--------------

Stand alone으로 설치하는 Coherence는 JCS내의 Coherence 버전과 동일해야 합니다. JCS 콘솔에서 WebLogic 버전을 확인합니다.

![](/assets/images/3tier/image51.png)

동일 버전의 Coherence SW를 다운 받습니다.

-   [Coherenc Download](https://www.oracle.com/technetwork/middleware/coherence/overview/index.html)

![](/assets/images/3tier/image52.png)

Coherence 설치 과정은 다음을 참고하시기 바랍니다. 여기에서는 설치 이후의 설정 과정만 설명하도록 하겠습니다.

-   [Installing Oracle Coherence for Java](https://docs.oracle.com/middleware/1212/coherence/COHDG/gs_install.htm#COHDG5660)

이 가이드에서는 coherence가 다음 위치에 설치 되었습니다.

![](/assets/images/3tier/image53.png)

Coherence 기동을 위한 스크립트를 작성합니다.

-   **Coherence*Web**을 사용할 것이기 때문에 session cache 설정을 해 줍니다.

-   Cloud 환경에서 multicast가 지원되지 않기 때문에 **unicast를 사용하는 WKA(Well Known Address)**를 설정해 줍니다. WKA 상세 설명과 여러 개의 WKA를 설정하는 방법은 다음 문서를 참고 하세요.

    -   [Using Well Known Addresses](https://docs.oracle.com/middleware/12213/coherence/COHDG/setting-cluster.htm#COHDG5454)

```
java -server -Xms512m -Xmx512m -cp /home/opc/fmw/coherence/lib/coherence.jar:/home/opc/fmw/coherence/lib/coherence-web.jar -Dcoherence.mode=prod -Dcoherence.management.remote=true -Dcoherence.session.localstorage=true -Dcoherence.enable.sessioncontext=true -Dcoherence.cacheconfig=default-session-cache-config.xml -Dcoherence.cluster=mycoh -Dcoherence.wka=10.0.3.3 com.tangosol.net.DefaultCacheServer
```

스크립트를 실행 시켜 Coherence를 기동 시킵니다. 아래와 유사한 로그가 보여질 것입니다.

![](/assets/images/3tier/image54.png)

coherence1 구성이 완료 되었으면 Custom Image를 생성해서 두번째 Instance를 생성합니다.

생성하는 과정은 생략하도록 하겠습니다. 생성된 후의 모습은 다음과 같습니다.

![](/assets/images/3tier/image55.png)

Coherence 구성은 완료된 상황이기 때문에 Coherence 설치 디렉토리로 이동하여 Coherence 구동 스크립트를 실행 시켜 줍니다.

JCS에서 Coherence 설정
======================

이 단계에서는 JCS의 세션을 Coherence에 저장하도록 설정하는 과정을 수행합니다. 
그렇게 하기 위해서는 WebLogic의 Managed Server들을 앞 단계에서 설정한 Coherence Cluster에 **Join** 되게 해 주어야 합니다.

이 설정은 WebLogic Console을 통해 수행해야 하는데 JCS가 Private Subnet에 생성 되었기 때문에 JCS 메뉴에 있는 "WebLogic Console" 열기를 통해서는 콘솔에 접속할 수 없습니다.

따라서 앞서 Web 설정에서 /console Location을 추가 해 두었기 때문에 웹서버(Apache)의 Public IP를 통해 웹로직 콘솔에 접속합니다.

-   http://[웹서버-Public-IP]/console

JCS의 WebLogic Cluster들은 Cohernece Cluster의 기본 구성을 사용하도록 설정되어 있습니다. 여기에서는 Default로 구성되는 Coherence를 사용하지 않고 Stand Alone으로 구성된 Coherence에 Join할 것이 때문에 아래 화면에서 설명하는 방법을 따라 Coherence Cluster에서 WebLogic Cluster를 멤버에서 제거해 줍니다.

![](/assets/images/3tier/image56.png)

![](/assets/images/3tier/image57.png)

![](/assets/images/3tier/image58.png)

멤버에서 잘 제외 되었는지 확인합니다.

![](/assets/images/3tier/image59.png)

서버 메뉴로 이동하여서 각 WebLogic Managed Server가 구동할 때 Coherence Cluster의 멤버로 Join할 수 있도록 WebLogic Managed Server의 Start Script 부분을 수정해 줍니다.

![](/assets/images/3tier/image60.png)

![](/assets/images/3tier/image61.png)

기본적으로 설정되어 있는 Start Script의 Arguments를 확인합니다.

![](/assets/images/3tier/image62.png)

이 Argument를 다음과 같이 수정합니다.

![](/assets/images/3tier/image63.png)

Argument 전체 부분을 변경하면 안되고, 아래 빨간색으로 표시된 부분만 변경 될 수 있도록 합니다. **기존 Argument에서 -Dtangosol.coherence.transport.reliable=tmb -Dtangosol.coherence.socketprovider=tcp 이 두 옵션은 제외해야 하니 주의 하세요.**

```
-Xms256m -Xmx8192m -XX:MaxMetaspaceSize=2048m -Djdk.tls.rejectClientInitiatedRenegotiation=true -Xloggc:/u01/data/domains/privJCS2_domainGC_privJCS2_server_1.log -XX:+UseGCLogFileRotation -XX:NumberOfGCLogFiles=4 -XX:GCLogFileSize=5m -Dweblogic.rjvm.enableprotocolswitch=true -Djava.net.preferIPv4Stack=true -Doracle.security.jps.db.connect.max.retry=720 -Doracle.security.jps.db.connect.retry.interval=10000 -Djps.auth.debug=false -DUSE_JAAS=false -Djps.combiner.optimize.lazyeval=true -Djps.combiner.optimize=true -Djps.authz=ACC -Djps.subject.cache.key=5 -Djps.subject.cache.ttl=600000 -Dweblogic.security.SSL.minimumProtocolVersion=TLSv1.2 -XX:+UnlockCommercialFeatures -XX:+FlightRecorder -verbose:gc -XX:+PrintGCDetails -XX:+PrintGCTimeStamps -Dweblogic.data.canTransferAnyFile=true -Djava.security.egd=file:/dev/./urandom -XX:CompileThreshold=8000 -XX:ReservedCodeCacheSize=1024m -Doracle.jdbc.fanEnabled=false -Dcoherence.session.localstorage=false -Dcoherence.enable.sessioncontext=true -Dcoherence.cluster=mycoh -Dtangosol.coherence.cluster=mycoh -Dcoherence.wka=10.0.3.3
```

스크립트 변경이 완료되었으면 웹로직 서버를 재기동해야 합니다. 아래와 같이 **Control** 탭으로 이동하여 해당 서버만 재기동 합니다.

![](/assets/images/3tier/image64.png)

![](/assets/images/3tier/image65.png)

서버가 정상적으로 기동되고 나면 세션 테스트을 위핸 샘플 애플리케이션을 배포해야 합니다.

애플리케이션 배포
=================

세션 테스트를 위햔 애플리케이션은 세션을 사용하는 어떤 애플리케이션을 이용하여도 무방합니다. 애플리케이션에서 Coherence*Web을 사용하게 하기 위해서는 **weblogic.xml** 에 다음과 같은 설정만 추가해 주면 됩니다.

```xml
<wls:session-descriptor>
  <wls:persistent-store-type>coherence-web</wls:persistent-store-type>
</wls:session-descriptor>
```


여기에서는 간단한 샘플 애플리케이션(cohweb)을 이용할 것입니다.

웹로직 콘솔의 **Deployments** 메뉴에서 애플리케이션을 배포합니다.

![](/assets/images/3tier/image66.png)

로컬 환경에서 JCS Cloud 환경으로 애플리케이션을 **Upload**해 줍니다.

![](/assets/images/3tier/image67.png)

![](/assets/images/3tier/image68.png)

![](/assets/images/3tier/image69.png)

![](/assets/images/3tier/image70.png)

애플리케이션을 Cluster에 배포합니다.

![](/assets/images/3tier/image71.png)

![](/assets/images/3tier/image72.png)

![](/assets/images/3tier/image73.png)

![](/assets/images/3tier/image74.png)

애플리케이션이 배포되고 나서는 애플리케이션을 시작 시켜야 서비스를 할 수 있습니다.

![](/assets/images/3tier/image75.png)

애플리케이션이 서비스 가능 상태가 되었습니다.

![](/assets/images/3tier/image76.png)

AD2의 JCS도 위와 마찬가지로 설정해 줍니다. 동일하게 애플리케이션도 배포해 줍니다.

Load Balancer 생성
==================

마지막으로 Load Balancer를 생성해 줍니다. 이 Load Balancer는 web1, web2를 Round Robin으로 발란싱하게 구성합니다.
Load Balancer는 **Sticky Session**을 지원하도록 설정할 수도 있지만, HA 태스트를 위해서 web1, web2로 Round Robin으로 돌리며 테스트를 진행할 예정이기 때문에 Sticky Session으로 설정하지 않습니다.

![](/assets/images/3tier/image77.png)

![](/assets/images/3tier/image78.png)

생성이 되고 나면 다음과 같이 보여집니다.

![](/assets/images/3tier/image79.png)

Test
====

이제 구성된 Load Balancer를 통해서 Application을 테스트 해 보겠습니다. Load Balancer의 Public IP로 접속합니다.

-   http://[Load-Balancer-Public-IP]/cohweb

다음과 같은 화면이 보일 것 입니다.

![](/assets/images/3tier/image80.png)

세션을 몇개 추가해 봅니다.

![](/assets/images/3tier/image81.png)

브라우저의 Refresh 버튼을 클릭하여 request를 계속 보내 봅니다. 다른 JCS Server 접속된 것을 확인할 수 있고, 세션이 유지되는 것을 확인 할 수 있습니다.

![](/assets/images/3tier/image82.png)

아래는 3 Tier의 각 컴포넌트를을 차례로 Down 시키면서 세션이 유지되는지 테스트하는 과정입니다.
각 Layer의 컴포넌트를 하나씩 내려 보면서 세션이 계속 유지되는지 테스트 해보시면 HA 아키텍처를 이해하는데 더욱 도움이 될 것입니다.

1. 먼저 정상 상황에서 서로 다른 두 WebLogic Server간의 세션 공유가 됨을 확인 합니다.
2. Coherence (Http Serssion 서버) 노드를 하나 다운 시키고 나서 세션이 공유 됨을 확인합니다.
3. 제일 앞단의 로드 발란서에서 웹서버 사이의 로그 발란싱 정책은 Round Robin 입니다. 세션에 상관없이 이 구간에서는 Round Robin으로 동작하지만 웹서버 WebLogic Server 구간은 Sticky Session이 적용됩니다. 따라서 현재 접속되어 있는 WebLogic Server 정보를 확인하고 해당 서버를 Down 시킴으로써, 요청이 다른 서버로 이동되게 합니다. 이 경우에도 정상적으로 세션이 유지됨을 확인 합니다.
4. 마지막으로 웹서버를 다운 시켜서 로드 발란서에서 살아있는 웹서버로만 요청이 이루어짐을 확인 합니다.

<iframe width="710" height="410" src="https://www.youtube.com/embed/EiXjE82FuCI" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

이상으로 모든 구성이 완료되었습니다. 
