# CHAPTER 3. 이벤트 소싱 I

> 데이터의 상태 변화 과정을 기록해야 하는 요구사항에서 이벤트 소싱을 적용할 수 있다.

## 감사와 이력

> 변경 사항을 기록하기 위해 이전 상태를 분리해서 기록해 둔 후 필요할 때 현제 상태와 비교하기 위한 방법

### 단일 테이블과 시퀀스

첫 번째 방법으로 TB_EMPLOYEE 테이블 식별자인 EMP_NO 외에 일련변호를 추가하고, 변경해야 할 때 기존 레코드를 유지하고 일련번호를 증가시켜 새로운 레코드를 추가하기.
- 일련번호 대신 시간으로 사용 가능

<figure><img src="../../.gitbook/assets/microservices-eventsourcing/t3-3.png" alt=""><figcaption></figcaption></figure>

1. 현재 상태의 마지막 일련번호 조회
2. 일련번호 값 증가
3. 파라미터 employee에 새로운 일련번호 할당
4. employeeDao를 사용해 최종 상태를 새로운 레코드로 추가

### 상태 테이블과 이력 테이블

두 번째 방법으로 현재 상태를 가진 TB_EMPLOYEE 테이블과 이력을 기록하는 TB_EMPLOYEE_HISTORY 테이블로 분리
- 기록 테이블은 현재 상태를 가진 테이블과 동일한 컬럼을 구성하고
- 단일 테이블-시퀀스 방법과 동일하게 상태 변화 순서를 구별하기 위해 일련번호를 추가

<figure><img src="../../.gitbook/assets/microservices-eventsourcing/t3-4-5.png" alt=""><figcaption></figcaption></figure>

1. 직원 정보와 직원 정보 이력에서 일련번호 조회
2. 이력 일련번호 증가
3. 저장 단계에서 변경 이전 직원 상태를 이력 테이블에 추가
4. 직원의 새로운 상태를 테이블에 저장

### 변경 값

세 번째 방법은 변경한 속성 이름과 값의 목록만 기록하기
- 전체 복사본이 아닌 변경한 속성만 선택해 기록하면 변경 속성을 찾기 위해 모든 속성의 값을 비교할 필요도 없고 데이터베이스도 효과적으로 사용 가능

<figure><img src="../../.gitbook/assets/microservices-eventsourcing/t3-7.png" alt=""><figcaption></figcaption></figure>

1. 서비스는 애그리게이트가 제공하는 메소드를 호출
2. 영향을 받은 속성 목록을 반환
3. 리포지토리가 제공하는 삽입 메소드에 전달

## 도메인 이벤트

> 도메인 주도 설계는 '도메인 이벤트'를 강조

변경 값을 기록하는 것은 같지만 변경의 단위를 비즈니스 처리 과정에서 발생한 결과로 정의
- 이벤트는 사용자가 무엇인가 처리하도록 시스템에 요청한 것임을 알 수 있는 힌트이면서 변경이 발생한 이유
- 시스템이 관리하는 정보의 변화를 도메인 이벤트로 기록하면 도메인 모델을 더욱 명확하게 표현 가능
- `Map<String, String>`과 같은 일반적인 객체 대신 `PasswordChanged`라는 이벤트 클래스로 정의

이벤트 클래스를 사용하면 클래스 이름만으로 도메인에서 어떤 일이 일어났는지 즉시 이해할 수 있다.

## 이벤트 소싱

> 이벤트 소싱은 도메인에서 발생하는 이벤트를 시스템의 상태 변화로 간주
>
> 불변식 유지 단위인 애그리게이트에서 발생한 모든 이벤트를 데이터베이스에 기록

<figure><img src="../../.gitbook/assets/microservices-eventsourcing/3-1.png" alt=""><figcaption></figcaption></figure>

애그리게이트를 저장하는 리포지토리와 애그리게이트에서 발생한 이벤트를 데이터베이스에 저장하는 것 또한 원자적이어야 하고, 일련의 흐름을 하나의 트랜잭션으로 처리해야 한다.

## 이벤트 소싱 구현

### 데이터 모델

> 이벤트 소싱은 애그리게이트의 속성을 컬럼으로 관리하지 않고, 애그리게이트에서 발생한 도메인 이벤트만 기록

데이터 모델
- 애그리게이트 식별: TB_CART
- 애그리게이트에서 발생한 도메인 이벤트를 저장: TB_CART_EVENT

이벤트는 JSON으로 직렬화해 PAYLOAD 컬럼에 저장
- 현 상태를 복원하는 이벤트 리플레이는 순서가 중요하므로 도메인 이벤트가 발생한 시간을 사용

