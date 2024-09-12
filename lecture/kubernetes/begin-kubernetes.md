# Begin Kubernetes

[대세는 쿠버네티스](https://inf.run/yW34) 강의를 요약한 내용입니다.
- [K8S 실습 자료실](https://kubetm.github.io/k8s/)
- [Install v1.27 (mac m series)](https://kubetm.github.io/k8s/02-beginner/cluster-install-case7/)
- [ Kubernetes](https://kubernetes.io/)

<figure><img src="../../.gitbook/assets/kubernetes/kubernetes-introduction.png" alt=""><figcaption></figcaption></figure>

# Why Kubernetes?

<figure><img src="../../.gitbook/assets/kubernetes/why-kubernetes.png" alt=""><figcaption></figcaption></figure>

**AS-IS**
- 서버 개수
  - 운영중인 각 서버가 시간에 따라 접속량이 달라질 경우에도 서비스마다 최대치의 서버 개수가 필요
- 장애 대비
  - 서버 장애 상황을 대비하여 한 대의 백업 서버도 추가로 필요
- 서비스 버전 업데이트
  - 서비스 중단이 허용되는 경우 모든 서버를 내리고, 업데이트 후 실행
  - 무중단 서비스가 필요할 경우 한 서버씩 내렸다가 업데이트 후 실행

**TO-BE**
- 서버 개수
  - 하루 각 서비스의 평균 트래픽을 계산하여 적절한 개수의 자원을 할당
- 장애 대비
  - `Auto scaling` 기능을 통해 트래픽 양에 따라 서비스 자원량을 변경
  - `Auto Healing` 기능을 통해 장애가 난 서버 위에 있는 서비스들을 다른 서버로 자동으로 이동
    - 한 서버에 장애가 나더라도 여분의 서버 한 대로 서비스 유지
- 서비스 버전 업데이트
  - Deployment 오브젝트를 통해 업데이트 방식에 대해 자동적으로 처리되도록 지원
- 그밖에도 여러 운영 자동화를 지원

> Kubernetes를 사용하면 운영 환경이 더욱 편리해지고, 서비스 효율이 증가
>
> - 서비스 효율로 서버가 적어지면 그만큼 유지보수 비용이 감소

---

# VM vs Container

**🔎 시스템 구조의 차이**

<figure><img src="../../.gitbook/assets/kubernetes/vm-container-1.png" alt=""><figcaption></figcaption></figure>

**VM**
> 각각의 OS를 띄워야 하는 구조
- Host OS 위에 VM 가상화를 위해 여러 하이퍼바이저가 존재

**Container**
> 한 OS 를 공유하는 구조
- Host OS 위에 컨테이너 가상화를 시켜주는 여러 소프트웨어가 존재 (보통 `Docker`를 가장 많이 사용)
  - 도커를 통해 컨테이너 이미지를 만들게 되는데, 이미지에는 *한 서비스와 그 서비스에 필요한 라이브러리*가 존재
  - 여러 컨테이너들 간에 호스트 자원을 분리해서 사용하도록 지원 (리눅스 Namespace, Cgroup을 사용한 격리)
    - Namespace: 커널에 관련된 영역을 분리
    - Cgroup: 자원에 대한 영역을 분리
- 컨테이너 가상화 솔루션은 OS에서 제공하는 자원 격리 기술을 이용해 컨테이너 단위로 서비스를 분리할 수 있도록 지원
- 개발 환경에 대한 걱정 없이 배포가 가능

**🔎 시스템 개발 사상적인 차이**

<figure><img src="../../.gitbook/assets/kubernetes/vm-container-2.png" alt=""><figcaption></figcaption></figure>

**VM**
- 일반적으로 한 가지의 언어로 개발된 여러 모듈들로 하나의 서비스가 동작
- 특정 모듈에 부하가 몰릴 경우 VM을 하나 너 생성
  - C 모듈만 확장을 하고 싶더라도 A, B 모듈이 같이 확장

**Container**
- 한 서비스를 만들 때 모듈별로 분리해서 컨테이너에 담는 것을 권장
  - 그 모듈에 맞는 최적화된 개발 언어를 사용
- 쿠버네티스는 여러 컨테이너들을 한 파드에 묶거나, 한 컨테이너만 파드에 담을 수 있음
  - 한 파드가 하나의 배포 단위이므로 필요한 파드만 확장 가능
- 시스템을 모듈별로 분리해서 개발했을 때 큰 효과를 발휘

<details>
<summary> Getting-Started</summary>

<figure><img src="../../.gitbook/assets/kubernetes/getting-started-kubernetes.png" alt=""><figcaption></figcaption></figure>

## Linux

**Install nodejs in CentOS**

```sh
yum install epel-release
yum -y install nodejs
```

**hello.js**

```js
var http = require('http');
var content = function(req, resp) {
 resp.end("Hello Kubernetes!" + "\n");
 resp.writeHead(200);
}
var w = http.createServer(content);
w.listen(8000);
```

**run hello.js**

```sh
node hello.js
```

## Docker

[docker hub](https://hub.docker.com/)

Dockerfile
```sh
FROM node:slim # docker hub 에서 가져올 이미지:버전
EXPOSE 8000 # 노출시킬 포트
COPY hello.js . # 현재 디렉토리의 hello.js 파일 복사
CMD node hello.js # 컨테이너 구동 시 실행시킬 명령어
```

**Run Docker Container**

```sh
# 이미지 빌드
# -t : 레파지토리/이미지명:버전 디렉토리
# . : Dockerfile 위치 (다른 이름으로 지정 시 파일 이름)
docker build -t kubetm/hello .

# 생성한 이미지 확인
docker images

# 컨테이너 구동
# -d : 백그라운드 모드
# -p : 포트변경 (노출:구동)
docker run -d -p 8100:8000 kubetm/hello

# 컨테이너 접속
docker ps
docker exec -it c403442e8a59 /bin/bash
```

**Push Docker Image**

```sh
docker login
docker push kubetm/hello
```

## Kubernetes

**Generate Pod**

```sh
apiVersion: v1
kind: Pod
metadata:
  name: hello-pod # 파드 종류
  labels:
    app: hello # 파드 이름
spec:
  containers:
  - name: hello-container # 컨테이너 이름
    image: kubetm/hello # docker hub에서 가져올 이미지
    ports:
    - containerPort: 8000 # 노출 포트
```

**Generate Service**

```sh
apiVersion: v1
kind: Service
metadata:
  name: hello-svc
spec:
  selector:
    app: hello # pod metadata.name.labels.app과 매칭하여 pod에 연결
  ports:
    - port: 8200 # 노출 포트
      targetPort: 8000 # 컨테이너 포트
  externalIPs:
  - 192.168.56.30 # 접속 IP
```
</details>

---

# Overview Kubernetes

<figure><img src="../../.gitbook/assets/kubernetes/overview-kubernetes.png" alt=""><figcaption></figcaption></figure>

## Object

쿠버네티스에서 서버 한 대는 `Master`로 사용하고, 다른 서버(`Node`)는 마스터에 연결
- 이것이 하나의 쿠버네티스 클러스터라는 개념에 묶임
- `Master`: 쿠버네티스의 전반적인 기능들을 컨트롤하는 역할
- `Node`: 자원을 제공하는 역할. 클러스터의 전체 자원을 늘리고 싶다면 노드를 계속 추가

`Namespace`
- 쿠버네티스 오브젝트들을 독립된 공간으로 분리
- 쿠버네티스 최소 배포 단위인 `Pod`가 존재
  - `Pod` 안에는 여러개의 컨테이너가 존재
  - `Pod`에서는 여러 앱(`Container`)이 동작
  - `Pod`에 문제가 생겨서 재성성되면 그 안에 데이터는 날라가므로, `Volume`을 `Pod`에 연결하여 데이터를 관리
- `Pod`들에게 외부로부터 연결이 가능하도록 `IP`를 할당해주는 `Service`가 존재
  - 서로 다른 `Namespace`에 있는 파드에는 연결 불가
- `ResourceQuota`, `LimitRange`를 달아서 한 `Namespace`에서 사용할 수 있는 자원의 양 한정
  - `Pod` 개수 제한, CPU/Memory 제한 등..
- `ConfigMap`, `Secret`을 통해 `Pod` 생성 시 `Container` 안에 환경 변수 값이나 파일을 만운팅

## Controller

> `Pod`들을 관리
>
> - 사용 용도에 따라 다양한 컨트롤러 제공

**Replication Controller, ReplicaSet**
- 가장 기본적인 컨트롤러
- `Pod`가 죽으면, 감지해서 다시 살려줌
- `Pod`의 개수를 늘리거나 줄임 (Scale In/Out)

**Deployment**
- 배포 후에 `Pod`들을 새 버전으로 업그레이드
- 업그레이드 중 문제 발생 시 쉬운 롤백 제공

**DemonSet**
- 한 노드에 `Pod`가 하나씩만 유지되도록 지원

**CronJob**
- 특정 작업을 주기적으로 수행하고 종료되도록 `Pod`에 `Job` 적용

---

# Pod

> Container
> 
> Label
> 
> NodeSchedule

<figure><img src="../../.gitbook/assets/kubernetes/object-pod.png" alt=""><figcaption></figcaption></figure>

## Container

<figure><img src="../../.gitbook/assets/kubernetes/container.png" alt=""><figcaption></figcaption></figure>

`Pod`
- Pod 안에는 하나의 독립적인 서비스를 구동할 수 있는 컨터이너들이 존재
  - 컨테이너들은 서비스가 연결될 수 있도록 포트를 보유
  - 한 컨테이너가 하나 이상의 포트를 가질 수 있지만, 한 Pod 내에서 컨테이너들끼리 중복 불가
  - Pod 안의 컨테이너는 한 호스트로 묶여있는 개념
- Pod 생성 시 고유 IP 주소 할당
  - 쿠버네티스 클러스터 내에서만 해당 IP를 통해 해당 Pod에 접근 가능
  - 외부에서는 해당 IP로 접근 불가
  - 만일 Pod에 문제가 생기면 시스템이 감지하여 Pod를 삭제시키고 다시 재생성하는데, 이때 IP 주소는 변경
- Pod 생성 스크립트
  
  ```sh
  apiVersion: v1
  kind: Pod # Pod 생성
  metadata:
   name: pod-1 # Pod 이름
  spec:
  containers: # 두 개의 컨테이너 생성
   - name: container1 # 컨테이너 이름
     image: tmkube/p8000 # 이미지 이름
     ports:
     - containerPort: 8000 # 컨테이너 포트
   - name: container2
     image: tmkube/p8080
     ports:
     - containerPort: 8080
  ```

- ReplicationController 생성 스크립트

  ```sh
  apiVersion: v1
  kind: ReplicationController
  metadata:
    name: replication-1
  spec:
    replicas: 1
    selector:
      app: rc
    template:
      metadata:
        name: pod-1
        labels:
          app: rc
      spec:
        containers:
        - name: container
          image: kubetm/init
  ```

## Label

<figure><img src="../../.gitbook/assets/kubernetes/label.png" alt=""><figcaption></figcaption></figure>

Label은 Pod 뿐 아니라 모든 오브젝트에 달 수 있는데, Pod에서 가장 많이 사용
- 목적에 따라 오브젝트들을 분류하고, 분류된 오브젝트만 따로 골라서 연결
- key/value 가 한 쌍으로 이루어짐
- 한 Pod에는 여러개의 Label을 달 수 있음
- ex) web, db, server 타입이 한 쌍으로 개발/상용 환경으로 나뉘어 구성

Service Example
- 특정 타입의 Pod만 사용할 경우, type:web인 Label이 달린 Pod들을 서비스에 연결하여 해당 서비스 정보를 전달
- 상용 환경을 담당하는 운영자라면, lo:product인 Label이 달린 Pod들을 서비스에 연결하여 해당 서비스 정보를 전달

Label 활용 스크립트
- Pod 생성 시 Label 등록

```sh
apiVersion: v1
kind: Pod
metadata:
  name: pod-2
  labels:
    type: web # key
    lo: dev # value
spec:
  containers:
  - name: container
    image: tmkube/init
```

- Service 생성 시 특정 라벨이 붙어있는 Pod 연결

```sh
apiVersion: v1
kind: Service
metadata:
  name: svc-1
spec:
  selector:
    type: web # key:value
  ports:
  - port: 8080
```

## Node Schedule

<figure><img src="../../.gitbook/assets/kubernetes/node-schedule.png" alt=""><figcaption></figcaption></figure>

Pod는 결국 여러 노드들 중에 한 노드에 올라가야 한다.
- 이 방법이 대해 직접 노드를 선택하는 방식과 쿠버네티스가 자동으로 지정해주는 방식이 존재

1️⃣ 직접 선택하는 방식
- 노드에 Label 등록 후, Pod 생성 시 노드를 지정

```sh
apiVersion: v1
kind: Pod
metadata:
  name: pod-3
spec:
  nodeSelector:
    kubernetes.io/hostname: k8s-node1 # 노드 라벨과 매칭되는 key: value
  containers:
  - name: container
    image: kubetm/init
```

2️⃣ 쿠버네티스의 스케줄러가 판단해서 지정하는 방식
- ex) Pod 생성 시 Pod에서 요구될 리소스의 사용량을 명시하여 메모리 사용량에 맞게 Pod 스케줄링
  
```sh
apiVersion: v1
kind: Pod
metadata:
  name: pod-4
spec:
  containers:
  - name: container
    image: kubetm/init
    resources:
      requests:
        memory: 2Gi # 2G 메모리 요구
      limits: # 자원의 성격에 따라 다르게 판단
        # memory: 초과 시 Pod 종료.
        # cpu: 초과 시 request 수치로 낮추고, 종료되진 않음
        memory: 3Gi # 최대 허용 메모리
```

...

Controller + Label + Node Schedule 스크립트

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-4                           # Pod 이름
  labels:                               # Label 
    type: web                           
    lo: dev  
spec:
  nodeSelector:                         # Node 직접 지정
    kubernetes.io/hostname: k8s-node1   
  containers:
  - name: container                     # 컨테이너 이름
    image: kubetm/init                  # 이미지 선택
    ports:
    - containerPort: 8080               
    resources:                          # 자원 사용량 설정
      requests:
        memory: 1Gi
      limits:
        memory: 1Gi
```

<details>
<summary> 📖 참고. kubectl</summary>

**Create**
- 같은 이름의 Pod가 존재할 경우 생성이 안됨

```sh
# 파일이 있을 경우
kubectl create -f ./pod.yaml

# 내용과 함께 바로 작성할 경우
kubectl create -f - <<END
apiVersion: v1
kind: Pod
metadata:
  name: pod1
spec:
  containers:
  - name: container
    image: kubetm/init
END
```

**Apply**
- 기존에 같은 이름의 Pod가 존재하면 업데이트

```sh
kubectl apply -f ./pod.yaml
```

**Get**

```sh
# 기본 Pod 리스트 조회 (Namepsace 포함)
kubectl get pods -n defalut

# 더 많은 내용 출력
kubectl get pods -o wide

# Pod 이름 지정
kubectl get pod pod1

# Json 형태로 출력
kubectl get pod pod1 -o json
```

**Describe**

```sh
# 상세 출력
kubectl describe pod pod1
```

**Delete**

```sh
# 파일이 있을 경우 생성한 방법 그대로 삭제
kubectl delete -f ./pod.yaml

# 내용과 함께 바로 작성한 경우 생성한 방법 그대로 삭제
kubectl delete -f - <<END
apiVersion: v1
kind: Pod
metadata:
  name: pod1
spec:
  containers:
  - name: container
    image: kubetm/init
END

# Pod 이름 지정
kubectl delete pod pod1
```

**Exec**

```sh
# Pod 이름이 pod1인 Container로 들어가기 (나올땐 exit)
kubectl exec pod1 -it /bin/bash

# (Pod에 Container가 두 개 이상 있을 경우) pod1의 con1 Container로 들어가기 
kubectl exec pod1 -c con1 -it /bin/bash
```
</details>