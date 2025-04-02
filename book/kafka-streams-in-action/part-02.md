# Part 2. 카프카 스트림즈 개발

# 카프카 스트림즈 개발

👉🏻 **스트림 프로세서 API**
- 카프카 스트림즈 애플리케이션을 신속하게 만들 수 있게 해주는 `고수준 API`
- 핵심은 키/값 쌍 레코드 스트림을 나타내는 `KStream 객체`
- 대부분의 메소드는 KStream 객체 레퍼런스를 반환해 플루언트 인터페이스(fluent interface) 스타일의 프로그래밍
  - 메소드 호출의 반환값이 원래 메소드를 호출한 인스턴스와 같음
- [Fluent Interface](https://martinfowler.com/bliki/FluentInterface.html)

.

👉🏻 **토폴로지 생성해보기**
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