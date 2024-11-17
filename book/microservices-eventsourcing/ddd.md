# CHAPTER I. 도메인 주도 설계

> 소프트웨어에서 모델은 해결해야 하는 문제를 바라보는 다양한 관심사 중 하나를 선택해서 문제를 설명하는데 꼭 필요한 것만 표현해야 한다.

1️⃣ **도메인 로직 패턴**
- 소프트웨어에서 도메인 로직을 어디에 두는게 좋은가?

{% hint style="info" %}

**도메인 로직을 구현하는 패턴에 대한 자세한 설명**

[Catalog of Patterns of Enterprise Application Architecture](https://martinfowler.com/eaaCatalog/)
- 메인 주도 설계에서 하위 도메인 분류(핵심/지원/일반 하위 도메인)를 마이크로서비스로 구현할 때 어떤 패턴을 사용할지 결정하는데 도움

{% endhint %}

👉🏻 **트랜잭션 스크립트 패턴**

- 클라이언트가 요청한 비즈니스 로직을 하나의 프로시저가 모두 처리
- 비즈니스 애플리케이션이 제공하는 기능 대부분을 반복적인 CRUD로 처리할 수 있는 경우 적합
- 서블릿과 JSP가 대표적인 사례

👉🏻 **테이블 모듈 패턴**

- 테이블 모듈은 **데이터베이스 테이블 단위로 비즈니스 로직을 처리**하는 클래스를 분리

👉🏻 **서비스 레이어 패턴**

- 서비스 레이어로 부르는 독립된 클래스에 **시스템 통합**과 **전체 흐름을 조정**하는 책임을 부여
- 현대 소프트웨어에서 가장 많이 사용하는 패턴
- 서비스 레이어는 **비즈니스 로직**뿐 아니라 **로깅, 권한 체크** 등과 같은 **공통 기능**을 구현
- 서비스 레이어의 책임은 **여러 리소스간 통합**과 **통합 결과에 따른 흐름 조정**처럼 여러 책임을 가져 복잡도가 증가

👉🏻 **도메인 모델 패턴**

- 데이터와 행위를 하나의 객체로 설계
- 다양한 규칙과 논리의 복잡한 관계를 **여러 객체에 분산**시키고 객체간 협력으로 구현
- 서비스 레이어는 도메인 객체에 명령을 전달하기 위한 준비 흐름만 있고, 도메인 객체가 제한을 하는 규칙을 검사

```kotlin
class CartService {
    fun addItem(cartId: String, 
                productNo: String, 
                productName: String, 
                quantity: Int) {
        val foundCart = cartDao.select(cartId)
        foundCart.addItem(productNo, productName, quantity)
        cartDao.update(foundCart)
    }
}

...

class Cart {
    private val items: MutableList<Item> = mutableListOf()

    fun addItem(productNo: String, productName: String, quantity: Int) {
        if (items.size >= 10) {
            throw ItemLimitExceedException()
        }
        items.add(Item(cartId, productNo, productName, quantity))
    }
}
```

2️⃣ 핵사고날 아키텍처

- 기술과 관계없는 비즈니스 로직과 기술에 의존하는 구성 요소인 어댑터 간 분리를 강조
  - 기술에 의존하는 어댑터는 **외부 요청을 수신해 비즈니스 로직을 시작**시키는 `인바운드 어댑터`
  - 비즈니스 로직을 실행하면서 **영구 저장소에 데이터를 저장하거나 다른 시스템과 협력**하는 `아웃바운드 어댑터`

3️⃣