### 애그리게이트와 이벤트 저장

도메인 이벤트는 과거에 발생한 사건으로 변경할 수 없는 객체여야 하므로 setter는 미제공
- 따라서, 도메인 이벤트 인스턴스를 생성하려면 **생성자의 파라미터로 모든 속성을 전달해야 하고 식별자도 생성자에서 할당해야 한다.**

Cart 애그리게이트가 제공하는 메소드를 호출하면 실행 결과인 도메인 이벤트 인스턴스를 생성하고, 이벤트 저장소인 TB_CART_EVENT 테이블에 저장하기 전까지 Cart 애그리게이트가 임시로 보관
- 메소드 실행에 오류가 없으면 요청에 대응하는 도메인 이벤트를 생성해 event 변수에 추가

**도메인 이벤트를 일반화시킨 Event 클래스**

```kotlin
abstract class Event {
    private val eventId: String = UUID.randomUUID().toString()
    private val time: Long = System.currentTimeMillis()
    private var cartId: String? = null

    fun getPayload(): String {
        return JsonUtil.toJson(this)
    }
}
```

**이벤트 추상 클래스와 상속**

```kotlin
class ItemAdded : Event()
class QuantityChanged : Event()
class ItemRemoved : Event()
```

Cart 애그리게이트가 네 번의 사용자 요청을 처리했을 때 TB_CART_EVENT 테이블에 기록한 도메인 이벤트 예시

<figure><img src="../../.gitbook/assets/microservices-eventsourcing/t3-9.png" alt=""><figcaption></figcaption></figure>

### 커맨드와 이벤트

> 커맨드: 소프트웨어가 어떤 일을 수행하게 하는 의도
>
> 이벤트: 수행한 결과

커맨드와 이벤트 모두 기술적으로 정보를 전달하는 목적을 가진 메시지이지만, 커맨드는 일반적으로 동기로 처리하고 이벤트는 비동기로 처리하는 차이

|커맨드|이벤트|
|---|---|
|행위를 실행하는 방법|이미 발생한 사실에 관한 설명|
|사이드 이펙트가 있는 작업|발생한 것이므로 과거형으로 네이밍|
|시스템 상태를 변경하는 의도를 가진 메시지|이벤트는 시스템 내에서 발생한 것|
|사용자 또는 시스템의 다른 부분에서 생성|메소드가 반환한 것이 아닌 명령을 실행한 결과|

커맨드는 도메인에서 식별한 동사를 주로 사용
- 카트와 관련된 동사로 카트 생성, 상품 추가, 상품 삭제, 수량 변경 등
- createCart, addItem, removeItem, changeQuantity..

이벤트는 단순히 커맨드에 의해 발생한 사건이므로 커맨드를 과거형으로 사용
- cartCreated, ItemAdded, ItemRemoved, QuantityCahrged

반대로 이벤트에서 시작하는 접근법으로 이벤트 스토밍이 있다.

.

비즈니스 프로세스는 식별한 이벤트와 연관된 커맨드 그리고 파생되는 이벤트까지 고려해야 한다.
- ex. 사용자 등록 UserRegistered or UserCreated 이벤트를 식별
- 위 이벤트에 반응해 가입 환영 이메일을 발송하는 sendWelcomMail 커맨드와 WelcomeMailSend 이벤트를 추가로 식별

{% hint style="info" %}

이벤트 소싱은 생각보다 많은 커맨드와 이벤트를 선언해야 하고

커맨드와 이벤트에 속성을 중복으로 선언해야 하는 단점이 존재

{% endhint %}

변경에 따른 영향도를 낮추는 것뿐만 아니라 설계 의도를 명확하게 표현하기 위해 `커맨드 객체`를 사용하는 것이 좋다.
- 커맨드 클래스를 사용하면 응집도, 결합도, 유지보수에 도움이 되므로 파라미터 목록을 대신할 커맨드 클래스를 사용하자

<figure><img src="../../.gitbook/assets/microservices-eventsourcing/3-5.png" alt=""><figcaption></figcaption></figure>

**command class**

```kotlin
abstract class Command {
    private var version: Long = 0
}

...

data class AddItem(
    @Transient var cartId: String? = null,
    var productNo: String,
    var productName: String,
    var price: Int,
    var quantity: Int = 1
) : Command()

...

data class ChangeQuantity(
    @Transient var cartId: String? = null,
    var productNo: String? = null,
    var quantity: Int = 0
) : Command()

...

data class RemoveItem(
    @Transient var cartId: String? = null,
    var productNo: String? = null
) : Command()
```

