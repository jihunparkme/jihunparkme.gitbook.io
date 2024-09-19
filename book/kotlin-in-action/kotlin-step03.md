# 코틀린 Step03 - 제네릭스, 애노테이션, 리플렉션

<details>
<summary>📒 제네릭스 요약</summary>

- 제네릭 함수와 클래스를 자바와 비슷하게 선언

```kotlin
fun <T> identity(value: T): T {
    return value
}

class Box<T>(val value: T)

@Test
fun `제네릭 함수`() {
    assertEquals(42, identity(42))
    assertEquals("Hello", identity("Hello"))
}

@Test
fun `제네릭 클래스`() {
    val intBox = Box(42)
    val stringBox = Box("Hello")

    assertEquals(42, intBox.value)
    assertEquals("Hello", stringBox.value)
}
```

- 자바와 마찬가지로 제네릭 타입의 타입 인자는 컴파일 시점에만 존재
- 타입 인자가 실행 시점에 지워지므로 타입 인자가 있는 타입(제네릭 타입)을 is 연산자를 사용해 검사 불가
- 인라인 함수의 타입 매개변수를 `refied`로 표시해서 실체화하면 실행 시점에 그 타입을 is로 검사하거나 java.lang.Class 인스턴스를 얻을 수 있다.
- 변성은 기저 클래스가 같고 타입 파라미터가 다른 두 제네릭 타입 사이의 상위/하위 타입 관계가 타입 인자 사이의 상위/하위 타입 관계에 의해 어떤 영향을 받는지를 명시하는 방법

```kotlin
open class Animal
class Dog : Animal()
class Box<out T>(val value: T)

@Test
fun `변성 테스트`() {
    val dogBox: Box<Dog> = Box(Dog())
    val animalBox: Box<Animal> = dogBox // out 키워드로 캐스팅 가능

    assertTrue(animalBox is Box<Animal>)
}
```

- 제네릭 클래스의 타입 파라미터가 `out` 위치에서만 사용되는 경우(생산자) 그 타입 파라미터를 `out`으로 표시해서 공변적으로 만들 수 있다.
    - 코틀린의 읽기 전용 List 인터페이스는 공변적이다.
    - 따라서 List<String>은 List<Any)의 하위 타입이다.
- 공변적인 경우와 반대로 제네릭 클래스의 타입 파라미터가 `in` 위치에서만 사용되는 경우(소비자) 그 타입 파라미터를 `in`으로 표시해서 반공변적으로 만들 수 있다.
- 함수 인터페이스는 첫 번째 타입 파라미터에 대해서는 반공변적이고, 두 번째 타입 파라미터에 대해서는 공변적이다.
    - 그래서 (Animal) → Int는 (Cat) → Number의 하위 타입이다.
- 코틀린에서는 제네릭 클래스의 공변성을 전체적으로 지정하거나(선언 지점 변성) 구체적인 사용 위치에서 지정할 수 있다. (사용 지점 변성)
- 제네릭 클래스의 타입 인자가 어떤 타입인지 정보가 없거나 타입 인자가 어떤 타입인지가 중요하지 않을 때 스타 프로젝션(`*`) 구문을 사용할 수 있다.
</details>

<details>
<summary>📕 애노테이션과 리플렉션 요약</summary>

- 코틀린에서 애노테이션을 적용할 때 사용하는 문법은 자바와 거의 동일
- 코틀린에서는 자바보다 더 넓은 대상에 애노테이션을 적용 가능 (ex. 파일과 식(expression))
- 애노테이션 인자로 원시 타입 값, 문자열, 이넘, 클래스 참조, 다른 애노테이션 클래스의 인스턴스, 그리고 지금까지 말한 여러 유형의 값으로 이뤄진 배열을 사용 가능
- `@get:Rule`을 사용해 애노테이션의 사용 대상을 명시하면 한 코틀린 선언이 여러 가지 바이트 코드 요소를 만들어내는 경우 정확히 어떤 부분에 애노테이션을 적용할지 지정 가능
- 애노테이션 클래스를 정의할 때는 본문이 없고 주 생성자의 모든 파라미터를 val 프로퍼티로 표시한 코틀린 클래스를 사용

```kotlin
@Target(AnnotationTarget.CLASS, AnnotationTarget.FUNCTION)
@Retention(AnnotationRetention.RUNTIME)
annotation class MyAnnotation(val name: String, val value: Int)
```

- 메타애노테이션을 사용해 대상, 애노테이션 유지 방식 등 여러 애노테이션 특성을 지정 가능

```kotlin
@Target(AnnotationTarget.CLASS, AnnotationTarget.FUNCTION) // 애노테이션 적용 대상
@Retention(AnnotationRetention.RUNTIME) // 애노테이션 유지 방식
@MustBeDocumented // 문서화 여부
annotation class MyAnnotation(val name: String, val value: Int)
```

- 리플렉션 API를 통해 실행 시점에 객체의 메소드와 프로퍼티를 열거하고 접근 가능
    - 리플렉션 API에는 클래스(KClass), 함수(KFunction) 등 여러 종류의 선언을 표현하는 인터페이스 제공
- 클래스를 컴파일 시점에 알고 있다면 `KClass` 인스턴스를 얻기 위해 ClassName::class를 사용
    - 하지만 실행 시점에 obj 변수에 담긴 객체로부터 KClass 인스턴스를 얻기 위해서는 obj.javaClass.kotlin을 사용
- `KFunction`과 `KProperty` 인터페이스는 모두 KCallable을 확장
    - KClassable은 제네릭 call 메소드를 제공
- `KCallable.callBy` 메소드를 사용하면 메소드를 호출하면서 디폴트 파라미터값을 사용 가능
- `KFunction0`, `KFunctiuon1` 등의 인터페이스는 모두 파라미터 수가 다른 함수를 표현하며, invoke 메소드를 사용해 함수 호출 가능
- `KProperty0`는 최상위 프로퍼티나 변수, `KProperty1`은 수신 객체가 있는 프로퍼티에 접근할 때 쓰는 인터페이스
    - 두 인퍼테이스 모두 GET 메소드를 사용해 프로퍼티 값을 가져올 수 있음
    - `KMutableProperty0`과 `KMutableProperty1`은 각각 KProperty0과 KProperty1을 확장하며, set 메소드를 통해 프로퍼티값을 변경할 수 있게 지원
</details>

---

