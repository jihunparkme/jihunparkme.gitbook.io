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

---

## 기본 인자와 함께 도움 함수 사용하기

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