### 커맨드와 유효성 검사

유효성 검사를 기술이 아닌 도메인 영역으로 정의하면 커맨드에서 유효성을 검사해 응집도를 높일 수 있다.

### 재수화(이벤트 리플레이)

애그리게이트 상태의 변화 기록인 도메인 이벤트를 데이터베이스에 빠짐없이 기록했으면, 이벤트를 리플레이해 애그리게이트의 현재 상태로 복원 가능
- 도메인 이벤트로 상태를 복원하는 것을 `재수화`(Rehydration)

시스템은 쓰기보다 읽기 빈도가 훨씬 높은데 이벤트 소싱을 적용하면 다양한 조건으로 데이터를 조회할 때 한계가 있다.
- 조회 성능과 개발 편의성을 해소하기 위해 도메인 이벤트에서 조회 전용 데이터를 미리 만드는 것을 `프로젝션`

`load` 메소드는 애그리게이트인 Cart 식별자를 사용하고 다섯 스텝으로 현재 상태로 복원
1. 애그리게이트와 매핑한 **CartJpo를 조회**
2. 도메인 이벤트와 매핑한 **CartEventJpo 목록을 발생한 시간순으로 조회**
3. CartJpo를 Cart **애그리게이트로 변환**
4. 조회한 CartEventJpo를 **도메인 이벤트로 변환**
5. Cart 애그리게이트가 제공하는 **재수화 메소드를 호출해 이벤트를 리플레이**

CartEventJpo를 도메인 이벤트로 변환하기 위해
- 도메인 이벤트를 저장할 때 역직렬화할 타입명을 가지는 TYPE 컬럼이 필요

```kotlin
@Entity
@Table(name = "TB_CART_EVENT")
data class CartEventJpo(
    @Id
    var eventId: String? = null,
    var cartId: String? = null,
    @Lob
    var payload: String? = null,
    var eventType: String? = null,
    var correlationId: String? = null,
    var sequence: Long = 0,
    var relayed: Boolean = false,
    var propagate: Boolean = false,
    var deleted: Boolean = false
) {
    constructor(cart: Cart, event: Event) : this() {
        this.eventId = event.eventId()
        this.cartId = cart.getCartId()
        this.payload = event.payload()
        this.eventType = event.typeName()
        this.correlationId = event.correlationId()
        this.sequence = event.sequence()
        this.relayed = event.relayed()
        this.propagate = event.propagate()
        this.deleted = false
    }

    fun toEvent(): Event {
        try {
            // 도메인 이벤트를 CartEventJpo로 변환하는 생성자에서 이벤트의 타입명을 복사해 이벤트 저장소에 함께 저장
            val event = JsonUtil.fromJson(this.payload, Class.forName(this.eventType)) as Event
            event.eventId(this.eventId)
            event.sequence(this.sequence)
            event.relayed(this.relayed)
            event.propagate(this.propagate)
            event.correlationId(this.correlationId)
            return event
        } catch (e: ClassNotFoundException) {
            e.printStackTrace()
        }
        throw IllegalStateException()
    }
}
```

애그리게이트가 네 번의 커맨드를 처리하고 TB_CART_EVENT 테이블에 기록한 도메인 이벤트

<figure><img src="../../.gitbook/assets/microservices-eventsourcing/t3-11.png" alt=""><figcaption></figcaption></figure>

애그리게이트에 커맨드 처리와 도메인 이벤트 리플레이 메소드를 분리하면 리플레이에 사용하는 도메인 이벤트는 상태 복원에만 적용하고 event 속성에 저장하지 않게 할 수 있다.

## 마이크로서비스 모듈

> 이벤트 소싱을 적용한 마이크로서비스의 핵심 모듈은 `aggregate`, `command`, `event`, `store` 패키지로 구분

<figure><img src="../../.gitbook/assets/microservices-eventsourcing/3-6.png" alt=""><figcaption></figcaption></figure>

package
- `aggregate`: 식별한 애그리게이트 클래스
- `command`: 애그리게이트를 생성하고 폐기할 때까지 영향을 주는 커맨드
- `event`: 커맨드에 영향을 받은 결과
- `store`: 이벤트 저장소, 애그리게이트, 이벤트 테이블과 매핑한 JPA 엔티티 클래스
- `service`: 전체 흐름을 조정하는 책임을 가지는 애플리케이션 서비스

## 이벤트 소싱과 단위 테스트

> 주요 비즈니스 로직은 여전히 도메인 객체에 있으므로 도메인 객체만 사용하는 단위 테스트를 구현 가능

