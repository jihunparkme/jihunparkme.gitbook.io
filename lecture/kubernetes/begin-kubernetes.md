# Begin Kubernetes

[대세는 쿠버네티스](https://inf.run/yW34) 강의를 요약한 내용입니다. 

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