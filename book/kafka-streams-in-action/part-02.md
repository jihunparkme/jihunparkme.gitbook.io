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