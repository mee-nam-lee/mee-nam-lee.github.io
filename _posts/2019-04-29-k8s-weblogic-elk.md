---
layout: post
title:  WebLogic on Oracle Kubernetes Engine(OKE) - 로그 모니터링 (ELK) 
categories: Cloud
tags: [WebLogic, Kubernetes, Oracle Kubernetes Engine, ELK, Elasticsearch, Logstash, Kibana]
---

이 문서에서는 **ELK(Elasticsearch, Logstash, Kibana)**를 통해 이전에 배포한 **WebLogic Operator**와 **WebLogic Domain** 환경의 로그를 수잡하여 모니터링하는 방법에 대해서 다룰 것이다.

# Architecture

![](/assets/images/kubeweblogic5/00_arch5.png)

# Kubernetes WebLogic Operator 구성 변경

**Kubernetes WebLogic Operator**에는 **ELK**를 통해 로그 모니터링을 하는 환경이 기본적으로 설정되어있다. 
WebLogic Operator POD 내에 logstach container를 구동하도록 설정되어 있는데 기본적으로는 이 구성이 **disable**로 되어있기 때문에 **elkIntegrationEnabled**라는 속성 항목을 **true**로 변경하여야 **logstash** container가 구동된다.
WebLogic Operator은 이전 과정에서 helm을 통해 이미 설치 되어있기 때문에 기존 설치된 weblogic-operator의 **upgrade**를 통해 속성 값을 변경해 줄 것이다.

```
helm upgrade weblogic-operator weblogic-operator/weblogic-operator --set elkIntegrationEnabled=true
```

**weblogic-operator**가 upgrade 되면서 REVISION 값이 변경되는 것을 확인할 수 있다.

![](/assets/images/kubeweblogic5/01_helm_upgrade.png)

속성 값이 잘 반영되었는지 다음으로도 확인할 수 있다.

![](/assets/images/kubeweblogic5/02_helm_get_values.png)

**history**로 보게 되면 REVISION 값이 변경되어있는 것을 확인 할 수 있다.

![](/assets/images/kubeweblogic5/03_upgrade_complete.png)

**weblogic-operator**의 POD 정보를 보면 **logstash** container가 기동되었기 때문에 Running Container 수가 2로 변경된 것을 볼 수 있다.

![](/assets/images/kubeweblogic5/04_operator2running.png)

# Elasticsearch, Kibana 배포

제공되는 샘플 스크립트 디렉토리로 이동하면 Elasticsearch와 Kibana를 배포할 수 있는 구성파일이 존재한다.

```
cd weblogic-kubernetes-operator/kubernetes/samples/scripts/elasticsearch-and-kibana
```

**elasticsearch_and_kibana.yaml** 파일을 열어서 Service 항목을 살펴보자. Elasticsearch와 Kibana를 위한 각각의 서비스 항목이 있을 것이다. 이 문서의 설명에서는 외부에서 Elasticsearch에 접속할 필요는 없기 때문에 elasticsearch 서비스는 그대로 두고 **kibana** 서비스만 아래와 같이 **LoadBalancer** type으로 수정해 준다.

```yaml
apiVersion: "v1"
kind: "Service"
metadata: 
  namespace: "default"
  name: "kibana"
  labels: 
    app: "kibana"
  annotations:
    service.beta.kubernetes.io/oci-load-balancer-shape: "100Mbps"
    service.beta.kubernetes.io/oci-load-balancer-backend-protocol: "HTTP"
spec: 
#  type: "NodePort"
  type: LoadBalancer
  ports:
    - port: 5601
      targetPort: 5601
  selector: 
    app: "kibana"
```
변경한 파일을 적용한다.
```
kubectl apply -f elasticsearch_and_kibana.yaml
```

![](/assets/images/kubeweblogic5/04_elk_apply.png)

생성된 POD를 확인한다. Elasticsearch container에 직접 접속하여 구성이 잘 되었는지 확인해 본다.

![](/assets/images/kubeweblogic5/05_elastic_logstash.png)

이제 Kibana 콘솔에 접속하기 위하여 Kibana 서비스의 External IP를 확인한다.

![](/assets/images/kubeweblogic5/06_kibana_svc.png)

브라우저에 다음과 같이 입력하여 상태가 **Green**이면 정상이다.

- http://[kibana-service-external-ip]:5601/status

![](/assets/images/kubeweblogic5/07_kibana_status.png)

식별할 Elasticsearch의 Index Pattern을 구성하고 **Create**를 클릭하면 바로 수집된 로그가 Kibana에서 보여진다.

![](/assets/images/kubeweblogic5/08_kibana_config.png)

**Discover** 탭에서 수집된 로그들을 아래와 같이 볼 수 있다.

![](/assets/images/kubeweblogic5/09_kibana_discover.png)

여기까지는 WebLogic Kubernetes Operator의 로그 수집 과정이었다. 

# WebLogic Server 로그 수집

이제 WebLogic Domain의 각 WebLogic Server들의 로그를 수집해 보자. 위 과정에서 사용한 **Logstash**는 WebLogic Kubernetes Operator 구성에 포함된 것으로, 이 설정을 이용해서는 외부의 WebLogic Domain 로그를 수집할 수 없다.
이 내장 **Logstash** 설정을 변경하는 대신 새롭게 WebLogic Domain 로그를 수집할 **Logstash**를 추가로 구성해 볼 것이다.

