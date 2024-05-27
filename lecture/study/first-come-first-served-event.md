# First Come First Served Event

[실습으로 배우는 선착순 이벤트 시스템](https://www.inflearn.com/course/%EC%84%A0%EC%B0%A9%EC%88%9C-%EC%9D%B4%EB%B2%A4%ED%8A%B8-%EC%8B%9C%EC%8A%A4%ED%85%9C-%EC%8B%A4%EC%8A%B5/dashboard) 강의를 듣고 요약한 내용입니다.

# Intro

**요구사항.**
- 선착순 100명에게 할인쿠폰을 제공하는 이벤트
- 선착순 100명에게만 지급되어야한다.
- 101개 이상이 지급되면 안된다.
- 순간적으로 몰리는 트래픽을 버틸 수 있어야한다.

**선착순 이벤트 진행 시 발생할 수 있는 문제점.**
- 쿠폰이 개수보다 더 많이 발급되었을 경우
- 이벤트 페이지 접속이 안 될 경우
- 이벤트랑 상관없는 페이지도 느려질 경우

**문제 해결.**
- 트래픽이 몰렸을 때 대처할 방법
- Redis 를 활용하여 쿠폰 발급 개수 보장하는 방법
- Kafka 를 활용하여 다른 페이지들에 대한 영향도를 줄이는 방법

# Race Condition

```java
@Service
@RequiredArgsConstructor
public class ApplyService {

    private final CouponRepository couponRepository;

    public void apply(Long userId) {
        final long count = couponRepository.count();

        if (count > 100) {
            return;
        }

        couponRepository.save(new Coupon(userId));
    }
}
```

아래 테스트에서 100의 결과를 예상했지만, `동시에 들어오는 요청들이 갱신 전 값을 읽고 데이터를 추가`하면서 예상했던 개수를 초과하는 현상이 발생하게 됩니다.

```java
@Test
void apply_multiple() throws InterruptedException {
    int threadCount = 1000;
    ExecutorService executorService = Executors.newFixedThreadPool(32);
    CountDownLatch latch = new CountDownLatch(threadCount);

    for (int i = 0; i < threadCount; i++) {
        long userId = i;
        executorService.submit(() -> {
            try {
                applyService.apply(userId);
            } finally {
                latch.countDown();
            }
        });
    }

    latch.await();

    final long count = couponRepository.count();

    assertThat(count).isEqualTo(100);
}
```

[Concurrency issues](https://jihunparkme.gitbook.io/docs/lecture/study/concurrency-issues) 에서 배운 것과 같이 Java Synchronized 를 적용해볼 수 있지만,

서버가 여러 대가 된다면 Race Condition 이 다시 발생하게 되므로 적절하지 않습니다.

.

또 다른 방법으로 MySQL, Redis 를 활용한 락을 구현해서 해결할 수도 있을 것 같지만, 

쿠폰 개수에 대한 정합성을 원하는데 락을 활용하여 구현하면, 발급된 쿠폰의 개수를 가져오는 것부터 쿠폰을 생성할 때까지 락을 걸어야 합니다.

이렇게 되면 락을 거는 구간이 길어지다보니 성능에 불이익(락이 풀릴 때까지 쿠폰 발급을 기다려야 하는)이 발생할 수 있습니다.

## Redis

Redis `incr` 명령어는 키에 대한 값을 1씩 증가시키는 명령어입니다.

Redis 는 `싱글스레드 기반`으로 동작하여 `레이스 컨디션을 해결`할 수 있을 뿐 아니라 incr 명령어는 `성능도 굉장히 빠른` 명령어입니다.

`incr` 명령어를 사용하여 발급된 쿠폰 개수를 제어한다면 성능도 빠르며 데이터 정합성도 지킬 수 있습니다.

```bash
> incr coupon_count
(integer) 1

> incr coupon_count
(integer) 2
```

[commit](https://github.com/jihunparkme/Study-project-spring-java/commit/300944b0f04b3cb9919fa66f0b917a74edc36816)

### Problems

**1) 쿠폰의 개수**

- 발급하는 쿠폰의 개수가 많아질수록 **데이터베이스에 부하**를 주게 됩니다.
- 해당 데이터베이스가 다른 곳에서도 사용되고 있다면 서비스 장애까지 발생할 수 있습니다.

**2) 짧은 시간에 많은 요청**

- 짧은 시간 내에 많은 요청이 들어오게 될 경우 DB 서버의 **리소스**를 많이 사용하게 되므로 **부하**가 발생하게 됩니다.
- 서비스 지연 혹은 오류가 발생할 수 있습니다.

## Kafka

**분산 이벤트 스트리밍 플랫폼**
- `이벤트 스트리밍`: 소스에서 목적지까지 이벤트를 실시간으로 스트리밍 하는 것

```bash
Producer ---> Topic <--- Consumer
```

### Start Kafka

**docker-compose.yml**

```bash
version: '3' # Docker Compose 파일 버전 지정
services: # 여러개의 Docker 컨테이너 서비스 정의
  zookeeper: # Zookeeper 서비스 정의
    image: wurstmeister/zookeeper:3.4.6
    container_name: zookeeper
    ports:
      - "2181:2181" # 호스트의 2181 포트를 컨테이너의 2181 포트와 바인딩
  kafka: # kafka 서비스 정의
    image: wurstmeister/kafka:2.12-2.5.0
    container_name: kafka
    ports:
      - "9092:9092" # 호스트의 9092 포트를 컨테이너의 9092 포트와 바인딩
    environment: # kafka 컨테이너의 환경 변수 설정
      KAFKA_ADVERTISED_LISTENERS: INSIDE://kafka:29092,OUTSIDE://localhost:9092 # 내/외부에서 접근할 수 있는 리스너 주소 설정
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: INSIDE:PLAINTEXT,OUTSIDE:PLAINTEXT # 리스너의 보안 프로토콜 매핑
      KAFKA_LISTENERS: INSIDE://0.0.0.0:29092,OUTSIDE://0.0.0.0:9092 # 컨테이너 내부에서 사용할 리스너 주소 설정
      KAFKA_INTER_BROKER_LISTENER_NAME: INSIDE # 브로커 간 통신에 사용할 리스너 이름
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181 # Kafka가 Zookeeper에 연결하기 위한 주소
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock # Docker 소켓을 컨테이너와 공유하여 Docker 이벤트를 관리할 수 있도록 설정

```

**example**

```bash
# 카프카 실행
$ docker-compose up -d

# 토픽생성
$ docker exec -it kafka kafka-topics.sh --bootstrap-server localhost:9092 --create --topic testTopic
Created topic testTopic.

# 프로듀서 실행
$ docker exec -it kafka kafka-console-producer.sh --topic testTopic --broker-list 0.0.0.0:9092
>Hello

# 컨슈머 실행
$ docker exec -it kafka kafka-console-consumer.sh --topic testTopic --bootstrap-server localhost:9092
Hello

# 카프카 종료
$ docker-compose down
```

### Producer

**api/KafkaProducerConfig.java**

```java
@Configuration
public class KafkaProducerConfig {

    @Bean
    public ProducerFactory<String, Long> producerFactory() {
        final Map<String, Object> config = new HashMap<>();

        config.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
        config.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
        config.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, LongSerializer.class);

        return new DefaultKafkaProducerFactory<>(config);
    }

    @Bean
    public KafkaTemplate<String, Long> kafkaTemplate() {
        return new KafkaTemplate<>(producerFactory());
    }
}
```

**api/CouponCreateProducer.java**

```java
@Component
@RequiredArgsConstructor
public class CouponCreateProducer {

    private final KafkaTemplate<String, Long> kafkaTemplate;

    public void create(Long userId) {
        kafkaTemplate.send("coupon_create", userId);
    }
}
```

[KafkaProducerConfig](https://github.com/jihunparkme/Study-project-spring-java/commit/565a8f6c64847ece55c26edf20f799c390f1247c)
[CouponCreateProducer](https://github.com/jihunparkme/Study-project-spring-java/commit/49773a3dc20e49869018a608ffc26addcb9141e4)

### Consumer

**consumer/KafkaConsumerConfig.java**

```java
@Configuration
public class KafkaConsumerConfig {

    @Bean
    public ConsumerFactory<String, Long> consumerFactory() {
        final Map<String, Object> config = new HashMap<>();

        config.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
        config.put(ConsumerConfig.GROUP_ID_CONFIG, "group_1");
        config.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
        config.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, LongDeserializer.class);

        return new DefaultKafkaConsumerFactory<>(config);
    }

    /**
     * 토픽으로부터 메시지를 전달받기 위한 kafka-listener 를 만드는 kafka-listener-container-factory 생성
      */
    @Bean
    public ConcurrentKafkaListenerContainerFactory<String, Long> kafkaListenerContainerFactory() {
        final ConcurrentKafkaListenerContainerFactory<String, Long> factory = new ConcurrentKafkaListenerContainerFactory<>();
        factory.setConsumerFactory(consumerFactory());

        return factory;
    }
}
```

**consumer/CouponCreatedConsumer.java**

```java
@Component
@RequiredArgsConstructor
public class CouponCreatedConsumer {

    private final CouponRepository couponRepository;

    @KafkaListener(topics = "coupon_create", groupId = "group_1")
    public void listener(Long userId) {
        couponRepository.save(new Coupon(userId));
    }
}
```

[using Consumer](https://github.com/jihunparkme/Study-project-spring-java/commit/e1e308a15284cf91564119c2d64f9ee21bde355a)