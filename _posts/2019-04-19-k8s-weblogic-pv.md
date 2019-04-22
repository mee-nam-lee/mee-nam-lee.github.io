---
layout: post
title:  Kubernetes에 WebLogic Domain 올리기 - Persistent Volume 사용
categories: cloud
tags: [WebLogic, Kubernetes, Oracle Kubernetes Engine]
---

이 전 예제에서는 Docker Image 내에 WebLogic Binary와 Domain 구성 정보가 모두 들어있는 Sample을 이용하였다. 
여기서는 WebLogic Binary는 Docker Image 내에 존재 하지만 Domain 정보는 Persistent Volume을 사용하여 외부에 구성하는 예제를 살펴볼 것 이다.

이전 글에서 사용한 **WebLogic Operator**의 Sample에 Persistent Volume을 사용하는 예제도 함께 들어있다. 
이 예제를 이용하여 Persistent Volume을 **Oracle Cloud Infrastructure**의 **File System**을 사용하도록 구성하는 방법으로 설명할 것이다.

# Prerequistes 

- Oracle Kubernetes Engine(OKE) 
- Oracle File System

# 이전 과정에서 했던 작업 돌아보기

이전 과정에서 수행 했던 작업들은 환경에 그대로 존재한다면 다시 수행할 필요 없이 그대로 이용하면 된다.

- Oracle Kubernetes Engine(OKE) 접속 확인
- WebLogic Operator 설치
- WebLogic Image 준비 : 이 예제에서는 **store/oracle/weblogic:12.2.1.3** 이미지를 Kubernetes WebLogic Operator Job 생성 시 직접 당겨오기 때문에 미리 준비해 둘 필요는 없다.
- WebLogic Domain 준비 : 이 예제에서는 **Kubernetes WebLogic Operator Job**이 돌면서 Persistent Volume에 Domain 구성 파일들을 생성한다.
- WebLogic Kubernetes Operator 샘플 Git Clone 
- WebLogic Domain Credential : 같은 id/password를 사용한다면 이전에 만든것을 재사용한다.

# File System 구성
도메인 구성 스크립트를 수행하기 전에 File System이 먼저 구성되어 있어야 한다.
**OKE**를 위해 구성된 **VCN (Virtual Cloud Network)**에 File System을 위한 별도의 **Subnet**을 구성한다.

![](/assets/images/kubeweblogic2/01_subnet.png)

![](/assets/images/kubeweblogic2/01_subnet2.png)

**File Storage** 메뉴로 이동해서 **File System**을 생성을 클릭한다.

![](/assets/images/kubeweblogic2/02_filestorage.png)

**Hide Details** 부분에 **Edit Details**가 있었을 것이다. 이 버튼을 클릭하여 File System 이름을 다음처럼 바꿔준다.

![](/assets/images/kubeweblogic2/03_filesystem1.png)

**Mount Target** 정보에서도 **Edit Details**를 클릭하여 Mount Target을 생상하도록 하고, Mount Target이 앞에서 생성한 **Subnet**에 생성되도록 해준다.

![](/assets/images/kubeweblogic2/03_filesystem2.png)

File System과 Mount Target이 생성되고 나면 다음과 같이 보여진다.
Mount Target의 **IP**는 다음 과정에서 **Persistent Volume**을 설정할 때 사용할 것이니 잘 기록해 둔다.

![](/assets/images/kubeweblogic2/04_mt.png)

**Export Path**가 아래와 같이 설정되었다. 이 Path도 **Persistent Volume** 설정 시에 필요한 정보이다. 이미 예제에서 사용하는 path (/shared)와 동일하게 생성하였기 때문에 다르게 생성하였을 경우에만 나중 사용을 위해 기록해 둔다.

![](/assets/images/kubeweblogic2/05_export.png)

OKE worker node에서 Mount Target에 접근하기 위해서는 **Security List**에 몇 가지 Rule 구성해 줘야 한다. 구성 방법은 아래 매뉴얼을 참고하면 된다.