- 하나의 커맨드가 여러 개의 도메인 이벤트를 발생시키는 경우 단위 테스트의 통과 조건으로 이벤트 개수를 검사
- 시스템 전체 구조가 아닌 도메인 모델만 이해하면 테스트 코드를 작성할 수 있으므로 독립적이면서 실용적

## 요약

{% hint style="info" %}

- 전통적인 방법으로 단일 테이블에 시퀀스를 추가해 이전 상태와 현재 상태를 기록 가능
- 데이터 관점에서 현재와 이력을 구분하는 **두 개의 테이블로 이전 상태와 현재 상태를 기록**
- 테이블 레코드 단위가 아닌 **변경된 값만 직렬화**해 변경을 기록
- 시스템에 변경을 요청하는 커맨드와 요청의 처리 결과인 이벤트
- 이벤트 소싱에서 커맨드와 이벤트를 처리하는 메소드를 각각 커맨드 핸들러, 이벤트 핸들러라고 한다.
- 변경된 값에 의미를 부여해 설계 의도를 명확하게 드러내는 도메인 이벤트를 소개
  - 도메인 이벤트로 변경 이력을 기록
- 이벤트 소싱에서 재수화와 프로젝션을 구분
- 도메인 이벤트로 변경을 기록하고 리플레이해 현재의 상태를 복원하는 원리를 알아보고 구현
- 이벤트 소싱은 자연스럽게 애그리게이트에 비즈니스 로직을 구현하는 효과가 있어 좀 더 독립적이면서 실용적인 단위 테스트가 가능

{% endhint %}

---

# CHAPTER 4. 이벤트 소싱 II

## 도메인 객체 라이프사이클

**애그리게이트 생성**

애그리게이트를 구성하는 엔티티와 값 객체는 모두 파라미터를 가지는 메소드를 제공하고 상태가 변할 때 이벤트를 발생
- 애그리게이트 생성도 커맨드를 이용하고 도메인 이벤트를 발생시켜야 한다.

**애그리게이트 삭제**

데이터 관점에서 이벤트 소싱은 TB_CART, TB_CART_EVENT 테이블의 레코드를 DELETE 구문으로 직접 삭제하지 않고 삭제되었다고 표기하는 `소프트 삭제`로 이력(deleted)을 유지

## 동시성과 이벤트 충돌

동시에 수정해 같은 속성을 변경할 때 나중에 처리된 요청이 이전 요청을 덮어쓰는 현상
- 데이터베이스에서는 이 현상을 갱신 분실(Lost Update)이라고 한다.
- 이벤트 소싱에서 이와 같은 현상을 이벤트 충돌(Event Conflicts)라고 한다.

이벤트간 충돌을 해결하려면 먼저 동시성 메커니즘을 이해해야 한다.
- 동시성 메커니즘은 비관적 잠금(Pessimistic Lock)과 낙관적 잠금(Optimistic Lock)으로 구분

**비관적 잠금(Pessimistic Lock)**

<figure><img src="../../.gitbook/assets/microservices-eventsourcing/4-3.png" alt=""><figcaption></figcaption></figure>

비관적 잠금에서 트랜잭션은 많은 충돌이 발생한다고 가정하고 데이터베이스가 제공하는 잠금 기능을 사용
- 어떤 프로세스가 데이터 변경을 목적으로 조회를 요청하고 변경을 완료할 때까지 다른 프로세스가 해당 데이터에 접근하지 못함
- 안정적이지만 낙관적 잠금에 비해 비즈니스 동시성 환경에 비적합

**낙관적 잠금(Optimistic Lock)**

<figure><img src="../../.gitbook/assets/microservices-eventsourcing/4-4.png" alt=""><figcaption></figcaption></figure>

낙관적 잠금에서 트랜잭션은 충돌이 발생하지 않는다고 가정하고 데이터를 변경하는 시점에만 잠금을 사용
- 버전 컬럼을 비교해 데이터 조회 후 업데이트를 요청하는 사이에 변경이 있었는지 확인
- 버전 컬럼의 값이 같으면 변경하고 그렇지 않으면 예외를 던져 데이터의 일관성을 유지
- 비관적 잠금에 비해 높은 성능을 제공해 대부분의 비즈니스 서비스는 낙관적 잠금을 사용




## 재수화 성능과 스냅샷

## 스냅샷 생성 전략

## 이벤트 소싱과 상수

## 도메인 이벤트와 버전

## 마이크로서비스 모듈

## 다건 처리와 성능

## 데이터 마이그레이션

## 백업과 아카이빙

## 이벤트 소싱과 추상화된 핵심

## 요약