- 테스트
- 입력/출력
- 그 밖의 코틀린 기능
- 스프링 프레임워크
- 코루틴과 구조적 동시성

# 테스트

## 테스트 클래스 수명주기 설정

> Junit 5의 테스트 수명주기를 기본값인 테스트 함수당 한 번 대신
>
> 클래스 인스턴스당 한 번씩 인스턴스화 하고 싶다면,
>
> `@TestInstance` 애노테이션이나 junit-platform.properties 파일의 `lifecycle.default` 속성을 설정하자.

Junit 5는 클래스에서 `@TestInstance` 애노테이션을 사용해 수명주기를 명시할 수 있다.
- 테스트 인스턴스의 수명주기를 PER_CLASS로 설정하면 테스트 함수의 양과 상관 없이 테스트 인스턴스가 딱 하나만 생성

```kotlin
@TestInstance(TestInstance.Lifecycle.PER_CLASS) // 모든 테스트의 테스트 클래스 인스턴스가 오직 하나
class JUnit5ListTests {
    // 인스턴스화와 원소 초기화가 오직 한번
    private val strings = listOf("this", "is", "a", "list", "of", "strings")

    // 테스트 할 때마다 실행 전 다시 초기화
    private lateinit var modifiable : MutableList<Int>

    @BeforeEach
    fun setUp() {
        // 테스트 할 때마다 실행 전 다시 초기화
        modifiable = mutableListOf(3, 1, 4, 1, 5)
        println("Before: $modifiable")
    }

    @AfterEach
    fun finish() {
        println("After: $modifiable")
    }

    @Test
    fun addElementsToList() {
        modifiable.add(9)
        modifiable.add(2)
        modifiable.add(6)
        modifiable.add(5)
        assertEquals(9, modifiable.size)
    }

    @Test
    fun size() {
        println("Testing size")
        assertEquals(6, strings.size)
        assertEquals(5, modifiable.size)
    }

    @Test
    fun accessBeyondEndThrowsException() {
        println("Testing out of bounds exception")
        assertThrows<ArrayIndexOutOfBoundsException> { strings[99] }
        assertEquals(6, strings.size)
    }
}
```

`@TestInstance`를 각 테스트 클래스에 반복하는 대신 모든 테스트의 수명주기를 properties 파일에 설정 가능
- 클래스 패스(src/test/resource)에 junit-platform.properties 파일을 생성하여 테스트 수명주기 설정 가능

```properties
junit.jupiter.testinstance.lifecycle.default = per_class
```

---

## 테스트에 데이터 클래스 사용하기

> 원하는 모든 속성을 캡슐화하는 데이터 클래스 생성하기

`assertAll` 함수의 장점은 Executable 인스턴스의 단언이 1개 이상 실패하더라도 Executable 인스턴스를 모두 실행

코틀린 data 클래스에는 이미 equals 메소드가 올바르게 구현되어 있으므로 테스트 프로세스 간소화
- 단 하나의 assertion이 모든 속성을 테스트

assertj에서 제공하는 contains 메소드를 이용해 컬렉션의 모든 원소를 확인할 수 있다.

```kotlin
data class Book(
    val isbn: String,
    val title: String,
    val author: String,
    val published: LocalDate
)

...

@Test
fun `check all elements in list`() {
    val badBook = books[2].copy(title = "Modern Java Cookbook")
    val found = arrayOf(books[2], books[0], books[1]) // Add badBook to see the assertion fail
    val expected = books
    assertThat(found).contains(*expected)
}
```

---

## 기본 인자와 함께 도움 함수 사용하기

> 각 인자에 기본값을 제공하도록 클래스를 수정하지 말고
>
> 기본값을 생성하는 팩토리 함수를 추가하자.

최상위 레벨의 유틸리티 클래스에 팩토리 함수를 위치시키면 테스트에서 팩토리 함수를 재활용할 수 있다.

```kotlin
fun createBook(
    isbn: String = "149197317X",
    title: String = "Modern Java Recipes",
    author: String = "Ken Kousen",
    published: LocalDate = LocalDate.parse("2017-08-26")
) = Book(isbn, title, author, published)

val mjr = createBook()

...

data class MultiAuthorBook(
    val isbn: String,
    val title: String,
    val authors: List<String>,
    val published: LocalDate
)

fun createMultiAuthorBook(
    isbn: String = "9781617293290",
    title: String = "Kotlin in Action",
    authors: List<String> = listOf("Dimitry Jeremov", "Svetlana Isakova"),
    published: LocalDate = LocalDate.parse("2017-08-26")
) = MultiAuthorBook(isbn, title, authors, published)
```

