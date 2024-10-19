# Kotlin Basic 

## Null 허용 타입

> 변수가 null 값을 갖지 못하게 하려면
>
> 안전 호출 연산자(?.)나 엘비스 연산자(?:)와 결합해서 사용하자.

👉🏻 **val 변수의 영리한 타입 변환(smart cast)**

- 널 할당이 불가능한 문자열 타입으로 영리한 변환 가능

```kotlin
val p = Person(first = "North", middle = null, last = "West")
if (p.middle != null) {
    val middleNameLength = p.middle.length
}
``` 

👉🏻 **var 변수가 널 값이 아님을 단언하기 `!!`**

- var 변수는 String 타입으로 영리한 타입 변환이 불가능
- 널 아님 단언 연산자(`!!`, not-null assertion operator)로 널 아님을 단언할 수 있지만, 코드 스멜이다
  - 변수가 널이 아닌 값으로 다뤄지도록 강제하고 해당 변수가 널이라면 예외를 던진다
  - 널 값에 이 연산자를 사용하는 것은 코틀린에서 NPE를 만날 수 있는 몇 가지 상황 중 하나
  - 가능하면 사용하지 않도록 노력하자.

```kotlin
var p = Person(first = "North", middle = null, last = "West")
if (p.middle != null) {
    val middleNameLength = p.middle!!.length
}
``` 

👉🏻 **var 변수에 안전 호출 연산자 사용하기 `?.`**

- 이 상황에서 안전 호출(`?.`, safe call)를 사용하는 것이 좋다.
- 결과 타입은 Type? 형태이고, null 이면 null을 반환

```kotlin
var p = Person(first = "North", middle = null, last = "West")
val middleNameLength = p.middle?.length
``` 

👉🏻 **var 변수에 안전 호출 연산자와 엘비스 연산자 사용하기 `?:`**

- `?:` : 왼쪽 식의 값을 확인해서 해당 값이 널이 아니면 그 값을 리턴, 널이라면 오른쪽 값을 리턴

```kotlin
var p = Person(first = "North", middle = null, last = "West")
val middleNameLength = p.middle?.length ?: 0
``` 

👉🏻 **안전 타입 변환 연산자 `as?`**

- 타입 변환이 올바르게 동작하지 않은 경우 ClassCastException이 발생하는 상황을 방지

```kotlin
val p1 = p as? Person
```

## 명시적 타입 변환

> 코틀린은 자동으로 기본 타입을 더 넓은 타입으로 승격하지 않는다. Int -> Long (X)
>
> 더 작은 타입을 명시적으로 변환하려면 toInt, toLong 등 구체적인 변환 함수를 사용하자.

```kotlin
val intVar: Int = 3
// val longVar: Long = intVal // failed Compile
val longVar: Long = intVar.toLong()

...

// 단 연산자 중복 시 명시적 타입 변환 불필요
val longSum = 3L + intVar
```

사용 가능한 타입 변환 메소드
- toByte(): Byte
- toChar(): Char
- toShort(): Short
- toInt(): Int
- toLong(): Long
- toFloat(): Float
- toDouble(): Double

## 중위(infix) 함수

> 코틀린에는 자바처럼 내장 거듭제곱 연산자가 없다.

👉🏻 **시그니처의 확장 함수를 정의**

```kotlin
fun Int.pow(x: Int) = toDouble().pow(x).toInt()
fun Long.pow(x: Int) = toDouble().pow(x).toLong()
```

👉🏻 **중위 연산자 infix 정의**

```kotlin
import kotlin.math.pow

infix fun Int.`**`(x: Int) = toDouble().pow(x).toInt()
infix fun Long.`**`(x: Int) = toDouble().pow(x).toLong()
infix fun Float.`**`(x: Int) = pow()
infix fun Double.`**`(x: Int) = pow()

fun Int.pow(x: Int) = `**`(x)
fun Long.pow(x: Int) = `**`(x)

...

@Test
fun `raise to pwoer`() {
    assertAll(
        { assertEquals(1, 2 `**` 0) },
        { assertEquals(2, 2 `**` 1) },
        { assertEquals(4, 2 `**` 2) },
        { assertEquals(8, 2 `**` 3) },

        { assertEquals(1L, 2L `**` 0) },
        { assertEquals(2L, 2L `**` 1) },
        { assertEquals(4L, 2L `**` 2) },
        { assertEquals(8L, 2L `**` 3) },

        { assertEquals(1F, 2F `**` 0) },
        { assertEquals(2F, 2F `**` 1) },
        { assertEquals(4F, 2F `**` 2) },
        { assertEquals(8F, 2F `**` 3) },

        { assertEquals(1.0, 2.0 `**` 0) },
        { assertEquals(2.0, 2.0 `**` 1) },
        { assertEquals(4.0, 2.0 `**` 2) },
        { assertEquals(8.0, 2.0 `**` 3) },

        { assertEquals(1, 2.pow(0)) },
        { assertEquals(2, 2.pow(1)) },
        { assertEquals(4, 2.pow(2)) },
        { assertEquals(8, 2.pow(3)) },

        { assertEquals(1L, 2L.pow(0)) },
        { assertEquals(2L, 2L.pow(1)) },
        { assertEquals(4L, 2L.pow(2)) },
        { assertEquals(8L, 2L.pow(3)) },
    )
}
```

