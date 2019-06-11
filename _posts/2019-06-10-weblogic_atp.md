---
layout: post
title:  WebLogic에서 JDBC를 이용하여 Autonomous Database 연결하기
categories: WebLogic
tags: [WebLogic, Autonomous Database, ATP, JDBC, Wallet]
---

Oracle **Autonomous Database (ADW, ATP)**를 연결하기 위해서는 **Wallet**이라는 인증 정보를 담고 있는 지갑이 필요하다. WebLogic Server에서 Database 연결을 위해서는 JDBC를 이용하는데, 아래에서는 Autonomous Database를 연결하기 위한 JDBC 설정 방법에 대해 설명 할 것이다. 

# 사용 환경
 - WebLogic Server 12.2.1.3.0
 - JDK 1.8.0_192
 - Autonomous Database : Autonomous Transaction Processiong (ATP)

# 사전 준비 사항

### JDBC Driver Upgrade

아래 문서에서 보듯이 WebLogic Server 12.2.1.3에서 Autonomous Database 연결을 위해서는 JDBC Driver Upgrade가 필요하다.

- [Java Connectivity with Autonomous Database (ATP or ADW) using 19c and 18.3 JDBC](https://www.oracle.com/technetwork/database/application-development/jdbc/documentation/atp-5073445.html#prereq)

JDBC Driver 18.3 버전을 다운 받아서 아래와 같이 WebLogic 디렉토리에 복사한다.
- [JDBC Driver Download](https://www.oracle.com/technetwork/database/application-development/jdbc/downloads/jdbc-ucp-183-5013470.html)

![Alt text](/assets/images/atp/05_jdbc.png)

### Wallet Download
Autonomous Database 연결을 위한 Wallet을 다운 받는다. 

![Alt text](/assets/images/atp/06_wallet_down.png)

**wallet**을 담고 있는 **zip** 파일이 다운로드 될 것이다. 이 파일을 원하는 위치에 복사하고 압축을 풀어준다.

![Alt text](/assets/images/atp/04_wallet_loc.png)

다운받은 zip 내에는 **tnsnames.ora** 파일이 들어있다. 
tnsnames.ora 파일은 다음과 같은 모습일 것이다.

![Alt text](/assets/images/atp/03_tnsnames.png)

이 서비스 중에서 원하는 작업 유형에 따라 서비스 명을 선택해서 접속하면 된다.
아래 예제에서는 **tp** suffix 서비스 명을 사용할 것이다. 사전 정의된 서비스 명의 사용 유형은 아래를 참고하면 된다.

- [Predefined Database Service Names for Autonomous Transaction Processing](https://docs.oracle.com/en/cloud/paas/atp-cloud/atpug/connect-predefined.html#GUID-9747539B-FD46-44F1-8FF8-F5AC650F15BE)

# JDBC Connection 생성

WebLogic Server Console에 접속해서 DataSouce를 생성해 줄 차례이다. WebLogic에서는 여러 타입의 DataSource를 제공하는데 여기에서는 **Generic** DataSource와 **UCP** DataSource를 사용할 것이다.

## Generic DataSource
**Generic DataSource** 타입을 선택한다.

![Alt text](/assets/images/atp/01_generic1.png)

원하는 DataSource Name과 JNDI Name을 입력한다.

![Alt text](/assets/images/atp/01_generic2.png)

JDBC Driver 유형을 선택한다. 

![Alt text](/assets/images/atp/01_generic3.png)

트랜잭션 옵션을 선택한다. 

![Alt text](/assets/images/atp/01_generic4.png)

**Database Name과 Host Name 란에 아무 값이나 입력한다. 이 값은 다음 페이지에서 변경해 줄 것이기 때문에 이 화면에서의 입력값은 의미 없다.** 

![Alt text](/assets/images/atp/01_generic5.png)

이전 화면에서 입력한 정보를 바탕으로 Connection URL이 생성되었을 것이다. 

![Alt text](/assets/images/atp/01_generic6.png)

이 URL을 다음과 같은 형태로 변경해 줘야 한다.
TNS_ADMIN에는 Wallet이 위치한 Location을 적어준다.


|Pattern | jdbc:oracle:thin:@dbname_tp?TNS_ADMIN=/users/test/wallet_dbname/ |
|에제 | jdbc:oracle:thin:@demo_tp?TNS_ADMIN=/u01/oracle/wallet |


변경 후에 **Test Configuration** 버튼을 클릭하여 연결이 잘 되는지 확인해 본다.

![Alt text](/assets/images/atp/01_generic7.png)

DataSource 배포할 WebLogic Instance를 선택한다.

![Alt text](/assets/images/atp/01_generic8.png)

DataSource가 배포되고 Connection이 연결되어 있는 상태를 모니터링 할 수 있을 것이다.

![Alt text](/assets/images/atp/01_generic9.png)

## UCP DataSource

이번에는 UCP 타입의 DataSource를 만들어 볼 것이다. 설정 화면이 Generic DataSource와 약간 다른 것을 볼 수 있을 것이다.

**UCP DataSource** 타입을 선택한다.

![Alt text](/assets/images/atp/02_ucp1.png)

원하는 DataSource Name과 JNDI Name을 입력하고 JDBC Driver 유형을 선택한다. 

![Alt text](/assets/images/atp/02_ucp2.png)

URL 부분에 위 예제에서 사용했던 Connection URL String을 입력한다.

![Alt text](/assets/images/atp/02_ucp3.png)

UCP 관련 옵션을 설정한다. 이 예제에서는 Pool Size 만 설정하였다.

![Alt text](/assets/images/atp/02_ucp4.png)

**Test Configuration** 버튼을 클릭하여 연결이 잘 되는지 확인해 본다.

![Alt text](/assets/images/atp/02_ucp5.png)

테스트에 성공하였다.

![Alt text](/assets/images/atp/02_ucp6.png)

모니터링에서도 Connection이 잘 만들어진 것을 확인해 볼 수 있다. 

![Alt text](/assets/images/atp/02_ucp7.png)

## 참고 자료 
- [Java Connectivity with Autonomous Database (ATP or ADW) using 19c and 18.3 JDBC](https://www.oracle.com/technetwork/database/application-development/jdbc/documentation/atp-5073445.html#prereq)





