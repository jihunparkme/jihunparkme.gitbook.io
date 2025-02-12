# CHAPTER 2. 코드 설계

# 3장. 재사용성

## Item 19. knowledge를 반복하여 사용하지 말라

{% hint style="info" %}

**knowledge**: 프로젝트 진행 시 정의한 모든 것

{% endhint %}

프로그램에서 중요한 `knowledge` 두 개를 뽑는다면
- 로직: 프로그램이 어떠한 식으로 동작하는지와 어떻게 보이는지
- 공통 알고리즘: 원하는 동작을 하기 위한 알고리즘

.

👉🏻 **모든 것은 변화한다**

- 변화는 우리가 예상하지 못한 곳에서 일어난다.
- 오늘날 대부분 프로젝트는 몇 달마다 요구 사항과 내부적인 구조를 계속해서 변경한다.
- 모든 것은 변화하고, 우리는 이에 대비해야 한다.

.

👉🏻 **반복해도 되는 코드**

- 두 코드가 같은 `knowledge`를 나타내는지, 다른 `knowledge`를 나타내는지를 
- "함께 변경될 가능성이 높은가? 따로 변경될 가능성이 높은가?"라는 질문으로 어느 정도 결정 가능

.

👉🏻 **단일 책임 원칙**

{% hint style="info" %}

코드를 추출해도 되는지를 확인할 수 있는 원칙으로, SOLID 원칙 중 하나

{% endhint %}

단일 책임 원칙은 우리에게 두 가지 사실을 알려준다.
- 서로 다른 곳에서 사용하는 knowledge는 독립적으로 변경할 가능성이 높다
  - 따라서, 비슷한 처리를 하더라도 완전히 다른 knowledge로 취급하는 것이 좋다.
- 다른 knowledge는 분리해 두는 것이 좋다.
  - 그렇지 않으면, 재사용해서는 안 되는 부분을 재사용하려는 유혹이 발생할 수 있다.

📖 **정리**

> 모든 것은 변화한다. 
>
> 따라서 공통 knowledge가 있다면, 이를 추출해서 변화에 대비하자.
>
> 여러 요소에 비슷한 부분이 있는 경우, 변경이 필요할 때 실수가 발생할 수 있다. 이런 부분은 추출하는 것이 좋다.
>
> 추가적으로 의도하지 않은 수정을 피하려면 또는 다른 곳에서 조작하는 부분이 있다면, 분리해서 사용하자.

## Item 20. 일반적인 알고리즘을 반복해서 구현하지 말라

👉🏻 **표준 라이브러리 살펴보기**
- 일반적인 알고리즘은 대부분 이미 다른 사람들이 정의해 놓았다.

.

👉🏻 **나만의 유틸리티 구현하기**
- 확장 함수의 장점
  - 함수는 상태를 유지하지 않으므로 행위를 나타내기 좋다(특히 사이드 이펙이 없는 경우 더욱)
  - 톱레벨 함수와 비교해서, 확장 함수는 구체적인 타입이 있는 객체에만 사용을 제한 가능
  - 수정할 객체를 아규먼트로 전달받아 사용하는 것보다 확장 리시버로 사용하는 것이 가독성 측면에서 좋음
  - 확장 함수는 객체에 정의한 함수보다 객체 사용 시, 자동 완성 기능 등으로 제안이 이루어지므로 쉡게 찾을 수 있음

📖 **정리**

> 일반적인 알고리즘 대부분 `stdlib`에 이미 정의되어 있을 가능성이 높다.
>
> 따라서 `stdlib`을 공부해 두면 좋다.
>
> stdlib에 없는 일반적인 알고리즘이 필요하거나, 특정 알고리즘을 반복해서 사용해야 하는 경우 프로젝트 내부에 직접 정의히자.
>
> 일반적으로 이런 알고리즘들은 `확장 함수`로 정의하자.

## Item 21. 일반적인 프로퍼티 패턴은 프로퍼티 위임으로 만들어라

코틀린은 코드 재사용과 관련해서 프로퍼티 위임이라는 새로운 기능을 제공
- 일반적인 프로퍼티의 행위를 추출해서 재사용 가능

대표적인 예로 지연 프로퍼티
- 코틀린에서는 stdlib `lazy` 프로퍼티 패턴으로 쉽게 구현

```kotlin
val value by lazy { createValue() }
```

.

프로퍼티 위임을 사용하면, 이외에도 변화가 있을 때 이를 감지하는 `observable` 패턴을 쉽게 생성 가능

```kotlin
var items: List<Item> by
    Delegates.observable(listOf()) { -, -, - ->
        notifyDataSetChanged()
    }
```

.

프로퍼티 위임은 다른 객체의 메서드를 활용해서 프로퍼티의 접근자(Getter, Setter)를 만드는 방식
- Getter ➔ getValue, Setter ➔ setValue 함수를 사용해서 생성

```kotlin
var token: String? by LoggingProperty(null)
var attempt: Int by LoggingProperty(0)

private class LoggingProperty<T>(var value: T) {
    operator fun getValue(
        thisRef: Any?,
        prop: KProperty<*>
    ): T {
        print("${prop.name} returned value $value")
        return value
    }

    operator fun setValue(
        thisRef: Any?,
        prop: KProperty<*>,
        newValue: T
    ) {
        val name = prop.name
        print("$name changed form $value to $newValue")
        value = newValue
    }
}
```

.

코틀린 stdlib에서 알아두면 좋은 프로퍼티 델리게이터
- lazy
- Delegates.observable
- Delegates.vetoable
- Delegates.notNull

📖 **정리**

> 프로퍼티 위임은 프로퍼티 패턴을 추출하는 일반적인 방법이라 많이 사용
>
> 코틀린 개발자라면 프로퍼티 위임이라는 강력한 도구와 관련된 내용을 잘 알아야 함
>
> 일반적인 패턴을 추출하거나 더 좋은 API 생성에 활용 가능









205