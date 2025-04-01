# Part 1. 카프카 스트림즈 시작하기

# 카프카 스트림즈

👉🏻 **빅데이터로의 전환**

빅데이터로의 전환으로 대량의 데이터를 벌크 처리할 수 있는 능력만으로는 충분하지 않다.

`카프카 스트림즈`는 이벤트별 레코드 처리를 수행할 수 있게 하는 라이브러리다.

.

👉🏻 **스트림 처리를 사용해야 할 경우와 사용하지 말아야 할 경우**

데이터에 신속하게 응답하거나 보고해야 하는 경우가 스트림 처리의 좋은 사용 사례
- 신용카드 사기(고정 패턴-위치, 소비 습관)
- 침입 탐지(비정상적인 동작 모니터링)
- 뉴욕시 마라톤과 같은 대규모 경주(센서 데이터 사용)
- 금융 업계(시장 가격과 방향 추적)

데이터가 도착하자마자 즉시 보고하거나 조치를 취해야 하는 경우 스트림 처리가 좋은 방법이다.

.

👉🏻 **구매 트랜잭션에서의 관점**

![Result](https://github.com/jihunparkme/jihunparkme.gitbook.io/blob/main/.gitbook/assets/kafka-streams-in-action/example-stream.png?raw=true 'Result')

📖 **요약**

> 카프카 스트림즈는 강력하고 복잡한 스틀미 처리를 위해 처리 노드를 조합한 그래프
> 
> 배치 처리는 강력하지만 데이터 작업에 대한 실시간 요구를 만족시키기에는 충분하지 않음
>
> 데이터, 키/값 쌍, 파티셔닝 및 데이터 복제를 분산하는 것은 분산 애플리케이션에서 매우 중요

## 카프카 빠르게 살펴보기

카프카는
- 내고장성(fault-tolerant)을 가진 견고한 `발행/구독 시스템`
- 하나의 카프카 노드를 `bloker`라 부르고, 여러 개의 가프카 브로커 서버가 `cluster`를 구성
- 카프카는 `producer`가 작성한 메시지를 토픽에 저장
- `consumer`는 토픽을 구독하며, 구독한 토픽에 메시지가 있는지 확인하기 위해 카프카에 접속

```text
데이터 허브로 카프카를 사용하면 아키텍처가 훨씬 간단해진다.
각 애플리케이션은 카프카에 읽고 쓰는 방법만 알고 있으면 된다.
애플리케이션을 추가하거나 제거해도 다른 애플리케이션의 데이터 처리에 영향을 주지 않는다.
```

### 카프카 아키텍처

👉🏻 **카프카는 메시지 브로커다**

- `브로커`는 상호 간 유익한 교환이나 거래를 위해 각자 반드시 알 필요가 없는 두 부분을 묶는 중개자
- `프로듀서`가 카프카에 메시지를 보내면 메시지가 저장되어 토픽 구독을 통해 `컨슈머`가 사용
- `주키퍼 노드 클러스터`는 카프카와 통신해 토픽 정보를 유지하고 클러스터의 브로커를 추적
- 카프카 토픽의 내부 기술은 카프카가 들어오는 레코드를 기록한 파일인 `로그`
  - 토픽에 들어오는 메시지의 부하를 관리하기 위해 `파티션`을 사용

.

👉🏻 **카프카는 로그다**
- 카프카의 기본 메커니즘은 `로그`
- 카프카의 맥락에서 로그는 추가만 가능한 시간순으로 완전히 정렬된 `레코드 시퀀스`
- 로그는 들어오는 레코드가 추가되는 파일로 새로 도착한 각 레코드는 마지막 레코드 뒤에 위치하고, 이 과정은 파일의 레코드를 시간순으로 정렬
- 카프카의 `토픽`은 토픽 이름으로 분리된 로그
  - 라벨이 붙은 로그라고 할 수 있다.
  - 로그가 머신 클러스터 간에 복제된 후 하나의 머신이 중지되면 해당 서버를 복구하는 것은 쉽다.
  - 단지 로그 파일을 재생하면 되므로..
  - 실패로부터 복구하는 기능은 정확하게 분산 커밋 로그의 역햘

.

👉🏻 **카프카와 파티션**
- 파티션은 카프카 디자인에서 중요한 부분
  - 성능에 필수적이며, 같은 키를 가진 데이터가 동일한 컨슈머에게 순서대로 전송되도록 보장
  - 같은 키를 가진 메시지를 함께 처리하도록 보장
- 토픽을 파티션으로 분할하면 기본적으로 병렬 스트림에서 토픽에 전달되는 데이터가 분할
  - 이는 카프카가 엄청난 처리량을 달성하는 비결
- 특정 토픽의 용량이 넘치지 않도록 토픽의 메시지를 여러 머신에 분산

.

👉🏻 **키에 의한 그룹 데이터 분할**
- 키가 null이면 카프카 프로듀서는 라운드 로빈 방식으로 선택된 파티션에 레코드를 씀
  - 메시지 키는 메시지가 이동할 파티션을 결정하는데 사용
- 키의 바이트를 해싱하고 파티션 수를 모듈로 계산해 파티션을 얻음
  - HashCode.(key) % 파티션 수
    - hashCode(AByte) % 2 = 0
    - hashCode(BByte) % 2 = 1
- 동일한 키를 가진 레코드가 항상 동일한 파티션에 순서대로 전송

.

👉🏻 **사용자 정의 파티셔너 작성하기**
- 복합 키를 사용하는 간단한 경우
- 파티셔닝과 관련해 특정 고객에 대한 모든 트랜잭션이 동일한 파티션으로 이동해야 하지만, 키 전체(고객ID, 구매 날짜)를 사용하면 이 작업을 수행 불가
- 파티션을 결정할 때 고객 ID만 사용하도록 사용자 정의 파티셔너 작성

  ```java
  public class PurchaseKeyPartitioner extends DefaultPartitioner {


      @Override
      public int partition(String topic, Object key, byte[] keyBytes, Object value, byte[] valueBytes, Cluster cluster) {
          Object newKey = null;
          if (key != null) { // 키가 널이 아니면 사용자 ID를 추출
              PurchaseKey purchaseKey = (PurchaseKey) key;
              newKey = purchaseKey.getCustomerId();
              keyBytes = ((String) newKey).getBytes(); // 새로운 값으로 키의 바이트 설정
          }
          // 슈퍼 클래스에 위임하여 업데이트된 키로 파티션을 반환
          return super.partition(topic, newKey, keyBytes, value, valueBytes, cluster); 
      }
  }
  ```

.

👉🏻 **사용자 정의 파티셔너 지정하기**
- 카프카 프로듀서 설정 시 파티셔너 지정
- 프로듀서 인스턴스별로 파티셔너 설정 가능
  - 프로듀서마다 각기 다른 파티셔너를 자유롭게 사용 가능

  ```bash
  partitioner.class=aaronp_2.partitioner.PurchaseKeyPartitioner
  ```

.

👉🏻 **정확한 파티션 수 정하기**
- 토픽 생성 시 사용할 파티션 수를 선택하는 것은 기술의 영역이면서 과학의 영역
- 핵심 고려사항 중 하나는 주어진 토픽에 들어오는 데이터의 양
- 데이터가 ㅁ낳을수록 처리량을 높이기 위해 더 많은 파티션이 필요
- 그러나 트레이드 오프(trade-off)는 존재
- [How to Choose the Number of Topics/Partitions in a Kafka Cluster?](https://www.confluent.io/blog/how-choose-number-topics-partitions-kafka-cluster/)

.

👉🏻 **분산 로그**
- 토픽이 분할되면 카프카는 하나의 머신에 모든 파티션을 할당하지 않음
- 카프카는 클러스터를 구성하는 여러 머신에 파티션을 분산
- 카프카는 레코드를 로그에 추가하므로 카프카는 이런한 레코드를 파티션별로 여러 머신에 분산
- 하나의 브로커에 저장하면 클러스터의 하나 혹은 그 이상의 머신에 데이터를 복제

.

👉🏻 **주키퍼: 리더, 팔로워, 복제**
- 카프카는 리더와 팔로워 브로커라는 개념이 존재
- 카프카에서 각 토픽 파티션별로 한 브로커가 다른 브로커의 리더로 선택
- 리더의 주요 임무 중 하나는 팔로워 브로커에 토픽 파티션의 복제를 할당하는 것
- 카프카가 클러스터 전반에 걸쳐 토픽에 대한 파티션을 할당하는 것처럼 카프카는 파티션도 여러 머신에 복제

.

👉🏻 **아파치 주키퍼**
- 아파치 주키퍼는 카프카의 아키텍처에 필수적이며 주키퍼를 사용해 **카프카가 리더 브로커를 확보**하고 **토픽 복제를 추적**할 수 있게 한다.
  - [zookeeper](https://zookeeper.apache.org/)
- 카프카 클러스터에서 브로커 중 하나는 `컨트롤러`로 선출
- 토픽 파티션에는 리더와 팔로워가 존재
  - 메시지 생성 시 카프카는 레코드 파티션의 리더가 있는 브로커에게 레코드를 보낸다.

.

👉🏻 **복제**
- 카프카는 클러스터의 브로커가 실패할 경우 데이터 가용성 보장을 위해 브로커 간에 레코드를 복제
- [Replicated Logs: Quorums, ISRs, and State Machines (Oh my!)](https://kafka.apache.org/documentation/#design_replicatedlog)

.

👉🏻 **컨트롤러의 책임**
- 컨트롤러 브로커는 토픽의 모든 파티션에 대한 리더/팔로어 관계를 설정
- 카프카 노드가 죽거나 응답하지 않으면, 할당된 모든 파티션이 컨트롤러 브로커에 의해 재할당
- 주키퍼는 카프카 운영의 다른 측면에도 관여
  - **클러스터 멤버십**: 클러스터에 가입하고 클러스터 멤버십을 유지 관리
    - 브로커를 사용할 수 없게 되면 주키퍼는 클러스터 멤버십에서 브로커를 제거
  - **토픽 설정**: 클러스터의 토픽을 트래킹
    - 브로커가 토픽의 리더인지, 토픽에 파티션이 몇 개인지, 토픽의 특정 설정이 업데이트되었는지 확인
  - **접근 제어**: 특정 토픽에 대해 누가 읽고 쓸 수 있는지 식별

.

👉🏻 **로그 관리**
- 카프카에서 오래된 데이터를 제거할 때 두 가지 방식이 있는데 전통적인 접근법인 로그 삭제, 그리고 압축이 존재
- 로그 삭제
  - 로그를 세그먼트로 나누어 가장 오래된 세그먼트를 삭제
  - 로그 분할 시간은 메시지에 포함된 타임스탬프를 기반으로 수행
  - 로그 롤링: 카프카 브로커 설정 시 지정하는 구성 설정 ([Broker Configs](https://kafka.apache.org/documentation/#brokerconfigs))
    - `log.roll.ms`: 주 설정(default. x)
    - `log.roll.hours`: 보조 설정. 주 설정이 없는 경우만 사용(default. 168h)
  - 세그먼트 삭제 우선순위
    - `log.retention.ms`: 로그 파일을 밀리초 단위로 보관하는 기간
    - `log.retention.minutes`: 로그 파일을 분 단위로 보관하는 기간
    - `log.retention.hours`: 시간 단위의 로그 파일 보존 기간
- 로그 압축
  - 대략적인(coarse-grained) 접근 방식을 사용하고, 시간이나 크기를 기반으로 전체 세그먼트를 삭제하는 대신
  - 좀 더 세밀한(fine-grained) 접근 방식을 사용하고 로그에서 키별로 오래된 레코드를 삭제
    - 로그 압축이 각 키별로 가장 최근의 메시지를 유지
  - [Log Compaction](https://kafka.apache.org/documentation/#compaction)

### 프로듀서로 메시지 보내기

프로듀서가 데이터를 카프카 클러스터로 보낼 수 있다.
- 카프카 프로듀서는 스레드 안전
- 모든 전송은 비동기

[SimpleProducer](https://github.com/bbejeck/kafka-streams-in-action/blob/master/src/main/java/bbejeck/chapter_2/producer/SimpleProducer.java)

```java
public class SimpleProducer {
    public static void main(String[] args) {

        Properties properties = new Properties();
        properties.put("bootstrap.servers", "localhost:9092");
        properties.put("key.serializer", "org.apache.kafka.common.serialization.StringSerializer");
        properties.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");
        properties.put("acks", "1");
        properties.put("retries", "3");
        properties.put("compression.type", "snappy");
        properties.put("partitioner.class", PurchaseKeyPartitioner.class.getName());

        PurchaseKey key = new PurchaseKey("12334568", new Date());

        try(Producer<PurchaseKey, String> producer = new KafkaProducer<>(properties)) {
            ProducerRecord<PurchaseKey, String> record = new ProducerRecord<>("some-topic", key, "value");

            Callback callback = (metadata, exception) -> {
                if (exception != null) {
                    exception.printStackTrace();
                }
            };

            // 프로듀서가 내부 버퍼에 레코드를 저장하면 즉시 반환
            Future<RecordMetadata> sendFuture = producer.send(record, callback);
        }
    }
}
```

.

👉🏻 **프로듀서 속성**

- **부트스트랩 서버**: 
  - 처음 클러스터에 연결하는 용도로만 사용
- **직렬화**:
  - 내부적으로 카프카는 키/값에 바이트 배열을 사용하므로 카프카에 정확한 직렬화기를 제공해 전송하기 전 객체를 바이트 배열로 변환
- **acks**:
  - 레코드 전송이 완료되었다고 생각하기 전에 프로듀서가 브로커로부터 기다리는 확인 수
  - `0`: 프로듀서가 어떤 확인 응답도 기다리지 않음
  - `1`: 브로커는 레코드를 로그에 기록하지만 팔로워의 레코드 커밋에 대한 확인 응답은 기다리지 않음
  - `all`: 브로커는 모든 팔로워가 레코드를 커밋할 때까지 대기
- **재시도**:
  - 배치 결과가 실패하는 경우 retries는 재전송 시도 횟수를 지정
- **압축 타입**:
  - `compression.type`은 적용할 압축 알고리즘을 지정
  - 보내기 전 배치를 압축하도록 프로듀서에 지시(개별 레코드가 아닌 배치 단위로 압축)
- **파티셔너 클래스**:
  - Partitioner 인터페이스를 구현하는 클래스 이름 지정
- [Producer Configs](https://kafka.apache.org/documentation/#producerconfigs)

### 컨슈머로 메시지 읽기

👉🏻 **오프셋 관리**
- KafkaProducer는 본질적으로 상태가 없지만, KafkaConsumer는 주기적으로 카프카에서 소비되는 메시지의 `오프셋`을 커밋해 일부 상태를 관리
- 오프셋은 메시지를 고유하게 식별하고 로그에서 메시지의 시작 위치를 나타낸다.
- 컨슈머는 받은 메시지의 오프셋을 주기적으로 커밋해야 한다.
- 오프셋 커밋은 컨슈머에 있어서 두 가지 의미가 존재
  - 커밋한다는 것은 컨슈머가 메시지를 완전히 처리했음을 의미
  - 커밋은 실패나 재시작 시 해당 컨슈머의 시작 지점도 나타냄
- 새로운 컨슈머 인스턴스가 있거나 일부 오류가 발생했고, 마지막 커밋한 오프셋을 사용할 수 없는 경우 컨슈머가 시작하는 위치는 설정에 따라 다름
  - `auto.offset.reset="earliest"`
    - 사용 가능한 가장 이른 오프셋부터 시작. 로그 관리 프로세스에 의해 아직 제거되지 않은 메시지
  - `auto.offset.reset="latest"`
    - 가장 최신 오프셋에서 메시지를 읽어서 기본적으로 컨슈머가 클러스터에 합류한 지점부터 유입된 메시지만 소비
  - `auto.offset.reset="none"`
    - 재설정 전략을 지정하지 않음. 브로커가 컨슈머에게 예외를 발생

.

👉🏻 **자동 오프셋 커밋**
- 자동 오프셋 커밋 방식이 기본값
- `enable.auto.commit` 프로퍼티로 설정 가능
- 짝을 이루는 설정 옵션은 `auto.commit.interval.ms`
  - 컨슈머가 오프셋을 커밋하는 주기를 지정(default. 5s)

.

👉🏻 **수동 오프셋 커밋**
- 수동 커밋된 오프셋에는 동기식, 비동기식 두 가지 유형이 존재
- 동기식(`commitSync()`)
  - 마지막 검색에서 반환된 모든 오프셋이 성공할 때까지 블로킹
  - 호출은 구독한 모든 토픽과 파티션에 적용
  - 맵에 지정된 오프셋, 파티션, 토픽만 커밋하도록 파라미터를 넘길 수도 있음
- 비동기식(`commitAsync()`)
  - 즉시 반환
- 수동 커밋을 사용하면 레코드가 처리된 것으로 간주되는 시기를 직접 제어 가능

.

👉🏻 **컨슈머와 파티션**
- 여러 애플리케이션이나 머신에 컨슈머를 분산하는 경우 모든 인스턴스의 총 스레드 수는 해당 토픽의 총 파티션 수를 넘지 않아야 한다.
  - 전체 파티션 수를 초과하는 스레드는 유휴 상태가 되기 때문
- 컨슈머가 실패하면 리더 브로커는 파티션을 다른 활성 컨슈머에게 할당

.

👉🏻 **리밸런싱**
- 컨슈머에게 토픽-파티션 할당을 추가 및 제거하는 프로세스
- [ThreadedConsumerExample](https://github.com/bbejeck/kafka-streams-in-action/blob/master/src/main/java/bbejeck/chapter_2/consumer/ThreadedConsumerExample.java)

  ```java
  public class ThreadedConsumerExample {

      private volatile boolean doneConsuming = false;
      private int numberPartitions;
      private ExecutorService executorService;

      public ThreadedConsumerExample(int numberPartitions) {
          this.numberPartitions = numberPartitions;
      }


      public void startConsuming() {
          executorService = Executors.newFixedThreadPool(numberPartitions);
          Properties properties = getConsumerProps();

          for (int i = 0; i < numberPartitions; i++) {
              Runnable consumerThread = getConsumerThread(properties);
              executorService.submit(consumerThread);
          }
      }

      private Runnable getConsumerThread(Properties properties) {
          return () -> {
              Consumer<String, String> consumer = null;
              try {
                  consumer = new KafkaConsumer<>(properties);
                  consumer.subscribe(Collections.singletonList("test-topic"));
                  while (!doneConsuming) {
                      ConsumerRecords<String, String> records = consumer.poll(5000);
                      for (ConsumerRecord<String, String> record : records) {
                          String message = String.format("Consumed: key = %s value = %s with offset = %d partition = %d",
                                  record.key(), record.value(), record.offset(), record.partition());
                          System.out.println(message);
                      }

                  }
              } catch (Exception e) {
                  e.printStackTrace();
              } finally {
                  if (consumer != null) {
                      consumer.close();
                  }
              }
          };
      }

      public void stopConsuming() throws InterruptedException {
          doneConsuming = true;
          executorService.awaitTermination(10000, TimeUnit.MILLISECONDS);
          executorService.shutdownNow();
      }


      private Properties getConsumerProps() {
          Properties properties = new Properties();
          properties.put("bootstrap.servers", "localhost:9092");
          properties.put("group.id", "simple-consumer-example");
          properties.put("auto.offset.reset", "earliest");
          properties.put("enable.auto.commit", "true");
          properties.put("auto.commit.interval.ms", "3000");
          properties.put("key.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
          properties.put("value.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");

          return properties;

      }

      public static void main(String[] args) throws InterruptedException {
          ThreadedConsumerExample consumerExample = new ThreadedConsumerExample(2);
          consumerExample.startConsuming();
          Thread.sleep(60000); //Run for one minute
          consumerExample.stopConsuming();
      }

  }
  ```

## 요약

1️⃣ 카프카는 메시지를 수신해 컨슈머의 요청에 쉽고 빠르게 응답할 수 있는 방식으로 메시지를 저장하는 `메시지 브로커`
- 메시지는 컨슈머로 전송 후에도 사라지지 않으며, 카프카의 메시지 보존은 메시지를 소비하는 시기와 빈도에 전적으로 의존

2️⃣ 카프카는 높은 처리량을 얻기 위해 `파티션`을 사용하고 `같은 키`를 사용해 메시지를 순서대로 그룹화하는 방법을 제공

3️⃣ `프로듀서`는 카프카에 메시지를 보내는 데 사용

4️⃣ 널 키는 라운드 로빈 파티션 할당을 의미
- 그렇지 않으면 프로듀서는 파티션 할당을 위해 키의 해시와 파티션 수의 모듈러 값을 사용

5️⃣ 컨슈머는 카프카에서 온 메시지를 읽는 데 사용

6️⃣ 컨슈머 그룹의 일원인 컨슈머는 메시지를 고르게 분산하기 위해 토픽-파티션 할당을 받음