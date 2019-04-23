---
layout: post
title:  DevOps Workshop (CI/CD 실습) - (2) - Lab 01
categories: DevOps
tags: [Developer Cloud, DevOps, CI/CD, Hands-on-lab]
---

이 Lab은 **Oracle Developer Cloud**와 Container기반의 **Application Container Cloud Service**를 사용하여 **DevOps** 자동화 과정의 핵심 부분인 **CI(Continuous Integration) / CD (Continuous Delivery)** 실습을 위한 Lab입니다. 

## 목표


- 외부 Git 저장소에서 코드 가져 오기 
- Developer Cloud Service 및 Oracle Application Container 클라우드 서비스를 사용하여 프로젝트 빌드 및 배포 

## 필수 아티팩트


- 강사가 제공한 각자에게 할당된 [Oracle Public Cloud 계정](env/env.md)을 확인하세요. 

## 특별주의 사항

- 복사하여 붙여 넣을 때 주의하십시오. 복사한 내용의 *이전*이나 *이후*에 **공백**을 넣게 되면 오류가 발생할 수 있습니다. 

# 마이크로 서비스 만들기 (Front 서비스)

## 초기 Git 저장소 생성


- 제공된 [`클라우드 환경 정보`](env/env.md)의 **Developer Cloud Service URL** 링크를 클릭합니다. **Identity Domain** 입력 부분이 자동으로 채워져 있을 것입니다. **Go**를 클릭하여 다음 로그인 페이지로 이동합니다.

![](/assets/images/devops/000.directaccess.png)

로그인을 위해서 다음 페이지가 보일 것입니다.

- 부여 받은 클라우드 계정의 ID와 패스워드를 입력하고 **Sign In**을 클릭합니다.

    - ID : lisa.jones
	- Password : 제공된 [`클라우드 환경 정보`](env/env.md)의 **Password** 컬럼을 참고합니다.

![](/assets/images/devops/000.idpw.png)

- Developer Cloud에 접속하면 프로젝트가 생성되어 있습니다. 강사에게 부여받은 프로젝트를 선택하여 해당 프로젝트로 이동합니다. (예: `Project`**01**) 

![](/assets/images/devops/001.landing.png)


-. 이미 Developer Cloud 페이지에 접속되어 있는 경우왼쪽 탐색 패널의 **Project**를 클릭하여 프로젝트 기본 페이지로 이동할 수 있습니다.

-. **REPOSITORIES**섹션의 오른쪽에서 **New Repository**를 클릭하여 새로운 Git 저장소를 만듭니다.

![](/assets/images/devops/002.createrepo.png)

- 새 저장소 마법사에서 다음 정보를 입력하고 **Create**을 클릭합니다. 

	- **Name:** `Rep##` -- 부여받은 Repository명을 입력합니다.
	- **Description:** `Microservice to provide UI Service`
	- **Initial content:** `Import existing repository`
	- **Enter the URL:** `https://github.com/pcdavies/JETTwitterQuickStart.git`

> 복사해서 붙여넣기를 할 때 불필요한 스페이스가 들어갈 수 도 있으니 주의하시기 바랍니다.

![](/assets/images/devops/003.newrepo.png)


- 기존 저장소를 기반으로하는 개발자 클라우드 서비스 내에 저장된 새로운 Git 저장소를 만들었습니다 

![](/assets/images/devops/004.repo.png)


## 기본 빌드 및 배포 프로세스 만들기

개발자 클라우드 서비스에서 관리하는 Git Repository에 소스 코드가 있으므로 마스터 분기를 커밋 할 때마다 트리거되는 빌드 프로세스를 만들어야합니다.  

### 기본 빌드 프로세스 만들기


- 탐색 패널에서 **Build**를 클릭하여 빌드 페이지에 액세스하고 **+ New Job** 을 클릭하십시오. 

![](/assets/images/devops/005.navibuild.png)


- New Job (새 작업) 팝업 창에서 Job Name에 대해 다음 처럼 입력하고 **Save**를 클릭하십시오. 
	- **Job Name:** **Rep##_build** -- 부여받은 Repository명을 `prefix`로 사용합니다.


> 복사해서 붙여넣기를 할 때 불필요한 스페이스가 들어갈 수 도 있으니 주의하시기 바랍니다.

![](/assets/images/devops/006.newbuildjob.png)

- 이제 작업 구성 화면에 배치됩니다. 

![](/assets/images/devops/007.newjob.png)


- **Source Control**탭을 클릭하십시오.
**Git** 라디오 버튼을 선택하십시오. 
리포지토리 섹션의 URL 드롭 다운에서 **Rep##.git**(`각자 생성한 Repository 를 선택해야 합니다. ## 부분 번호확인 필수`) 를 선택하십시오. 

![](/assets/images/devops/008.srcctrl.png)


- **Triggers**탭을 클릭하십시오. **Based on SCM polling schedule**를 클릭하십시오.

![](/assets/images/devops/009.trigger.png)


- **Build Steps**탭을 클릭하고**Add Build Step**를 클릭 한 다음 **Execute shell**을 선택합니다. 

![](/assets/images/devops/010.steps.png)


- Command textarea에 다음을 입력하십시오 :`npm install` 

![](/assets/images/devops/011.npm.png)


- **Post Build**탭을 클릭하십시오. **Archive the artifacts**을 ​​선택하십시오. 아카이브 할 파일에`**/target/*`을 입력하고 압축 유형으로 **GZIP**가 선택되어 있는지 확인하십시오. 

