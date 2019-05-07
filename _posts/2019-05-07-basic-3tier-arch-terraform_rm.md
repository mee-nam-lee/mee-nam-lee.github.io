---
layout: post
title:  Oracle Cloud Infrastructure Resource Manager를 이용하여 Terraform 구성 리소스 Provision 하기
categories: Cloud
tags: [Oracle Cloud, Virtual Cloud Network,IaaS, 3 Tier Architecture, HA, Terraform, Resource Manager] 
---

이전 문서 [Oracle IaaS와 Java Cloud Service를 사용하여 3 Tier 아키텍쳐 구축하기 (Terraform을 통한 Provision)](/cloud/2019/basic-3tier-arch-terraform/)에서는 몇 차례 기고를 통해 설명했던 **Oracle Cloud 상에 3 Tier 아키텍쳐 구성하기**에 필요한 각 컴포넌트들을 하나 하나 수동으로 생성하는 것이 아닌 **Terraform**을 통해 Batch 형태로 구성하는 과정을 설명했었다.

보통 Terraform을 이용한 Provision을 위해서는 Terraform 환경을 별도로 구축하거나 Local 환경에 설치하여 작업하지만, Oracle Cloud Infrastructure에서는 별도의 Terraform 환경 구성 없이도 Terraform 구성 파일만을 Oracle Cloud에 업로드 하여 리소스를 생성할 수 있도록 하는 기능을 제공하고 있다. Oracle Cloud Infrastructure는 이를 **Resource Manager**라는 이름으로 제공한다.

