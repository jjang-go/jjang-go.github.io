---
title: CKA 정리(Core concepts)
date: 2023-04-14 00:00:00 +0900
last_modified_at: 2024-04-16 16:50:00 +0900
categories: [자격증, Cloud, CKA]
tags: [docker, kubernetes, k8s, cka, 자격증]
---

# CKA 자격증

강의 : Udemy - [Certified Kubernetes Administrator (CKA) with Practice Tests](https://www.udemy.com/share/101Xtg3@i_PWod_lMIUhcyrSIngElFmre9WNNhaMnXwaoIwwianw3_xF22Gsc1h4Z6SsVULmiA==/)

## Cluster Architecture

### Kubernetes Architecture

- K8S의 목적 : 응용 프로그램을 `컨테이너형식`으로 호스트 - 자동화된 방식
- 요구사항에 따라 응용프로그램의 많은 인스턴스를 쉽게 배포 및 다양한 서비스간 쉽게 통신가능
- ETCD : HA(고가용성) Key - Value 저장소 => `Key - Value 형식`으로 정보를 저장하는 데이터베이스
- kube-scheduler : 리소스 요구 사항, 노드 용량, 정책, 제약조건, 테인트등 규칙에 근거하여 컨테이너를 설치하기 위해 `올바른 노드를 식별`
- Controller-Manager
  - Node-Controller : 새 노드를 클러스터에 온보딩하고 `노드가 사용 불가능하거나 파괴되는 상황을 처리`
  - Replication-Controller : 원하는 컨테이너 수가 `복제그룹에서 항상 실행되도록 보장`
- kube-apiserver : 클러스터 내에서 모든 작업을 Orchestration, `주기적으로 kubelet으로부터 상태를 모니터링`함
- Runtime Engine : 클러스터 내 `모든 노드(master,worker)에 설치되어있음`, 대표적으로 Docker
- kubelet : 클러스터의 각 노드에서 실행되는 에이전트
  - kube-apiserver의 지시를 듣고 필요한대로 노드에서 컨테이너를 배포 또는 파괴
  - kube-proxy : 작업자 노드에서 실행되는 응용 프로그램은 서로 통신가능하게함

#### ETCD

##### For Beginner

- 분산되고 신뢰할 수 있으며 키워드 가치가 있는 저장소
- 단순, 안전, 신속
- key - value 저장소
  - 키 값 쌍을 저장 : `./etcdctl set key1 value1`
  - 키 값 쌍을 회수 : `./etcdctl get key1`
- ETCD는 2379포트를 통해 사용
- 기본 클라이언트 : ETCDCTL : etcd의 버전 확인 후 사용
- version
  - v0 : 2023.08
  - v2.0 : 2015.02(RAFT합의 알고리즘 재설계)
  - v3.0 : 2017.01(최적화와 성능향상)
  - 2018.11(etcd -> CNCF)

##### In earnest

- 클러스터에 관한 정보 저장(`Node`, `Pods`, `Configs`, `Secrets`, `Accounts`, `Roles`, `Bindings`)
- kube-controller을 실행할 때 얻게 되는 모든 정보는 etcd서버에서 얻음
- etcd-master pod 조회 `kubectl get pods -n kube-system`
- `kubectl exec etcd-master -n kube-system etcdctl get / --prefix -keys-only` : etcd 데이터베이스에서 루트 키(/)를 기준으로 모든 키를 가져오는 명령, `-keys-only` 플래그는 키의 값 대신 키 자체만 반환

#### kube-apiserver

- 주요관리 구성요소, 기타 데이터 저장소와 직접 상호작용하는 유일한 구성요소
- kubectl 명령 실행 -> kube-apiserver에 도달 -> kube admin이 요청을 인증 및 유효성 확인 -> etcd cluster에서 `데이터 회수해 요청된 정보로 응답`
- `요청의 인증과 유효성을 확인`하고 기타등등의 데이터 스토어에서 데이터를 검색하고 업데이트
- Scheduler, kube-controller-manager은 kube-apiserver를 이용해 각 영역의 클러스터에서 업데이트 수행
- kube-apiserver는 많은 매개변수로 실행됨
  - etcd, kubelet의 인증 파일 위치가 매개변수로 연결됨
  - etcd-servers=https://127.0.0.1:2379로 연결
- `kubectl get pods -n kube-system`
- `cat /etc/kubernetes/manifests/kube-apiserver.yaml`
- `ps -aux | grep kube-apiserver`

#### kube-controller-manager

- k8s의 다양한 컨드롤러를 관리
- 시스템 내 다양한 구성 요소의 상태를 지속적으로 모니터링하고 시스템 전체를 원하는 기능 상태로 만드는 것
- node-controller와 replication-controller가 있음

##### node-controller

- kube-apiserver를 통해 `node의 상태 모니터링 및 application이 계속 실행되도록 함`
- node-controller는 `5초마다 node의 상태를 확인`
- node 상태가 갑자기 `NotReady`상태가 되었을 때 40초 후에야 신호 잡힘
- node 상태가 수신불가로 표시되면 다시 뜰 때까지 5분이 걸림
- 할당된 노드의 POD를 제거하고 복제본으로 프로비전

##### replication-controller

- ReplicaSet의 상태를 모니터링하고 원하는 수의 pod를 세트 내에서 항상 사용가능 하도록 함 - Pod가 죽으면 다른 Pod가 생김

#### kube-scheduler

- node 내 pod의 스케줄 관리를 책임짐
- 어떤 pod가 어느 node에 들어갈지만 결정(`pod를 node에 상주시키지 않음` - kubelet의 역할임)
- 리소스 요구사항에 맞는 node에 pod를 배치하는 역할

#### kubelet

- kube-apiserver와 통신을 통해 pod 배포 및 삭제, 모니터링 정보를 일정간격으로 공유함

#### kube-proxy

- Kubernetes 클러스터 내에서 네트워크 트래픽을 관리하는 중요한 구성 요소
- 서비스 간 통신을 가능하게 하고 로드 밸런싱을 수행
- 서비스의 가상 IP 주소와 실제 실행 중인 pod 사이의 연결을 관리
- 네트워크 패킷을 적절한 pod로 전달하여 클러스터 내의 애플리케이션들이 원활하게 통신할 수 있도록 함
- Kubernetes Cluster의 각 노드에서 실행되는 프로세스

### Create and Configure Pods

#### Pod

- k8s에서 만들 수 있는 가장 작은 물체
- app의 단일 인스턴스
- k8s는 worker node에 직접 container를 배포하지 않고, pod라 불리는 객체로 캡슐화되어 배포됨
- spin up시 기존의 pod에 인스턴스를 추가하지 않고 새 pod를 배포함
- 규모 키울려면 새pod 배포, 줄이려면 기존pod 파괴
- 보통 컨테이너와 1:1로 연결됨, 여러개로 구성도 가능함

##### How to create a pad with yaml

- k8s는 yaml파일을 pod, replicaset, deployment, service등 개체 생성을 위한 입력으로 사용
- k8s yaml파일은 항상 4개의 상위레벨 포함(root레벨 속성들)
  - apiVersion
  - kind
  - metadata
  - spec

|    kind    | apiVersion |
| :--------: | :--------: |
|    Pod     |     v1     |
|  Service   |     v1     |
| ReplicaSet |   app/v1   |
| Deployment |   app/v1   |

```yaml
# pod-definition.yml
apiVersion: v1 #만들려는 게 무엇이냐에 따라 올바른 API 버전을 사용해야 함
kind: Pod # 만들려는 개체유형 입력
metadata: # 개체의 데이터
  name: myapp-pod
  labels: # label 기반으로 pod를 필터링 가능(key - value로 label작성)
    app: myapp
    type: front-end
spec: # 개체와 관련된 추가정보를 입력
  containers: # 컨테이너 목록
    - name: nginx-controller # 컨테이너 이름
      image: nginx # 사용하고자하는 컨테이너 이미지명
```

- `kubectl create -f pod-definition.yml`을 통해 pod 생성
- `kubectl get pods`를 통해 pod목록 확인
- `kubectl describe pod myapp-pod`를 통해 생성한 pod 정보 확인

```yaml
# pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    app: nginx
    tier: frontend
spec:
  containers:
    - name: nginx
      image: nginx
```

- container image 수정방법
  1. kubectl edit po <수정할 컨테이너>를 통해서 수정한다.
  2. 생성 시 사용했던 yaml파일을 수정하여 다시 apply를 한다.

#### kube-controller

- k8s의 두뇌
- k8s를 모니터링하고 `그에 따라 반응하는 프로세스`

##### replication-controller

- k8s cluster에 있는 `단일 pod의 다중 인스턴스를 실행하도록 도와줌 - 고가용성 제공`
- pod를 하나만 가질 계획이여도 Replication-Controller 사용가능(pod가 하나여도 Replication-Controller는 `기존 pod가 고장 났을 때 자동으로 새 pod를 불러옴`)
- pod의 갯수 상관없이 `pod가 항상 실행되도록 보장`함
- `서로 다른 node의 여러 pod에 걸쳐 부하를 분산하는데 도움`이 되고 수요가 증가하면 `앱 sale조정도 가능`
- Replication-Controller | Replica Set
  - 용도는 같지만 다름
  - Replication-Controller는 구식 기술로 Replica Set으로 대체되고 있음
  - 복제를 설정하는 새로운 권장 방법

```yaml
# rc-definition.yml
apiVerison: v1
kind: ReplicationController
metadata: # Replication Controller metadata
  name: myapp-rc
  labels:
    app: myapp
    type: front-end
spec: # Replication Controller spec
  template: # pod 정의 - 아래 metadata, spec은 pod에 대한 정보
    metadata: # Pod metadata
      name: myapp-pod
      labels:
        app: myapp
        type: front-end
    spec: # pod spec
      containers:
        - name: nginx-container
          image: nginx
  replicas: 3 # template으로 정의 된 부분을 갯수만큼 생성
```

- `kubectl create -f rc-definition.yml`을 이용하여 생성
- `kubectl get replicationcontroller`를 통해 생성 확인
- `kubectl get pods`를 통해 생성된 복제본 pod 확인

##### ReplicaSet

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata: # replicaset metadata
  name: myapp-replicaset
  labels:
    app: myapp
    type: front-end
spec: # replicaset spec
  template: # pod 정보 정의
    metadata: # pod metadata
      name: myapp-pod
      labels:
        app: myapp
        type: front-end
    spec: # pod spec
      containers: # container information
        - name: nginx-container
          image: nginx
  replicas: 3
  # replicaset는 selector가 필수
  selector:
    matchLabels: # 레이블 형식으로 작성
      type: front-end
  # 복제 컨트롤러와 복제본 세트의 가장 큰 차이점
  # replicationcontroller는 selector가 필수는 아니지만 사용은 가능
```

- `kubectl create -f replicaset-definition.yml`을 통해 replicaset 생성
- `kubectl get replicaset`을 통해 생성 확인
- `kubectl get pods`를 통해 pod 확인
- 복제본 수 변경방법
  1. 생성한 yaml파일에서 변경 후 `kubectl replace -f 파일명`을 통해 변경
  2. `kubectl scale --replicas=6 -f 파일명`을 통해 변경
  3. `kubectl scale --replicas=6 replicaset replicaset이름`을 통해 변경

#### deployment

- 애플리케이션을 배포하고 관리하기 위한 리소스
- 원하는 수의 애플리케이션 인스턴스를 선언적으로 관리가능
- 시스템은 자동으로 해당 상태를 유지하려고 시도함
- 이를 통해 업데이트, 롤백 등의 작업을 쉽게 할 수 있으며, 애플리케이션의 가용성과 확장성을 향상시킬 수 있음
- Pod의 집합으로 구성되며, ReplicaSet을 사용하여 Pod의 복제본을 관리
- app 배포와 운영 자동화
- ReplicaSet yaml과 비슷, Kind만 변경?

#### Service

- app 안팎의 다양한 구성 요소 간의 통신을 가능하게함
- app을 다른 app 또는 사용자와 연결하는 걸 도와줌

##### Service Type

###### NodePort

- Service가 내부 port를 node의 port에 접근 가능하게 함

```yaml
# service-definition.yml
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  type: NodePort
  ports:
    - targetPort: 80
      port: 80 # 필수 입력
      nodePort: 30080
  selector: # service를 사용하고자하는 pod의 metadata에 정의한 라벨을 selector에 입력
    app: myapp
    type: front-end
```

- `kubectl create -f service-definition.yml`로 서비스 생성
- `kubectl get services`로 생성된 서비스 확인
- `curl http://Cluster-IP:30080`으로 웹 서비스 접속 확인
- pod가 여러개면 service와 pod들의 label이 같은 key-value이면 됨

###### Cluster IP

- pod그룹에 접근하기 위한 단일 인터페이스를 제공하는 서비스 유형
- 애플리케이션의 다른 계층 간의 통신이 가능해지며, pod의 IP 주소가 동적으로 변경되어도 연결성이 유지
- 각 서비스는 고유한 IP와 이름을 할당받아, 다른 pod가 이 이름을 사용하여 서비스에 접근가능해짐
- 마이크로서비스 기반 애플리케이션의 효과적인 배포와 확장성이 보장됨

```yaml
apiVersion: v1
kind: Service
metadata:
  name: back-end
spec:
  type: ClusterIP
  ports:
    - targetPort: 80
      port: 80
  selector:
    app: myapp
    type: back-end
```

###### LoadBalancer

```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  type: LoadBalancer
  ports:
    - targetPort: 80
      port: 80
      nodePort: 30080
```

#### Namespace

- default : 사용자가 명시적으로 네임스페이스를 지정하지 않을 때 기본적으로 사용되는 네임스페이스 - k8s 초기 설정 시 자동 생성됨
- kube-system : 시스템 자체에 의해 생성되고 사용되는 핵심 컴포넌트들이 실행되는 네임스페이스
- kube-pulic : 클러스터 전체에서 공개적으로 액세스 가능한 리소스를 저장하는 네임스페이스로, 모든 사용자(인증되지 않은 포함)에게 읽기가 허용됨
- 명령을 이용한 namespace활용법
  - namespace 생성법
    1. yaml을 통한 생성
       ```yaml
       apiVersion: v1
       kind: Namespace
       metadata:
         name: dev
       ```
    2. `kubectl create namespace dev`로 namespace 생성
  - `kubectl get pods --namespace=네임스페이스명`을 사용 시 해당 namespace에 있는 pod를 조회
  - `kubectl create -f 파일명 --namespace=네임스페이스명`을 사용 시 해당 namespace에 yaml파일에 기재된 정보를 바탕으로 resource생성
  - yaml파일의 metadata를 통한 방법
    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: myapp-pod
      namespace: dev
      labels:
        app: myapp
        type: front-end
    spec:
      containers:
        - name: nginx-container
          image: nginx
    ```

#### Imperative vs Declarative

##### Imperative(명령적 접근법)

- Create Objects

  ```shell
  kubectl run --image=nginx nginx
  kubectl create deployment --image=nginx nginx
  kubectl expose deployment nginx --port 80
  ```

- Update Objects

  ```shell
  kubectl edit deployment nginx
  kubectl scale deployment nginx --replicas=5
  kubectl set image deployment nginx nginx=nginx:1.18
  ```

###### Imperative Object Configuration Files

- Create Objects

  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: myapp-pod
    labels:
      app: myapp
      type: front-end
  spec:
    containers:
      - name: nginx-container
        image: nginx
  ```

  ```shell
  kubectl create -f nginx.yaml
  kubectl replace -f nginx.yaml
  kubectl delete -f nginx.yaml
  ```

- Update Objects
  - create, replace 사용 시 개체 미리 확인 필요
  ```shell
  kubectl edit deployment nginx
  kubectl replace -f nginx.yaml
  kubectl replace --force -f nginx.yaml
  kubectl create -f nginx.yaml
  kubectl replace -f nginx.yaml
  ```

##### Declarative(선언적 접근법)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
    type: front-end-service
spec:
  containers:
    - name: nginx-container
      image: nginx:1.18
```

- Create Objects

  ```shell
  kubectl apply -f nginx.yaml
  kubectl apply -f /path/to/config-files
  ```

- Update Objects
  ```shell
  kubectl apply -f nginx.yaml
  ```

#### Kubectl Apply

- 로컬 구성파일과 k8s의 라이브 개체를 정의
- create, replace명령 대신 kubectl apply를 사용하여 객체를 관리하는 명령을 적용
- 개체가 아직 존재하지 않는 경우 개체를 만들 수 있을만큼 지능적
- 여러 개체 구성 파일을 단일 파일 대신 directory 경로로 지정가능
