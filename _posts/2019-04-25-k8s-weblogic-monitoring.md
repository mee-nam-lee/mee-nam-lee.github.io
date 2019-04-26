---
layout: post
title:  WebLogic on Oracle Kubernetes Engine(OKE) - 모니터링 (Prometheus / Grafana) 
categories: Cloud
tags: [WebLogic, Kubernetes, Oracle Kubernetes Engine, Prometheus, Grafana]
---

이 예제에서는 이전에 생성한 WebLogic Domain의 여러 매트릭들을 모니터링 하는 기능을 추가 해 볼 것이다.
WebLogic Server는 다양한 매트릭을 MBean으로 제공하고 있고 일반적으로 많은 모니터링 툴들이 JMX를 이용하여 이 MBean 정보를 읽어가는 형태로 모니터링을 수행한다. 

Prometheus도 Generic JMX Exporter를 제공하고 있고, 이를 통해서 WebLogic Server 모니터링이 가능하지만, 이를 위해서는 WebLogic Server 쪽에 JMX 모니터링을 위한 몇가지 옵션들을 구성해 줘야 한다.

WebLogic Server에서는 JMX를 통한 모니터링 이외에 REST API 기반으로 MBean 정보를 읽거나 수정할 수 있는 Management Interface도 함께 제공하고 있는데, 여기서 소개할 **WebLogic Monitoring Exporter**가 **Management REST API** 기반으로 구현해 둔 Prometheus 용 Exporter다. 

