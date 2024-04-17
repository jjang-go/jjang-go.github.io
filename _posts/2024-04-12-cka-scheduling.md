---
title: CKA 정리(Scheduling)
date: 2023-04-16 16:52:00 +0900
last_modified_at: 2024-04-17 23:29:00 +0900
categories: [자격증, Cloud, CKA]
tags: [docker, kubernetes, k8s, cka, 자격증]
---

# Scheduling

강의 : Udemy - [Certified Kubernetes Administrator (CKA) with Practice Tests](https://www.udemy.com/share/101Xtg3@i_PWod_lMIUhcyrSIngElFmre9WNNhaMnXwaoIwwianw3_xF22Gsc1h4Z6SsVULmiA==/)

## Manual Scheduling

```yaml
# pod-definition.yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
    labels:
      name: nginx
spec:
  containers:
  - name: nginx
    image: nginx
    ports:
    - containerPort: 8080
  nodeName: node02
```

- nodeName 속성을 사용하여 pod에 node를 지정
- nodeName으로 node를 지정 시 이후 수정 불가
- binding object를 생성하고 api post로 요청
  ```yaml
  # Pod-bind-definition.yaml
  apiVersion: v1
  kind: Binding
  metadata:
    name: nginx
  target:
    apiVersion: v1
    kind: Node
    name: node02
  ```

### Labels and Selectors

- Labels와 Selector는 그룹화하는 표준방법
- key - value 형태인 Labels을 작성하여 여러 리소스를 그룹화
- Selector를 이용하여 그룹화를 필터링하는 데 도움이 됨

```yaml
# pod-definition.yaml
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp
  labels:
    app: App1
    function: Front-end
spec:
  containers:
    - name: simple-webapp
      image: simple-webapp
      ports:
        - containerPort: 8080
```

- `kubectl get pods --selector app=App1`을 통해 pod 조회 가능

```yaml
# replicaset-definition.yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: simple-webapp
  labels:
    app: App1
    function: Front-end
  annotations: # 주석 - 정보 수집목적으로 다른 세부 사항을 기록하는데 사용
    buildversion: 1.34
spec:
  replicas: 3
  selector:
    matchLabels:
      app: App1
  template:
    metadata:
      labels:
        app: App1
        function: Front-end
    spec:
      containers:
        - name: simple-webapp
          image: simple-webapp
```

### Taints and Tolerations

#### taint

- node마다 설정가능
- 설정한 node에는 pod를 scheduling 할 수 없음
- Taints - Node
  - `kubectl taint nodes node-name key=value:{taint-effect}`
  - taint-effect 종류
    - `NoSchedule`
    - `PreferNoSchedule`
    - `NoExecute`
