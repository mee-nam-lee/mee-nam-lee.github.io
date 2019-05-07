---
layout: post
title:  Oracle IaaS와 Java Cloud Service를 사용하여 3 Tier 아키텍쳐 구축하기 (Terraform을 통한 Provision)
categories: Cloud
tags: [Oracle Cloud, Virtual Cloud Network,IaaS, 3 Tier Architecture, HA, Java Cloud Service, Terraform] 
---

이전 문서 [Oracle IaaS와 Java Cloud Service를 사용하여 3 Tier 아키텍쳐 구축하기 (Regional Subnet 사용)](/cloud/2019/basic-3tier-arch-regional/)에서는 각 Tier에 필요한 모든 컴포넌트들을 **Oracle Cloud Console**을 통해 Step by Step으로 생성하였었다.

여기에서는 이전 문서에서 설명한 모든 필요 컴포넌트를 **Terraform**을 이용해서 프로비전 하는 과정에 대해서 설명할 것이다.

생성되어질 컴포컨트들의 **아키텍쳐는 Regional Subnet을 사용할 때와 동일**하다. 따라서 생성된 후의 모습은 다음과 같을 것이다.

# Architecture

![](/assets/images/3tier/regional/architecture.png)

# Prerequisites

- Oracle Cloud Infrastucture CLI (oci)
- Terraform (OCI provider / paas provider)

# OCI CLI / Terraform 설치

설치 관련은 다음 문서를 참고한다. 

 - [Getting Started with the Terraform Provider](https://docs.cloud.oracle.com/iaas/Content/API/SDKDocs/terraformgetstarted.htm)


# 3-Tier 예제 소스 

예제 소스를 다음에서 다운 받는다.

```
> git clone https://github.com/mee-nam-lee/terraform_oci_paas.git
> cd terraform_oci_paas/oci_with_paas
```

생성되는 각 컴포턴트는 이전 예제와 동일한 이름을 사용하였다. 컴포넌트 상세는 이전 글을 참고한다.

## Terraform 파일 설명

| 파일명        | 생성되는 Resource    | 설명     |
|-------------|--------------------| --------|
| env.sh      |                    | Oracle Cloud에 연결하기 위한 정보가 들어있는 파일 |
| provider.tf |                    | oci, oraclepaas provider 구성 정보 |
| variables.tf |                   | 다른 terraform 파일에서 참고하는 variable 정도들이 들어있음 |
| data.tf      |                   | OCI 정보 조회 |
| vcn.tf       | VCN               | VCN 생성 |
| subnet.tf    | Subnet            | 3개의 Subnet 생성 |
| routetable.tf | RouteTable       | 2개 (PublicRT, PrivateRT) RouteTable 생성 |
| securitylist.tf  | SecurityList     | 3개 (PublicSL, PrivateSL, DBSL)의 Security List와 Ingress, Egress Rule 정의 |
| internetgw.tf  | Internet Gateway   |     |
| servicegw.tf   | Service Gateway    |     |
| db.tf          | DB System          | DemoDB 생성 |
| object.tf      | ObjectStorage      | JCS Backup용 Object Storage Bucket 2개 생성 |
| jcs.tf         | Java Cloud Service | AD1, AD2에 각각 JCS 1개 생성 |
| compute_web.tf | Compute Instance   | Web용 Custom Image로 Compute Instance 2개 생성 |
| compute_coh.tf | Compute Instance   | Coherence 용 Custom Image로 Compute Instance 2개 생성 |
| loadbalancer.tf | LoadBalancer      | Loadbalancer 생성 후 Web 인스턴스에 연결하는 BackendSet 구성 |
| output.tf       |                   | 필요 정보 출력 |


## env.sh 수정

**env.sh** 파일을 각자의 환경에 맞게 수정한다.

![](/assets/images/3tier/terraform/01_env_sh.png)

수정한 환경 변수 파일을 실행 시키고 Terraform Provider를 설치한다.

```
> source env.sh
> terraform init
```
![](/assets/images/3tier/terraform/01_terraform_init.png)

## variables.tf 확인

**variables.tf** 파일에는 사용하는 Compute Shape 및 Admin Password 등의 변수들이 선언되어 있다.
이 파일을 열어 각 변수를 원하는 값으로 편집해도 된다.

참고로 Web용 Compute와 Coherence용 Compute는 이전 과정에서 만들어 둔 Custom Image를 참조하도록 Custom Image의 OCID로 설정되어 있다.
Custom Image를 사용하지 못하거나, 다른 Image를 사용하려면 아래 부분을 수정하면 된다.

```
variable "instance_image_ocid" {
  type = "map"

  default = {
    // custom image OCID
    web= "ocid1.image.oc1.phx.aaaaaaaa5bxabwjdny6dpaf2xr63rvxoofxslzkahjnfqlcpodfl5oobualq"
    coherence ="ocid1.image.oc1.phx.aaaaaaaacp2wktef46wf255fh3lu3df6rea52dd2zjcqz34wmoj7tigbxeka"
  }
}
```

변수들을 살펴봤으면 이제 적용해 보자

## Terraform Plan

```
> terraform plan
```

![](/assets/images/3tier/terraform/02_terraform_plan.png)

## Terraform Apply

apply command로 리소스를 생성한다. Database와 Java Cloud Service 생성에는 약간의 시간이 걸린다.

```
> terraform apply
```

![](/assets/images/3tier/terraform/03_apply.png)

![](/assets/images/3tier/terraform/04_apply.png)

# 생성 확인

Oracle Cloud Console에 접속해서 리소스들이 잘 생성되었는지 확인한다.

## Network

![](/assets/images/3tier/terraform/05_network.png)

## Load Balacner

![](/assets/images/3tier/terraform/06_loadbalancer.png)

## Object Storage

![](/assets/images/3tier/terraform/07_object.png)

## DB System

![](/assets/images/3tier/terraform/08_db.png)

## Compute

![](/assets/images/3tier/terraform/09_compute.png)

## Java Cloud Service

![](/assets/images/3tier/terraform/10_jcs.png)

# 애플리케이션 배포 및 Test

이제 필요한 리소스들은 다 생성되었고 
Web과 Coherence 인스턴스에 접속해서 서비스를 구동시키고, 웹로직 서버에 애플리케이션 배포하고 테스트 하는 과정만 남아있다.
이 과정은 이전 글을 참고하여 진행하면 된다.

# 환경 지우기

테스트가 완료되었으면 모든 환경을 지운다.

```
> terraform destroy
```

![](/assets/images/3tier/terraform/11_destroy.png)

리소스 삭제에도 약간의 시간이 걸린다. 모두 완료되면 다음과 같이 표시 될 것이다.

![](/assets/images/3tier/terraform/12_destroy_2.png)

# 이전 문서 참고

- [Oracle IaaS와 Java Cloud Service를 사용하여 멀티 AD(Availability Domain)에 3 Tier 아키텍쳐 구축하기](/cloud/2019/basic-3tier-arch/)
- [Oracle IaaS와 Java Cloud Service를 사용하여 3 Tier 아키텍쳐 구축하기 (Regional Subnet 사용)](/cloud/2019/basic-3tier-arch-regional/)

# 참고 자료

- [Terraform Oracle Cloud Infrastructure Provider](https://www.terraform.io/docs/providers/oci/)
- [Terraform Oracle Cloud Platform Provider](https://www.terraform.io/docs/providers/oraclepaas/index.html)
