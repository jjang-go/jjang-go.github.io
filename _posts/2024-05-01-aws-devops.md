---
title: AWS DevOps
date: 2023-05-01 20:54:00 +0900
#last_modified_at: 2024-04-24 16:05:00 +0900
categories: [Cloud, AWS]
tags: [aws, cicd, devops]
---

itwill - [AWS DevOps](https://www.e-itwill.com/course/course_view.jsp?id=437&cid=12&ch=course)

# AWS 배포 기본 지식 학습

## 학습 목표(AWS)
- Cloud Service를 활용하기 위해 기본지식 학습 - `AWS`, `Linux`, `Network`
- Cloud Service에 내 프로젝트를 단순 배포하기 위해 환경을 구축 - `EC2`
- Cloud Service에 내 프로젝트를 배포를 간편하게 한다. - `shell script`
- Cloud Service에 환경 구축없이 내 프로젝트를 배포 - `Elastic Beanstalk` PaaS
- Cloud Service에 배포 자동화를 구축 (CI/CD) - `Github Action`
- Cloud Service에 무중단 배포 구성 - `Load Balancer`, `Rolling Update`
- 정적 IP할당을 위해 Network Load Balancer를 활용

## 최종 목표
프로젝트(code, test code) -> github push -> Github Action(test, build, deploy) -> aws : LB(트래픽 부하분산)-EC2

### 전산실 구