---

## 여러 데이터에 JUnit 5 테스트 반복하기

> Junit 5에는 쉼표로 구분된 값(CSV)과 팩토리 메소드가 포함된 옵션과 함께 데이터 소스를 명시할 수 있는 파라미터화된 테스트가 있다.

👉🏻 **CSV 데이터를 사용해 파리미터화된 테스트 수행**

```kotlin
@ParameterizedTest
@CsvSource("1, 1", "2, 1", "3, 2",
    "4, 3", "5, 5", "6, 8", "7, 13",
    "8, 21", "9, 34", "10, 55")
fun `first 10 Fibonacci numbers (csv)`(n: Int, fib: Int) =
    assertThat(fibonacci(n)).isEqualTo(fib)
```

Junit 5에서는 팩토리 메소드를 사용해 테스트 데이터를 생성할 수 있다.

👉🏻 **파라미터 소스로서 인스턴스 함수에 접근**

```kotlin
private fun fibnumbers() = listOf(
    Arguments.of(1, 1), Arguments.of(2, 1),
    Arguments.of(3, 2), Arguments.of(4, 3),
    Arguments.of(5, 5), Arguments.of(6, 8),
    Arguments.of(7, 13), Arguments.of(8, 21),
    Arguments.of(9, 34), Arguments.of(10, 55))
    
@ParameterizedTest(name = "fibonacci({0}) == {1}")
    @MethodSource("fibnumbers")
    fun `first 10 Fibonacci numbers (instance method)`(n: Int, fib: Int) =
        assertThat(fibonacci(n)).isEqualTo(fib)
```

테스트 수명주기가 기본 옵션인 Lifecycle.PER_METHOD 라면 테스트 데이터 소스 함수를 동반 객체 안에 위치시켜야 한다.

```kotlin
companion object {
    // needed if parameterized test done with Lifecycle.PER_METHOD
    @JvmStatic
    fun fibs() = listOf(
        Arguments.of(1, 1), Arguments.of(2, 1),
        Arguments.of(3, 2), Arguments.of(4, 3),
        Arguments.of(5, 5), Arguments.of(6, 8),
        Arguments.of(7, 13), Arguments.of(8, 21),
        Arguments.of(9, 34), Arguments.of(10, 55))
}

@ParameterizedTest(name = "fibonacci({0}) == {1}")
@MethodSource("fibs")
fun `first 10 Fibonacci numbers (companion method)`(n: Int, fib: Int) =
    assertThat(fibonacci(n)).isEqualTo(fib)
```

---

## 파라미터화된 테스트에 data 클래스 사용하기

> 입력 값과 예상 값을 감싸는 data 클래스를 만들고
>
> 만든 data 클래스 기반의 테스트 데이터를 생성하는 함수를 테스트 메소드 소스로서 사용하자.

```kotlin
@JvmOverloads
tailrec fun fibonacci(n: Int, a: Int = 0, b: Int = 1): Int =
when (n) {
    0 -> a
    1 -> b
    else -> fibonacci(n - 1, b, a + b)
}
```

👉🏻 **입력과 예상되는 출력을 담는 데이터 클래스 정의**
- 데이터 클래스는 이미 toString이 재정의되어 있으므로
- 입력과 출력 쌍을 나타내는 data 클래스를 인스턴스화하는 파라미터화된 테스트를 사용하는 테스트 메소드를 작성할 수 있다.

```kotlin
data class FibonacciTestData(val number: Int, val expected: Int)

...

@ParameterizedTest
@MethodSource("fibonacciTestData")
fun `check fibonacci using data class`(data: FibonacciTestData) {
    assertThat(fibonacci(data.number)).isEqualTo(data.expected)
}

private fun fibonacciTestData() = Stream.of(
    FibonacciTestData(number = 1, expected = 1),
    FibonacciTestData(number = 2, expected = 1),
    FibonacciTestData(number = 3, expected = 2),
    FibonacciTestData(number = 4, expected = 3),
    FibonacciTestData(number = 5, expected = 5),
    FibonacciTestData(number = 6, expected = 8),
    FibonacciTestData(number = 7, expected = 13)
)
```

