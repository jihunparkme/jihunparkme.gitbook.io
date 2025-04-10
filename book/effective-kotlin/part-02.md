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
        println("${prop.name} returned value $value")
        return value
    }

    operator fun setValue(
        thisRef: Any?,
        prop: KProperty<*>,
        newValue: T
    ) {
        val name = prop.name
        println("$name changed form $value to $newValue")
        value = newValue
    }
}

@Test
fun loggingProperty() {
    token = "abe" // token changed form null to abe
    attempt = 10 // attempt changed form 0 to 10
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

## Item 22. 일반적인 알고리즘을 구현할 때 제네릭을 사용하라

> 타입 아규먼트(타입 파라미터)를 사용하는 함수를 `제네릭 함수`라고 부른다.

타입 파라미터는 컴파일러에 타입과 관련된 정보를 제공하여 컴파일러가 타입을 조금이라도 더 정확하게 추측할 수 있도록 지원

```kotlin
public inline fun <T> Iterable<T>.filter(
    predicate: (T) -> Boolean
): List<T> {
    return filterTo(ArrayList<T>(), predicate)
}

public inline fun <T, C : MutableCollection<in T>> Iterable<T>.filterTo(
    destination: C, 
    predicate: (T) -> Boolean
): C {
    for (element in this) if (predicate(element)) destination.add(element)
    return destination
}
```

.

👉🏻 **제네릭 제한**

타입 파라미터의 중요한 기능 중 하나는 구체적인 타입의 서브타입만 사용하게 타입을 제한하는 것
- 타입 제한이 걸리므로 내부에서 해당 타입이 제공하는 메서드를 사용할 수 있다.

```kotlin
fun <T : Comparable<T>> Iterable<T>.sorted(): List<T> {
    //
}

fun <T, C : MutableCollection<in T>>
Iterable<T>.toCollection(destination: C): C {
    //
}

class ListAdapter<T: ItemAdapter>( /* ... */ ) {
    //
}
```

많이 사용하는 제한으로는 `Any`가 있는데, `nullable`이 아닌 타입을 나타낸다.

```kotlin
public inline fun <T, R : Any> Iterable<T>.mapNotNull(
    transform: (T) -> R?
): List<R> {
    return mapNotNullTo(ArrayList<R>(), transform)
}
```

📖 **정리**

> 코틀린 자료형 시스템에서 타입 파라미터는 굉장히 중요한 부분
>
> 일반적으로 이를 사용해서 type-safe 제네릭 알고리즘과 제네릭 객체를 구현
>
> 타입 파라미터는 구체 자료형의 서브타입을 제한 가능
>
> 이렇게 하면 특정 자료형이 제공하는 메서드를 안전하게 사용 가능

## Item 23. 타입 파라미터의 섀도잉을 피하라

프로퍼티와 파라미터가 같은 이름을 가질 수 있다.
- 이렇게 되면 지역 파라미터가 외부 스코프에 있는 프로퍼티를 가린다.
- 이를 `섀도잉`이라고 부른다.

```kotlin
class Forest(val name: String) {
    
    fun addTree(name: String) {
        //...
    }
}
```

이 경우, addTree 함수가 클래스 타입 파라미터인 T를 사용하는 것이 좋다.

```kotlin
class Forest(T: Tree) {
    
    fun addTree(tree: T) {
        //...
    }
}

val forest = Forest<Birch>()
forest.addTree(Birch())
forest.addTree(Spruce()) // type mismatch ERROR
```
📖 **정리**

> 타입 파라미터 섀도잉이 발생한 코드는 이해하기 어려울 수 있다.
>
> 타입 파라미터가 섀도잉되는 경우 코드를 주의해서 살펴보자.

## Item 24. 제네릭 타입과 variance 한정자를 활용하라

`invariant`는 제네릭 타입으로 만들어지는 타입들이 서로 관련성이 없다는 의미
- 만일 어떤 관련성을 원한다면, `out` 또는 `in` 이라는 variance 한정자를 붙인다.

`out`은 타입 파라미터를 `covariant`(공변성)으로 만든다.
- ex. A가 B의 서브 타입일 때, Cup\<A\>가 Cup\<B\>의 서브타입

`in` 한정자는 타입 파라미터를 `contravariant`(반변성)으로 만든다.
- ex. A가 B의 서브 타입일 때, Cup\<A\>가 Cup\<B\>의 슈퍼타입

![Result](https://github.com/jihunparkme/jihunparkme.gitbook.io/blob/main/.gitbook/assets/kotlin/variance.png?raw=true 'Result')

📖 **정리**

코틀린은 타입 아규먼트의 관계에 제약을 걸 수 있는 강력한 제네릭 기능을 제공

> 코틀린에서 제공하는 타입 한정자
>
> - 타입 파라미터의 기본적인 `variance` 동작은 `invariant`
>   - 만일 Cup\<T\>라면, 타입 파라미터 T는 invariant
>   - A가 B의 서브타입이라고 할 때 Cup\<A\>와 Cup\<B\>는 아무 관계를 갖지 않음
> - `out` 한정자는 타입 파라미터를 `covariant`하게 만듦
>   - 만일 Cup\<T\>라면, 타입 파라미터 T는 covariant
>   - A가 B의 서브타입이라고 할 때 Cup\<A\>는 Cup\<B\>의 서브타입
>   - covariant 타입은 out 위치에 사용 가능
> - `in` 한정자는  타입 파라미터를 `contravariant`하게 만듦
>   - 만일 Cup\<T\>라면, 타입 파라미터 T는 contravariant
>   - A가 B의 서브타입이라고 할 때 Cup\<B\>는 Cup\<A\>의 슈퍼타입
>   - contravariant 타입은 in 위치에 사용 가능
>
> 코틀린에서는
>
> - List, Set 타입 파라미터는 `covariant`(out 한정자)
>   - Map 값 타입을 나타내는 타입 파라미터도 covariant(out 한정자)
>   - 단, Mutable.. 타입 파라미터는 `invariant`
> - 함수 타입의 파라미터 타입은 `contravariant`(in 한정자)
>   - 리턴 타입은 `contravariant`(out 한정자)
> - 리턴만 되는 타입에는 `covariant`(out 한정자) 사용
> - 허용만 되는 타입에는 `contravariant`(in 한정자) 사용

## Item 25. 공통 모듈을 추출해서 여러 플랫폼에서 재사용하라

공통 모듈을 사용하면 여러 플랫폼에서 코드를 재사용할 수 있다.
- 따라서 이러한 접근 방법은 코드를 한 번만 작성해서 공통 논리 또는 공통 알고리즘을 재사용할 수 있게 해주는 강력한 도구

# 4장. 추상화 설계

> 컴퓨터 과학에서 추상화(abstraction)은 복잡한 자료, 모듈, 시스템 등으로부터 핵심적인 개념 또는 기능을 간추려 내는 것을 말한다.

추상화는 복잡성을 숨기기 위해 사용되는 단순한 형식을 의미
- 대표적인 예로 인터페이스

프로그래밍에서 추상화를 사용하는 목적
- 복잡성을 숨기기 위해
- 코드를 체계화하기 위해
- 만드는 사람에게 변화의 자유를 주기 위해

## Item 26. 함수 내부의 추상화 레벨을 통일하라

간단한 추상화를 추출해서 가독성을 향상
- 함수는 간단해야 한다.
- 함수는 작아야 하며, 최소한의 책임만을 가져야 한다.

함수를 추출하면 재사용과 테스트가 쉬워짐

📖 **정리**

> 함수, 클래, 모듈 등의 다양한 방식을 통해 추상화를 분리
> 
> 각각의 레이어가 너무 커지는 것은 좋지 않다.
> 
> 작고 최소한의 책임만 갖는 함수가 이해하기 쉽다.

## Item 27. 변화로부터 코드를 보호하려면 추상화를 사용하라

상수로 추출할 때의 장점
- 이름을 붙일 수 있고,
- 나중에 해당 값을 쉽게 변경 가능

.

👉🏻 **함수**

많이 사용되는 알고리즘은 간단한 확장 함수로 사용 가능

```kotlin
fun Context.toast(
    message: String,
    duration: Int = Toast.LENGTH_LONG
) {
    Toast.makeText(this, message, duration).show()
}
```

메시지를 출력하는 더 추상적인 방법

```kotlin
fun Context.showMessage(
    message: String,
    duration: MessageLength = MessageLength.LONG
) {
    val toastDuration = when(duraion) {
        SHORT -> Length.LENGTH_SHORT
        LONG -> Length.LENGTH_LONG
    }
    Toask.makeText(this, message, toastDuration).show()
}

enum class MessageLength { SHORT, LONG }
```

.

👉🏻 **클래스**

구현을 추상화할 수 있는 더 걍력한 방법은 클래스
- 클래스가 함수보다 더 강력한 이유는 `상태`를 가질 수 있으며, `많은 함수`를 가질 수 있다는 점 때문

클래스 생성을 위임할 수 있음

```kotlin
@Inject lateinit var messageDisplay: messageDisplay
```

mock 객체를 활용해서 해당 클래스에 의존하는 다른 클래스의 기능을 태스트 가능

```kotlin
val messageDisplay: MessageDisplay = mockk()
```

메시지를 출력하는 더 다양한 종류의 메서드를 만들 수 있음

```kotlin
messageDisplay.setChristmasMode(true)
```

.

👉🏻 **인터페이스**

거의 모든 것이 인터페이스로 표현
- `listOf` 함수는 `List`를 리턴
- 컬렉션 처리 함수는 `Iterable` 또는 `Collection` 확장 함수로서 `List`, `Map` 등을 리턴
- 프로퍼티 위임은 `ReadOnlyProperty` 또는 `ReadWriteProperty` 뒤에 숨겨짐

인터페이스 뒤에 객체를 숨김으로써 실질적인 구현을 추상화하고, 사용자가 추상화된 것에만 의존하게 만들 수 있음
- 즉, 결합(coupling)을 줄일 수 있음

인터페이스 도입

```kotlin
interface MessageDisplay {
    fun show(
        message: String,
        duraion: MessageLength = LONG
    )
}

class ToastDisplay(val context: Context): MessageDisplay {
    override fun show(
        message: String,
        duraion: MessageLength
    ) {
        val toastDuration = when(duraion) {
            SHORT -> Length.SHORT
            LONG -> Length.LONG
        }
        Toast.makeText(context, message, toastDuration).show()
    }
}

enum class MessageLength { SHORT, LONG }
```

테스트할 때 인터페이스 페이킹이 클래스 모킹보다 간단하므로, 별도의 모킹 라이브러리를 사용하지 않아도 된다.
- 선언과 사용이 분리되어 있으므로, 실제 클래스를 자유롭게 변경 가능

.

👉🏻 **ID 만들기(nextId)**

ID 타입을 쉽게 변경할 수 있게 클래스를 사용하는 것이 좋음

```kotlin
data class Id(private val id: Int)

private var nextId: Int = 0
fun getNextId(): Id = Id(next++)
```

더 많은 추상화는 더 많은 자유를 주지만, 이를 정의하고, 사용하고, 이해하는 것은 어려워졌다.

.

👉🏻 **추상화가 주는 자유**

추상화를 하는 몇 가지 방법
- 상수로 추출
- 동작을 함수로 래핑
- 함수를 클래스로 래핑
- 인터페이스 뒤에 클래스 숨기기
- 보편적인 객체(universal object)를 특수한 객체(specialistic object)로 래핑

구현 시 활용가능한 여러 도구
- 제네릭 타입 파라미터 사용
- 내부 클래스를 추출
- 생성을 제한(ex. 팩토리 함수로만 객체를 생성할 수 있게 만들기)

.

👉🏻 **추상화의 문제**

추상화에는 단점도 존재
- 추상화는 자유를 주지만, 코드를 이해하고 수정하기 어렵게 만든다.

어떤 방식으로 추상화를 하려면 코드를 읽는 사람이 해당 개념을 배우고, 잘 이해해야 한다.
- 다른 방식으로 추상화를 하려면 또 해당 개념을 배우고, 잘 이해해야 한다.

추상화는 많은 것을 숨길 수 있는 테크닉
- 생각할 것을 어느 정도 숨겨야 개발이 쉬워지는 것도 사실이지만, 너무 많은 것을 숨기면 결과를 이해하는 자체가 어려워짐

추상화를 사용할 수 있는 규칙
- 많은 개발자가 참여하는 프로젝트는 이후 객체 생성과 사용 방법을 변경하기 어려움
  - 따라서 추상화 방법을 사용하는 것이 좋고, 최대한 모듈과 부분을 분리하자
- 의존성 주입 프레임워크를 사용하면 생성이 얼마나 복잡한시는 신경 쓰기 않아도 됨
- 테스트를 하거나 다른 애플리케이션을 기반으로 새로운 애플리케이션을 만든다면 추상화를 사용하는 것이 좋음
- 프로젝트가 작고 실험적이라면, 추상화를 하지 않고도 직접 변경해도 괜찮음

📖 **정리**

> 추상화를 사용하는 것은 굉장히 어렵지만 이를 배우고 이해해야 한다.
>
> 다만 추상적인 구조를 사용하면 결과를 이해하기 어렵다.
>
> 추상화를 사용할 때의 장점과 단점을 모두 이해하고, 프로젝트 내에서 그 균현을 찾아야 한다.
>
> 추상화가 너무 많거나 너무 적은 상황 모두 좋은 상황은 아니다.

## Item 28. API 안정성을 확인하라

`Experimental` 메타 어노테이션을 사용해서 사용자들에게 아직 해당 요소가 안정적이지 않다는 것을 알려 주는 것이 좋다.

```kotlin
@Experimental(level = Experimental.Level.WARNING)
annotation class ExperimentalNewApi

@ExperimentalNewApi
suspend fun getUsers(): List<User> {
    //...
}
```

API의 일부를 변경해야 한다면, 전환하는 데 시간을 두고 `Deprecated` 어노테이션을 활용해서 사용자에게 미리 알려 줘야 한다.

```kotlin
@Deprecated("Use suspending getUsers instead")
fun getUsers(callback: (List<User>)->Unit) {
    //...
}
```

직접적인 대안이 있다면, IDE가 자동 전환을 할 수 있게 `ReplaceWith`를 붙여 주는 것도 좋다.

```kotlin
@Deprecated("Use suspending getUsers instead", ReplaceWith("getUsers()"))
fun getUsers(callback: (List<User>)->Unit) {
    //...
}
```

📖 **정리**

> 모듈과 라이브러리를 만드는 사람과 이를 사용하는 사람들 사이에 커뮤니케이션이 중요
>
> 커뮤니케이션은 버전 이름, 문서, 어노테이션 등을 통해 확인 가능
>
> 또한, 안정적인 API에 변경을 가할 때는 사용자가 적응할 충분한 시간이 필요

## Item 29. 외부 API를 wrap해서 사용하라

잠재적으로 불안정하다고 판된되는 외부 라이브러리 API를 Wrap해서 사용하자.

장점 (자유와 안정성을 얻을 수 있음)
- 문제가 있다면 래퍼만 변경하면 되므로, API 변경에 쉽게 대응
- 프로젝트의 스타일에 맞춰 API 형태 조정 가능
- 특정 라이브러리에서 문제 발새애 시, 래퍼를 수정해서 다른 라이브러리를 사용하도록 코드를 쉽게 변경 가능성
- 필요 시 쉽게 동작을 추가하거나 수정 가능

단점
- 래퍼를 따로 정의
- 다른 개발자가 프로젝트를 다룰 때, 어떤 래퍼들이 있는지 따로 확인 필요
- 래퍼들은 프로젝트 내부에서만 유효하므로, 문제가 생겨도 질문 불가

라이브러리가 얼마나 안정적인지 확인할 수 있는 가장 기본적인 휴리스틱은 버전 번호와 사용자 수

## Item 30. 요소의 가시성을 최소화하라

작은 인터페이스는 배우기 쉽고 유지하기 쉽다
- 기능이 많은 클래스보다 적은 클래스를 이해하는 것이 쉬움
- 유지보수와 테스트가 쉽고 적어짐

세터만 private으로 만드는 코드는 굉장히 많이 사용
- 일반적으로 코틀린에서는 구체 접근자의 가시성을 제한해서 모든 프로퍼티를 캡슐화하는 것이 좋다.

```kotlin
// ...
var elementsAdded: Int = 0
    private set
```

가시성이 제한될수록 클래스의 변경을 쉽게 추척 가능하며, 프로퍼티의 상태를 더 쉽게 이해 가능
- 병렬 프로그래밍 시 안전

.

👉🏻 **가시성 한정자 사용하기**

클래스 멤버의 경우 4개의 가시성 한정자 사용 가능
- public: 어디서나 볼 수 있음(default)
- private: 클래스 내부에서만 볼 수 있음
- protected: 클래스와 서브클래스 내부에서만 볼 수 있음
- internal: 모듈 내부에서만 볼 수 있음

톱레벨 요소에는 세 가지 가시성 한정자 사용 가능
- public: 어디서나 볼 수 있음(default)
- private: 클래스 내부에서만 볼 수 있음
- protected: 클래스와 서브클래스 내부에서만 볼 수 있음

코틀린에서 `모듈`이란 함께 컴파일되는 코틀린 소스를 의미
- Gradle 소스 세트
- Maven 프로젝트
- IntellijJ IDEA 모듈
- Ant 태스크 한 번으로 컴파일되는 파일 세트

📖 **정리**

> 요소의 가시성은 최대한 제한적인 것이 좋다.
>
> 보이는 요소들은 모두 public API로서 사용되며, 아래 이유로 최대한 단순한 것이 좋다.
>
> - 인터페이스가 작을수록 이를 공뷰하고 유지하는 것이 쉬움
> - 최대한 제한이 되어 있어야 변경이 쉬움
> - 클래스의 상태를 나타내는 프로퍼티가 노출되어 있다면, 클래스가 자신의 상태를 책임질 수 없음
> - 가시성이 제한되면 API의 변경을 쉽게 추적 가능

## Item 31. 문서로 규약을 정의하라

함수가 무엇을 하는지 명확하게 설명하고 싶다면, KDoc 주석을 붙이자

.

👉🏻 **규약**
- 어떤 행위를 설명하면 사용자는 이를 일종의 약속으로 취급하며, 이를 기반으로 스스로 자유롭게 생각하던 예측을 조정
- 이처럼 예측되는 행위를 요소의 규약(contract of an element)이라고 함
- 규약을 정의하는 것은 양쪽 모두에게 좋은 일

규약을 설정하지 않는다면
- 클래스를 사용하는 사람은 스스로 할 수 있는 것과 할 수 없는 것을 모르므로, 구현의 세부적인 정보에 의존
- 클래스를 만든 사람은 사용자가 무엇을 할지 알 수 없으므로 사용자의 구현을 망칠 위험이 있음

.

👉🏻 **규약 정의하기**

규약 정의하기
- 이름
- 주석과 문서
- 타입

.

👉🏻 **주석을 써야 할까?**
- 주석을 함께 사용하면 요소(함수 또는 클래스)에 더 많은 내용의 규약을 설명 가능
- 함수 이름과 파라미터만으로 정확하게 표현되는 요소에는 따로 주석을 넣지 않는 것이 좋음
- 규약이 잘 정리된 함수
  
  ```kotlin
  /**
   * Returns a new read-only list of given elements.  
   * The returned list is serializable (JVM).
   * @sample samples.collections.Collections.Lists.readOnlyList
   */
   public fun <T> listOf(vararg elements: T): List<T> = if (elements.size > 0) elements.asList() else emptyList()
  ```

.

👉🏻 **KDoc 형식**

KDoc 주석의 구조
- 첫 번째 부분은 요소에 대한 요약 설명
- 두 번째 부분은 상세 설명
- 이어지는 줄은 모두 태그로 시작(추가 설명 용도)

[KDoc syntax﻿ Block tags](https://kotlinlang.org/docs/kotlin-doc.html#block-tags)

공식적인 코틀린 문서 생성 도구 이름은 `Dokka`
- [GitHub](https://github.com/Kotlin/dokka)
- [Doc.](https://kotlin.github.io/dokka/1.6.10/)

📖 **정리**

> 외부 API 구현 시 규약을 잘 정의하자.
>
> 이러한 규약은 이름, 문서, 주석, 타입을 통해 구현 가능
>
> 규약은 사용자가 객체를 사용하는 방법을 쉽게 이해하는 등 요소를 쉽게 예측 가능
>
> 규약은 요소가 현재 어떻게 동작하고, 앞으로 어떻게 동작할지를 사용자에게 전달받아 이를 기반으로 사용자는 요소를 확실하게 사용할 수 있고, 규약에 없는 부분을 변경할 수 있는 자유를 얻음

## Item 32. 추상화 규약을 지켜라

📖 **정리**

> 프로그램을 안정적으로 유지하고 싶다면 규약을 지키자
>
> 규약을 깰 수 밖에 없다면 이를 잘 문서화하자.

# 5장. 객체 생성

## Item 33. 생성자 대신 팩토리 함수를 사용하라

생성자의 역할을 대신 해주는 함수를 **팩토리 함수**라고 부른다.

생성자 대신 팩토리 함수 사용 시 장점
- 함수에 이름을 붙일 수 있다.
- 함수가 원하는 형태의 타입을 리턴할 수 있다.
- 호출될 때마다 새 객체를 만들 필요가 없다.
- 아직 존재하지 않은 객체를 리턴할 수 있다.
- 객체 외부에 팩토리 함수를 만들면, 그 가시성을 원하는 대로 제어할 수 있다.
- 인라인으로 만들 수 있으며, 그 파라미터들을 reified로 만들 수 있다.
- 생성자로 만들기 복잡한 객체도 만들 수 있다.
- 원하는 때에 생성자를 호출할 수 있다.

팩토리 함수는 굉장히 강력한 객체 생성 방법

팩토리 함수의 종류
- `companion` 객체 팩토리 함수
- 확장 팩토리 함수
- 톱레벨 팩토리 함수
- 가짜 생성자
- 팩토리 클래스의 메서드

.

👉🏻 **Companion 객체 팩토리 함수**

- 팩토리 함수를 정의하는 가장 일반적인 방법
  - 코틀린에서 이러한 접근 방법을 인터페이스에도 구현 가능

  ```kotlin
  class MyLinkedList<T>(
    val head: T,
    val tail: MyLinkedList<T>?
  ) {
    companion object {
        fun <T> of(vararg elements: T): MyLinkedList<T>? {
            /* ... */
        }
    }
  }

  // 사용
  val list = MyLinkedList.of(1, 2)
  ```

팩토리 함수에 자주 사용되는 이름들
- `from`: 파라미터를 하나 받고, 같은 타입의 인스턴스 하나를 리턴
- `of`: 파라미터 여러개를 받고, 이를 통합해 인스턴스를 생성
- `valueOf`: from, of와 비슷한 기능을 하면서도, 의미를 조금 더 쉽게 읽을 수 있는 함수
- `instance` or `getInstance`: 싱글턴으로 인스턴스 하나를 리턴
- `createInstance` or `newInstance`: getInstance처럼 동작하지만, 싱글턴이 적용되지 않아 함수 호출마다 새로운 인스턴스 생성
- `getType`: getInstance처럼 동작하지만, 팩토리 함수가 다른 클래스에 있을 때 사용
- `newType`: newInstance처럼 동작하지만, 팩토리 함수가 다른 클래스에 있을 때 사용

companion 객체는 인터페이스를 구현할 수 있으며, 클래스를 상속받을 수도 있다.
- companion 객체를 만드는 팩토리 함수

```kotlin
abstract class ActivityFactory {
    abstract fun getIntent(context: Context): Intent

    fun start(context: Context) {
        val intent = getIntent(context)
        context.startActivity(intent)
    }

    fun startForResult(activity: Activity, requestCode: Int) {
        val intent = getIntent(activity)
        activity.startActivityForResult(intent, requestCode)
    }
}

class MainActivity: AppCompatActivity() {
    //...

    companion object: ActivityFactory() {
        override fun getIntent(context: Context): Intent = 
            Intent(context, MainActivity::class.java)
    }
}

// Use
val intent = MainActivity.getIntent(context)
MainActivity.start(context)
MainActivity.startForResult(activity, requestCode)
```

추상 companion 객체 팩토리는 값을 가질 수 있어서, 캐싱을 구현하거나 테스트를 위한 가짜 객체 생성 가능

.

👉🏻 **확장 팩토리 함수**

- 이미 companion 객체가 존재할 때, 이 객체의 함수처럼 사용할 수 있는 팩토리 함수를 만들어야 할 때가 있다.

```kotlin
interface Tool {
    companion object { /* ... */ }
}
```

- companion 객체를 활용해서 확장 함수를 정의 가능

```kotlin
fun Tool.Companion.createBigTool( /*...*/ ): BigTool {
    //...
}

Tool.createBigTool()
```

이러한 코트를 활용하면 팩토리 메서드를 만들어서 외부 라이브러리를 확장 가능
- 다만 companion 객체를 확장하려면 (적어도 비어있는) companion 객체가 필요

.

👉🏻 **톱레벨 팩토리 함수**

- 대표적인 예로 listOf, setOf, mapOf

```kotlin
public fun <T> listOf(
    vararg elements: T
): List<T> = if (elements.size > 0) elements.asList() else emptyList()
```

.

👉🏻 **가짜 생성자**

코틀린의 생성자는 톱레벨 함수와 같은 형태로 사용
- 따라서 톱레벨 함수처럼 참조

```kotlin
class A
val a = A()

val reference: ()->A = ::A
```

List를 생성자처럼 사용하는 코드
- 톱레벨 함수는 생성자처럼 보이고, 생성자처럼 작동
- 하지만, 팩토리 함수와 같은 모든 장점을 가짐
- 이것을 `가짜 생성자`(**fake constructor**)라고 부름

```kotlin
List(4) { "User$it" }

public inline fun <T> List(
    size: Int, 
    init: (index: Int) -> T)
: List<T> = MutableList(size, init)

public inline fun <T> MutableList(
    size: Int, 
    init: (index: Int) -> T
): MutableList<T> {
    val list = ArrayList<T>(size)
    repeat(size) { index -> list.add(init(index)) }
    return list
}
```

진짜 생성자 대신 가짜 생성자를 만드는 이유
- 인터페이스를 위한 생성자를 만들고 싶을 때
- reified 타입 아규먼트를 갖게 하고 싶을 때

이를 제외하면, 가짜 생성자는 진짜 생성자처럼 동작해야 한다
- 생성자처럼 보여야 하며 생성자와 같은 동작을 해야 함
- 캐싱, nullable 리턴, 서브 클래스 리턴 등의 기능까지 포함해서 객체를 만들고 싶다면, companion 객체 팩토리 메서드처럼 다른 이름을 가진 팩토리 함수를 사용하는 것을 권장
- 기본 생성자를 만들 수 없는 상황 또는 생성자가 제공하지 않는 기능으로 생성자를 만들어야 하는 상황에만 가짜 생성자를 사용하자

.

👉🏻 **팩토리 클래스의 메서드**

```kotlin
data class Student(
    val id: Int,
    val name: String,
    val surname: String,
)

class StudentsFactory {
    var nextId = 0
    fun next(name: String, surname: String) = 
        Student(nextId++, name, surname)
}

val factory = StudentsFactory()
var s1 = factory.next("Marcin", "Moskala")
var s2 = factory.next("Igor", "Wojda")
```

팩토리 클래스는 프로퍼티를 가질 수 있다.
- 이를 활용하면 다양한 종류로 최적화하고, 다양한 기능 도입이 가능
- 캐싱 활용, 이전에 생성한 객체를 복제해서 객체 생성 등 객체 생성 속도 향상 등..

📖 **정리**

> 가짜 생성자, 톱레벨 팩토리 함수, 확장 팩토리 함수 등 일부는 신중하게 사용 필요
>
> 팩토리 함수를 정의하는 가장 일반적인 방법은 companion 객체를 사용하는 것

## Item 34. 기본 생성자에 이름 있는 옵션 아규먼트를 사용하라

객체를 정의하고 생성하는 방법을 지정할 때 사용하는 가장 기본적인 방법은 `기본 생성자`를 사용하는 것

디폴트 아규먼트가 점층적 생성자보다 좋은 이유
- 파라미터들의 값을 원하는 대로 지정 가능
- 아규먼트를 원하는 순서로 지정 가능
- 명시적으로 이름을 붙여서 아규먼트를 지정하므로 의미가 훨씬 명확

```kotlin
class Pizza(
    val size: String,
    val cheese: Int = 0,
    val olives: Int = 0,
    val bacon: Int = 0,
)
```

📖 **정리**

> 일반적인 프로젝트에서 기본 생성자를 사용해 객체를 생성
>
> 디폴트 아규먼트는 더 짧고, 명확하고 사용하기 쉽다.

## Item 35. 복잡한 객체를 생성하기 위한 DSL을 정의하라

📖 **정리**

> DSL은 언어 내부에서 사용할 수 있는 특별한 언어
>
> 복잡한 객체는 물론 HTML, 복잡한 설정 등의 계층 구조를 갖는 객체를 간단하게 표현 가능
>
> 하지만, DSL 구현은 해당 DSL이 익숙하지 않은 개발자에게 혼란과 어려움을 줄 수 있음
>
> 따라서 DSL은 복잡한 객체를 만들거나, 복잡한 계층 구조를 갖는 객체 생성 시에만 활용하는 것을 권장
>
> 좋은 DSL을 만드는 작업은 굉장히 어렵지만, 잘 정의된 DSL은 프로젝트에 굉장히 큰 도움

# 6장. 클래스 설계

## Item 36. 상속보다는 컴포지션을 사용하라

보통 간단한 행위를 재사용하기 위해 슈퍼클래스를 만들어서 공통되는 행위를 추출

이 경우 문제 없이 동작은 하지만, 몇 가지의 단점이 존재
- 상속은 하나의 클래스만을 대상으로 가능
- 상속은 클래스의 모든 것을 가져옴
- 상속은 이해하기 어려움

대표적인 대안은 `컴포지션`(composition)
- 컴포지션을 사용한다는 것은 객체를 프로퍼티로 갖고, 함수를 호출하는 형태로 재사용하는 것을 의미
- 추가 코드를 적절하게 처리하는 것이 조금 어려울 순 있지만, 실행을 더 명확하게 예측 가능한 장점
- 또, 하나의 클래스 내부에서 여러 기능을 재사용 가능

```kotlin
class Progress {
    fun showProgress() { /* */ }
    fun hideProgress() { /* */ }
}

class ProfileLoader {
    val progress = Progress()

    fun load() {
        progress.showProgress()
        progress.hideProgress()
    }
}

class ImageLoader {
    val progress = Progress()

    fun load() {
        progress.showProgress()
        progress.hideProgress()
    }
}
```

.

👉🏻 **오버라이딩 제한하기**

- 상속은 허용하지만, 메서드는 오버라이드하지 못하게 만들고 싶은 경우
- 메서드에 `open` 키워드를 사용
- `open` 클래스는 open 메서드만 오버라이드 가능

```kotlin
open class Parent {
    fun a() {}
    open fun b() {}
}

class Child: Parent() {
    override fun a() {} // Error
    override fun b() {}
}
```

📖 **정리**

> 1️⃣ 컴포지션은 더 안전
> 
> - 다른 클래스의 내부적인 구현에 의존하지 않고 외부에서 관찰되는 동작에만 의존
>
> 2️⃣ 컴포지션은 더 유연
>
> - 상속은 한 클래스만 대상으로 할 수 있지만, 컴포지션은 여러 클래스를 대상으로 가능
> - 상속은 모든 것을 받지만, 컴포지션은 필요한 것만 받을 수 있음
> - 슈퍼클래스의 동작을 변경하면, 서브클래스의 동작에도 큰 영향
>
> 3️⃣ 컴포지션은 더 명시적
> - 리시버를 명시적으로 활용할 수밖에 없으므로 메서드가 어디에 있는 것인지 확실하게 인지
> - this 키워드 불필요
>
> 4️⃣ 컴포지션은 생각보다 번거로움
> - 객체를 명시적으로 사용해야 하므로, 대상 클래스에 일부 기능을 추가할 때 이를 포함하는 객체의 코드 변경 필요
>
> 5️⃣ 상속은 다형성을 활용
>
> 일반적으로 OOP에서는 상속보다 컴포지션을 사용하는 것이 좋다.
> - 코틀린에서는 더욱 이런 규칙을 지켜 주는 것이 좋음
>
> 그렇다면 상속은 언제 사용하지?
> - 명확한 'is-a 관계'일 때 상속을 사용

## Item 37. 데이터 집합 표현에 data 한정자를 사용하라

data 한정자를 붙이면, 다음과 같은 몇 가지 함수가 자동으로 생성
- toString
- equals & hashCode
- copy
- componentN(component1, component2 ...)

.

👉🏻 **튜플 대신 데이터 크 ㄹ래스 사용하기**

- 데이터 클래스는 튜플보다 많은 것을 제공
- 구체적으로 코틀린의 튜플은 Serializable을 기반으로 만들어지며
- toString을 사용할 수 있는 제네릭 데이터 클래스

코틀린에서 클래스는 큰 비용없이 사용할 수 있는 좋은 도구
- 따라서 클래스를 활용하는 데 두려움을 갖지 말고, 적극적으로 활용 권장

## Item 38. 연산 또는 액션을 전달할 때는 인터페이스 대신 함수 타입을 사용하라

함수 타입을 사용하는 코드는 더 많은 자유를 얻을 수 있다.

✅ 람다 표현식 또는 익명 함수로 전달

```kotlin
setOnClickListener { /*..*/ } 
setOnClickListener(fun(view) { /*..*/ })
```

✅ 함수 레퍼런스 또는 제한된 함수 레퍼런스로 전달

```kotlin
setOnClickListener(::println)
setOnClickListener(this::showUsers)
```

✅ 선언된 함수 타입을 구현한 객체로 전달

```kotlin
class ClickListener: (View)->Unit {
    override fun invoke(view: View) {
        // ...
    }
}

setOnClickListener(ClickListener())
```

타입 별칭을 사용하면 함수 타입도 이름을 붙일 수 있다.

```kotlin
typealias OnClick = (View) -> Unit
```

📖 **정리**

> 자바에서 사용하기 위한 API를 설계할 때는 함수 타입보다 SAM(Single-Abstract Method)을 사용하는 것이 합리적
>
> 하지만, 이외의 경우에는 함수 타입을 사용하는 것이 좋다.

## Item 39. 태그 클래스보다는 클래스 계층을 사용하라

상수모드를 가진 클래스를 꽤 많이 볼 수 있다.
- 이러한 상수 모드를 태그(tag)라고 부르며, 태그를 포함한 클래스를 태그 클래스라고 부름

코틀린은 일반적으로 태그 클래스보다 sealed 클래스를 많이 사용
- 한 클래스에 여러 모드를 만드는 방법 대신 각각의 모드를 여러 클래스로 만들고 타입 시스템과 다형성을 활용
- 이러한 클래스에는 sealed 한정자를 붙여서 서브클래스 정의를 제한

```kotlin
sealed class ValueMatcher<T> {
    abstract fun match(value: T): Boolean
    
    class Equal<T>(val value: T): ValueMatcher<T>() {
    	override fun match(value: T): Boolean = 
        	value == this.value
    }
    
    class NotEqual<T>(val value: T): ValueMatcher<T>() {
    	override fun match(value: T): Boolean = 
        	value != this.value
    }
    
    class EmptyList<T>(val value: T): ValueMatcher<T>() {
    	override fun match(value: T): Boolean = 
        	value is List<*> && value.isEmpty()
    }
    
    class NotEmptyList<T>(val value: T): ValueMatcher<T>() {
    	override fun match(value: T): Boolean = 
        	value is List<*> && value.isNotEmpty()
    }
}
```

.

👉🏻 **`sealed` 한정자**

- sealed 한정자는 외부 파일에서 서브클래스를 만드는 행위 자체를 모두 제한
- 외부에서 추가적인 서브클래스를 만들 수 없으므로 타입이 추가되지 않을 것이라는 게 보장
- 확장 함수를 사용해서 클래스에 새로운 함수를 추가하거나, 클래스의 다양한 변경을 쉽게 처리 가능

📖 **정리**

> 코틀린에서는 태그 클래스보다 타입 계층을 사용하는 것이 좋다.
>
> 그리고 일반적으로 이러한 타입 계층을 만들 때는 `sealed` 클래스를 사용
>
> 상태 패턴과는 다른데, 타입 계층과 상태 패턴은 실질적으로 함께 사용하는 협력 관계라고 할 수 있음
>
> 하나의 뷰를 가지는 경우보다는 여러 개의 상태로 구분할 수 있는 뷰를 가질 때 많이 활용

## Item 40. equals 규약을 지켜라

코틀린에는 두 가지 종류의 동등성이 존재
- `구조적 동등성`
  - equals 메서드와 이를 기반으로 만들어진 ==, != 연산자로 확인하는 동등성
  - nullable일 경우 a?.equals(b) ?: (b === null) 로 변환
  - 아닐 경우 a.equals(b) 로 변환
- `레퍼런스적 동등성`
  - ===, !=== 로 확인하는 동등성
  - 두 피연산자가 같은 객체를 가리키면 true

.

👉🏻 **equals가 필요한 이유**

Any 클래스에 구현되어 있는 equals 메서드는 디폴트로 ===처럼 두 인스턴스가 완전히 같은 객체인지를 비교
- data 한정자를 붙여서 데이터 클래스로 정의하면 자동으로 같은 동등성으로 동작
  - 데이터 클래스는 내부에 어떤 값을 가지고 있는지가 중요
- 데이터 클래스의 동등성은 모든 프로퍼티가 아니라 일부 프로퍼티만 비교해야 할 때도 유용

`equals`를 직접 구현해야 하는 경우를 정리해 보면, 다음과 같습니다
- 기본적으로 제공되는 동작과 다른 동작을 해야 하는 경우
- 일부 프로퍼티만으로 비교해야 하는 경우
- data 한정자를 붙이는 것을 원하지 않거나, 비교해야 하는 프로퍼티가 기본 생성자에 없는 경우

.

👉🏻 **equals 구현하기**

- 특별한 이유가 없는 이상 직접 equals를 구현하는 것은 좋지 않다.
- 기본적으로 제공되는 것을 그대로 쓰거나, 데이터 클래스로 만들어서 사용하는 것이 좋다.
- 직접 구현해야 한다면, 반사적/대칭적/연속적/일관적 동작을 하는지 확인하자.

## Item 41. hashCode 규약을 지켜라

hashCode 규약

1️⃣ 어떤 객체를 변경하지 않았다면, `hashCode`는 여러 번 호출해도 그 결과가 항상 같아야 한다.

2️⃣ equals 메서드의 실행 결과로 두 객체가 같다고 나온다면, `hashCode` 메서드의 호출 결과도 같다고 나와야 한다.

- 일관성 유지를 위해 hashCode가 필요
- hashCode는 equals와 같이 일관성 있는 동작을 해야 함
- 같은 요소는 반드시 같은 hashCode를 가져야 함

.

👉🏻 **hashCode 구현하기**
- 일반적으로 모든 해시 코드의 값을 더한다.
- 더하는 과정마다 이전까지의 결과에 31을 곱한 뒤 더한다.

```kotlin
class DateTime(
    private var millis: Long = 0L,
    private var timeZone: TimeZone? = null
) {
    private var asStringCache = ""
    private var changed = false
    
    override fun equals(other: Any?): Boolean =
        other is DateTime &&
            other.millis == millis &&
            other.timeZone == timeZone
           
    override fun hashCode(): Int {
        var result = millis.hashCode()
        result = result * 31 + timeZone.hashCode()
        return result
    }   
}
```

이때 유용한 함수로 코틀린/JVM의 `Objects.hashCode`를 사용하면 해시를 계산해 줍니다.

```kotlin
override fun hashCode(): Int = 
    Objects.hash(timeZone, millis)
```

## Item 42. compareTo의 규약을 지켜라

1️⃣ 비대칭적 동작
- a >= b이고 b >= a 라면, a == b

2️⃣ 연속적 동장
- a >= b이고 b >= c 라면, a >= c

3️⃣ 코넥스적 동작
- 두 요소는 어떤 확실한 관계를 가지고 있어야 한다.
- a >= b 또는 b >= a 중 적어도 하나는 항상 true

.

👉🏻 **compareTo를 따로 정의해야 할까?**

- 코틀린에서 compareTo를 따로 정의해야 하는 상황은 거의 없다.
  - 일반적으로 어떤 프로퍼티 하나를 기반으로 순서를 지정하는 것으로 충분하기 때문
  - 예를 들어, sortedBy를 사용하면, 원하는 키로 컬렉션을 정렬
- 여러 프로퍼티를 기반으로 정렬해야 한다면 `sortedWith` 함수를 사용

```kotlin
val sorted = names
            .sortedWith(compareBy({ it.surname }, { it.name }))
```

.

👉🏻 **compareTo 구현하기**

- compareTo를 구현할 때 유용하게 활용할 수 있는 톱레벨 함수가 있다.
  - 두 값을 단순하게 비교하기만 한다면, `compareValues` 함수 활용 가능
- 더 많은 값을 비교하거나, selector를 활용해서 비교하고 싶다면, `compareValuesBy`를 사용

```kotlin
class User(
    val name: String,
    val surname: String
): Comparable<User> {
    override fun compareTo(other: User): Int =
        compareValues(surname, other.surname)
}

...

class User(
    val name: String,
    val surname: String
): Comparable<User> {
    override fun compareTo(other: User): Int =
        compareValuesBy(this, other, { it.surname }, { it.name })
}
```

특별한 논리를 구현해야 하는 경우 이 함수가 다음 값을 리턴하도록 하자
- 0: 리시버와 other가 같은 경우
- 양수: 리시버가 other보다 큰 경우
- 음수: 리시버가 other보다 작은 경우

## Item 43. API의 필수적이지 않은 부분을 확장 함수로 추출하라

멤버와 확장의 가장 큰 차이점은 확장은 따로 가져와서 사용해야 한다는 점.

1️⃣ 임포트해서 사용한다는 특징 덕분에 확장은 같은 타입에 같은 이름으로 여러 개 만들 수 있다.
- 따라서, 여러 라이브러리에서 여러 메서드를 받을 수도 있고 충돌이 발생하지 않는다는 장점도 생긴다.
- 하지만, 같은 이름으로 다른 동작을 하는 확장이 있다는 것은 위험할 수 있다.
  - 위험 가능성이 있다면 그냥 멤버 함수로 만들어서 사용하자.
  - 그렇게 하면 컴파일러가 항상 확장 대신 멤버 함수를 호출할 것이다.

2️⃣ 확장은 가상이 아니라는 것
- 즉, 파생 클래스에서 오버라이드 불가

3️⃣ 확장 함수는 클래스가 아닌 타입에 정의하는 것
- 그래서 nullable 또는 구체적인 제네릭 타입에도 확장 함수를 정의 가능

4️⃣ 확장은 클래스 레퍼런스에서 멤버로 표시되지 않는다
- 필수적이지 않은 요소를 확장 함수로 추출하면, 어노테이션 프로세스로부터 숨겨진다.

📖 **정리**

> 멤버와 확장 함수의 차이를 비교하면, 다음과 같다.
>
> - 확장 함수는 읽어 들여야 한다.
> - 확장 함수는 virtual이 아니다.
> - 멤버는 높은 우선 순위를 가진다.
> - 확장 함수는 클래스 위가 아니라 타입 위에 만들어진다.
> - 확장 함수는 클래스 레퍼런스에 나오지 않는다.
>
> API의 필수적인 부분은 멤버로 두는 것이 좋지만, 필수적이지 않은 부분은 확장 함수로 만드는 것이 여러모로 좋다.

## Item 44. 멤버 확장 함수의 사용을 피하라

어떤 클래스에 대한 확장 함수를 정의할 때, 멤버로 추가하는 것은 좋지 않다.
- 확장 함수는 첫 번째 아규먼트로 리시버를 받는 단순한 일반 함수로 컴파일된다.

```kotlin
// 컴파일 전
fun String.isPhoneNumber(): Boolean = 
        length == 7 && all { it.isDigit() }

// 컴파일 후
fun isPhoneNumber('$this': String): Boolean = 
        '$this'.length == 7 && '$this'.all { it.isDigit() }
```

확장 함수를 클래스 멤버로 정의하는 것은 DSL을 만들 때를 제외하면 사용하지 않는 것이 좋다.
- 가시성을 제한하지 못하므로 확장 함수를 사용하는 형태를 어렵게 만든다.
- 확장 함수의 가시성을 제한하고 싶다면, 멤버로 만들지 말고, 가시성 한정자를 붙여주자.

```kotlin
/* 
 * 멤버 확장 함수를 사용하는 경우
 */
class PhoneBookCorrect {
    fun String.isPhoneNumber() =
        length == 7 && all { it.isDigit() }
}

PhoneBookCorrect().apply { "1234567890".test() }

/* 
 * 멤버가 아닌 확장 함수를 사용하는 경우
 */
class PhoneBookCorrect {
    // ...
}

private fun String.isPhoneNumber() =
            length == 7 && all { it.isDigit() }
```

✅ 멤버 확장을 피해야 하는 몇 가지 타당한 이유

1️⃣ 레퍼런스를 지원하지 않는다.

```kotlin
val refX = PhoneBookInCorrect::isPhoneNumber // 오류
val boundedRefX = book::isPhoneNumber // 오류
```

2️⃣ 암묵적 접근 시, 두 리시버 중 어떤 리시버가 선택될지 혼동된다.

```
class A {
    val a = 10
}
class B {
    val a = 20
    val b = 30

    fun A.test() = a + b
}
```

3️⃣ 확장 함수가 외부에 있는 다른 클래스를 리시버로 받을 때, 해당 함수가 어떤 동작을 하는지 명확하지 않다.

```kotlin
class A {
    // ...
}
class B {
    // ...

    fun A.update() = ... // A, B 중 어떤 것을 업데이트?
}
```

📖 **정리**

> 멤버 확장 함수를 사용하는 것이 의미가 있는 경우에는 사용해도 괜찮다.
>
> 하지만, 일반적으로 그 단점을 인지하고, 사용하지 않는 것이 좋다.
>
> 가시성을 제어하려면 가시성과 관련된 한정자를 사용하자.
>
> 클래스 내부에 확장 함수를 배치한다고, 외부에서 해당 함수를 사용하지 못하게 제한되는 것이 아니다.