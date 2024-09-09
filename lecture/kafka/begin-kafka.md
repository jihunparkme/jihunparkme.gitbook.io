# Begin Kafka

[아파치 카프카 for beginners](https://www.inflearn.com/course/아파치-카프카-입문/dashboard) 강의를 요약한 내용입니다. 

<https://kafka.apache.org/>

# Kafka?

**Before Kafka**

> 소스 애플리케이션, 타겟 애플리케이션 개수가 늘어날 수록 데이터 라인이 많아지게 된다.
- 이 때 배포와 장애 대응이 어려워지고,
- 데이터 전송 시 프로토콜 포맷 파편화가 심해짐
- 추후 데이터 포맷에 변경사항이 있을 경우 유지보수에 어려워진다.

<figure><img src="../../.gitbook/assets/kafka/before-kafka.png" alt=""><figcaption></figcaption></figure>


After Kafka

> 소스/타겟 애플리케이션의 커플링을 약하게 하기 위해 출시
> 
- 위와 같은 복잡함을 해결하기 위해 링크드인에서 내부적으로 개발하여 오픈소스로 제공

<figure><img src="../../.gitbook/assets/kafka/kafka.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/kafka/kafka-cluster.png" alt=""><figcaption></figcaption></figure>

https://developers.redhat.com/learning/learn:apache-kafka:kafka-101/resource/resources:what-are-partitions