- [Configuring VCN Security List Rules for File Storage](https://docs.cloud.oracle.com/iaas/Content/File/Tasks/securitylistsfilestorage.htm?Highlight=mount%20target%20security%20list)

여기에서는 **편의상** Mount Target이 속한 Subnet(nfs-sn)이 Worker Node subnet이 사용하는 **Security List**와 같은 것을 사용하도록 설정하였기 때문에 이미 모든 Worker Node들에서 부터의 모든 Protocol을 받을 수 있게 되어있다.

![](/assets/images/kubeweblogic2/06_securitylist.png)

Security List를 다르게 해 줬다면 문서에서 설명하는 대로 Rule들을 지정해 줘야 한다.

# PV 사용 도메인 구성

WebLogic Operator의 Sample Script 디렉토리로 이동한다.

Domain을 생성하기 앞서서 **Persistent Volume**을 먼저 생성해야 하기 때문에 아래 단계를 수행한다.

```
> cd weblogic-kubernetes-operator/kubernetes/samples/scripts/create-weblogic-domain-pv-pvc
```
위 디렉토리에서 **create-pv-pvc-inputs.yaml** 파일을 열어 다음 부분을 수정한다.

```yaml
weblogicDomainStorageType: NFS

# Mount Target의 IP
weblogicDomainStorageNFSServer: 10.0.13.3

# Mount Target의 Export Path, 위애서 설명한 데로 생성했다면 그대로 두면 된다.
weblogicDomainStoragePath: /shared
```

수정된 input 파일을 기반으로 다음 스크립트를 실행한다.
```
> ./create-pv-pvc.sh -i create-pv-pvc-inputs.yaml -o ./pv-pvc-output –e
```

![](/assets/images/kubeweblogic2/07_pv_pvc.png)

output 디렉토리로 이동하여 생성된 yaml 파일을 확인하고 kubectl로 적용한다.

```
> kubectl apply -f ./domain1-weblogic-sample-pv.yaml
> kubectl apply -f ./domain1-weblogic-sample-pvc.yaml
```

![](/assets/images/kubeweblogic2/08_apply_pv_pvc.png)

잘 생성되었는지 확인한다.

![](/assets/images/kubeweblogic2/09_get_pv_pvc.png)

이제 도메인을 생성할 단계이다.

도메인 생성을 위해 해당 스크립트가 위치한 디렉토리로 이동한다.

```
> cd weblogic-kubernetes-operator/kubernetes/samples/scripts/create-weblogic-domain/domain-home-on-pv
```
디렉토리에서 **create-domain-job-template.yaml** 파일을 열어 Docker Hub의 **Repository Secret** 정보를 추가해 준다.  

```yaml
      imagePullSecrets:
        # secret name으로 변경
        - name: mnleecred
```
탬플릿을 저장한 후 도메인 생성을 위한 스크립트를 실행할 것이다.

**create-domain.sh**에는 WebLogic Domain 구성 파일들을 **Persistent Volume**에 만드는 작업을 수행하는 Job을 생성하고 실행하는 부분이 들어가 있다. 이 Job에서는 Docker Hub의 **store/oracle/weblogic:12.2.1.3** 이미지를 당겨와서 Domain 구성 파일을 만들게 되는데 이 이미지를 Pull하기 위해서는 Docker Hub로 들어가서 **License Agreement**를 Agree해주는 과정이 먼저 되어 있어야 한다.

![](/assets/images/kubeweblogic2/10_proceed_checkout.png)

![](/assets/images/kubeweblogic2/10_proceed_checkout2.png)

![](/assets/images/kubeweblogic2/10_proceed_checkout3.png)

**위처럼 pull 커맨드가 보여진 후 도메인 생성 스크립트를 수행한다.**

> 참고 사항
> shared 디렉토리의 내용을 확인하기 위해 Public Subnet에 Bastion Host를 하나 생성하고 Shared File System을 Mount하여 내용을 확인해 보면 이해에 도움이 될 것이다.
>
> Bastion Host가 생성되었다면 다음 Command를 수행하여 Shared File System을 Mount한다.
> ```
> sudo mount 10.0.13.3:/shared /mnt/shared
> ```
> 디렉토리에 아직은 아무 내용도 없는 것을 볼 수 있다.
>
>![](/assets/images/kubeweblogic2/11_empty_shared.png)


이제 도메인 생성 스크립트를 실행한다. Job이 수행되고 /shared 디렉토리에 Domain을 생성하고 Job이 종료될 것이다.

```
./create-domain.sh -i create-domain-inputs.yaml -o ./create-domain-output
```

![](/assets/images/kubeweblogic2/12_job_completed.png)

Job이 잘 수행되었는지는 pod 상태로도 확인할 수 있다.

![](/assets/images/kubeweblogic2/12_job_completed2.png)

> 
> 이 시점에서 Shared 디렉토리를 살펴보면 Domain 디렉토리와 관련 파일들이 생성되어 있음을 볼 수 있다.
>
>![](/assets/images/kubeweblogic2/13_shared_domain.png)
>

이제 도메인을 기동시켜 보는 단계만 남아있다. 
**create-domain-output** 디렉토리로 이동하여 다음을 실행한다.

```
> cd ./create-domain-output/weblogic-domains/domain1
> kubectl apply -f domain.yaml
```
Admin Server와 Managed Server 두개가 Running 중인 것을 확인할 수 있다.

![](/assets/images/kubeweblogic2/13_domain_running.png)

이전 과정에서 만들어 둔 LoadBalancer Serivce를 사용해서 외부에서 접속해 볼 수 있다.

![](/assets/images/kubeweblogic2/14_ex_service.png)

# 애플리케이션 배포

이전 과정에서 처럼 WebLogic Console을 통해 애플리케이션을 Upload하지 않고, **Persistent Volume**으로 사용하는 domain 디렉토리에 애플리케이션을 복사하고, 모든 Managed Server가 해당 애플리케이션을 바라보도록 배포해 볼 것이다.
먼저 배포할 애플리케이션을 **/shared**의 특정 위치로 복사한다. 

복사할 때는 Public Subnet에서 구동중인 **Bastion Server**를 활용하여 공유되고 있는 /shared에 접속하는 방법을 통하면 된다.

![](/assets/images/kubeweblogic2/15_app_copy.png)

복사가 되어었으면 Admin Server Service의 External IP를 통해 WebLogic Console에 접근하고 애플리케이션을 배포해 보자.
domain 디렉토리 내에 복사해 둔 애플리케이션이 보일 것이다.

![](/assets/images/kubeweblogic2/16_app_deploy1.png)

애플리케이션을 cluster에 배포하고, 배포 속성 중에 애플리케이션 참조 위치를 특정 디렉토리로 하는 옵션을 선택한 후 애플리케이션 배포를 완료한다.

![](/assets/images/kubeweblogic2/16_app_deploy2.png)

애플리케이션을 **start** 시키고, **domain1-cluster1-lb-ext** 서비스의 External IP를 통해 애플리케이션을 호출해 본다.

![](/assets/images/kubeweblogic2/17_test.png)

# 확장 / 축소

도메인의 Cluster에서 Managed Server의 수를 늘리거나 줄이기 위해서는 다음과 같이 domain을 정보를 수정하면 된다. kubectl의 edit를 이용해서 cluster의 **replica** 수를 변경해 볼 것이다.

```
> kubectl edit domain domain1
```
기본 editor 창으로 들어갈 것이다. replica를 찾아서 수를 변경한다.

![](/assets/images/kubeweblogic2/18_scale1.png)

기본 2로 되어있던 수를 3으로 변경하고 저장 (wq)한 후 editor를 빠져 나간다.

![](/assets/images/kubeweblogic2/18_scale2.png)

domain이 변경되었다. 

잠시 후 pod를 확인하면 domain1-managed-server3이 생성되고 Running 중인 것을 확인 할 수 있다.

![](/assets/images/kubeweblogic2/19_managed3.png)

애플리케이션에서도 새창을 열어 호출해 보면 새롭게 구동된 managed-server3으로 로드 발란싱 되는 것을 볼 수 있다.

![](/assets/images/kubeweblogic2/20_managed3_test.png)

# 로그 확인

Domain 관련 각종 로그들은 /shared/logs 디렉토리에 위치한다.

![](/assets/images/kubeweblogic2/21_logs.png)

# 참고 자료

- [Oracle WebLogic Server Kubernetes Operator](https://oracle.github.io/weblogic-kubernetes-operator/)

