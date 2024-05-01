---
title: CKA 정리(Logging and Monitoring)
date: 2023-04-21 20:12:00 +0900
# last_modified_at: 2024-04-21 23:29:00 +0900
categories: [Cloud, 자격증, CKA]
tags: [docker, kubernetes, k8s, cka, 자격증]
---

# Logging and Monitoring

강의 : Udemy - [Certified Kubernetes Administrator (CKA) with Practice Tests](https://www.udemy.com/share/101Xtg3@i_PWod_lMIUhcyrSIngElFmre9WNNhaMnXwaoIwwianw3_xF22Gsc1h4Z6SsVULmiA==/)

## Monitor Cluster Components

### Monitoring

- k8s는 풀 기능이 탑재된 모니터링 솔루션 없음
- 오픈소스솔루션 사용해야함(Metric SErver, Prometheus, Elastic Stack)
- 과거에는 Heapster 라는 모니터링 프로젝트를 사용했으나, 현재는 사용하지 않고 Metric Server 라는 간소화된 방식을 사용

#### Metrics Server

- 인메모리 모니터링 솔루션, 디스크에 지표를 저장x
- 설치방법
  1. `minikube addons enable metrics-server`명령으로 metric-server 활성화
  2. `git clone https://github.com/kubernetes-incubator/metrics-server.git && kubectl create -f ..`

## Application Logs

- `kubectl logs -f <podName>` 명령어로 log확인
- `kubectl logs -f <podName> -c <containerName>`
