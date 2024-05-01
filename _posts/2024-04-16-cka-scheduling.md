---
title: CKA 정리(Scheduling)
date: 2023-04-16 16:52:00 +0900
last_modified_at: 2024-04-21 23:29:00 +0900
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

- taint : node마다 설정가능하며 설정한 node에는 pod를 scheduling 할 수 없음
- toleration : taint 설정을 무시가능
- master node는 어떠한 pod도 스케줄링 되지 않음 - 클러스터 처음 설정 시 taint가 자동으로 설정되어 pod가 지정되는 걸 막음
- `kubectl describe node kubemaster | grep Taint` 명령을 통해 master node의 Taint확인 가능
- Taints - Node
  - `kubectl taint nodes node-name key=value:{taint-effect}`
  - taint-effect 종류
    - `NoSchedule`
      - toleration을 적용하지 않은 어떠한 새 pod도 해당 node에 스케줄링 되지 않음
      - 기존 pod에는 영향 없으며, 새 pod에만 배치에 제한됨
      - 특정 작업에 특화된 node를 유지하거나, 특정 pod가 특정 노드에만 배치되도록 하고 싶을 때 유용
    - `PreferNoSchedule`
      - taint를 toleration 하지 않은 pod를 가능한 해당 node에 배치 하지 않으려함
      - 다른 모든 node가 사용 불가하거나 해당 node만이 pod의 요구사항을 충족 시 예외적으로 배치가능
      - 선호되지 않는 node에 pod가 배치되는 것을 최소화하려할 때 유용
    - `NoExecute`
      - 해당 taint를 toleration하지 않은 pod는 해당 node에 스케줄링 불가하며 기존에 실행중인 pod도 쫓겨남
      - node의 하드웨어 장애 및 유지보수를 위해 node를 비워야 할 때 유용
- Toleration
  - `kubectl taint nodes node1 app=blue:NoSchedule`의 toleration 설정법
    ```yaml
    # pod-definition.yaml
    apiVersion:
    kind: Pod
    metadata:
      name: myapp-pod
    spec:
      containers:
        - name: nginx-container
          image: nginx
      tolerations:
        - key: "app"
          operator: "Equal"
          value: "blue"
          effect: "NoSchdule"
    ```

### Node Selectors

- 특정 pod를 특정 node에서 실행하기 위함
- node에 적용 된 label을 새로 생성할 pod의 nodeSelector에 설정
- `kubectl label nodes <node-name> <label-key>=<label-value>`를 통해 node에 label적용
- 예시
  ```yaml
  # pod-definition.yaml
  apiVersion:
  kind: Pod
  metadata:
    name: myapp-pod
  spec:
    containers:
      - name: data-processor
        image: data-processor
    nodeSelector:
      size: Large
  ```
  - `kubectl create -f pod-definiton.yaml`

### Node Affinity

- nodeSelector와 비슷하게 node label기반으로 스케줄링
- 필드
  - `requiredDuringSchedulingIgnoredDuringExecution`
    - 조건을 만족하는 node에만 pod가 배치될 수 있음
    - 예를 들어, 특정 CPU를 가진 node에만 Pod를 스케줄링하고 싶을 때 사용
    - Pod가 실행 중일 때 노드의 속성이 변경되어도 Pod는 해당 노드에서 계속 실행됨
  - `preferredDuringSchedulingIgnoredDuringExecution`
    - 지정된 규칙은 스케줄러가 가능한 한 준수하려고 시도하지만, 규칙을 만족하는 노드가 없을 경우 다른 노드에 파드를 스케줄링 가능
    - `requiredDuringSchedulingIgnoredDuringExecution`보다 유연하며 스케줄링의 선호도를 지정하는 데 사용됨
    - 예를 들어, 파드를 특정 지역의 노드에 우선적으로 스케줄링하되, 해당 지역에 사용 가능한 노드가 없는 경우 다른 지역의 노드에도 배치가능

### Taints and Tolerations vs Node Affinity

|               |                                               Taints and Tolerations                                                |                               Node Afinity                                |
| :-----------: | :-----------------------------------------------------------------------------------------------------------------: | :-----------------------------------------------------------------------: |
| 목적과 사용처 | 주로 node에 pod를 스케줄링하지<br>않도록 하기위해 사용<br>특정node가 특정 pod에만 배치되도록 <br>하는데 초점을 맞춤 |   pod가 특정 조건을 만족하는 node에 <br>스케줄링 되도록 유도하는데 사용   |
|   동작 방식   |                                  Taint는 node에 설정,<br>Toleration은 pod에서 설정                                  |              pod spec에 설정<br>노드 선택을 위한 조건을 제시              |
|  선택과 제한  |                                       node의 pod 스케줄링을 제한하는 데 사용                                        | 스케줄링 선호도나 요구사항을 정의하여<br> 더 세밀하게 node선댁을 조정가능 |

### Resource Requirements and Limits