## Pair 인스턴스

> 중위(infix) to 함수로 Pair 클래스의 인스턴스를 생성할 수 있다.

코틀린은 Pair 인스턴스의 리스트로부터 맵을 생성하는 mapOf와 같은 맵 생성을 위한 최상위 함수 몇 가지를 제공

```kotlin
fun <K, V> mapOf(vararg pairs: Pair<K, V>): Map<K, V>
```

Pair는 first, second 이름의 두 개의 원소를 갖는 데이터 클래스이다.

```kotlin
data class Pair<out A, out B> : Serializable
```

Pair 클래스는 두 개의 인자를 받는 생성자를 사용해서 Pair 클래스를 생성할 수 있지만, to 함수를 사용하는 것이 일반적이다.

```kotlin
public infix fun <A, B> A.to(that: B): Pair<A, B> = Pair(this, that)
```

👉🏻 **mapOf 인자인 pair를 생성하기 위해 to 함수 사용하기**

```kotlin
@Test
fun `create map using infix to function`() {
    // to를 사용한 Pair 생성
    val map = mapOf("a" to 1, "b" to 2, "c" to 2)

    assertAll(
        { assertTrue(map.containsKey("a")) },
        { assertTrue(map.containsKey("b")) },
        { assertTrue(map.containsKey("c")) },
        { assertTrue(map.containsValue(1)) },
        { assertTrue(map.containsValue(2)) },
    )
}

@Test
fun `create a Pair form constructor vs to function`() {
    val p1 = Pair("a", 1) // 생성자를 사용한 Pair 생성
    val p2 = "a" to 1 // to를 사용한 Pair 생성

    assertAll(
        { assertEquals(p1.first, "a") },
        { assertEquals(p1.second, 1) },
        { assertEquals(p2.first, "a") },
        { assertEquals(p2.second, 1) },
        { assertEquals(p1, p2) },
    )
}
```

⚠️ Pair는 데이터 클래스이므로 구조 분해를 통해서만 개별 원소에 접근할 수 있다.

```kotlin
@Test
fun `destructuring a Pair`() {
    val pair = "a" to 1
    val (x, y) = pair

    assertEquals(x, "a")
    assertEquals(y, 1)
}
```

{% hint style="info" %}

**Triple**

세 개의 값을 나타내는 Triple이라는 이름의 클래스도 코틀린 표준 라이브러리에 들어 있다.

{% endhint %}

## const와 val의 차이

> 런타임보다 컴파일 타임에 변수가 상수임을 나타내야 한다

컴파일 타임 상수에 const 변경자를 사용하자.
- val 키워드는 변수에 한 번 할당되면 변경이 불가능함을 나타내지만 이러한 할당은 실행 시간에 일어난다.

👉🏻 **컴파일 타임 상수 정의하기**
- const가 val 키워드를 대체하는 것이 아니라 **반드시 같이 쓰여야 한다.**

```kotlin
class Task(val name: String, _priority: Int = DEFAULT_PRIORITY) {

    companion object {
        // 컴파일 타임 상수
        const val MIN_PRIORITY = 1
        const val MAX_PRIORITY = 5
        const val DEFAULT_PRIORITY = 3
    }

    // 사용자 정의 설정자(setter)를 사용하는 속성
    var priority = validPriority(_priority)
        set(value) {
            field = validPriority(value)
        }

    // private 검증 함수
    private fun validPriority(p: Int) =
        p.coerceIn(MIN_PRIORITY, MAX_PRIORITY)
}

...

@Test
fun `priority range`() {
    var task = Task("Default Task")
    assertEquals(3, task.priority)

    task = Task("Priority Task", 4)
    assertEquals(4, task.priority)

    task = Task("Low Priority Task", 0)
    assertEquals(1, task.priority)

    task = Task("High Priority Task", 6)
    assertEquals(5, task.priority)

    task = Task("Update Priority Task", 2)
    task.priority = 5
    assertEquals(5, task.priority)

    task = Task("Update Priority Task", 2)
    task.priority = 10
    assertEquals(5, task.priority)

    task = Task("Update Priority Task", 4)
    task.priority = -1
    assertEquals(1, task.priority)
}
```

## 사용자 정의 획득자와 설정자 생성하기

> 코틀린은 모든 것이 기본적으로 public 이다.
>
> 데이터 은닉 원칙을 침해하는 딜레마를 클래스에서 필드는 직접 선언할 수 없도록 하는 방법으로 해결하였다.

