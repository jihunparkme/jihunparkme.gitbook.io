# Domain Driven Design

# DDD & Micro Service

**마이크로서비스 설계에서 도메인 주도 설계 활용**

<figure><img src="../../.gitbook/assets/micro-service/ddd-design.png" alt=""><figcaption></figcaption></figure>

`전략적 설계`를 이용한 마이크로서비스 `식별`
- 마이크로서비스을 어떤 기준으로 분리할 것인가?
  - Bounded	Context, Ubiquitous Language
- 서비스 간은 어떻게 연계할 것인가?
  - Context	Mapping
- 어떤 이벤트에 의해 마이크로서비스는 서로 반응하는가?
  - Domain Event

> 참고 도서. 
> 
> [도메인 주도 설계 핵심](https://www.yes24.com/Product/Goods/48577718)

`전술적 설계`와 미이크로서비스 `내부 설계`
- 마이크로서비스 내부는 어떻게 설계 할 것인가?
  - Domain Model

> 참고 도서. 
> 
> [도메인 주도 개발 시작하기](https://www.yes24.com/Product/Goods/108431347)
>
> [도메인 주도 설계 첫걸음](https://www.yes24.com/Product/Goods/109708596)
>
> [도메인 주도 설계 철저 입문](https://www.yes24.com/Product/Goods/93384475)

# 도메인 주도 설계(전략적 설계)

구축해야 하는 소프트웨어와 시스템을 위해서는 문제를 이해
- `Problem`: 그 조직의 비즈니스 전략과 소프트웨어를 통해서 얻고자 하는 가치
- `Business Domain`: 기업의 주요 활동 영역, 회사가 제공하는 서비스
- `Sub domain`: 비즈니스 활동의 세분화된 영역, 제공하는 서비스 단위
  - ex. 회원, 고객, 상품, 주문, 배송..

> Problem -> Business -> Domain -> Sub Domain
>
> 시스템에서 사용하는 모든 개념들이 비즈니스 도메인을 표현해야 한다.

.

## Sub Domain

핵심(Core)
- 회사만의 차별성. 복잡성이 높지만 경쟁력 제공
- 핵심 인재 할당과 진보된 엔지니어링 기술 적용
- ex) 우버의 손님 매칭 서비스, 구글의 검색 순위 알고리즘 ..

일반(Generic)
- 모든 회사가 같은 방식으로 수행하는 비지니스 활동
- 복잡하고 구현하기 어려우나 주로 오픈 소스를 사용하고, 경쟁력을 제공하지는 않음
- ex) 인증, 권한 부여 ..

지원(Supporting)	
- 회사 비지니스 지원 활동
- 간단한 기능으로 경쟁우위 제공하지 않음
- ex) CRUD, ETL

## Ubiquitous Language

도메인 지식(멘탈 모델)을 코드로 구현하기 위해 여러 변환 과정을 거치는 대신 그대로 코드로 표현
- 도메인을 코드로 설명하기 위한 단일화된 체계
- 기술 용어 대신 유비쿼터스 언어를 사용
- 유비쿼터스 언어: 동의어와 같이 모호하지 않고, 정확하고 일관성있는 언어
  - ex. 정책 -> 규제 규칙 / 보험계약, 사용자 -> 방문자, 비회원, 회원

## Domain Model

- 특정 도메인을 개념적으로 표현한 것
- 효과적인 모델은 그 목적을 달성하는 데 **필요한 세부사항만 포함**(목적에 필요한 정보만 제공)
  - ex. 지하철 노선도
- 모델은 본질적으로 **추상화의 결과**. 모델로 실세계의 복잡성을 관리
- 도메인 모델링: 유비쿼터스 언어로 비즈니스 도메인 모델 구축

## Bounded Context (BC)

<figure><img src="../../.gitbook/assets/micro-service/bounded-context.png" alt=""><figcaption></figcaption></figure>

- BC는 유비쿼터스 언어(모델)의 일관성이 유지되는 경계
- 유비쿼터스 언어를 여러 개의 작은 언어로 나눈 다음, 각 언어를 적용할 수 있는 명시적인 bounded Context에 할당
  - 유비쿼터스 언어의 용어, 원칙, 비지니스 규칙은 해당 바운디드 컨텍스트 내에서만 일관성

  ```bash
  ├── Order
  │   └── Customer: 주문하는 사람 -> Buyer
  │
  ├── Accounting
  │   └── Customer: 시스템 고객 -> Member
  │
  ├── Delivery
  │   └── Customer: 배송 받는 사람 -> Recipient
  └── 
  ```

- BC는 컴포넌트 개발의 수명주기를 분리할 필요가 있거나 기능이 독립적으로 확장되어야 할 경우 세분화될 수 있음
- 각 BC의 수명 주기는 독립적

Sub Domain 과의 차이
- 하위 도메인은 비지니스 전략에 의해 정의되는 반면 바운디드 컨텍스트는 소프트웨어 엔지니어에 의해 설계
- 하위 도메인은 발견, 바운디드 컨텍스트는 설계

## Context Mapping

- BC는 독립적으로 발전할 수 있지만 **서로 상호작용**을 해야 한다. 
  - **각 BC는 접점**이 있는데 이것이 **contract**
- BC의 연동/통합 : 각 바운디드 컨텍스트에서 작업하는 팀간의 관계를 의미
  - 협력형 패턴
    - 파트너십
    - 공유 영역
  - 공급자, 소비자 패턴
    - Supplier(Upstream) - Customer(Downstream)
    - 순응주의자(Conformist) 패턴: 힘의 균형은 상류에 존재
    - 충돌방지계층(ACL, Anti-Corruption Layer) 패턴: 하류가 이에 순응하지 않은 경우
    - OHS(Open Host Service) 패턴: 힘이 사용자 측에 있을 경우 제공자는 사용자를 보호하고 가능한 최고의 서비스를 제공
  - 분리형 노선
    - Separated Way
    - 커뮤니케이션의 어려움
    - 협력과 연동보다 특정 기능을 중복으로 두는 것이 더 저렴한 경우

# 도메인 주도 설계(전술적 설계)

## Transaction Script Pattern

- 간단한 비지니스 로직 구현(절차 지향 스크립트), 데이터베이스 직접 접근
  - 단순하고 이해하기 쉬운 장점
  - 비즈니스 로직이 복잡할수록 트랜잭션 간 로직이 중복되기 쉽고, 중복 코드가 동기화되지 않으면 일
관성 없는 동작 발생
  - 코어 하위 도메인에는 적용하지 않는 것을 권장
- 대안으로 Active Record, Domain Model 패턴의 등장

```java
db.Excute(@"UPDATE Users SET last_visit=@p1
WHERE user_id=@p2",vistedOn,userId);
db.Excute(@"INSERT INTO VisiteLog(user_id,visit_date)
VALUES(@p1,@p2)", userId,visitedOn);
```

## Active Record Pattern

<figure><img src="../../.gitbook/assets/micro-service/active-record-pattern.png" alt=""><figcaption></figcaption></figure>

- Anemic Domain Model(빈약한 도메인 모델)
- 복잡한 자료 구조를 표현
  - 자료 구조 외에도 CRUD, ORM 연계 등 데이터 접근 로직 구현
    - 사용자 입력의 유효성을 검사하는 CRUD 작업 같은 간단한 비즈니즈 로직
  - 본질적으로는 DB 접근을 최적화하는 Transaction	Script Pattern 유형
- Entity 에 약간의 비즈니스 로직을 포함해도 되지만, 대부분의 비지니스 로직과 흐름은 응용 서비스의 행위에 의해 처리
- 비즈니스 로직은 단순하지만 복잡한 자료 구조인 지원 하위 도메인, 일반 하위 도메인에 적합

```java
User user = new User();
user.setName(userDetails.name);
user.setEmail(userDetails.email);
user.save();
```

## Domain Model Pattern

> 전술적 도메인 주도 설계(Tactical Domain-Driven Design) 패턴 - 에반스

<figure><img src="../../.gitbook/assets/micro-service/domain-model-pattern.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/micro-service/domain-model-example.png" alt=""><figcaption></figcaption></figure>

- 행위(behavior) + 자료구조(data)를 통해 비지니스 로직 구현
- POJO(Plain Old Java Object)로 구성
  - 복잡한 인프라, 기술적 관심사는 피하고 비즈니스 로직으로만 구성
- 응용 서비스에서는 대부분 업무 흐름 제어만 하며, 주요 비지니스 로직은 도메인 모델에 위임하여 처리

## Aggregate Pattern

<figure><img src="../../.gitbook/assets/micro-service/aggregate-pattern.png" alt=""><figcaption></figcaption></figure>

- 도메인 모델 패턴 적용 시 도메인 모델이 점점 복잡하고 비대해짐(Big ball of mud)
- 도메인 주도의 Aggregate 단위로 복잡성을 구분하여 관리하는 패턴
- 대부분 한개의 Entity(Aggregate Root)와 여러 개의 VO로 구성

### VO(Value Object)

여러 개의 VO로 구성된 엔티티 예시

```java
class Person {
  private PersonId id;
  private Name name;
  private PhoneNumber landline;
  private PhoneNumber mobile;
  private EmailAddress email;
  private Height height;
  private CountryCode country;

  public Person(...) {}
}
```

VO 예시

```java
class PhoneNumber {
  String number;

  public PhoneNumber(...) {
    // validate phone number
  }
}
```

- VO는 고유의 식별자를 가지지 않고, 개념적으로 완전한 하나를 표현
- 상태를 변경할 수 없고, 단순히 값만을 갖는 읽기 전용 불변(immutable) 객체
  - 객체 변경 시 객체 자체를 완전히 교체
- 분산되기 쉬운 비즈니스 로직을 한 곳에 묶어주고 안전한 코드 작성
- 대부분의 경우에 적용 가능
  - 다른 객체(엔티티, VO)의 속성을 표현하는 요소로 사용
- 명료성 향상, 명확한 의도 전달, 유효성 검사, 비즈니스 로직 표현, 유비쿼터스 언어를 사용한 비즈니스 도메인 개념 표현에 대한 이점

### Entity

```java
class Person {
  public PersonId id;
  private Name name;
  
  public Person(PersonId id, Name name) {
    this.id = id;
    this.name = name;	
  }
}
```

- 도메인의 고유 개념 표현
- 다른 객체와 구별할 수 있는 식별자(고유 식별자)를 갖는 객체
- 주문에서 배송지 정보가 변경되어도 주문번호는 변경되지 않음
- 자신의 생명주기를 가짐

### Aggregate

<figure><img src="../../.gitbook/assets/micro-service/aggregate.png" alt=""><figcaption></figcaption></figure>

```java
class Target {
  private UserId customer; // 다른 Aggregate는 식별자를 통해 참조
  private List<ProductId> products;
  private UserId assignAgent;
  private List<Message> messages; // 같은 Aggregate는 엔티티로 참조
}
```

- 관련 객체를 하나로 묶은 군집
- 데이터의 일관성을 보호하고, 데이터 변경 시 Aggregate 단위로 처리
- Aggregate Root(Entity)를 통해 Aggregate 내의 다른 Entity 및 VO 접근
- 데이터 변경의 단위, 트랜잭션 단위가 되는 연관된 객체 묶음
- 설계 시 고려사항
  - 하나의 Transaction에서는 하나의 Aggregate만 수정
    - Transaction 일관성과 성공 보장
  - 하나의 일을 잘 수행할 수 있도록 작게 설계
    - 성능 향상과 확장에 용이
  - 한 Aggregate에서 다른 Aggregate의 참조는 식별자(id)를 통해서만 참조
    - 하나의 Transaction 내에서 여러 Aggregate 수정 방지
  - 하나의 Transaction에서 여러 개의 Aggregate이 갱신되어야 하는 경우, 다른 Aggregate 갱신은 비동기 통신을 활용해서 결과적 일관성을 맞춰야 함

### Domain Event

<figure><img src="../../.gitbook/assets/micro-service/domain-event.png" alt=""><figcaption></figcaption></figure>

- 비지니스 도메인에서 일어난 이벤트를 설명하는 메시지(`'과거형'`으로 명명)
- Aggregate의 퍼블릭 인터페이스의 일부, Aggregate는 자신의 Domain Event를 발행

### Domain Service

<figure><img src="../../.gitbook/assets/micro-service/domain-service.png" alt=""><figcaption></figcaption></figure>

- 특정 엔티티/VO에 속하지 않는 도메인 로직 또는 복수의 Aggregate 관련 비지니스 로직 제공
- 어떤 계산이나 분석을 위해 다양한 시스템 구성요소의 호출을 조율
- 상태가 없는 객체(stateless object)
  
### Repository

- 도메인 모델의 영속성을 처리
- 도메인 모델을 사용하기 위해 Repository를 통해 도메인 객체를 조회한 후 도메인 객체의 기능 실행
- 도메인 객체(Aggregate)에 대한 생명주기, 즉 영속성 관리(등록, 조회, 수정, 삭제 시 Aggregate의 일관성 유지)
- Spring Data JPA Repository Interface

# MSA Pattern

마이크로서비스 내부 구조 정의
- `전술적 패턴`은 **비즈니스 로직을 모델링하고 구현하는 다양한 방법**
- 시스템을 구성하기 위해서는 **비즈니스 로직 외에도 여러 기능들이 필요**
  - 사용자와 상호작용(입/출력)
  - 다양한 저장소에 상태 저장
  - 외부 시스템과 연동
- 이런 필요에 이해 비즈니스 로직이 다른 구성요소에 흩어지기 쉬움

> 아키텍처 패턴은 코드의 구성 원칙을 도입하고 이들 사이의 명확한 경계를 제시

## Layered architecture

- 고전적이고, 가장 대중적인 아키텍쳐
- **3 계층**으로 구성
  - 1 계층: 프리젠테이션(MVC 컨트롤러에 속함)
  - 2 계층: 비즈니스 로직(가장 중요한 계층)
  - 3 계층: 데이터 액세스
  - 근접 레이어에만 접근 가능하고, Top-down 순방향으로 의존
- 레이어드 아키텍처는 **비즈니스 로직과 데이터 접근 계층 간에 의존성**이 존재
  - 비즈니스 로직 구현이 **트랜잭션 스크립트**, **엑티브 레코드**인 경우 적합한 아키텍처
  - 하지만, 결국 데이터 액세스 계층에 비지니스 로직이 점점 몰리게 된다
  - 중요한 것(잘 변하지 않는 것, 비즈니스 로직)이 덜 중요한 것(가변적인 것, 기술)에 의존하여 변화에 대응할 수 없는 비유연성을 내포하는 문제가 발생
- 위 문제를 해결하기 위한 **의존성 역전의 원칙**(DIP)의 등장
  - 비즈니스 로직을 구현하는 상위 수준의 모듈은 하위 수준의 모듈에 의존해서는 안된다.
  - 인터페이스 활용: 비즈니스 로직은 데이터 엑세스 계층의 인터페이스를 정의
    - 비즈니스 로직은 인터페이스에 의존하고 데이터 액세스 계층은 인터페이스를 구현

<figure><img src="../../.gitbook/assets/micro-service/layered-architecture.png" alt=""><figcaption></figcaption></figure>

## Hexagonal Architecture

<figure><img src="../../.gitbook/assets/micro-service/hexagonal-architecture.png" alt=""><figcaption></figcaption></figure>

- port and adapter pattern
  - 외부 영역: 프레젠테이션, 데이터 접근
  - 내부 영역: 응용, 비즈니스 로직
- 외부 영역의 어댑터가 다양하게 변경됨으로 **가변성을 수용**
- 비즈니스 로직 구현을 도메인 모델 패턴으로 한 경우 적합

## Hexagonal Micro Service Architecture

<figure><img src="../../.gitbook/assets/micro-service/msa-hexagonal-architecture.png" alt=""><figcaption></figcaption></figure>

- 내부 영역과 외부 영역을 분리하여 구성
- 내부 영역 구성
  - **응용 서비스 클래스**(유스케이스 흐름 처리)
  - POJO 중심의 **도메인 모델**(비즈니스 로직)
  - 데이터 처리 I/F (**Repository**)
- 외부 영역 구성
  - 기술 가변성을 수용한 어댑터 클래스
  - 데이터 처리 구현
  - API 발행, 통신 구현
  - 이벤트 전송 및 리스너

## CQRS

<figure><img src="../../.gitbook/assets/micro-service/cqrs.png" alt=""><figcaption></figcaption></figure>

- Command Query Responsibility Segregation
  - Command Model: 비즈니스 로직 구현, 강력한 일관성
  - Query Model: 읽기 모델(인메모리 캐시, 일반 파일, materialized view, 읽기 전용)
- 모델별 저장소 동기화 필요: 동기(동시 쓰기)/비동기식(MessageQueue)
- 확장 가능, 읽기 성능 향상, 다양한 저장소 활용 가능한 장점

## Event-Driven Architecture

- 이벤트를 생산하는 모듈과 이벤트에 대응하는 모듈을 분리하고 상호 독립적으로 동작하여 병렬 처리를 촉진
- 발신자와 수신자를 장소와 시간에서 쉽게 분리 가능
- 장점
  - 느슨한 결합으로 확장성, 수정 가능성, 내결함성에 많은 이점을 제공
- 단점
  - 부주의하게 적용 시 모노리스를 분산된 커다란 진흙 덩어리로 만들 수 있음
  - 로컬 복잡성: 개별 마이크로서비스의 복잡성
  - 글로벌 복잡성: 전체 시스템의 복잡성, 서비스 간의 상호작용과 의존성
- 고려해야 할 장애 상황
  - 네트워크 장애, 서버 장애, 이벤트 순서 꼬임, 이벤트 중복 등
  - 데이터 소실 처리
    - 큐에 전달 안됨, 전달되어도 큐가 다운됨 -> 이벤트 저장소 활용
    - 메시지를 꺼내 처리 전에 장애 발생 -> 클라이언트 확인 응답 모드 활용
    - 데이터 에러로 인해 메시지 저장 실패 -> ACID 트랜잭션 적용

**비동기 관련 문제**
- 데이터 처리는 정상이지만 이벤트가 전달되지 않았을 경우 일관성을 맞추기 위해 적용할 수 있는 방법들

데이터 처리와 이벤트 발행을 하나의 트랜잭션으로

<figure><img src="../../.gitbook/assets/micro-service/async-problem.png" alt=""><figcaption></figcaption></figure>

Outbox Pattern 적용

> 서비스가 외부 시스템이나 마이크로서비스와 통신할 때 직접 메시지를 보내는 대신 
> Outbox라는 데이터베이스 테이블에 이벤트나 메시지를 저장하고, 별도의 프로세스가
> 이 Outbox 테이블을 폴링하여 저장된 메시지를 읽고 외부 시스템으로 전송하는 방식

- 아웃박스라는 저장소에 도메인 이벤트를 저장
- 메시지 릴레이가 아웃박스에 있는 이벤트를 발행

<figure><img src="../../.gitbook/assets/micro-service/outbox-pattern.png" alt=""><figcaption></figcaption></figure>

Saga Pattern 적용

> 마이크로서비스들끼리 이벤트를 주고 받아 특정 마이크로서비스에서의 작업이 실패하면
> 이전까지의 작업이 완료된 마이크서비스들에게 보상(complemetary) 이벤트를
> 소싱함으로써 분산 환경에서 원자성(atomicity)을 보장하는 패턴

- 여러 트랜잭션에 걸쳐 있는 비즈니스 프로세스 처리 문제 해결
- 분산 트랜잭션의 일관성 보장: 보상 트랜잭션

<figure><img src="../../.gitbook/assets/micro-service/saga-pattern.png" alt=""><figcaption></figcaption></figure>