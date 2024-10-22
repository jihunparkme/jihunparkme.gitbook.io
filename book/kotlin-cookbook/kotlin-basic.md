# Kotlin Basic 

사용 라이브러리

```groovy
testImplementation("org.assertj:assertj-core:3.26.3")
testImplementation("org.hamcrest:hamcrest:2.2")
```

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

---

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

---

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

---

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

# 객체 지향 프로그래밍

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

---

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

---

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

---

## 지원 속성 기법

> 클래스의 속성을 클라이언트에게 노출하고 싶지만, 해당 속성을 초기화하거나 읽는 방법을 제어하고 싶다면
>
> 같은 타입의 속성을 하나 더 정의하고 사용자 정의 획득자와 설정자를 이용해 원하는 속성에 접근하자.

👉🏻 **생성 즉시 초기화되지 않도록 message 속성과 같은 타입의 널 허용 _message 속성을 추가**

```kotlin
class Customer(val name: String) {
    private var _messages: List<String>? = null // null 허용 private 속성 초기화

    val messages: List<String> // 불러올 속성
        get() { // private 함수
            if (_messages == null) {
                _messages = loadMessages()
            }
            return _messages!!
        }

    private fun loadMessages(): MutableList<String> =
        mutableListOf(
            "Initial contact",
            "Convinced them to use Kotlin",
            "Sold training class. Sweet."
        ).also { println("Loaded messages") }
}

@Test
fun `지연 로딩의 어려운 버전`() {
    // messages 를 처음 로딩(messages 를 바로 불러오려면 apply 함수를 사용)
    val customer = Customer("Fred").apply { messages }
    assertEquals(3, customer.messages.size) // messages 에 다시 접근
}
```

.

👉🏻 **lazy 대리자 함수로 쉽게 지연로딩을 구현**

```kotlin
class Customer(val name: String) {
    val messages: List<String> by lazy { loadMessages() }

    private fun loadMessages(): MutableList<String> =
        mutableListOf(
            "Initial contact",
            "Convinced them to use Kotlin",
            "Sold training class. Sweet."
        ).also { println("Loaded messages") }
}

@Test
fun `lazy 대리자를 사용한 지연 로딩`() {
    val customer = Customer("Fred")
    assertEquals(3, customer.messages.size)
}
```

---

## 연산자 중복(Overloading)

> 코틀린의 연산자 중복(overloading) 매커니즘을 사용해서 +, * 등의 연산자와 연관된 함수를 구현할 수 있다.

👉🏻 **함수 재정의**

많은 연산자가 코틀린에서 함수로 구현되어 있고, 기호를 사용하면 해당 연산자와 연관된 함수에 처리를 위임
- 모든 연산자 함수 재정의 시 `operator` 키워드는 필수(equals 제외)

```kotlin
data class Point(val x: Int, val y: Int)

operator fun Point.unaryMinus() = Point(-x, -y)

@Test
fun `Point의 unaryMinus 연산자 재정의`() {
    val point = Point(10, 20)
    assertEquals(Point(-10, -20), -point)
}
```

.

👉🏻 **확장 함수**

자신이 작성하지 않은 클래스에 연산과 관련 함수를 추가하고 싶다면 확장 함수를 사용할 수 있다.
- Complex 클래스의 이와 같은 기존 함수에 연산을 위임하는 확장 함수

```kotlin
implementation("org.apache.commons:commons-math3:3.6.1")
```

