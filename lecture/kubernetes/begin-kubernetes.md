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