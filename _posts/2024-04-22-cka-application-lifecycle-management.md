---
title: CKA 정리(Application Lifecycle Management)
date: 2023-04-22 11:33:00 +0900
# last_modified_at: 2024-04-21 23:29:00 +0900
categories: [자격증, Cloud, CKA]
tags: [docker, kubernetes, k8s, cka, 자격증]
---

# Application Lifecycle Management

강의 : Udemy - [Certified Kubernetes Administrator (CKA) with Practice Tests](https://www.udemy.com/share/101Xtg3@i_PWod_lMIUhcyrSIngElFmre9WNNhaMnXwaoIwwianw3_xF22Gsc1h4Z6SsVULmiA==/)

## Rolling Update and Rollbacks in Deployments

### Rollout

- 첫 deployment를 생성하면 Rollout을 trigger
- 새 Rollout은 새 deployment revision을 생성
- status & history 확인 명령어

  ```shell
  # status 확인
  kubectl rollout status deployment/<deploymentName>

  # history 확인
  kubectl rollout history deployment/<deploymentName>
  ```

#### Deployment Strategy

- 새 버전 업그레이드 방법

  1. 기존 버전 모두 파괴 후 새 버전 배포(recreate strategy)
     - 업그레이드 과정에서 사용자에 접근이 불가능해짐
       ```yaml
       apiVersion: apps/v1
       kind: Deployment
       metadata:
         name: recreate-deployment # Deployment의 이름
       spec:
         replicas: 3 # Deployment가 관리할 Pod의 수
         strategy:
           type: Recreate # Deployment 전략 유형
         selector:
           matchLabels:
             app: myapp # 이 Deployment가 관리할 Pod를 선택하는 데 사용되는 레이블
         template:
           metadata:
             labels:
               app: myapp # Pod 템플릿의 레이블
           spec:
             containers:
               - name: myapp-container # 컨테이너의 이름
                 image: myapp:1.0.0 # 사용할 이미지
       ```
  2. 기존 버전을 하나씩 새 버전으로 교체하는 방식(rolling update)
     - 기본 배포전략
       ```yaml
       apiVersion: apps/v1
       kind: Deployment
       metadata:
         name: rollingupdate-deployment # Deployment의 이름
       spec:
         replicas: 3 # Deployment가 관리할 Pod의 수
         strategy:
           type: RollingUpdate # Deployment 전략 유형
           rollingUpdate:
             maxSurge: 1 # 업데이트 과정에서 추가될 수 있는 Pod의 최대 수 (절대값 또는 비율)
             maxUnavailable: 0 # 업데이트 과정에서 사용할 수 없게 될 최대 Pod 수 (절대값 또는 비율)
         selector:
           matchLabels:
             app: myapp # 이 Deployment가 관리할 Pod를 선택하는 데 사용되는 레이블
         template:
           metadata:
             labels:
               app: myapp # Pod 템플릿의 레이블
           spec:
             containers:
               - name: myapp-container # 컨테이너의 이름
                 image: myapp:1.0.0 # 사용할 이미지
       ```

##### command

```shell
# create
kubectl create -f deployment-definition.yaml

# get
kubectl get deployments

# update
kubectl apply -f deployment-definition.yaml
kubectl set image deployment/myapp-deployment nginx=nginx:1.9.1

# status
kubectl rollout status deployment/myapp-deployment
kubectl rollout history deployment/myapp-deployment

# rollback
kubectl rollout undo deployment/myapp-deployment
```

## Configure Application

### Commands and Arguments

#### Commands

- docker run 명령에 명령을 추가 - 이미지에 지정된 기본 명령을 재정의
  - 5초 간 실행
    ```shell
    # docker run ubuntu [COMMAND]
    # 5초 간 sleep 실행
    docker run ubuntu sleep 5
    ```
- 영구적인 명령 실행은 CMD를 정의하고 빌드해야함

  - 정의

    ```dockerfile
    FROM Ubuntu

    # CMD 입력 방식

    # CMD command param1 param2
    CMD sleep 5

    # CMD ["command", "param1", "param2"]
    # CMD ["sleep","5"]
    ```

  - 빌드 및 실행

    ```shell
    # build
    docker build -t ubuntu-sleeper .

    # run
    docker run ubunntu-sleeper
    ```

  - ENTRYPOINT : 진입점 명령 - 실행될 명령을 지정가능

    ```dockerfile
    FROM Ubuntu

    ENTRYPOINT ["sleep"]

    # CMD를 이용하여 기본값 지정 - param 입력 하지 않았을때를 문제발생을 예방함
    CMD ["5"]
    ```

    - `docker run ubuntu-sleeper 10` 명령 시 sleep 10이 적용됨
    - `docker run --entrypoint sleep2.0 ubuntu-sleeper 10`을 이용하여 sleep 진입점을 sleep2.0으로 변경가능 -> `sleep2.0 10`이 실행됨

  - 이미지 생성 후 pod로 생성
    ```yaml
    # pod-definition.yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: ubuntu-sleeper-pod
    spec:
      containers:
        - name: ubuntu-sleeper
          image: ubuntu-sleeper
          command: ["sleep2.0"] # ENTRYPOINT를 재정의할때 사용
          args: ["10"] # ENTRYPOINT에 사용 될 매개변수
    ```

### Configure Environment Variables & ConfigMaps in Applications

#### ENV Variables in Kubernetes

- Plain Key Value
  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: simple-webapp-color
  spec:
    containers:
      - name: simple-webapp-color
        image: simple-webapp-color
        ports:
          - containerPort: 8080
        env: #환경변수 설정
          - name: App_COLOR
            value: pink
  ```
- ConfigMap
  ```yaml
  env:
    - name: APP_COLOR
      valueFrom:
        configMapKeyRef:
  ```
- Secrets
  ```yaml
  env:
    - name: APP_COLOR
      valueFrom:
        secretKeyRef:
  ```

#### ConfigMaps

- k8s의 key value 쌍의 구성 데이터를 전달하는 데 사용
- configMap을 생성 후 pod에 주입
- configmap 생성방법

  - command

    ```shell
    # --from-literal : 명령 자체에서 key value 쌍을 지정하는 데 사용
    # kubectl create configmap <configName> --from-literal=<key>=<value>
    kubectl create configmap app-config --from-literal=APP_COLOR=blue

    # --from-file : 필요한 데이터를 포함한 파일의 경로 지정
    # kubectl create configmap <configName> --from-file=<path-to-file>
    kubectl create configmap app-config --from-file=app_config.properties
    ```

  - yaml

    ```yaml
    # config-map.yaml
    apiVersion: v1
    kind: ConfigMap
    metadata:
      name: app-config
    data:
      APP_COLOR: blue
      APP_MODE: prod
    ```

    ```shell
    # create
    kubectl create -f ./config-map.yaml

    # view
    kubectl get configmaps
    kubectl describe configmaps
    ```

- ConfigMap in Pods

  - config data를 pod에 넣는 법

    - env

      ```yaml
      envFrom:
        - configMapRef:
            name: app-config
      ```

    - single env

      ```yaml
      env:
        - name: APP_COLOR
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: APP_COLOR
      ```

    - volume

      ```yaml
      volumes:
        - name: app-config-volume
          configMap:
            name: app-config
      ```

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
        envFrom: # 생성한 configmap 적용
          - configMapRef:
              name: app-config
  ```

### Secrets

#### Configure Secrets in Applications

- 민감한 정보를 저장하는데 사용
- 인코딩된 형식으로 저장됨 점만 제외하면 ConfigMaps와 비슷
- ConfigMap방식과 똑같이 secret생성 후 pod에 주입함
- 명령적 접근법

  ```shell
  # kubectl create secret generic <secretName> --from-literal=<key>=<value>
  kubectl create secret generic app-secret --from-literal=DB_Host=mysql --from-literal=DB_User=root --from-literal=DB_Password=paswrd

  # kubectl create secret generic <secretName> --from-file=<path-to-file>
  kubectl create secret generic app-secret --from-file=app_secret.properties
  ```

- 선언적 접근법

  ```yaml
  # secret-data.yaml
  apiVersion: v1
  kind: Secret
  metadata:
    name: app-secret
  data:
    # echo -n 'mysql' | base64
    DB_Host: bXlzcWw= # mysql
    # echo -n 'root' | base64
    DB_User: cm9vdA== # root
    # echo -n 'paswrd' | base64
    DB_Password: cGFzd3Jk # paswrd
  ```

  `kubectl create -f secret-data.yaml`

- view

  ```shell
  kubectl get secrets
  kubectl describe secrets
  ```

- Secrets in Pods

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
        envFrom:
          - secretRef:
              name: app-secret
  ```