여기에서 사용하는 WebLogic Domain은 이전 글에서 생성한 **Persistent Volume**을 사용하는 WebLogic Domain이다. 따라서 이 WebLogic Domain의 log들은 **Persistent Volume**, **/shared/logs**에 저장되도록 설정되어있다.

추가로 구성하는 **Logstash**는 shared log 디렉토리 내의 로그들을 수집하도록 구성할 것이다.

## Logstash 구성 

추가의 **Logstach** 구성을 위한 **logstash.yaml** 파일을 다음과 같이 작성한다.
이 파일에서는 **/shared**를 참조하는 미리 만들어진 **weblogic-domain-storage-volume**을 mount할 것이다.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: logstash
  namespace: default
spec:
  replicas: 1 
  selector:
    matchLabels:
      app: "logstash"
  template: 
    metadata:
      labels:
        app: logstash
    spec:
      volumes:
      - name: weblogic-domain-storage-volume
        persistentVolumeClaim:
          claimName: domain1-weblogic-sample-pvc
      containers:
      - name: logstash
        image: logstash:6.6.0
        args: ["-f", "/shared/logstash/logstash.conf"]
        env:
        - name: ELASTICSEARCH_HOST
          value: elasticsearch.default.svc.cluster.local
        - name: ELASTICSEARCH_PORT
          value: "9200"
        imagePullPolicy: IfNotPresent
        volumeMounts:
        - mountPath: /shared
          name: weblogic-domain-storage-volume
        ports:
        - containerPort: 5044
          name: logstash
```

위 파일을 적용하기 앞서서 logstash 시작에 필요한 커스텀 구성 파일 **logstash.conf** 파일을 작성해 주어야 한다.
이 파일은 **Persistent Volume**으로 mount될 **/shared** 디렉토리 내에 폴더를 만들어 구성하고 이를 참조해서 Logstash가 구동되도록 할 것이다.

**Public Subnet**에 구성해 둔 **Bastion Server**에 접속하여 Shared File System에 접속해서 작업하도록 한다. (접속 방법은 이전 글 참고)

shared 폴더로 이동하여 **logstash** 폴더를 추가해 주고 **logstash.conf** 파일을 생성한다.

> 참고 :  
> Bastion Server에서 사용하는 Mount Path는 꼭 /shared가 아닌 다른 위치에 mount해서 사용해도 된다.
> 아래 예시에서는 /mnt/shared에 Mount되어있다.

![](/assets/images/kubeweblogic5/14_logstash_conf.png)

**logstash.conf** 파일의 내용을 다음과 같이 작성해 준다.

```conf
input {
  file {
    path => "/shared/logs/domain1/*.log*"
    start_position => "beginning"
    type => "weblogic-server"
  }
}

filter {
  grok {
       match => [ "message", "<%{DATA:log_timestamp}> <%{WORD:log_level}> <%{WORD:thread}> <%{HOSTNAME:hostname}> <%{HOSTNAME:servername}> <%{DATA:timer}> <<%{DATA:kernel}>> <> <%{DATA:uuid}> <%{NUMBER:timestamp}> <%{DATA:misc}> <%{DATA:log_number}> <%{DATA:log_message}>" ]
  }
  date {
     match => [ "timestamp", "MM-dd-YYYY'T'HH:mm:ss.SSSZ" ]
     locale => "en"
     timezone => "UTC"
     target => "@timestamp"
  }
  mutate {
     remove_field => ["message","timestamp"]
  }
}

output {
  elasticsearch {
    hosts => ["${ELASTICSEARCH_HOST}:${ELASTICSEARCH_PORT}"]
  }
  stdout { codec => rubydebug }
}
```

파일 준비가 완료되었으면 logstash POD를 생성한다.

```
kubectl apply -f logstash.yaml
```
다음과 깉이 새로운 logstash Pod가 생성될 것이다.

![](/assets/images/kubeweblogic5/10_logstash_pod.png)

logstash에서 Log를 잘 수집하고 있는지 Container 로그를 살펴본다. 로그가 수집되고 있는 상황이 계속 Logging 될 것이다.

![](/assets/images/kubeweblogic5/11_logstash_logs.png)

**Kibana**에 들어가서 로그를 살펴보면 아래와 같이 **webLogic-server** type의 로그가 들어온 것을 볼 수 있다.

![](/assets/images/kubeweblogic5/12_kibana_logs.png)

그 동안 쌓여있는 WebLogic Log들이 많았기 때문에 기존에 수집한 WebLogic Operator 로그와 수를 비교해 보면 WebLogic Server 로그가 상대적으로 매우 많은 것을 볼 수 있다.

![](/assets/images/kubeweblogic5/13_kibana_graph.png)

이제 로그 수집을 위한 모든 준비가 다 되었으니 원하는 형태로 로그를 보기 위해 **Dashboard**를 꾸며서 보기만 하면 된다.
**Dev Tool**을 이용해 Elasticsearch를 API 기반으로 조회해 보고 결과를 확인해 보는 것도 좋을 것이다.

![](/assets/images/kubeweblogic5/16_kibana_devtool.png)

아래는 샘플로 두개의 Graph를 넣어 만든 Dashboard 이다.

![](/assets/images/kubeweblogic5/15_kibana_dashboard.png)

여기까지 Kubernetes에 배포된 WebLogic Server의 로그 모니터링에 대한 방법에 대해 다루었다.

# 참고 자료

- [Oracle WebLogic Server Kubernetes Operator](https://oracle.github.io/weblogic-kubernetes-operator/)




