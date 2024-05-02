---
title: CKA 정리(Cluster Maintenance)
date: 2023-04-24 16:07:00 +0900
#last_modified_at: 2024-04-24 16:05:00 +0900
categories: [Cloud, 자격증, CKA]
tags: [docker, kubernetes, k8s, cka, 자격증]
---

# Cluster Maintenance

강의 : Udemy - [Certified Kubernetes Administrator (CKA) with Practice Tests](https://www.udemy.com/share/101Xtg3@i_PWod_lMIUhcyrSIngElFmre9WNNhaMnXwaoIwwianw3_xF22Gsc1h4Z6SsVULmiA==/)

## Cluster Upgrade Management

### OS Upgrades

- SW 기반 upgrade, 패치적용(보안 등) - 유지보수 목적
- node가 5분 이상 다운되면 해당 node에서 pod를 종료
- master node는 node가 오프라인이 될 때마다 최대 5분까지 기다림
- pod가 ReplicaSet의 일부라면 다른 node에서 재구성됨
- cluster 내 다른 node로 pod 옮기는 법(다른 node로 pod 재현시킴)

  ```shell
  # <nodeName>노드 안의 pod를 다른 node에 재현 되도록 함
  # <nodeName>노드에는 명령이 실행 된 뒤에는 schedule이 불가
  kubectl drain <nodeName>

  # <nodeName>노드에 schedule이 다시 가능하도록 설정
  kubectl uncordon <nodeName>
  ```

#### Kubernetes Releases

- `v1.11.3` : 1. Major 버전, 2. Minor 버전, 3. Patch 버전
- k8s github repository에 모든 릴리즈 확인 가능
- 구성요소마다 버전이 다를 수 있음

### Cluster Upgrade Process

- kube-apiserver는 controlplane은 주요 구성요소이고 다른 구성요소들과 통신하는 구성요소이기 때문에 어떤 다른 구성요소도 kube-apiserver보다 높은 버전은 안됨

  - kube-apiserver가 v1.29라면
  - kube-controller-manager, kube-scheduler는 v1.28, v1.29버전을 사용 가능하고
  - kubelet, kube-proxy는 v1.27, v1.28, v1.29버전 까지 사용가능
  - 한 번에 Minor 버전 하나씩 upgrade 권장
  - cluster upgrade : Master Node -> Worker Node
  - Master Node
    - Master Node upgrade되는 동안 controlplane의 구성요소인 스케줄러나 컨트롤러 관리자는 잠시 다운됨
    - Master Node가 잠시 다운되도 Cluster 상의 Worker Node나 App이 영향 받지 않음
  - Worker Node
    - Worker Node upgrade 시 다양한 전략이 사용됨
      - **한번에 모두 upgrade**
        - pod가 다운됨 - 사용자 app 접속 불가 - upgrade 완료 후 새 pod 배포되면 사용가능
        - `가동 중지 시간`이 필요한 전략
      - **Worker Node 하나씩 upgrade**
        - upgrade 할 node의 pod를 다른 node로 옮긴 후 upgrade
      - **새 node 연결 후 기존 node를 삭제**
        - cloud환경에서 유용

- master node upgrade

  ```shell
  # scheduling 안되도록 설정
  k drain <MasterNodeName> --ignore-daemonsets

  # https://v1-29.docs.kubernetes.io/ko/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/

  # upgrade할 버전 결정 - Ubuntu
  apt update
  apt-cache madison kubeadm

  # 만약 upgrade 할 버전이 없다면
  #
  # echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/<which-version-to-upgrade>/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
  #
  # https://v1-29.docs.kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/ 확인하기
  #
  # 아래 명령어 실행 후 위의 upgrade 결정 단계를 한번 더 해서 버전 확인 하기
  echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.29/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list

  # 설치 할 버전을 입력
  apt-mark unhold kubeadm && \
  apt-get update && apt-get install -y kubeadm=1.29.0-0.0 && \
  apt-mark hold kubeadm

  # 설치 후 버전 확인
  kubeadm version

  # upgrade 계획 확인
  kubeadm upgrade plan 1.29.0

  # upgrade
  kubeadm upgrade apply v1.29.0

  # kubelet과 kubectl 업그레이드 - Ubuntu
  apt-mark unhold kubelet kubectl && \
  apt-get update && apt-get install -y kubelet=1.29.0-0.0 kubectl=1.29.0-0.0 && \
  apt-mark hold kubelet kubectl

  # kubelet 재시작
  sudo systemctl daemon-reload
  sudo systemctl restart kubelet

  # 다시 scheduling 가능하도록 설정
  kubectl uncordon <MasterNode>
  ```

- worker node upgrade

  ```shell
  # master node에서 업그레이드 할 worker node에 scheduling이 안되도록 설정
  kubectl drain <WorkerNodeName>

  # worker node 접속
  ssh <WorkerNodeName>

  # https://v1-29.docs.kubernetes.io/ko/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/

  # upgrade할 버전 결정 - Ubuntu
  apt update
  apt-cache madison kubeadm

  # 만약 upgrade 할 버전이 없다면
  #
  # echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/<which-version-to-upgrade>/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
  #
  # https://v1-29.docs.kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/ 확인하기
  #
  # 아래 명령어 실행 후 위의 upgrade 결정 단계를 한번 더 해서 버전 확인 하기
  echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.29/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list

  # 설치 할 버전을 입력
  apt-mark unhold kubeadm && \
  apt-get update && apt-get install -y kubeadm=1.29.0-0.0 && \
  apt-mark hold kubeadm

  # 설치 후 버전 확인
  kubeadm version

  # upgrade 계획 확인
  kubeadm upgrade plan 1.29.0

  # upgrade
  kubeadm upgrade apply v1.29.0

  # kubelet과 kubectl 업그레이드 - Ubuntu
  apt-mark unhold kubelet kubectl && \
  apt-get update && apt-get install -y kubelet=1.29.0-0.0 kubectl=1.29.0-0.0 && \
  apt-mark hold kubelet kubectl

  # kubelet 재시작
  sudo systemctl daemon-reload
  sudo systemctl restart kubelet

  # worker node 종료
  exit

  # 다시 scheduling 가능하도록 설정
  kubectl uncordon <WorkerNode>
  ```

### Backup and Restore

- 명령적 접근법보다 선언적 접근법이 선호됨
- 모든 object에 대한 resource구성을 복사해 저장도 가능
  ```shell
  kubectl get all --all-namespaces -o yaml > all-deploy-service.yaml
  ```

#### Backup - ETCD

```shell
# trusted-ca-file, cert-file, key-file 경로 확인
kubectl describe pods -n kube-system etcd-controlplane
# trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
# cert-file=/etc/kubernetes/pki/etcd/server.crt
# key-file=/etc/kubernetes/pki/etcd/server.key

# etcd backup
ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/etcd/server.crt --key=/etc/kubernetes/pki/etcd/server.key \
  snapshot save snapshot.db

ETCDCTL_API=3 etcdctl snapshot save snapshot.db

# etcd status
ETCDCTL_API=3 etcdctl snapshot status snapshot.db
```

#### Restore - ETCD

```shell
# 복원 전 kube-apiserver를 중단 - cluster 재시작하기 위해
service kube-apiserver stop

# data-dir 경로 확인
kubectl describe pods -n kube-system etcd-controlplane
# data-dir=/var/lib/etcd

# 복원
ETCDCTL_API=3 etcdctl \
  snapshot restore snapshot.db \
  --data-dir /var/lib/etcd
```