```kotlin
import org.apache.commons.math3.complex.Complex

internal class ComplexOverloadOperatorsKtTest {

    operator fun Complex.plus(c: Complex) = this.add(c)
    operator fun Complex.plus(d: Double) = this.add(d)
    operator fun Complex.minus(c: Complex) = this.subtract(c)
    operator fun Complex.minus(d: Double) = this.subtract(d)
    operator fun Complex.div(c: Complex) = this.divide(c)
    operator fun Complex.div(d: Double) = this.divide(d)
    operator fun Complex.times(c: Complex) = this.multiply(c)
    operator fun Complex.times(d: Double) = this.multiply(d)
    operator fun Complex.times(i: Int) = this.multiply(i)
    operator fun Double.times(c: Complex) = c.multiply(this)
    operator fun Complex.unaryMinus() = this.negate()

    private val first  = Complex(1.0, 3.0)
    private val second = Complex(2.0, 5.0)

    @Test
    internal fun plus() {
        val sum = first + second
        assertEquals(sum, Complex(3.0, 8.0))
    }

    @Test
    internal fun minus() {
        val diff = second - first
        assertEquals(diff, Complex(1.0, 2.0))
    }

    @Test
    internal fun negate() {
        val minus1 = -Complex.ONE
        assertThat(minus1.real).isCloseTo(-1.0, offset(0.000001))
        assertThat(minus1.imaginary).isCloseTo(0.0, offset(0.000001))
    }

    @Test
    internal fun `Euler's formula`() {
        val iPI = Complex.I * FastMath.PI
        assertTrue(Complex.equals(iPI.exp(), -Complex.ONE, 0.00001))
    }
}
```

---

## 지연 초기화 lateinit

> 널 비허용으로 선언된 클래스 속성은 생성자에서 초기화되어야 하지만, 속성에 할당할 값의 정보가 충분하지 않다면
>
> 나중 초기화를 위해 `lateinit`을 사용할 수 있다.
>
> 단, 의존성 주입의 경우 유용하지만, 일반적으로는 지연 평가 같은 대안을 권장

👉🏻 **스프링 컨트롤러 테스트**

```kotlin
@SpringBootTest(WebEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
class OfficerControllerTests {
    @Autowired // autowiring 에 의한 초기화
    lateinit var client: WebTestClient 

    @Autowired // autowiring 에 의한 초기화
    lateinit var repository: OfficerRepository

    @Before
    fun setUp() {
        repository.addTestData()
    }

    @Test
    fun `GET to route returns all offices in db`() {
        client.get().uri("route")
            ...
    }
    
    ...
}
```

.

`lateinit` 변경자는 클래스 몸체에서만 선언되고 사용자 정의 획득자와 설정자가 없는 var 속성에서만 사용할 수 있다.
- lateinit 사용 가능한 속성 타입은 널 할당이 불가능한 타입
  - 기본 타입에는 lateinit 사용 불가
- lateinit 추가 시 해당 변수가 처음 사용되기 전에 초기화 가능
  - 사용 전 초기화에 실패하면 예외 발생

👉🏻 **lateinit 속성 동작 방식**

```kotlin
class LateInitDemo {
    lateinit var name: String

    fun initializeName() {
        println("Before assignment: ${::name.isInitialized}")
        name = "World"
        println("After assignment: ${::name.isInitialized}")
    }
}

...

internal class LateInitDemoTest {

    @Test
    internal fun `초기화 전에 속성 접근 시 예외 발생`() {
        assertThrows<UninitializedPropertyAccessException> {
            LateInitDemo().name
        }
    }

    @Test
    internal fun `초기화 후 속성 접근`() {
        assertDoesNotThrow { LateInitDemo().apply { name = "Dolly" } }
    }

    @Test
    fun `속성 레퍼런스에 isInitialized 사용`() {
        // Before assignment: false
        // After assignment: true
        LateInitDemo().initializeName()
    }
}
```

{% hint style="info" %}

**lateinit, lazy 차이**

`lateinit` 변경자는 var 속성에 사용되고, `lazy` 대리자는 속성에 처음 접근할 때 평가되는 람다를 받는다.

초기화 비용은 높은데 lazy를 사용한다면 초기화는 반드시 실패한다.
- 또한, `lazy`는 val 속성에 사용할 수 있는 반면, `lateinit`은 var 속성에만 적용 가능

`lateinit` 속성은 속성에 접근할 수 있는 모든 곳에서 초기화 가능
- 객체 바깥쪽에서도 초기화 가능

{% endhint %}

---

## equals 메소드 구현

> 논리적으로 동등한 인스턴스인지 확인 가능한 equals 재정의를 위해 
>
> 레퍼런스 동등 연산자(`==`), 안전 타입 변환 함수(`as?`), 엘비스 연산자(`?:`)를 다 같이 사용

👉🏻 **Any class**

```kotlin
open class Any {
    open operator fun equals(other: Any?): Boolean
    open fun hashCode(): Int
    open fun toString(): String
}
```

equals 문법에서 equalsa 구현은 반사성(reflexive), 대칭성(symmetirc), 추이성(transitive), 일관성(consistent)이 있어야 하고, 널도 적절하게 처리할 수 있어야 한다.

.

👉🏻 **equals 구현의 좋은 예 (KotlinVersion 클래스의 equals)**
- equals 함수가 재정의되면 hashCode 함수도 재정의

```kotlin
override fun equals(other: Any?): Boolean {
    // 레퍼런스 동등성 확인
    if (this === other) return true
    // 인자를 변환하거나 널을 리턴하는 안전 타입 변환 연산자(as?) 사용
    // 엘비스 연산자(?:)로 널 체크 및 동등성 확인
    val otherVersion = (other as? KotlinVersion) ?: return false
    // 인스턴스의 속성 비교
    return this.version == otherVersion.version
}

// 완벽한 동작을 위한 hashCode 구현
override fun hashCode(): Int = version
```

.

👉🏻 **equals, hashCode 구현해 보기**

```kotlin
class Customer(val name: String) {

    override fun equals(other: Any?): Boolean {
        if (this === other) return true
        val otherCustomer = (other as? Customer) ?: return false
        return this.name == otherCustomer.name
    }

    override fun hashCode(): Int = name.hashCode()
}
```

---

## 싱글톤 생성하기

> 클래스 하나당 인스턴스를 하나만 만드려면 class 대신 object 키워드를 사용하자.

싱글톤 디자인 패턴은 특정 클래스의 인스턴스를 오직 하나만 존재하도록 메커니즘을 정의
- 클래스의 모든 생성자를 private 로 정의
- 필요하다면 해당 클래스를 인스턴스화하고 그 인스턴스 레퍼런스를 리턴하는 정적 팩토리 메소드를 제공

👉🏻 **코틀린에서 싱글톤 선언**

```kotlin
object MySingleton {
    val myProperty = 3

    fun myFunction() = "Hello"
}
```

👉🏻 **싱글톤을 위해 object를 디컴파일한 코드**

```kotlin
public final class MySingleton {
   @NotNull
   public static final MySingleton INSTANCE = new MySingleton(); // INSTANCE 속성 생성
   private static final int myProperty = 3;

   private MySingleton() { // private 생성자
   }

   public final int getMyProperty() {
      return myProperty;
   }

   @NotNull
   public final String myFunction() {
      return "Hello";
   }

   static {

   }
}

...

@Test
fun `코틀린에서 싱글톤 멤버에 접근하기`() {
    MySingleton.myFunction()
    MySingleton.myProperty
}

@Test
fun `자바에서 싱글톤 멤버에 접근하기`() {
    MySingleton.INSTANCE.myFunction();
    KParameter.Kind.INSTANCE.myProperty();
}
```

{% hint style="info" %}

**코틀린 object와 인자 전달**

코틀린 object는 생성자를 가질 수 없기 때문에 쉽게 인자를 전달할 수 있는 방법이 없다.

{% endhint %}

---

## Nothing에 관한 야단법석

> 코틀린의 Nothing 클래스

```kotlin
public class Nothing private constructor()
```
- `Nothing`의 인스턴스는 존재하지 않는다.
- 결코 존재할 수 없는 값을 나타내기 위해 `Nothing`을 사용할 수 있다.

.

Nothing 클래스는 아래 두 가지 상황에서 사용될 수 있다

👉🏻 **코틀린에서 예외 던지기**

```kotlin
fun doNothing(): Nothing = throw Exception("Nothing at all")
```

- 리턴 타입을 반드시 명시해야 하지만 리턴하지 않으므로 리턴 타입은 Nothing

👉🏻 **변수에 널을 할당할 때 구체적인 타입을 명시하지 않는 경우**

```kotlin
val x = null
```

- x에 대한 다른 정보가 없으므로 추론된 x의 타입은 `Nothing?`
- 코틀린에서 Nothing 클래스는 실제로 다른 모든 타입의 하위 타입

# 함수형 프로그래밍

## 알고리즘에서 fold 사용하기

> fold 함수를 사용해 시퀀스나 컬렉션을 하나의 값으로 축약할 수 있다.

`fold` 함수는 배열 또는 반복 가능한 컬렉션에 적용할 수 있는 `축약 연산`

```kotlin
inline fun <R> Iterable<T>.fold(
    initial: R,
    operation: (acc: R, T) -> R
): R
```

- `fold`는 두 개의 인자를 받는다.
  - 첫 번째는 누적자의 초기값
  - 두 번째는 두 개의 인자를 받아 누적자를 위해 새로운 값을 리턴하는 함수

👉🏻 **fold를 사용해 정수의 합 계산하기**

```kotlin
@Test
fun `sum`() {
    // 초기값은 0이고, 2개의 인자를 받는 람다 함수를 제공
    fun sum(vararg nums: Int) =
            nums.fold(0) { acc, n ->
                println("acc = $acc, n = $n")
                acc + n
            }

    /**
     * acc = 0, n = 3
     * acc = 3, n = 1
     * acc = 4, n = 4
     * acc = 8, n = 1
     * acc = 9, n = 5
     * acc = 14, n = 9
     */
    val numbers = intArrayOf(3, 1, 4, 1,5 , 9)
    assertEquals(numbers.sum(), sum(*numbers))
}
```

👉🏻 **fold를 사용해 팩토리얼 구현하기**

```kotlin
@Test
fun `factorial`() {
    fun factorialFold(n: Long): BigInteger =
        when(n) {
            0L, 1L -> BigInteger.ONE // n이 0 or 1 일 경우
            else -> (2..n).fold(BigInteger.ONE) { acc, i ->
                acc * BigInteger.valueOf(i)
            }
        }

    assertEquals(BigInteger.valueOf(24), factorialFold(4))
}
```

👉🏻 **fold를 사용해 피보나치 수 계산하기**

```Kotlin
@Test
fun `fibonacci`() {
    fun fibonacci(n: Long) =
        (2 until n).fold(1 to 1) { (prev, curr), _ ->
            curr to (prev + curr) }.second

    // 1, 1, 2, 3, 5, 8 ,13, 21, 34, 55
    assertEquals(55, fibonacci(10))
}
```

## reduce 함수를 사용해 축약하기

> 비어 있지 않는 컬렉션의 값을 축약하고 싶지만 누적자의 초기값을 설정하고 싶지 안다면,
>
> fold 대신 reduce 연산을 사용

`reduce` 함수는 `fold` 함수와 사용 목적도 같고 비슷하다.
- 큰 차이점은 `reduce` 함수에는 누적자의 초기값 인자가 없는 것
- 따라서 누적자의 초기값은 컬렉션의 첫 번째 값으로 초기화

```kotlin
inline fun <S, T : S> Iterable<T>.reduce(
    operation: (acc: S, T) -> S
): S
```

`reduce` 함수는 누적자를 컬렉션의 첫 번째 값으로 초기화할 수 있는 경우에만 사용 가능
- 인자와 함께 함수가 호출되면 첫 번째 인자가 누적자로 초기화되며 다른 값들은 한 번에 하나씩 누적자에 더해진다.
- 인자 없이 호출될 경우 예외를 던진다.

👉🏻 **reduce로 구현한 sum 함수**

```kotlin
@Test
fun `sum using reduce`() {
    fun sumReduce(vararg nums: Int) =
        nums.reduce {acc, i -> acc + i }

    val numbers = intArrayOf(3, 1, 4, 1, 5 ,9)
    assertAll(
        { assertEquals(numbers.sum(), sumReduce(*numbers)) }, // Int 배열 검증
        { assertThrows<UnsupportedOperationException> { // 인자가 없어서 예외 발생
            sumReduce()
        }
    })
}
```

{% hint style="info" %}

**reduce 사용**

컬렉션의 첫 번째 값으로 누적자를 초기화하고 컬렉션의 다른 값에 추가 연산을 필요로 하지 않는 경우에만 reduce를 사용하자.

{% endhint %}

## 꼬리 재귀 적용하기

> 재귀 프로세스를 실행하는 데 필요한 메모리를 최소화하고 싶다면, `tailrec` 키워드를 추가하자.

꼬리 재귀로 알려진 접근법은 콜 스택에 새 스택 프레임을 추가하지 않게 구현하는 특별한 종류의 재귀이다.

👉🏻 **꼬리 호출 알고리즘을 사용하는 팩토리얼 구현**
- 팩토리얼 연산의 누적자 역할을 하는 두 번째 인자가 필요
- 마지막 평가 식은 더 작은 수와 증가된 누적자를 이용해 자신 스스로를 호출

```kotlin
@JvmOverloads // 자바에서 오직 1개의 인자로 호출할 수 있도록
    tailrec fun factorial(n: Long, acc: BigInteger = BigInteger.ONE): BigInteger =
        when (n) {
            0L -> BigInteger.ONE
            1L -> acc
            else -> factorial(n - 1, acc * BigInteger.valueOf(n)) // 꼬리 재귀 호출
        }

...

@Test
fun `factorial tests`() {
    assertAll(
        { assertThat(factorial(0), `is`(BigInteger.ONE)) },
        { assertThat(factorial(1), `is`(BigInteger.ONE)) },
        { assertThat(factorial(2), `is`(BigInteger.valueOf(2))) },
        { assertThat(factorial(5), `is`(BigInteger.valueOf(120))) },
        // ...
        { assertThat(factorial(15000).toString().length, `is`(56130)) },
        { assertThat(factorial(75000).toString().length, `is`(333061)) },
    )
}
```

tailrec 변경자를 적용할 수 있는 함수의 자격 요건
- 해당 함수는 반드시 수행하는 마지막 연산으로서 자신을 호출해야 한다.
- try/catch/finally 블록 안에서는 tailrec을 사용할 수 없다.
- 오직 JVM 백엔드에서만 꼬리 재귀가 지원된다.

{% hint style="info" %}

**tailrec 키워드**

컴파일러에게 해당 재귀 호출을 최적화해야 한다고 알려준다.

자바에서 똑같은 알고리즘을 재귀로 작성하면 메모리 제약이 있다.

{% endhint %}