👉🏻 **사용자 정의 획득자와 설정자**

속성 정의 문법

```kotlin
var <propertyName>[: <PropertyType> [= <property_initializer>]
    [<getter>]
    [<setter>]
```

```kotlin
class Task(val name: String) {
    // 사용자 정의 설정자
    var priority = 3
        set(value) {
            field = value.coerceIn(1..5)
        }

    // 파생 속성을 위한 사용자 정의 획득자
    val lowPriority
        get() = priority < 3
}

...

class Task(val name: String) {
    // 사용자 정의 설정자
    var priority = 3
        set(value) {
            field = value.coerceIn(1..5)
        }

    // 파생 속성을 위한 사용자 정의 획득자
    val lowPriority
        get() = priority < 3
}

@Test
fun `create task instance using apply`() {
    val task = Task("Default Task").apply { priority = 4 }
    assertEquals(4, task.priority)
}

@Test
fun `create task instance using setter`() {
    val task = Task("Valid Priority Task")
    task.priority = 4
    assertEquals(4, task.priority)
}

@Test
fun `priority range`() {
    val task = Task("Default Task").apply { priority = 2 }
    assertEquals(2, task.priority)
    assertTrue(task.lowPriority)

    task.priority = 6
    assertEquals(5, task.priority)
    assertFalse(task.lowPriority)

    task.priority = 0
    assertEquals(1, task.priority)
    assertTrue(task.lowPriority)
}
```

## 데이터 클래스 정의하기

> 클래스를 정의할 때 data 키워드로 `equals`, `hashCode`, `toString` 등을 갖춘 엔티티를 나타낼 수 있다.

코틀린은 데이터를 담는 특정 클래스의 용도로 나타내기 위해 `data` 키워드를 제공한다.

```kotlin
data class Product(
    val name: String,
    var price: Double,
    var onSale: Boolean = false
)
```

.

data 키워드로 생성되는 함수들..

👉🏻 **주 생성자에 선언된 속성 바탕으로 생성되는 `equals`, `hashCode` 함수**
  
    ```kotlin
    @Test
    fun `check equivalence`() {
        val p1 = Product("baseball", 10.0)
        val p2 = Product("baseball", 10.0, false)

        assertEquals(p1, p2)
        assertEquals(p1.hashCode(), p2.hashCode())
    }

    @Test
    fun `create set to check equals and hashcode`() {
        val p1 = Product("baseball", 10.0)
        val p2 = Product(price = 10.0, onSale = false, name = "baseball")

        val products = setOf(p1, p2)
        assertEquals(1, products.size)
    }
    ```

👉🏻 **클래스의 속성 값을 보여주는 `toString` 함수**

👉🏻 **`copy` 함수**
- 원본과 같은 속성 값으로 시작해서 copy 함수에 제공된 **속성 값만을 변경해 새로운 객체를 생성**하는 인스턴스 메소드

```kotlin
@Test
fun `change price using copy`() {
    val p1 = Product("baseball", 10.0)
    val p2 = p1.copy(price = 12.0)
    assertAll(
        { assertEquals("baseball", p2.name) },
        { assertEquals(p2.price, 12.0) },
        { assertFalse(p2.onSale) },
    )
}
```

- copy 함수는 **얕은 복사를 수행**

```kotlin
data class OrderItem(val product: Product, val quantity: Int)

@Test
fun `data copy function is shallow`() {
    val item1 = OrderItem(Product("baseball", 10.0), 5)
    val item2 = item1.copy()

    assertAll(
        { assertTrue(item1 == item2) }, // OrderItem 인스턴스는 동등
        { assertFalse(item1 === item2) }, // copy 함수로 생성한 OrderItem 은 다른 객체
        { assertTrue(item1.product == item2.product) }, // 같은 내부 Product 인스턴스를 공유
        { assertTrue(item1.product === item2.product) } // 두 OrderItem 인스턴스에 있는 Product 는 같은 객체
    )
}
```

**👉🏻 구조 분해를 위한 `component` 함수**

```kotlin
@Test
fun `destructure using component functions`() {
    val p = Product("baseball", 10.0)

    val (name, price, sale) = p // product 구조 분해
    assertAll(
        { assertEquals(p.name, name) },
        { assertEquals(p.price, price) },
        { assertFalse(sale) }
    )
}
```

{% hint style="info" %}

**참고**

- 원할 경우 equals, hashCode, toString, copy, \_componentN\_ 함수를 자유롭게 재정의 가능
- 데이터 클래스는 기본적으로 데이터가 담긴 클래스를 손쉽게 표현
- 코틀린 표준 라이브러리에는 2-3개의 제네릭 타입 속성을 담는 Pair, Triple 이라는 데이터 클래스가 존재

{% endhint %}