![](/assets/images/devops/012.post.png)


- **Save**을 클릭하여 구성을 완료하십시오. 

![](/assets/images/devops/013.save.png)


- 빌드는 1-2 분 내에 자동으로 시작됩니다. 자동으로 시작되지 않으면 **Build Now**버튼을 클릭하십시오. 

![](/assets/images/devops/014.buildnow.png)


- 상태는 다음 다이어그램과 유사하게 바뀝니다. 

![](/assets/images/devops/015.queue.png)


빌드 작업을 시작하려면 몇 분 정도 기다려야합니다. 

![](/assets/images/devops/016.running.png)


- 빌드가 완료 될 때까지 몇 분이 소요됩니다. 다음 단계로 진행하기 전에 빌드가 완료 될 때까지 기다리십시오 - **배포 구성을 생성하기 위해 빌드 아티팩트가 필요하므로**. 

![](/assets/images/devops/017.complete.png)


### 기본 배포 프로세스 만들기 


- **Deploy**를 클릭하여 배포 페이지에 액세스하고 **+ New Configuration**버튼을 클릭하십시오. 

![](/assets/images/devops/018.navideploy.png)


- 다음 데이터를 입력하십시오. 

	- **Configuration Name:** `a##` (예 : **a01**)
	- **Application Name:** `a##` (예 : a01) `## 부분의 번호룰 Rep## 번호와 같게 생성하세요. 할당 받은 번호를 사용합니다.` 


> 복사해서 붙여넣기를 할 때 불필요한 스페이스가 들어갈 수 도 있으니 주의하시기 바랍니다.


![](/assets/images/devops/019.deployname.png)


- **Deployment Target**의 오른쪽 옆에있는 **[New]**버튼을 클릭하고 **Application Container Cloud ...를 선택하십시오.**

![](/assets/images/devops/020.deployaccs.png)


- 다음 정보를 입력하고 **Test Connection**를 클릭하십시오. 

	- 부여 받은 [`클라우드 환경 정보`](env/env.md)를 사용합니다.
	- **Data Center:** `your datacenter, (em2 또는 us2)`
	- **Identity Domain:** `your identity domain`, (gse000 으로 시작하는 정보)
	- **Username:** `lisa.jones`
	- **Password:** `password of the cloud user`, that is the password you are using


![](/assets/images/devops/021.accsconn.png)


- 성공하면 **Use Connection**버튼을 클릭하십시오. 

![](/assets/images/devops/022.useconn.png)


- Application Container Cloud Service 속성에서 다음을 설정합니다. 

	- **Runtime** to `Node`
	- **Subscription** to `Hourly`
	- **Type**에서 `Automatic`으로 설정하고 Deploy stable build only **체크**


**런타임이 Node 인지 다시 한번 확인**

![](/assets/images/devops/023.deploynodejs.png)


- **Job**에서 선택하십시오.이 이름은 위의 빌드 작업과 일치해야합니다 (예 : **`Rep##_build`** ). 

- **Artifact**에서 선택하십시오.이 이름은 위의 아카이브 아티펙트와 일치해야합니다 (예 :`target/jet-quickstart-client-dist.zip`). 

![](/assets/images/devops/024.deployjobname.png)


- **Include ACCS Deployment**상자를 체크하고 다음 json을 추가하십시오. 

```json
{
	"memory":"1G",
	"instances":"1"
}
```

![](/assets/images/devops/024.json.png)

- 위의 사항을 확인하고 **Save** 버튼을 클릭합니다.

![](/assets/images/devops/025.deploysave.png)

- 기어 마크가 있는 아이콘을 클릭한 다음 **Start** 를 누릅니다.

![](/assets/images/devops/026.deploystart.png)

- 배포하는 작업이 큐에 들어갈 것입니다. 조금 기다리면 **Starting application** 메시지가 **Last deployment succeeded**로 바뀔 것입니다. 만약 배포작업이 실패하면 도움을 요청하십시오.

![](/assets/images/devops/027.deploysuccess.png)

배포가 완료되면 배포명 좌측의 아이콘이 Green으로 변경됩니다.
배포된 서비스를 확인하기 위하여 **Deploy to ACCS** 옆의 링크를 클릭합니다. 아래 이미지의 Yellow로 마킹된 부분입니다.

![](/assets/images/devops/028.accessaccs.png)

- ACCS Service Console 이 보여질 것입니다. 그리고 **a## (예 : `a01`)** 라는 새로운 애플리케이션이 배포된 것을 볼 수 있습니다.

![](/assets/images/devops/030.accsconsole.png)

##  Verify the Working Service

- 애플리케이션 패널에서 애플리케이션에 대한 URL을 볼 수 있습니다. 예를들면 https://a##-{your-identity-domain}.apaas.{your-data-center}.oraclecloud.com

![](/assets/images/devops/037.url.png)

- 링크를 클릭하여 해당 url을 엽니다. 다음과 같이 애플리케이션이 정상적으로 브라우징 되면 성공입니다.

![](/assets/images/devops/039.result.png)


**축하합니다. 첫 마이크로 서비스가 완성되었습니다.** 

# DevOps Workshop Lab 

- [DevOps Workshop 개요](./DevOpsWorkshop_1.html)
- [Lab 01로 이동](./DevOpsWorkshop_2.html)
- [Lab 02로 이동](./DevOpsWorkshop_3.html)