# 입력/출력

## use로 리소스 관리하기

> 확실하게 리소스를 닫고 싶지만 코틀린은 자바의 try-with-resource 구문을 지원하지 않는다.
>
> kotlin.io 패키지의 use 또는 java.io.Reader의 `useLines` 확장 함수를 사용하자.

**File.useLines**

- useLines의 첫 번째 선택적 인자는 문자 집합이며 기본값은 UTF-8
- 두 번째 인자는 파일의 줄을 나타내는 Sequence를 제네릭 인자 T로 매핑하는 람다

```kotlin
public inline fun <T> File.useLines(
    charset: Charset = Charsets.UTF_8, 
    block: (Sequence<String>) -> T
): T {
    contract {
        callsInPlace(block, InvocationKind.EXACTLY_ONCE)
    }
    // BufferedReader를 생성하고 BufferedReader의 use 함수에 처리를 위임
    return bufferedReader(charset).use { block(it.lineSequence()) }
}

...

fun get10LongestWordsInDictionary() =
    File("/usr/share/dict/words").useLines { line ->
        line.filter { it.length > 20 }
            .sortedByDescending(String::length)
            .take(10)
            .toList()
    }

@Test @EnabledOnOs(OS.MAC)
internal fun `10 longest words in dictionary`() {
    get10LongestWordsInDictionary().forEach { word -> println("$word (${word.length})") }
}
```

**use 함수의 시그니처**

```kotlin
public inline fun <T : Closeable?, R> T.use(block: (T) -> R): R
```

---

## 파일에 기록하기

> File 클래스의 확장 함수에는 일반적인 **자바 입출력 메소드**뿐만 아니라 **출력 스트림**과 **Writer를 리턴하는 확장 함수**가 있다.

`forEachLine` 함수를 사용해서 파일을 순회할 수 있다.
- 파일이 매우 크지 않다면 File의 `readLines`를 호출해 모든 줄이 담긴 컬렉션을 획득할 수도 있다.
- `useLines` 함수를 사용해 파일의 줄마다 호출되는 함수를 제공할 수 있다.
- 파일의 크기가 작다면 `readText` 또는 `readBytes`를 사용해 전체 내용을 각각 문자열이나 바이트 배열로 읽어올 수 있다.

파일에 존재하는 내용을 모두 교체하고 싶다면 `writeText` 함수를 사용

```kotlin
File("myfile.txt).writeText("My data")
```

File 클래스에는 파일에 데이터를 추가하는 `appendText`라는 확장 함수가 있다.
- `writeText`와 `appendText` 함수는 `writeBytes`와 `appendBytes`에 기록 작업을 위임힌다.
- `writeBytes`와 `appendBytes`는 기록이 끝나면 `use` 함수를 사용해 파일을 확실히 닫는다.

개발자는 `OutputStreamWriter`와 `BufferedWriter`를 리턴하는 `writer`(printWriter)와 `bufferedWriter` 함수를 사용할 수도 있다.

```kotlin
File(fileName).printWriter().use { writer ->
    writer.println(data) }
```

# 그 밖의 코틀린 기능

## 코틀린 버전 알아내기

> 코트를 작성해 현재 사용 중인 코틀린 버전을 알려면,
>
> KotlinVersion 클래스 동반 객체의 CURRENT 속성을 사용하자.

👉🏻 **코틀린 버전 비교하기**

```kotlin
@Test
internal fun `comparison of KotlinVersion instances work`() {
    val v12 = KotlinVersion(major = 1, minor = 2)
    val v1341 = KotlinVersion(1, 3, 41)
    assertAll(
        { assertTrue(v12 < KotlinVersion.CURRENT) },
        { assertTrue(v1341 <= KotlinVersion.CURRENT) },
        { assertEquals(KotlinVersion(1, 3, 41),
            KotlinVersion(major = 1, minor = 3, patch = 41)) }
    )
}

@Test
internal fun `current version is at least 1_3`() {
    assertTrue(KotlinVersion.CURRENT.isAtLeast(major = 1, minor = 3))
    assertTrue(KotlinVersion.CURRENT.isAtLeast(major = 1, minor = 3, patch = 40))
}
```

---