- [Overview of Resource Manager](https://docs.cloud.oracle.com/iaas/Content/ResourceManager/Concepts/resourcemanager.htm)

여기에서는 이전에 작성한 Terraform 구성 파일들을 Oracle Cloud Infrastructure(OCI)의 **Resource Manager**로 생성하여 원하는 리소스를 Provision하는 과정에 대해 설명하려고 한다.

> OCI의 Resource Manager에서는 현재까지는 **oci** provider만 지원되고, 추가적인 Plugin을 설치할 수 없기 때문에 이전 과정에서 사용한 **oraclepaas**  Provider 부분은 제외해야 한다. 즉 Java Cloud Service(JCS) 부분의 Provision은 **Resource Manager**를 통해서 수행할 수 없다.
> 따라서 여기에서는 JCS 부분을 제외한 다른 리소스들만 생성해 볼 것이다.


# Prerequisites

- Oracle Cloud Infrastucture

# 3-Tier 예제 소스 

이전 과정을 진행하지 않았었다면 여기서 새롭게 예제 소스를 다운 받는다.

```
> git clone https://github.com/mee-nam-lee/terraform_oci_paas.git
> cd terraform_oci_paas/oci_resourcemanager
```

## Terraform 파일 설명

사용하는 Terraform Configuration 파일은 이전과 동일하지만 **Resource Manager**를 통해 생성할 수 없는 PaaS 부분인 Java Cloud Service(JCS) 구성파일은 제외되었다.

| 파일명        | 생성되는 Resource    | 설명     |
|-------------|--------------------| --------|
| ~~env.sh~~      |                    | **(삭제됨)** Oracle Cloud에 연결하기 위한 정보가 들어있는 파일 |
| provider.tf |                    | **(수정됨)** oci, oraclepaas provider 구성 정보 |
| variables.tf |                   | **(수정됨)** 다른 terraform 파일에서 참고하는 variable 정도들이 들어있음 |
| data.tf      |                   | OCI 정보 조회 |
| vcn.tf       | VCN               | VCN 생성 |
| subnet.tf    | Subnet            | 3개의 Subnet 생성 |
| routetable.tf | RouteTable       | 2개 (PublicRT, PrivateRT) RouteTable 생성 |
| securitylist.tf  | SecurityList     | 3개 (PublicSL, PrivateSL, DBSL)의 Security List와 Ingress, Egress Rule 정의 |
| internetgw.tf  | Internet Gateway   |     |
| servicegw.tf   | Service Gateway    |     |
| db.tf          | DB System          | DemoDB 생성 |
| object.tf     | ObjectStorage      | JCS Backup용 Object Storage Bucket 2개 생성 |
| ~~jcs.tf~~         | Java Cloud Service | **(삭제됨)** AD1, AD2에 각각 JCS 1개 생성 |
| compute_web.tf | Compute Instance   | Web용 Custom Image로 Compute Instance 2개 생성 |
| compute_coh.tf | Compute Instance   | Coherence 용 Custom Image로 Compute Instance 2개 생성 |
| loadbalancer.tf | LoadBalancer      | Loadbalancer 생성 후 Web 인스턴스에 연결하는 BackendSet 구성 |
| output.tf       |                   | 필요 정보 출력 |


## provider.tf

위에서 설명한 데로 **Resource Manager**에서는 OCI Provider만 사용할 수 있고, Parameter로 **user_ocid**, **fingerprint**, **private_key_path** 같은 보안 관련 파라미터를 구성파일 내에 두지 못하게 하기 때문에 이전 파일에서 보안 관련 파라미터들을 제거해야 한다.

> OCI 내에서 작업이 진행되는 것이기 때문에 이 정보가 요구되지 않는다.

변경된 파일은 다음과 같다.

```
provider "oci" {
  tenancy_ocid         = "${var.tenancy_ocid}"
  region               = "${var.region}"
  disable_auto_retries = "${var.disable_auto_retries}"
}
```

## variables.tf

이전 Local Terraform Provision에서는 TF_VAR 환경 변수를 참조하여 변수를 사용하도록 구성하였기 때문에 variables.tf에 default 값을 설정하지 않았다.
**Resource Manager**에서는 환경 변수 Export를 통한 변수 참조가 가능하지 않기 때문에 variable 값을 셋팅해줘야 하는데, 다음의 3가지 방법으로 수행할 수 있다.

1. tfvars 파일 생성
2. **Resource Manager**에서 variable 값을 콘솔을 통해서 입력
3. variables.tf의 default 값 setting

여기에서는 3번 방법으로 수행하였다. 따라서 variables.tf 파일도 수정이 되어야 한다.
사용하지 않는 variable들을 삭제하고 default 값을 설정해준 variables.tf 파일은 다음과 같다.

![](/assets/images/3tier/oci_rm/01_variables.png)

## Terraform Configuration Archive 생성

준비된 Terraform Configuration 파일을 zip으로 묶는다.

```
> zip ../../tf_oci_rm.zip *.tf 
```
![](/assets/images/3tier/oci_rm/02_zip.png)

# Resource Manager 생성

Oracle Cloud Console에 접속하여 **Resource Manager** 메뉴로 이동한다. **Create Stack**을 클릭하여 Stack을 하나 생성한다.

![](/assets/images/3tier/oci_rm/03_stack.png)

Stack 이름을 입력하고, 압축해두었던 zip 파일을 업로드한다. 그러고 나면 variable 값이 파싱되어서 화면에 보여질 것이다. 값들을 확인한 후 하단의 **Create** 버튼을 클릭한다.

![](/assets/images/3tier/oci_rm/04_create_stack.png)

Stack이 하나 만들어졌다. 

## Variable 설정

variables.tf 파일로 default 값들을 설정하였지만 보안 상의 이유로 configuration file에 정의할 수 없는 **ssh_public_key** 같은 변수는 공백으로 남겨져 있다.
이 값은 Resource Manager의 콘솔을 통해 값을 셋팅하도록 할 것이다.
**Variables** 메뉴에서 **Edit Variables**를 클릭하여 변수 값을 설정해 준다.

![](/assets/images/3tier/oci_rm/10_var.png)

**ssh_public_key** 값을 입력하고 저장한다.

![](/assets/images/3tier/oci_rm/11_save_var.png)

## Terraform Plan

생성한 Stack으로 이동하여 Plan/Apply/Destroy 작업을 수행할 수 있는 **Job**을 하나 생성한다. 

![](/assets/images/3tier/oci_rm/05_job.png)

먼저 **Plan** 부터 생성한다.

![](/assets/images/3tier/oci_rm/06_plan.png)

Job이 **Accepted**가 되었고 곧 **In Progress** 상태로 전환될 것이다.

![](/assets/images/3tier/oci_rm/07_job_accepted.png)

진행 사항을 다음처럼 Log에서 확인할 수가 있다.

![](/assets/images/3tier/oci_rm/08_log1.png)

![](/assets/images/3tier/oci_rm/08_log2.png)

## Terraform Apply

**apply** Job도 위에서 처럼 생성한다.

![](/assets/images/3tier/oci_rm/09_apply.png)

**Apply** Job의 진행사항도 Log에서 다음과 같이 확인이 된다.

![](/assets/images/3tier/oci_rm/12_apply_log.png)

이제 기다리기만 하면 원하는 리소스들이 생성되어 있을 것이다.

**outputs.tf**에 정의한 output 결과는 **Outputs** 메뉴에서 확인 할 수 있다.

![](/assets/images/3tier/oci_rm/13_output.png)

# 생성 확인

Oracle Cloud Console에 접속해서 리소스들이 잘 생성되었는지 확인한다.
생성된 내용은 이전과 동일한 내용이니 여기에서는 생략하도록 하겠다.

# Terraform destroy : 환경 지우기

**Destroy** 역시 Job으로 생성해서 수행한다.


# 이전 문서 참고

- [Oracle IaaS와 Java Cloud Service를 사용하여 멀티 AD(Availability Domain)에 3 Tier 아키텍쳐 구축하기](/cloud/2019/basic-3tier-arch/)
- [Oracle IaaS와 Java Cloud Service를 사용하여 3 Tier 아키텍쳐 구축하기 (Regional Subnet 사용)](/cloud/2019/basic-3tier-arch-regional/)
- [Oracle IaaS와 Java Cloud Service를 사용하여 3 Tier 아키텍쳐 구축하기 (Terraform을 통한 Provision)](/cloud/2019/basic-3tier-arch-terraform/)

# 참고 자료

- [Terraform Oracle Cloud Infrastructure Provider](https://www.terraform.io/docs/providers/oci/)
- [Overview of Resource Manager](https://docs.cloud.oracle.com/iaas/Content/ResourceManager/Concepts/resourcemanager.htm)
