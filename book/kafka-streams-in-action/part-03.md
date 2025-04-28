# Part 3. 카프카 스트림즈 관리

**Table of contents**
- 기본적인 카프카 모니터링
  - 인터셉터
- 메트릭
- 카프카 스트림즈 디버깅 기술

# 7장. 모니터링과 성능

## 기본적인 카프카 모니터링

카프카 스트림즈 API는 카프카의 일부이므로 애플리케이션을 모니터링할 때 카프카도 일부 모니터링해야 한다.

[Monitoring](https://kafka.apache.org/documentation/#monitoring)

.

👉🏻 **컨슈머와 프로듀서 성능 측정**
- 프로듀서와 컨슈머의 성능은 처리량과 관련이 있다.
- `프로듀서`의 경우 프로듀서가 얼마나 빠르게 브로커로 메시지를 보내느냐가 큰 관심사
  - 분명 처리량이 높으면 높을수록 더 낮다
- `컨슈머`의 경우 브로커에서 얼마나 빠르게 메시지를 읽을 수 있느냐가 성능에 영향
  - 컨슈머 성능을 측정하는 또 다른 방법은 컨슈머 지연
- 프로듀서가 브로커에 기록하는 속도와 컨슈머가 메시지를 읽는 속도 차이를 `컨슈머 지연`이라고 부른다

.

👉🏻 **컨슈머 지연 확인하기**
- 컨슈머 지연을 확인하기 위해 카프카는 편리한 명령줄 도구를 제공
  - kafka-consumer-groups.sh
- 활성화 상태의 모든 컨슈머 그룹을 찾기 위해 `list` 명령을 사용
  
```bash
kafka-installed-path/bin/kafka-consumer-groups.sh
                --bootstrap-server localhost:9092
                --list
```

- 조회할 컨슈머 그룹 이름을 선택하고 다음 명령을 실행

```bash
kafka-installed-path/bin/kafka-consumer-groups.sh
            --bootstrap-server localhost:9092
            --group GROUP-NAME
            --describe
```

- 작은 지연이나 일정한 지연은 문제가 안 되지만, 시간이 지남에 따라 계속 증가하는 지연은 컨슈머에게 더 많은 리소스를 제공해야 한다

.

👉🏻 **프로듀서와 컨슈머 가로채기**

- `인터셉터`는 디버깅을 위한 일반적인 제일선 도구는 아니지만 카프카 스트리밍 애플리케이션의 동작을 관찰하는 데 유용할 수 있으며, 자신만의 도구 세트에 추가할 만한 유용한 도구이다.
- `인터셉터`를 사용하는 좋은 예제는 카프카 스트림즈 애플리케이션이 카프카 토픽으로 다시 생산하는 메시지 오프셋을 추적하는 데 사용

**컨슈머 인터셉터**
- 컨슈머 인터셉터는 가로채기를 위해 두 가지 접근점을 제공

1️⃣ `ConsumerInterceptor.onConsume()`
- 브로커에서 조회한 시점과 `Consumer.poll()` 메소드가 메시지를 반환하기 전 `ConsumerRecords`에서 읽는다.
- `ConsumerConfig.INTERCEPTOR_CLASSES_CONFIG`를 통해 하나 이상의 `ConsumerInterceptor` 구현자 클래스의 컬렉션으로 지정

```java
ConsumerRecords<String, String> poll(long timeout) {
  ConsumerRecords<String, String> consumerRecords = 
    ...consuming records
  // 인터셉터 체인을 통해 레코드를 실행하고 결과를 반헌
  return interceptors.onConsume(consumerRecords);
}
```
  
2️⃣ `ConsumerInterceptor.onCommit()`
- 컨슈머가 브로커에게 오프셋을 커밋하면 브로커는 토픽, 파티션 및 커밋된 오프셋과 관련된 메타데이터와 함께 정보가 포함된 `Map<TopicPartition, OffsetAndMetadata>`를 반환

**로깅 목적으로 사용되는 간단한 ConsumerInterceptor 예시**

```java
// StockTransactionConsumerInterceptor.java

public class StockTransactionConsumerInterceptor implements ConsumerInterceptor<Object, Object> {
    //...

    /**
     * 레코드가 처리되기 전에 컨슈머 레코드와 메타데이터를 로깅
     */
    @Override
    public ConsumerRecords<Object, Object> onConsume(ConsumerRecords<Object, Object> consumerRecords) {
        LOG.info("Intercepted ConsumerRecords {}", buildMessage(consumerRecords.iterator()));
        return consumerRecords;
    }

    /**
     * 카프카 스트림즈 컨슈머가 브로커에 오프셋을 커밋할 때 커밋 정보를 로깅
     */
    @Override
    public void onCommit(Map<TopicPartition, OffsetAndMetadata> map) {
        LOG.info("Commit information {}", map);
    }
    //...
}
```

**프로듀서 인터셉터**
- `ProducerInterceptor.onSend()` 및 `ProducerInterceptor.onAcknowledgement()` 두 가지 접근 지점이 존재
- 체인상에 있는 각 프로듀서 인터셉터는 이전 인터셉터에서 반환된 객체를 받는다.

**간단히 로깅하는 ProducerInterceptor 예제**

```java
// ZMartProducerInterceptor.java

public class ZMartProducerInterceptor implements ProducerInterceptor<Object, Object> {
    /**
     * 메시지를 브로커에 전송하기 바로 전에 로깅
     */
    @Override
    public ProducerRecord<Object, Object> onSend(ProducerRecord<Object, Object> record) {
        LOG.info("ProducerRecord being sent out {} ", record);
        return record;
    }

    /**
     * 브로커 수신 확인 또는 생산 단계 동안 브로커 측에서 오류가 발생했는지 로깅
     */
    @Override
    public void onAcknowledgement(RecordMetadata metadata, Exception exception) {
        if (exception != null) {
            LOG.warn("Exception encountered producing record {}", exception);
        } else {
            LOG.info("record has been acknowledged {} ", metadata);
        }
    }
}
```

- ProducerInterceptor는 ProducerConfig.INTERCEPTOR_CLASSES_CONFIG에 지정하고, 하나 이상의 ProducerInterceptor 클래스의 컬렉션으로 설정

✅ 참고

> 카프카 스트림즈 애플리케이션에서 인터셉터를 구성할 때 컨슈머와 프로듀서 인터셉터의 속성 이름의 프리픽스를
>
> `StreamsConfig.consumerPrefix(ConsumerConfig.INTERCEPTOR_CLASSES_CONFIG)` 와
>
> `StreamsConfig.producerPrefix(ProducerConfig.INTERCEPTOR_CLASSES_CONFIG)` 로 지정해야 한다.

부수적으로 인터셉터는 카프카 스트림즈 애플리케이션의 모든 레코드에서 작동하므로 로깅 인터셉터의 출력이 중요하다.
- 인터셉터 결과는 소스 코드를 설치한 로그 디렉토리에 있는 consumer_interceptor.log 및 producer_interceptor.log 로 출력

## 애플리케이션 메트릭

메트릭 카테고리 살펴보기
- 스레드 메트릭
  - 평균 커밋, 폴링, 처리 작업 시간
  - 초당 생성한 태스크 수, 초당 종료된 태스크 수
- 태스크 메트릭
  - 초당 평균 커밋 횟수
  - 평균 커밋 시간
- 프로세서 노드 메트릭
  - 평균 및 최대 처리 시간
  - 초당 평균 처리 작업 수
  - 포워드 레이트
- 상태 저장소 메트릭
  - put, get, flush 작업의 평균 실행 시간
  - put, get, flush 작업의 초당 평균 실행 횟수

[Monitor Kafka Streams Applications in Confluent](https://docs.confluent.io/platform/current/streams/monitoring.html#)

.

👉🏻 **메트릭 구성**
- 카프카 스트림즈는 이미 성능 메트릭을 수집하는 메커니즘을 제공

설정한 레벨에 따른 간으한 메트릭
- 메트릭 수집의 기본 레벨은 INFO

|매트릭 카테고리|DEBUG|INFO|
|---|---|---|
|스레드|O|O|
|태스크|O||
|프로세서 노드|O||
|상태 저장소|O||
|레코드 캐시|O||

메트릭을 위한 구성 변경

```java
private static Properties getProperties() {
    Properties props = new Properties();
    props.put(StreamsConfig.CLIENT_ID_CONFIG, "zmart-metrics-client-id"); // 클라이언트 아이디
    props.put(ConsumerConfig.GROUP_ID_CONFIG, "zmart-metrics-group-id"); // 그룹 아이디
    props.put(StreamsConfig.APPLICATION_ID_CONFIG, "zmart-metrics-application-id"); // 애플리케이션 아이디
    props.put(StreamsConfig.METRICS_RECORDING_LEVEL_CONFIG, "DEBUG"); // 메트릭 로그 레벨
    props.put(StreamsConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092"); // 브로커 접속 설정
    props.put(StreamsConfig.producerPrefix(ProducerConfig.INTERCEPTOR_CLASSES_CONFIG), Collections.singletonList(ZMartProducerInterceptor.class));
    return props;
}
```

- 카프카 스트림즈 애플리케이션의 전체 범위를 측정하는 기본 메트릭이 있으며, DEBUG 레벨에서 메트릭 수집을 설정하려면 그 전에 성능 영향을 신중하게 고려해야 함

.

👉🏻 **수집한 메트릭 확인 방법**
- 카프카 스트림즈 애플리케이션의 메트릭을 수집하면 메트릭 리포터에게 배포
- 기본 메트릭 리포터를 JMX(Java Management Extensions)를 통해 제공
- 메트릭 접근 예제
  - [StockPerformanceStreamsAndProcessorMetricsApplication.java](https://github.com/bbejeck/kafka-streams-in-action/blob/master/src/main/java/bbejeck/chapter_7/StockPerformanceStreamsAndProcessorMetricsApplication.java)

.

👉🏻 **JMX(Java Management Extensions) 사용**
- 자바 VM에서 실행되는 프로그램의 동작을 보는 표준 방법
- JMX를 사용해 자바 VM 성능도 확인 가능
- 즉, JMX는 실행 중인 프로그램의 일부를 노출하는 인프라를 제공
- 모니터링 수행을 위해 [VisualVM](https://visualvm.github.io/), [JConsole](https://docs.oracle.com/en/java/javase/13/management/using-jconsole.html#GUID-77416B38-7F15-4E35-B3D1-34BFD88350B5), [JMC](https://docs.oracle.com/javacomponents/jmc-5-5/jmc-user-guide/jmc.htm#JMCCI111) 사용