## 반복적으로 람다 실행하기

> 주어진 람다 식을 여러 번 실행하고 싶다면 코틀린 내장 repeat 함수를 사용하자.

```kotlin
/**
 * times: 반복할 횟수
 * action: 실행할 (Int) -> Unit 형식의 함수
 */
@kotlin.internal.InlineOnly
public inline fun repeat(times: Int, action: (Int) -> Unit) {
    contract { callsInPlace(action) }

    for (index in 0 until times) {
        action(index)
    }
}

...

/*
 * Counting: 0
 * Counting: 1
 * Counting: 2
 * Counting: 3
 * Counting: 4
 */
repeat(5) {
    println("Counting: $it")
}
```

---

## 완벽한 when 강제하기

👉🏻 **컴파일러에게 else 절을 요구하도록 강요**

```kotlin
val <T> T.exhaustive: T
    get() = this

@Test
fun `test`() {
    fun printMod3Exhaustive(n: Int) {
        when (n % 3) {
            0 -> println("$n % 3 == 0")
            1 -> println("$n % 3 == 1")
            2 -> println("$n % 3 == 2")
            else -> println("Houston, we have a problem...")
        }.exhaustive
    }

    (1..10).forEach { printMod3Exhaustive(it) }
}
```

---

## 실행 가능한 클래스 만들기

> 클래스에서 단일 함수를 간단하게 호출하고 싶다면,  
>
> 함수를 호출할 클래스에서 `invoke` 연산자 함수를 재정의하자.

```kotlin
data class Assignment(
    val name: String,
    val craft: String
)

data class AstroResult(
    val message: String,
    val number: Int,
    val people: List<Assignment>
)

...

class AstroRequest {
    companion object {
        private const val ASTRO_URL = "http://api.open-notify.org/astros.json"
    }

    // invoke 연산자 함수를 통해 실행 가능한 클래스로 사용
    operator fun invoke(): AstroResult =
        Gson().fromJson(URL(ASTRO_URL).readText(), AstroResult::class.java)
}

...

@Test
fun `get people in space`() {
    // val request = AstroRequest() // AstroRequest 클래스 인스턴스화
    // val result = request() // 함수처럼 클래스를 호출(invoke 호출)
    var result = AstroRequest()()
    assertAll(
        { assertEquals("success", result.message) },
        { assertTrue { result.number >= 0 } },
        { assertEquals(result.number, result.people.size) }
    )
    }
```

`invoke` 연산자 함수를 제공하고 클래스 레퍼런스에 괄호를 추가하면 클래스 인스턴스를 바로 실행할 수 있다.
- 원한다면 필요한 인자를 추가한 invoke 함수 중복도 추가할 수 있다.

---

## 경과 시간 측정하기

> 코드 블록이 실행되는 데 걸린 시간을 알고 싶다면, measureTimeMillis 또는 measureNanoTime 함수를 사용하자.

**measureTimeMillis 함수의 구현**

```kotlin
public inline fun measureTimeMillis(block: () -> Unit): Long {
    contract {
        callsInPlace(block, InvocationKind.EXACTLY_ONCE)
    }
    val start = System.currentTimeMillis()
    block()
    return System.currentTimeMillis() - start
}
```

👉🏻 **코드 블록의 경과 시간 측정하기**

```Kotlin
fun doubleIt(x: Int): Int {
    Thread.sleep(100L)
    println("doubling $x with on thread ${Thread.currentThread().name}")
    return x * 2
}

/**
 * This machine has 10 processors
 * doubling 1 with on thread main
 * ...
 * doubling 8 with on thread main
 * Sequential stream took 845ms
 * doubling 6 with on thread main
 * doubling 8 with on thread ForkJoinPool.commonPool-worker-2
 * ...
 * doubling 1 with on thread ForkJoinPool.commonPool-worker-6
 * Parallel stream took 106ms
 */
fun main() {
    println("This machine has ${Runtime.getRuntime().availableProcessors()} processors")

    var time = measureTimeMillis {
        IntStream.rangeClosed(1, 8)
            .map { doubleIt(it) }
            .sum()
    }
    println("Sequential stream took ${time}ms")

    time = measureTimeMillis {
        IntStream.rangeClosed(1, 8)
            .parallel()
            .map { doubleIt(it) }
            .sum()
    }
    println("Parallel stream took ${time}ms")

}
```

