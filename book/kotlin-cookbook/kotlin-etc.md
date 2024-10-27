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

---

## 파라미터화된 테스트에 data 클래스 사용하기


# 입력/출력

## use로 리소스 관리하기

---

## 파일에 기록하기

# 그 밖의 코틀린 기능

# 스프링 프레임워크

# 코루틴과 구조적 동시성
