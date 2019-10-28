---
layout: post
title:  Private Load Balancer를 이용한 MySQL Read Replica 로드 발란싱 하기
categories: Cloud
tags: [Oracle Cloud, OCI, MySQL, Read Replica, Private Load Balancer]
---

이 문서에서는 **Private Subnet** 상에 MySQL Master와 2개의 Slave(**Read Replica**)를 구성하고, Read Replica들 간의 로드발란싱을 위해 **Private Load Balancer**를 통해 접속하는 방법에 대해 기술 할 것이다.

여기에서는 MySQL의 Replication을 구성하는 내용에 대해서는 다루지 않을 것이다. 이 내용은 MySQL 공식 페이지를 참고하면 될 것이다.

아래 그림과 같이 Private Subnet에 Master 노드와 두개의 Slave를 구성해 두었다.

![](/assets/images/mysql/replica/architecture.png)

> MySQL은 Community Edition을 이용하였다. 설치 관련은 다음을 참고한다.
> ![](/assets/images/mysql/replica/01_install.png)

구성된 MySQL 각 노드에 접속하여서 현재 상태를 살펴본다.
Replication 구성은 **www** Database에 대해서 구성되어있다. 

## Master

Public Subnet에 존재하는 MySQL Client를 통해 Master 노드로 Remote로 접속한다.
Master Node의 Private IP는 10.0.2.2 이다.

> ![](/assets/images/mysql/replica/master.png)

## Slave1

Slave 노드에 Remote로 접속해 본다. Slave1의 Private IP는 10.0.2.3 이다.

> ![](/assets/images/mysql/replica/slave1.png)

## Slave2

Slave 노드에 Remote로 접속해 본다. Slave2의 Private IP는 10.0.2.4 이다.

> ![](/assets/images/mysql/replica/slave2.png)

# Private Load Balancer 생성

Slave를 Private Load Balancer를 통해 접속 할 수 있도록 Private Subnet에 Private Load Balancer를 다음과 같이 구성한다.

> ![](/assets/images/mysql/replica/private_lb.png)

Backend 서버 연결과 Health Check에 사용되는 포트를 3306으로 설정한다.

> ![](/assets/images/mysql/replica/private_lb2.png)

MySQL Client에서 3306을 통해 로드발란서에 연결하기 위해서 Listener 포트도 3306으로 구성한다.

> ![](/assets/images/mysql/replica/private_lb3.png)

로드발란서가 생성되었으면 다음과 같이 Load Balancer를 통해 Slave에 접속해 본다. Round Robin 정책을 사용했기 때문에 접속될 때 마다, Slave가 교대로 접속될 것이다.

위에서 생성한 Private Load Balancer의 Private IP는 10.0.2.7 이다.

> ![](/assets/images/mysql/replica/connect_via_lb.png)

이제 Replication이 잘 이루어지는지 Master에서 Data를 Insert 해보자

> ![](/assets/images/mysql/replica/master_insert.png)

각 Slave에서 확인해보면 방금 Master에서 insert한 데이터가 보이는 것을 확인할 수 있다.

## Slave1

> ![](/assets/images/mysql/replica/slave1_result.png)

## Slave2

> ![](/assets/images/mysql/replica/slave2_result.png)


# 참고 자료
- [MySQL Replication](https://dev.mysql.com/doc/refman/8.0/en/replication.html)