{% hint style="info" %}

더 정확한 성능 측정을 원한다면 오픈 JDK의 자바 마이크로벤치마크 도구[JMH, Java Microbenchmark Harness](https://github.com/openjdk/jmh)프로젝트를 사용하자.

{% endhint %}

---

## 스레드 시작하기

**스레드 확장 함수의 시그니처**

```kotlin
public fun thread(
    start: Boolean = true,
    isDaemon: Boolean = false,
    contextClassLoader: ClassLoader? = null,
    name: String? = null,
    priority: Int = -1,
    block: () -> Unit
): Thread 
```

👉🏻 **다수의 스레드를 임의의 간격으로 시작하기**

```kotlin
/**
 * Thread-0 for 0 after 2ms
 * Thread-1 for 1 after 345ms
 * Thread-2 for 2 after 370ms
 * Thread-3 for 3 after 153ms
 * Thread-4 for 4 after 138ms
 * Thread-5 for 5 after 879ms
 */
(0..5).forEach { n ->
    val sleepTime = Random.nextLong(range = 0..1000L)
    thread {
        Thread.sleep(sleepTime)
        println("${Thread.currentThread().name} for $n after ${sleepTime}ms")
    }.join()
}
```

Thread 확장 함수는 자신의 본문에서 생성한 스레드를 리턴하므로, 스레드의 join 메소드를 사용해서 모든 스레드를 순차적으로 호출하게 만들 수 있다.

```kotlin
(0..5).forEach { n ->
    thread {
    // ...
    }.join() // 리턴된 스레드의 invoke 메소드를 사용할 수 있다.
}
```

---

## TODO로 완성 강제하기

👉🏻 **TODO 함수의 구현**
- 효율성을 이유로 소스는 인라인되어 있고, 함수가 호출될 때 NotImplementedError 발생

```kotlin
public inline fun TODO(reason: String): Nothing = 
    throw NotImplementedError("An operation is not implemented: $reason")

...

fun main() {
    TODO(reason = "none, really")
}
```

---

## Random의 무작위 동작 이해하기

👉🏻 **nextInt 함수**

```kotlin
@Test
internal fun `nextInt with no args gives any Int`() {
    val value = Random.nextInt()
    assertTrue(value in Int.MIN_VALUE..Int.MAX_VALUE)
}

@Test
internal fun `nextInt with a range gives value between 0 and limit`() {
    val value = Random.nextInt(10)
    assertTrue(value in 0..10)
}

@Test
internal fun `nextInt with min and max gives value between them`() {
    val value = Random.nextInt(5, 10)
    assertTrue(value in 5..10)
}

@Test
internal fun `nextInt with range returns value in range`() {
    val value = Random.nextInt(7..12)
    assertTrue(value in 7..12)
}
```

👉🏻 **시드 값과 함께 난수 생성기 사용하기**

```kotlin
@Test
internal fun `Random function produces a seeded generator`() {
    val r1 = Random(12345)
    val nums1 = (1..10).map { r1.nextInt() }

    val r2 = Random(12345)
    val nums2 = (1..10).map { r2.nextInt() }

    // println(nums1)
    assertEquals(nums1, nums2)
}
```

# 스프링 프레임워크

코틀린으로 스프링 애플리케이션을 작성할 때 사용할 수 있는 몇 가지 기술들

## 확장을 위해 스프링 관리 빈 클래스 오픈하기

> 확장을 위해 자동으로 필요한 스프링 관리 클래스를 열어주는 코틀린 스프링 플러그인을 빌드 파일에 추가하자

코틀린은 기본적으로 **정적으로 결합**한다.
- 클래스가 open 키워드를 사용해 확장을 위한 열림으로 표시되지 않으면 메소드 **재정의 또는 클래스 확장이 불가능**하다.
- 코틀린은 이 문제를 `all-open` 플러그인으로 해결한다.
- `all-open` 플러그인은 클래스와 클래스에 포함된 함수에 명시적으로 open 키워드를 추가하지 않고 명시적인 open 애노테이션으로 클래스를 설정한다.

`kotlin-spring` 플러그인은 아래 애노테이션으로 클래스를 열도록 설정되어 있다.
- @Component
- @Async
- @Transactional
- @Cacheable
- @SpringBootTest

```groovy
implementation("org.jetbrains.kotlin.plugin.spring:org.jetbrains.kotlin.plugin.spring.gradle.plugin:2.0.21")
```

{% hint style="info" %}

`kotlin-spring`이 제공하는 것보다 더 많이 필요하다면 all-open 플러그인도 추가할 수 있지만 거의 필요 없다.

{% endhint %}

---

## 코틀린 data 클래스로 퍼시스턴스 구현하기

> data 클래스로 JPA를 사용하고 싶다면 kotlin-jpa 플러그인을 추가하자.

JPA 관점에서 data 클래스는 두 가지 문제가 있다.

1️⃣ JPA는 모든 속성에 기본값을 제공하지 않는 이상 기본 생성자가 필수지만 data 클래스는 기본 생성자가 없다.
- no-arg 플러그인은 인자가 없는 생성자를 추가할 클래스를 선택할 수 있고
- 기본 생성자 추가를 호출하는 애너테이션을 정의할 수도 있다.

kotlin-jpa 플러그인은 no-arg 플러그인을 기반으로 만들어 졌고, 아래 애노테이션으로 자동 표시된 클래스에 기본 생성자를 추가한다.
- `@Entity`
- `@Embeddable`
- `@MappedSuperclass`

```kotlin
kotlin("plugin.jpa") version "1.2.71"
```

2️⃣ val 속성과 함께 data 클래스를 생성하면 불변 객체가 생성되는데, JPA는 불변 객체와 더불어 잘 동작하도록 설계되지 않았다.
- 엔티티로 사용하고 싶은 코틀린 클래스에 (data 클래스 대신) 필드 값을 변경할 수 있게 속성에 var 타입을 사용하는 단순 클래스 사용을 추천

👉🏻 **데이터베이스 테이블로 매핑되는 코틀린 클래스**

```kotlin
@Entity
class Article(
    var title: String,
    var headline: String,
    var content: String,
    @ManyToOne var author: User,
    var slug: String = title.toSlug(),
    var addedAt: LocalDateTime = LocalDateTime.now(),
    @Id @GeneratedValue var id: Long? = null)

@Entity
class User(
    var login: String,
    var firstname: String,
    var lastname: String,
    var description: String? = null,
    @Id @GeneratedValue var id: Long? = null)
```

ref. [Building web applications with Spring Boot and Kotlin](https://spring.io/guides/tutorials/spring-boot-kotlin)

---

## 의존성 주입하기

> 코틀린 스프링은 생성자 주입을 제공하지만 필드 주입에는 lateinit var 구조를 사용해야 한다.
>
> 선택적인 빈은 널 허용 타입으로 선언한다.

클래스에서 생성자가 하나뿐이라면 스프링이 자동으로 클래스의 유일한 생성자에 모든 인자를 자동으로 오토와이어링하기 때문에 @Autowired 애노테이션을 사용할 필요가 없다.

👉🏻 **스프링으로 의존성 오토와이어링하기**

```kotlin
/** 단일 생성자를 갖는 클래스 */
@RestController
class GreetingController(val service: GreetingService) { /* ... */ }

/** 명시적으로 오토와이어링 */
@RestController
class GreetingController(@Autowired val service: GreetingService) { /* ... */ }

/** 오토와이어링 생성자 호출. 주로 다수의 의존성을 갖는 클래스 */
@RestController
class GreetingController @Autowired constructor(val service: GreetingService) { 
    // ...
}

/** 필드 주입(비추천하지만 유용할 수 있다) */
@RestController
class GreetingController {
    @Autowired 
    lateinit var service: GreetingService

    // ...
}
```

클래스의 속성이 필수가 아니라면 해당 속성을 널 허용 타입으로 선언할 수 있다.

👉🏻 **선택 가능한 파라미터를 갖는 컨트롤러 함수**

```kotlin
@GetMapping("/hello")
fun greetUser(@RequestParam name: String?) = 
    Greeting(service.sayHello(name ?: "World"))

...

@DataJpaTest
class RepositoriesTests @Autowired constructor (
    val entityManager: TestEntityManager,
    val userRepository: UserRepository,
    val articleRepository: ArticleRepository) {
    // ...
}

@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
class IntegrationTests(@Autowired val restTemplate: TestRestTemplate) {
    // ...
}
```

# 코루틴과 구조적 동시성
