# Part 2. 카프카 스트림즈 개발

# 카프카 스트림즈 개발

## 스트림 프로세서 API
- 카프카 스트림즈 애플리케이션을 신속하게 만들 수 있게 해주는 `고수준 API`
- 핵심은 키/값 쌍 레코드 스트림을 나타내는 `KStream 객체`
- 대부분의 메소드는 KStream 객체 레퍼런스를 반환해 플루언트 인터페이스(fluent interface) 스타일의 프로그래밍
  - 메소드 호출의 반환값이 원래 메소드를 호출한 인스턴스와 같음
- [Fluent Interface](https://martinfowler.com/bliki/FluentInterface.html)

.

## 토폴로지 생성해보기
- 카프카 스트림즈 애플리케이션을 만드는 첫 번째 단계는 소스 노드를 만드는 것
- 소스 노드는 애플리케이션을 통해 유입되는 레코드를 토픽에서 소비하는 역할
  
1️⃣ 스트림 소스 정의하기
- 토픽 이름 지정 외에도 카프카의 레코드를 역직렬화하기 위해 `Serde` 객체 제공

```java
KStream<String, String> simpleFirstStream = 
    builder.stream("src-topic", Consumed.with(stringSerde, tringSerde));
```

2️⃣ 유입 텍스트를 대문자로 매핑하기
- 노드의 입력값을 mapValues 호출을 통해 결괏값으로 만드는 새로운 노드를 생성
- 초깃값의 변환된 복사본을 받음

```java
KStream<String, String> upperCasedStream = 
        simpleFirstStream.mapValues(String::toUpperCase);
```

3️⃣ 싱크 노드 생성
- 싱크 프로세서는 레코드를 다시 카프카에 보냄

```java
upperCasedStream.to( "out-topic", Produced.with(stringSerde, stringSerde));
```

지금까지의 과정을 플루언트 인터페이스 스타일로 프로그래밍하기
```java
builder.stream("src-topic", Consumed.with(stringSerde, tringSerde))
    .mapValues(String::toUpperCase)
    .to( "out-topic", Produced.with(stringSerde, stringSerde));
```

.

👉🏻 **카프카 스트림즈 설정**
- `StreamsConfig.APPLICATION_ID_CONFIG`
  - 카프카 스트림즈 애플리케이션을 식별하며 전체 클러스터에 대해 고유한 값
- `StreamsConfig.BOOTSTRAP_SERVERS_CONFIG`
  - hostname:port 쌍 또는 쉼표로 구분된 다중의 쌍
  - 카프카 스트림즈 애플리케이션에 카프카 클러스터의 위치를 제공

.

👉🏻 **Serde 생성**

```java
Serde<String> stringSerde = Serdes.String();
```
- Serdes 클래스를 사용해 키와 값을 직렬화/역직렬화에 필요한 Serde 인스턴스 생성
- 아래 타입에 대한 기본 구현을 제공
  - String
  - Byte array
  - Long
  - Integer
  - Double
  
카프카 스트림즈 애플리케이션의 대부분에서 볼 수 있는 일반적인 패턴
1. StreamsConfig 인스턴스 생성
2. Serde 객체 생성
3. 처리 토폴로지 구성
4. 카프카 스트림즈 프로그램 시작

## 사용자 데이터로 작업하기

![Result](https://github.com/jihunparkme/jihunparkme.gitbook.io/blob/main/.gitbook/assets/kafka-streams-in-action/example-stream.png?raw=true 'Result')

👉🏻 **토폴로지 구성하기**
- 소스 노드 만들기
  - 고객의 개인 정보를 보호하기 위해 신용카드번호를 마스킹하는 책임

```java
KStream<String,Purchase> purchaseKStream = 
        // 쉼표로 구분된 이름 목록 또는 토픽 이름과 일치하는 정규 표현식을 대신 제공
        streamsBuilder.stream("transactions", 
        // 토픽 이름만 제공하면 설정 매개변수를 통해 제공된 기본 Serdes 사용
        Consumed.with(stringSerde, purchaseSerde))
        // 하나의 타입 매개변수를 취해 해당 객체를 새로운 값으로 매핑
        // 만일 새 값을 만드는데 새 키/값 쌍을 생성하거나 키를 포함하려면 KStream.map 메소드를 사용
        .mapValues(p -> Purchase.builder(p).maskCreditCard().build());
```

👉🏻 **두 번째 프로세서 만들기**

```java
KStream<String, PurchasePattern> patternKStream = 
    purchaseKStream.mapValues(purchase -> 
    PurchasePattern.builder(purchase).build());

// KStream 인스턴스의 데이터를 카프카 토픽에 쓰는 데 사용하는 싱크 노드를 정의
patternKStream.to("patterns", 
    Produced.with(stringSerde,purchasePatternSerde));
```

👉🏻 **세 번째 프로세서 만들기**

```java
KStream<String, RewardAccumulator> rewardsKStream = 
        purchaseKStream.mapValues(purchase -> 
        RewardAccumulator.builder(purchase).build());

rewardsKStream.to("rewards", 
        Produced.with(stringSerde,rewardAccumulatorSerde));
```

👉🏻 **마지막 프로세서 만들기**
- [ZMartKafkaStreamsApp](https://github.com/bbejeck/kafka-streams-in-action/blob/master/src/main/java/bbejeck/chapter_3/ZMartKafkaStreamsApp.java)

```java
purchaseKStream.to("purchases", Produced.with(stringSerde,purchaseSerde));
```

.

✅ **사용자 정의 Serde 생성하기**
- 카프카는 데이터를 바이트 배열 형식으로 전송
  - 데이터 형식이 JSON 이므로 토픽에 데이터를 보낼 때 먼저 객체를 JSON으로 변환하고 바이트 배열로 변환하는 방법을 카프카에게 알려줘야 한다.
  - 반대로, 소비한 바이트 배열을 JSON으로 변환한 다음 프로세서에서 사용할 객체 타입으로 변환하는 방법을 명시해야 한다.
- 즉, `Serde`는 데이터를 다른 형식으로 변환하기 위해 필요
- Serde를 만들려면 Deserializer\<T\>와 Serializer\<T\> 인터페이스를 구현해야 한다.

```java
// 직렬화
public class JsonSerializer<T> implements Serializer<T> {
    private Gson gson;

    public JsonSerializer() {
        GsonBuilder builder = new GsonBuilder();
        builder.registerTypeAdapter(FixedSizePriorityQueue.class, new FixedSizePriorityQueueAdapter().nullSafe());
        gson = builder.create();
    }

    @Override
    public void configure(Map<String, ?> map, boolean b) { }

    @Override
    public byte[] serialize(String topic, T t) {
        // 객체를 바이트로 직렬화(객체를 JSON으로 변환 후 문자열에서 바이트를 가져옴)
        return gson.toJson(t).getBytes(Charset.forName("UTF-8"));
    }

    @Override
    public void close() { }
}

...

// 역직렬화
public class JsonDeserializer<T> implements Deserializer<T> {
    private Gson gson;
    private Class<T> deserializedClass;
    private Type reflectionTypeToken;

    public JsonDeserializer(Class<T> deserializedClass) {
        this.deserializedClass = deserializedClass;
        init();

    }

    public JsonDeserializer(Type reflectionTypeToken) {
        this.reflectionTypeToken = reflectionTypeToken;
        init();
    }

    private void init () {
        GsonBuilder builder = new GsonBuilder();
        builder.registerTypeAdapter(FixedSizePriorityQueue.class, new FixedSizePriorityQueueAdapter().nullSafe());
        gson = builder.create();
    }

    public JsonDeserializer() { }

    @Override
    @SuppressWarnings("unchecked")
    public void configure(Map<String, ?> map, boolean b) {
        if(deserializedClass == null) {
            deserializedClass = (Class<T>) map.get("serializedClass");
        }
    }

    @Override
    public T deserialize(String s, byte[] bytes) {
        if(bytes == null){
            return null;
        }
        Type deserializeFrom = deserializedClass != null ? deserializedClass : reflectionTypeToken;
        // 바이트 배열을 기대하는 클래스의 인스턴스로 역직렬화
        return gson.fromJson(new String(bytes), deserializeFrom);
    }

    @Override
    public void close() { }
}

...

JsonDeserializer<Purchase> purchaseJsonDesirializer = 
        new JsonDeserializer<>(Purchase.class);
JsonSerializer<Purchase> purchaseJsonSirializer = 
        new JsonSerializer<>();
Serde<Purchase> purchaseSerde = 
        Serdes.serdeForm(purchaseJsonSerializer, purchaseJsonDesirializer);
```

## 대화형 개발

Printed는 stdout에 출력하는 `Printed.toSysOut()` 혹은 파일에 결과를 기록하는 `Printed.toFile(filePath)` 두 가지 정적 메소드를 제공

```java
patternKStream.print(Printed.<String, PurchasePattern>toSysOut()
                .withLabel("patterns"));

rewardsKStream.print(Printed.<String, RewardAccumulator>toSysOut()
                .withLabel("rewards"));

purchaseKStream.print(Printed.<String, Purchase>toSysOut()
                .withLabel("purchases"));
```

## 다음 단계

👉🏻 **구매 필터링**

- Predicate<K, V> 인스턴스를 매개변수로 사용하는 KStream 메소드 사용

  ```java
  KeyValueMapper<String, Purchase, Long> purchaseDateAsKey = (key, purchase) -> 
        purchase.getPurchaseDate().getTime();

  // $5.00 미만인 구매를 필터링하고 Long 값의 구매 날짜를 키로 선택
  KStream<Long, Purchase> filteredKStream = 
        purchaseKStream.filter((key, purchase) -> 
        purchase.getPrice() > 5.00).selectKey(purchaseDateAsKey);
  ```

.

👉🏻 **키 생성하기**

`KStream.selectKey` 메소드를 통해 새로운 키와 동일한 값을 갖는 레코드를 만드는 KStream 인스턴스를 반환
- 키 타입을 (Long으로) 변경했으므로 KStream.to 메소드에서 사용된 serde 타입도 변경

```java
KeyValueMapper<String, Purchase, Long> purchaseDateAsKey = (key, purchase) -> 
    purchase.getPurchaseDate().getTime();

// $5.00 미만인 구매를 필터링하고 Long 값의 구매 날짜를 키로 선택
KStream<Long, Purchase> filteredKStream = 
    purchaseKStream.filter((key, purchase) -> 
    purchase.getPrice() > 5.00).selectKey(purchaseDateAsKey);

filteredKStream.print(Printed.<Long, Purchase>
        toSysOut().withLabel("purchases"));
filteredKStream.to("purchases", 
        Produced.with(Serdes.Long(), purchaseSerde));
```

.

👉🏻 **Foreach 액션**
- KStream.foreach는 ForeachACtion<K, V>인스턴스를 사용하며, 터미널 노드의 또 다른 예
  - 제공된 ForeachACtion 인스턴스를 사용해 받는 각 레코드에 대해 작업을 수행하는 간단한 프로세서

```java
ForeachAction<String, Purchase> purchaseForeachAction = (key, purchase) ->
    SecurityDBService.saveRecord(
        purchase.getPurchaseDate(), 
        purchase.getEmployeeId(), 
        purchase.getItemPurchased()
    );


purchaseKStream.filter((key, purchase) -> 
    purchase.getEmployeeId()
    .equals("000000"))
    .foreach(purchaseForeachAction);
```

📖 **요약**

> KStream.`mapValues` 함수를 사용하면 들어오는 레코드값을 가능한 다른 타입의 새로운 값으로 매핑
> - 메소드인 KStream.`map은` 동일한 작업을 수행하지만 키와 값을 모두 새로운 것으로 매핑
>
> `predicate`는 매개변수로 객체를 받아들이고 해당 객체가 주어진 조건과 일치하는지 여부에 따라 true 또는 false를 반환하는 구문
>
> KStream.`selectKey` 메소드를 사용하면 기존 키를 수정하거나 새 키를 생성

## 이벤트

[ZMartKafkaStreamsAddStateApp](https://github.com/bbejeck/kafka-streams-in-action/blob/master/src/main/java/bbejeck/chapter_4/ZMartKafkaStreamsAddStateApp.java)

👉🏻 **스트림의 상태**
- 스트림 처리에서 추가된 문맥을 상태라고 부른다.
- 상태의 개념은 데이터베이스 테이블 같은 정적 리소스의 이미지를 떠올릴 수 있다.
- 가장 기본적인 상태 유지 함수는 `KStream.transformValues`

![Result](https://github.com/jihunparkme/jihunparkme.gitbook.io/blob/main/.gitbook/assets/kafka-streams-in-action/transformingState.jpg?raw=true 'Result')

- `transformValues` 프로세서는 로컬 상태에 저장된 정보를 사용하여 들어오는 레코드를 업데이트
- 의미상으로는 mapVAlues()와 동일하지만 몇 가지 예외가 존재
  - 한 가지 차이점은 transformValues가 StateStore 인스턴스에 접근해서 작업을 완료하는 것

.

👉🏻 **고객 보상의 상태 유지 예시**

- 보상을 분배하기 위해 로컬 상태를 사용하여 누적 포인트와 마지막 구매 날짜를 추적
- 여기서 `transformValues`의 역할
  - 값 변환기를 초기화
  - 상태를 사용해 Purchase 객체를 RewardAccumulator로 매핑

```java
public class RewardAccumulator {
    private String customerId;
    private double purchaseTotal;
    private int totalRewardPoints; // 포인트
    private int currentRewardPoints;
    private int daysFromLastPurchase; // 마지막 구매 날짜
    //...
}
```

.

👉🏻 **Transformer**
- 여기서 키가 채워지지 않으므로 라운드 로빈 할당은 주어진 고객에 대한 트랜잭션이 동일한 파티션에 들어가지 않음을 의미
- 이 문제를 해결하는 방법은 고객 ID로 데이터를 다시 분할하는 것

```java
public class PurchaseRewardTransformer implements ValueTransformer<Purchase, RewardAccumulator> {

    private KeyValueStore<String, Integer> stateStore;
    private final String storeName;
    private ProcessorContext context;

    public PurchaseRewardTransformer(String storeName) {
        Objects.requireNonNull(storeName,"Store Name can't be null");
        this.storeName = storeName;
    }

    /** 처리 토폴로지를 만들 때 생성된 상태 저장소 찾기 **/
    @Override
    @SuppressWarnings("unchecked")
    public void init(ProcessorContext context) {
        this.context = context;
        stateStore = (KeyValueStore) this.context.getStateStore(storeName);
    }

    /** 상태를 사용해 Purchase 객체를 RewardAccumulator에 매핑하기  **/
    @Override
    public RewardAccumulator transform(Purchase value) {
        RewardAccumulator rewardAccumulator = RewardAccumulator.builder(value).build();
        // 1. 고객 ID별로 누적된 포인트 조회
        Integer accumulatedSoFar = stateStore.get(rewardAccumulator.getCustomerId());

        if (accumulatedSoFar != null) {
            // 2. 현재 거래에 대한 포인트를 합산하고 합계를 표시
            rewardAccumulator.addRewardPoints(accumulatedSoFar);
        }
        // 3. RewardAccumulator 보상 포인트를 새로운 총 보상 포인트로 설정
        // 4. 고객 ID별로 새 총점을 로컬 상태 저장소에 저장
        stateStore.put(rewardAccumulator.getCustomerId(), rewardAccumulator.getTotalRewardPoints());

        return rewardAccumulator;

    }

    @Override
    @SuppressWarnings("deprecation")
    public RewardAccumulator punctuate(long timestamp) {
        return null;  //no-op null values not forwarded.
    }

    @Override
    public void close() {
        //no-op
    }
}

...

KStream<String, RewardAccumulator> statefulRewardAccumulator = 
        transByCustomerStream.transformValues(() -> 
                new PurchaseRewardTransformer(rewardsStateStoreName), rewardsStateStoreName);
```

.

👉🏻 **데이터 리파티셔닝**
- 레코드를 리파티셔닝하려면 먼저 원본 레코드의 키를 변경하거나 바꾼 다음 레코드를 새로운 토픽에 쓴다.
  - 그 다음으로, 해당 레코드를 다시 소비
- [StreamPartitioner](https://kafka.apache.org/10/javadoc/org/apache/kafka/streams/processor/StreamPartitioner.html)를 사용하면 키 대신 값 또는 값의 일부를 이용해서 분할하는 다른 파티션 전략 적용 가능

.

**카프카 스트림즈의 리파티셔닝**
- 카프카 스트림즈에서 리파티셔닝은 KStream.through()를 사용해 쉽게 수행
  - 중간 토픽을 생성하고 현재 KStream 인스턴스는 해당 토픽에 레코드를 기록

![Result](https://github.com/jihunparkme/jihunparkme.gitbook.io/blob/main/.gitbook/assets/kafka-streams-in-action/kstreamThroughDemo.jpg?raw=true 'Result')

- 반환된 KStream 인스턴스는 중간 토픽을 즉시 소비하기 시작
- 중간 토픽을 사용하기 위해 내부적으로 싱크 노드와 소스 노4드를 생성

**KStream.through 메소드 사용하기**

```java
RewardsStreamPartitioner streamPartitioner = new RewardsStreamPartitioner();

KStream<String, Purchase> transByCustomerStream = 
        purchaseKStream.through( "customer_transactions", 
                Produced.with(stringSerde, purchaseSerde, streamPartitioner));
```

**StreamPartitioner 사용하기**
- 키 대신 Purchase 객체에 있는 고객 ID를 사용해 특정 고객의 모든 데이터가 동일한 상태 저장소에 저장되도록 적용
- 여기서 핵심은 상태를 사용해 레코드를 업데이트하고 수정할 때 해당 레코드가 같은 파티션에 있어야 한다는 것

```java
public class RewardsStreamPartitioner implements StreamPartitioner<String, Purchase> {
    @Override
    public Integer partition(String key, Purchase value, int numPartitions) {
        // 고객 ID로 파티션을 결정
        return value.getCustomerId().hashCode() % numPartitions;
    }
}
```

✅ 참고

> 리파티셔닝이 가끔 필요하긴 하지만, 데이터가 중복되거나 프로세싱 오버헤드가 발생한다.
>
> 가능하면 mapValue(), transformValues(), flatMapValues() 사용을 권장
>
> map(), transform(), flatMap()은 자동으로 리파티셔닝을 유발할 수 있기 때문

.

👉🏻 **보상 프로세서 업데이트**

```java
// 상태를 가진 변환 사용
KStream<String, RewardAccumulator> statefulRewardAccumulator = 
        transByCustomerStream.transformValues(() -> 
            new PurchaseRewardTransformer(rewardsStateStoreName), rewardsStateStoreName);

// 결과를 토픽에 기록
statefulRewardAccumulator.to("rewards", Produced.with(stringSerde, rewardAccumulatorSerde));
```

![Result](https://github.com/jihunparkme/jihunparkme.gitbook.io/blob/main/.gitbook/assets/kafka-streams-in-action/through-processor.png?raw=true 'Result')

## 상태 저장소 사용하기

상태의 두 가지 중요한 속성인 데이터 지역성(data locality)과 실패 복구(failure recovery)를 살펴보자

👉🏻 **데이터 지역성**
- 데이터 지역성은 스트림 처리에 필요
- 중요한 점은 스트리밍 애플리케이션이 때로 상태를 필요로 하지만 처리가 이뤄지는 곳은 로컬이어야 한다.

![Result](https://github.com/jihunparkme/jihunparkme.gitbook.io/blob/main/.gitbook/assets/kafka-streams-in-action/dataLocality.jpg?raw=true 'Result')

.

👉🏻 **실패 복구와 내결함성**
- 각 프로세서에는 로컬 데이터 저장소가 있으며, 변경로그 토픽은 상태 저장소를 백업하는 데 사용
- kafkaProducer는 레코드를 배치로 보내며 기본적으로 레코드는 캐시
- 캐시를 플러시할 때만 카프카 스트림즈가 레코드를 저장소에 기록하므로 주어진 키에 대한 최신 레코드만 유지
- 카프카 스트림이 제공하는 상태 저장소는 지역성과 내결함성 요구사항을 모두 충족
- 상태 저장소는 또한 백업과 빠른 복구를 위해 토픽을 사용

![Result](https://github.com/jihunparkme/jihunparkme.gitbook.io/blob/main/.gitbook/assets/kafka-streams-in-action/failureRecovery.jpg?raw=true 'Result')
- 프로세서는 자체 로컬 상태 저장소와 비공유 아키텍처가 있으므로 두 프로세스 중 하나가 실패하면 다른 프로세스는 영향을 받지 않음
- 또한, 각 저장소는 토픽에 복제된 키/값을 가지며 프로세스가 실패하거나 다시 시작할 때 잃어버린 값을 복구하는 데 사용
  - 오류를 복구하는 기능은 스트림 애플리케이션에서 중요
  - 카프카 스트림즈는 로컬 인메모리 저장소의 데이터를 내부 토픽으로 유지하므로 실패 또는 재시작 후 작업을 다시 시작할 때 데이터가 다시 채워짐

.

👉🏻 **상태 저장소 사용하기**
- 고수준 DSL을 사용하는 경우 보통 `Materialized` 클래스를 사용
- 저수는 프로세서 API로 작업 시 `StoredBuilder`를 사용

```java
String rewardsStateStoreName = "rewardsPointsStore";
// StateStore 공급자 생성
KeyValueBytesStoreSupplier storeSupplier = 
        Stores.inMemoryKeyValueStore(rewardsStateStoreName);

// StoreBuilder를 생성하고 키와 값의 타입을 명시
StoreBuilder<KeyValueStore<String, Integer>> storeBuilder = 
        Stores.keyValueStoreBuilder(storeSupplier, Serdes.String(), Serdes.Integer());

// 상태 저장소를 토폴로지에 추가
builder.addStateStore(storeBuilder);
```

.

👉🏻 **추가적인 키/값 저장소 공급자**
- `Stores.inMemoryKeyValueStore` 메소드 외에도 정적 팩토리 메소드를 사용해 저장소 공급자 생성 가능
  - Stores.persistentKeyValueStore
  - Stores.persistentWindowStore
  - Stores.lruMap
  - Stores.persistentSessionStore
- 모든 영구 StateStore 인스턴스가 [RocksDB](https://rocksdb.org/)를 사용해 로컬 스토리지를 제공

.

👉🏻 **상태 저장소의 내결함성**
- 모든 `StateStoreSupplier` 타입은 기본적으로 로깅이 활성화
  - 로깅은 저장소의 값을 백업하고, 내결함성을 제공하기 위한 변경로그로 사용되는 카프카 토픽을 의미
  - 서버를 복구하고 카프카 스트림즈 애플리케이션을 다시 시작하면 해당 인스턴스의 상태 저장소가 원래 내용으로 복원

.

👉🏻 **변경로그 토픽 설정하기**
- 상태 저장소에 대한 변경로그는 `withLoggingEnabled(Map<String, String> config)` 메소드를 통해 설정 가능
- 토픽에 대해 가능한 모든 설정 매개변수를 사용 가능
- 카프카 스트림즈는 변경로그 토픽 생성을 자동으로 처리
- ✅ 참고: 상태 저장소에서 영구적으로 레코드를 제거하고 싶다면 put(key, null) 작업을 실행

로그 세그먼트의 데이터 보전에 대한 기본 설정은 일주일이고, 크기는 무제한
- 변경로그 토픽이 10GB의 보존 크기와 2일의 보존 기간을 갖도록 설정하는 방법

    ```java
    Map<String, String> changeLogConfigs = new HashMap<>();
    changeLogConfigs.put("retention.ms", "172800000"); // 2일
    changeLogConfigs.put("retention.bytes", "10000000000"); // 10GN

    // StoreBuilder 사용 시
    storeBuilder.withLoggingEnable(changeLogConfigs);

    // Materialized 사용 시
    Meterialized.as(Stores.inMemoryKeyValueStore("foo").withLoggingEnable(changeLogConfigs));
    ```
- 압축 토픽은 토픽을 정리하는 데 다른 방법을 사용
  - 로그 세그먼트를 크기나 시간별로 삭제하는 대신 **로그 세그먼트를 각 키별로 최신 레코드만 유지하는 방식으로 압축**
  - 동일한 키를 가진 이전 레코드는 삭제
  - 기본적으로 카프카 스트림즈는 삭제 정책이 `compact`인 변경로그 토픽을 생성
- 그러나, 많은 고유 키를 가진 변경로그 토픽이 있다면 로그 세그먼트의 크기가 계속 커지면서 압축으로 충분하지 않을 수 있음
  - 이 경우 `delete`, `compact` 클린업 정책을 명시

    ```java
    Map<String, String> changeLogConfigs = new HashMap<>();
    changeLogConfigs.put("retention.ms", "172800000");
    changeLogConfigs.put("retention.bytes", "10000000000");
    changeLogConfigs.put("cleanup.policy", "compact,delete");
    ```

## 스트림 조인하기

👉🏻 **조인을 위한 키 생성**
- 키를 생성하려면 스트림의 구매 데이터에서 고객 ID를 선택

```java
KStream<String, Purchase>[] filteredCoffeePurchase = 
    transactionStream.selectKey((k,v)-> 
      v.getCustomerId()).filter(coffeePurchase);

KStream<String, Purchase>[] filtereDelectronicPurchase = 
    transactionStream.selectKey((k,v)-> 
      v.getCustomerId()).filter(electronicPurchase);
```

- 새로운 키를 생성하는 메소드(selectKey, map, transform) 호출 때마다 내부 Boolean 플래그가 true로 설정된다.
- 이 boolean 플래그 설정을 사용해 조인, 리듀스 또는 집계 연산을 수행하면 자동으로 리파티셔닝을 처리

.

👉🏻 **조인 구성하기**

![Result](https://github.com/jihunparkme/jihunparkme.gitbook.io/blob/main/.gitbook/assets/kafka-streams-in-action/joinToplogy.jpg?raw=true 'Result')

- 업데이트된 토폴로지에서 카페와 전자제품 프로세서 모두는 레코드를 조인 프로세서로 전달
- 조인 프로세서는 상태 저장소 2개를 사용해 다른 스트림의 레코드와 일치하는 항목을 검색한다.