---
layout: post
title:  Docker Compose를 사용한 WebLogic, MySQL 개발 환경
categories: WebLogic
tags: [WebLogic, MySQL, Docker, Docker Compose] 
---

Local PC 환경에서 WebLogic을 데이터베이스와 함께 테스트하는 환경을 구축할 필요가 있을 때, 내 PC에 각종 소프트웨어들을 다 설치할 필요 없이 표준 이미지로 제공되는 Docker Image 이용하여 구동만 하면 매우 편리할 때가 많다. 

여기에서는 MySQL을 DB로 사용하는 WebLogic 환경을 구축하는 간단한 방법을 설명할 것이다.

# Docker-Compose 파일 생성

Docker Compose 구성 파일에서 사용하는 필요한 내용들을 미리 준비해 둔다.

## 필요 파일
 - Docker Images : MySQL과 WebLogic Image는 [Docker Hub](http://dockerhub.com)에서 받을 수 있다.
 - MySQL JDBC Driver : [Download Connector/J](https://dev.mysql.com/downloads/connector/j/) 
     - 다운 받은 jar 파일을 작업 디렉토리에 복사한다. 
     - 작업 디렉토리는 WebLogic Container에 Volume으로 Attach 할 것이다.
- domain.properties 파일 : 도메인 계정 정보를 담는 파일이다. 아래 내용을 참고해서 작업 디렉토리 내에 생성해 둔다.

### domain.properties

```
username=weblogic
password=welcome1
```

### docker-compose.yml

```yml
version: '3'

services:
  mysql:
    image: mysql
    container_name : some-mysql
    environment:
      MYSQL_ROOT_PASSWORD: welcome1
      MYSQL_DATABASE: mysql
    volumes:
      - ./mysqldata:/var/lib/mysql
      - .:/mnt/tmp
    ports:
      - "3306:3306"
      - "33060:33060"
    hostname: mysql
    command: --default-authentication-plugin=mysql_native_password
  weblogic:
    image: store/oracle/weblogic:12.2.1.3-dev
    container_name: myweblogic
    environment:
      PRE_CLASSPATH: "/u01/oracle/properties/mysql-connector-java-8.0.16.jar:${PRE_CLASSPATH}"
    depends_on:
      - mysql
    volumes:
      - .:/u01/oracle/properties
    ports:
      - "7001:7001"
      - "9002:9002"
    stdin_open: true
    tty: true
```
파일이 완료되었으면 컨테이너를 구동한다.

```
> docker-compose up -d
```

# MySQL User 생성
애플리케이션에서 사용할 DB User를 생성한다. 이 User는 WebLogic에서 DataSource를 생성할 때 사용할 것이다.
현 작업 디렉토리가 MySQL Container에 Volume으로 연결되어 있으므로 사용자를 생성할 **createuser.sql** 파일을 현 디렉토리 내에 다음 내용으로 생성한다. 별도의 필요한 DDL/DML이 있다면 여기에 추가하면 좋을 것이다.

```sql
CREATE user 'user1' IDENTIFIED BY 'welcome1';
GRANT ALL PRIVILEGES ON mysql.* TO 'user1'@'%';

```

다음 command를 실행하면 사용자가 추가 될 것이다. 새롭게 생성되는 사용자는 **mysql_native_password** plugin을 기본하도록 설정되어 있다.

```
> docker exec -it some-mysql sh -c "mysql -uroot -pwelcome1 mysql < /mnt/tmp/createuser.sql"
```

![](/assets/images/mysql/01_mysql_user.png)

# DataSource 생성

WebLogic Console에 접속해서 DataSource를 생성한다.

[https://localhost:9002/console](https://localhost:9002/console)로 접속한다.

![](/assets/images/mysql/02_ds1.png)

적당한 드라이버를 선택한다.

![](/assets/images/mysql/02_ds2.png)

사용자 생성 시 사용했던 Password를 입력한다.

![](/assets/images/mysql/02_ds3.png)

Connection을 Test 해본다. 입력이 잘 되었다면 성공할 것이다.

![](/assets/images/mysql/02_ds4.png)

**Finish**를 클릭하면 DataSource가 생성된다.

이제 애플리케이션에서 Connection을 얻어다 사용하기만 하면 된다.