- Resource requests
  - pod 생성 시 필요한 cpudhk memory 양을 지정 가능
  - 컨테이너에 대한 리소스 요청이라고 함
  - 스케줄 컨트롤러가 pod를 배치해달라는 요청받으면 사용 가능한 node를 찾음
    - pod가 node에 배치되면 pod는 사용 가능한 양의 자원을 보장받음
  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: simple-webapp-color
    labels:
      name: simple-webapp-color
  spec:
    containers:
      - name: simple-webapp-color
        image: simple-webapp-color
        ports:
          - containerPort: 8080
  ```
- Resource cpu

  - 1개의 cpu는 1vCPU하나와 같음
  - 1 AWS vCPU = 1 GCP Core = 1 Azure Core = 1 Hyperthread

- Resource memory

  - 접미사를 이용해 지정 (256mebibyte -> 256Mi)
  - 1G (Gigabyte) = 1,000,000,000 bytes
  - 1M (Megabyte) = 1,000,000 bytes
  - 1K (Kilobyte) = 1000 bytes
  - 1Gi (Gibibyte) = 1,073,741,824 bytes
  - 1Mi (Mebibyte) = 1,048,576 bytes
  - 1Ki (Kibibyte) = 1,024 bytes

- Resource limits
  - cpu,memory 갯수를 제한시킴
  - 컨테이너가 여러 개면 각 컨테이너별로 요청이나 제한을 설정 가능
  - cpu 초과 시 시스템이 cpu를 조절해 지정된 한도를 넘지 않도록 함
  - 메모리는 초과 시 OOM(Out of Memory)에러로 종료 됨
    ```yaml
    # pod-definition.yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: simple-webapp-color
      labels:
        name: simple-webapp-color
    spec:
      containers:
        - name: simple-webapp-color
          image: simple-webapp-color
          ports:
            - containerPort: 8080
          resources:
            requests: #제한
              memory: "1Gi"
              cpu: 1
            limits: # 요청
              memory: "2Gi"
              cpu: 2
    ```

### DaemonSet

- ReplicaSet이랑 같음
- 여러 개의 instance pod를 배포하도록 도와줌
- 클러스터의 node마다 pod를 하나씩 실행
- 클러스터에 새 node가 추가될 때마다 pod복제본이 자동으로 해당 node에 추가됨
- node를 제거 시 pod도 같이 제거 됨
- DaemonSet 생성방법은 ReplicaSet 생성방법과 비슷
  - DaemonSet
    ```yaml
    apiVersion: apps/v1
    kind: DaemonSet
    metadata:
      name: monitoring-daemon
    spec:
      selector:
        matchLabels:
          app: monitoring-agent
      template:
        meatadata:
          labels:
            app: monitoring-agent
        spec:
          containers:
            - name: monitoring-agent
              image: monitoring-agent
    ```
  - ReplicaSet
    ```yaml
    apiVersion: apps/v1
    kind: ReplicaSet
    metadata:
      name: monitoring-daemon
    spec:
      selector:
        matchLabels:
          app: monitoring-agent
      template:
        meatadata:
          labels:
            app: monitoring-agent
        spec:
          containers:
            - name: monitoring-agent
              image: monitoring-agent
    ```

### Static Pod

- kubelet은 node를 독립적으로 관리 가능
- pod에 관한 정보를 저장하는 서버디렉토리 : `/etc/kubernetes/manifests`
- kubelet이 스스로 만든 pod는 api서버의 간섭이나 나머지 k8s cluster 구성 요소의 간섭 없음
- ReplicaSet이나 deployment를 생성 불가능
- pod이름 뒤 node의 이름이 붙으면 static pod임
- `ps -ef | grep kubelet`명령을 이용해서 kubelet의 config파일을 열어서 `staticPodPath` 확인

  ```shell
  controlplane ~ ✖ ps -ef | grep kubelet
  root        4406       1  2 03:30 ?        00:00:25 /usr/bin/kubelet --bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf --config=/var/lib/kubelet/config.yaml --container-runtime-endpoint=unix:///var/run/containerd/containerd.sock --pod-infra-container-image=registry.k8s.io/pause:3.9
  root       13593   11716  0 03:48 pts/0    00:00:00 grep --color=auto kubelet

  controlplane ~ ➜  cat /var/lib/kubelet/config.yaml
  apiVersion: kubelet.config.k8s.io/v1beta1
  ...
  ...
  staticPodPath: /etc/kubernetes/manifests
  ...
  ```

### Multiple Schedulers

- k8s cluster는 한번에 여러 스케줄러를 가질 수 있음
- 여러개일 경우 이름이 반드시 달라야함 - 각각의 스케줄러를 구분 가능하도록
- 스케줄러 설정 파일을 통해 각각의 스케줄러의 이름을 설정함
- 이름을 지정하지 않는다면 `default-scheduler`
- pod 생성 시 `schedulerName`를 통해 스케줄러를 지정가능
- 생성 중 문제 발생 시 `kubectl logs <schedulerName>`명령어를 통해 확인 가능
