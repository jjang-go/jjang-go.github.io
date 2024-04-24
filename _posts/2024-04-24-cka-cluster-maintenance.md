---
title: CKA 정리(Cluster Maintenance)
date: 2023-04-24 16:07:00 +0900
#last_modified_at: 2024-04-24 16:05:00 +0900
categories: [자격증, Cloud, CKA]
tags: [docker, kubernetes, k8s, cka, 자격증]
---

# Cluster Maintenance

강의 : Udemy - [Certified Kubernetes Administrator (CKA) with Practice Tests](https://www.udemy.com/share/101Xtg3@i_PWod_lMIUhcyrSIngElFmre9WNNhaMnXwaoIwwianw3_xF22Gsc1h4Z6SsVULmiA==/)

## Cluster Upgrade Management

### OS Upgrades

- SW 기반 업그레이드, 패치적용(보안 등) - 유지보수 목적
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