- [About the WLS RESTful Management Interface](https://docs.oracle.com/middleware/1221/wls/WLRUR/overview.htm#WLRUR111)

- [WebLogic Monitoring Exporter](https://github.com/oracle/weblogic-monitoring-exporter)

지금부터는 **WebLogic Monitoring Exporter**를 이용하여 **Premetheus**에서 WebLogic Domain을 모니터링 해 볼 것이다.

# Architecture

![](/assets/images/kubeweblogic4/00_arch.png)

# Prerequistes 

- Oracle Kubernetes Engine(OKE) 
- Oracle File System
- WebLogic Monitoring Exporter

# WebLogic Monitoring Exporter 구성

**WebLogic Monitoring Exporter**는 모니터링할 WebLogic Server에 배포되어 돌아가는 **Web Application**으로 만들어져 있다. 
먼제 제공되는 소스를 받아서 배포 가능 형태로 빌드를 수행해야 한다.

다음에서 소스를 받아서 빌드를 수행한 후 배포용 Archive를 만든다.

```
git clone https://github.com/oracle/weblogic-monitoring-exporter.git
cd weblogic-monitoring-exporter

mvn install
```
**wls-exporter-1.0-SNAPSHOT.jar** 파일이 생성되고 local mvn repository에 install이 된다.

![](/assets/images/kubeweblogic4/10_mvn_install.png)

**webapp** 디랙토리로 이동하여 war 파일을 packaging 한다.
```
cd webapp
mvn package
```

다음과 같이 target 디렉토리 내에 **wls-exporter.war**이 생성된다.

![](/assets/images/kubeweblogic4/11_mvn_package.png)

현재 WebLogic Domain이니 **Persistent Volume**을 사용하는 구성으로 되어있기 때문에 빌드된 war 파일을 NFS File System으로 Copy하여 배포하여도 되고, WebLogic Console을 통해서 war 파일 **upload** 방법을 선택하여 배포해도 된다. 원하는 방법으로 배포를 수행한다.

>
> WebLogic Admin Console 접근 법은 이전 글을 참고한다. admin-server를 참고하는 LoadBalancer Type의 서비스가 존재해야 한다. 이미 삭제 했다면, 이전 글을 참고하여 다시 생성한다.
>

애플리케이션은 Admin Server와 Cluster 모두를 Target으로 지정한다. 
배포 후 애를리케이션을 **Start** 시키면 다음과 같이 Admin Server와 Cluster에서 **Active** 상태로 서비스될 수 있음이 보여진다.

![](/assets/images/kubeweblogic4/05_exporter_deploy.png)

배포한 애플리케이션이 잘 동작하는지 확인해 본다. LoadBalancer Type의 Service의 External IP를 이용하여 다음과 같이 접속하면 된다. 

> 애플리케이션이 Admin Server와 Cluster 모두에 배포되어있기 때문에 둘 중 어떤 서비스를 이용해도 관계 없다

- http://[Service External IP]:[Port]/wls-exporter/

다음과 같은 화면이 보일 것이다. 
위 **WebLogic Monitoring Exporter** 소스 디렉토리 내의 다음 위치에 가면 모니터링할 매트릭 정의를 위한 구성 파일 샘플이 존재한다. 

-  weblogic-monitoring-exporter/samples/configurations

**Append** 옵션을 선택하여 샘플 구성 파일을 선택하고 **Submit**을 클릭하면 해당 모니터링 매트릭이 Export 매트릭으로 계속 추가가 된다.

![](/assets/images/kubeweblogic4/04_exporter_config.png)

이제 **metrics**라는 상단의 링크를 클릭해 보면 metric이 다음과 같은 형태로 추출되고 있는 것을 볼 수 있다.

![](/assets/images/kubeweblogic4/04_wls_exporter_metrics.png)

## Monitoring Metric 추가

Sample로 제공되는 metric 외에 다른 metric를 추가하거나 변경하고자 한다면 제공된 구성 파일을 바탕으로 metric을 조정하면 된다.
예를 들어 제공된 jvm.yml에서 metric를 추가하고 싶다면, 아래의 WebLogic Server **WLST (Weblogic Script Tool)** 툴을 통해 다른 Metric들의 이름을 확인하고 추가해 주면 된다.

![](/assets/images/kubeweblogic4/12_wls_mbean.png)

**HeapSizeMax**와 **Uptime**을 추가한다면 파일을 다음과 같이 수정하면 된다.

```yml
# jvm_custom.yml
metricsNameSnakeCase: true
queries:
- JVMRuntime:
    key: name
    prefix: jvm_
    values: [heapFreeCurrent, heapFreePercent, heapSizeCurrent, heapSizeMax, uptime]
```
수정한 구성파일을 WebLogic Monitoring Exporter에 적용하고 매트릭이 잘 추출되는지 확인해 본다.

![](/assets/images/kubeweblogic4/13_metric_added.png)

WebLogic Domain 쪽에서의 준비는 완료가 되었다.

# Prometheus 배포 

이제는 Prometheus를 **Oracle Kubernetes Engine**에 배포할 것이다.
배포를 위한 구성 파일은 다음에 존재한다.

-  weblogic-monitoring-exporter/samples/kubernetes

먼저 Prometheus를 위하는 **monitoring**이라는 namespace를 만들고 RBAC를 위한 Role을 생성해 줄 것이다.

다음을 수행한다.

```
cd weblogic-monitoring-exporter/samples/kubernetes
kubectl apply -f monitoring-namespace.yaml
```

다음과 같이 생성된다. 

![](/assets/images/kubeweblogic4/01_create_ns.png)

이제 필요한 Role을 생성해야 하는데 제공된 스크립트는 WebLogic Domain은 **weblogic-domain** namespace를 사용하고 WebLogic Operator는 **weblogic-operator** namespace를 사용하는 환경 기반으로 작성되어있다.

이전 글에서 설명한 내용을 따라서 WebLogic Operator와 WebLogic Domain을 배포했다면 여기서는 모두 **default** namespace를 사용하는 것으로 하였었기 때문에 namespace 참조 부분을 모두 수정해 줘야 한다.

Namespace를 변경한 sample configuration은 아래와 같다.

```yaml
---
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: weblogic-domain-cluster-role
rules:
- apiGroups: ["weblogic.oracle"]
  resources: ["domains"]
  verbs: ["get", "list"]
---
#
# creating role-bindings for cluster role
#
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: domain-cluster-rolebinding
subjects:
- kind: ServiceAccount
  name: default
  namespace: default
  apiGroup: ""
roleRef:
  kind: ClusterRole
  name: weblogic-domain-cluster-role
  apiGroup: "rbac.authorization.k8s.io"
---
#
# creating role-bindings
#
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: weblogic-domain-operator-rolebinding
  namespace: default
subjects:
- kind: ServiceAccount
  name: default
  namespace: default
  apiGroup: ""
roleRef:
  kind: ClusterRole
  name: cluster-admin
  apiGroup: "rbac.authorization.k8s.io"
---
```

변경된 파일을 반영한다.

```
kubectl apply -f crossnsrbac.yaml
```
Role이 생성되고 Bind 되었다.

![](/assets/images/kubeweblogic4/02_rbac.png)

이제 Prometheus를 배포할 차례이다. 
제공되는 **prometheus-deployment.yaml**에 보면 Service 항목이 존재하는데 이 서비스의 Type이 **NodePort**로 되어있다. 기억하겠지만 Oracle Kubernetes Engine을 **Private Subnet**에 생성 했기 때문에 NodePort를 사용하여서는 외부에서 바로 접속할 수가 없다.

따라서 이 부분을 **LoadBalancer** Type으로 변경해 줘야 한다.

변경된 Service 구성 부분은 다음과 같다.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: prometheus
  namespace: monitoring
  annotations:
    service.beta.kubernetes.io/oci-load-balancer-shape: "100Mbps"
    service.beta.kubernetes.io/oci-load-balancer-backend-protocol: "HTTP"
spec:
  type: LoadBalancer
  ports:
  - port: 9090
    protocol: TCP
    targetPort: 9090
#    nodePort: 32000
  selector:
    app: prometheus
#  type: NodePort
```
변경된 파일을 적용한다.

```
kubectl apply -f prometheus-deployment.yaml
```

다음과 같이 정의된 리소스들이 생성되는 것을 확인할 수 있다.

![](/assets/images/kubeweblogic4/03_prometheus.png)

서비스 항목도 살펴보자. External IP 부분이 확인되면 이 IP를 통해서 브라우저에서 접속한다.

![](/assets/images/kubeweblogic4/07_prometheus_svc.png)

다음으로 접속해 본다.

- http://[prometheus-service-external-ip]:9090

모니터링 항목을 선택하는 화면이 나올 것이다. 

![](/assets/images/kubeweblogic4/06_prometheus.png)

메트릭을 확인하기 전에 먼저 **Status/Target** 메뉴를 먼저 확인해 보자. WebLogic Server에 배포한 **WebLogic Monitoring Exporter**에 의해서 메트릭이 추출되고 있음을 확인할 수 있다.

![](/assets/images/kubeweblogic4/08_prometheus_target.png)

다시 Graph 메뉴로 이동하여 모니터링 하고자 하는 매트릭을 선택한다.

![](/assets/images/kubeweblogic4/09_monitoring.png)

# Grafana 구성

Grafana를 배포할 것이다. 
**grafana-deployment.yaml**에 보면 Service 항목이 존재하는데 이 서비스의 Type이 **NodePort**로 되어있다. 이 부분도  **LoadBalancer** Type으로 변경해 줘야 한다.

변경된 Service 구성 부분은 다음과 같다.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: grafana
  namespace: monitoring
  annotations:
    service.beta.kubernetes.io/oci-load-balancer-shape: "100Mbps"
    service.beta.kubernetes.io/oci-load-balancer-backend-protocol: "HTTP"
spec:
  type: LoadBalancer
  ports:
  - port: 3000
    protocol: TCP
    targetPort: 3000
#    nodePort: 31000
  selector:
    name: grafana
#  type: NodePort
```

![](/assets/images/kubeweblogic4/14_grafana.png)

브라우저로 접속한다

- http://[grafana-external-service-ip]:[port]
- Username : admin
- Password : admin

![](/assets/images/kubeweblogic4/15_grafana_login.png)

좌측 메뉴의 **Datasource** 메뉴로 이동한다.

![](/assets/images/kubeweblogic4/19_add_datasource.png)

다음과 같이 입력하고 **Add**를 클릭한다.

- Name : Prometheus
- Type : Prometheus
- URL : http://prometheus:9090
- Access : proxy

![](/assets/images/kubeweblogic4/16_add_datasource.png)

Data Source Edit 창에서 Prometheus Stats Dashboard를 Import 할 수 있다.
여기에서는 이 Dashboard는 사용할 것이 아니기 때문에 Import 하지 않아도 된다.

![](/assets/images/kubeweblogic4/17_import_datasource.png)

좌측 메뉴의 **Dashboards** 메뉴에서 **New**를 선택하여 Dashboard를 새롭게 하나 생성한다.
Dashboard 이름은 나중에 수정하고 먼저 **Graph** 하나를 추가해 볼 것이다.

![](/assets/images/kubeweblogic4/20_new_dashboard.png)

상단의 **Panel Title**을 클릭하면 **Edit** 할 수 있는 메뉴가 나타난다. 여기서 **Edit**를 클릭하면 하단에 Graph를 수정할 수 있는 속성 탭들이 나타난다.

Data Source Drop Down 메뉴에서 **Prometheus**를 선택하고 텍스트 박스에 Metric 항목의 이름을 입력한다. 아래 예는 **jvm_heap_free_current**를 사용했다. 우측의 **눈** 모양 아이콘을 클릭하면 상단의 그래프에 Metric이 표시된다.

![](/assets/images/kubeweblogic4/21_metric_add.png)

Panel Title은 **General** 탭에서 수정한다.
![](/assets/images/kubeweblogic4/22_pane_title.png)

Metric이 다 추가되었으면 Dashboard의 이름을 변경해준다. Dashboard 옆 **설정** 아이콘을 클릭하면 **Settings** 메뉴가 나온다. 
여기에서 Name 항목을 원하는 이름으로 설정하면 된다.

![](/assets/images/kubeweblogic4/23_dashboard_name.png)

설정이 완료되고 나면 아래와 같이 보이게 될 것이다.

![](/assets/images/kubeweblogic4/18_grafana_dashboard.png)

여기까지 Kubernetes에 배포된 WebLogic Server 환경을 Custom Exporter와 Open Source Monitoring 솔루션인 **Prometheus**를 사용하여 모니터링하고 **Grafana**를 이용하여 더욱 보기 좋은 형태의 Dashboard를 만드는 작업에 대해 다루었다.

# 참고 자료

- [Oracle WebLogic Server Kubernetes Operator](https://oracle.github.io/weblogic-kubernetes-operator/)
- [WebLogic Monitoring Exporter](https://github.com/oracle/weblogic-monitoring-exporter)



