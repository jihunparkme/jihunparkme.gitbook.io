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


# 스프링 프레임워크

# 코루틴과 구조적 동시성
