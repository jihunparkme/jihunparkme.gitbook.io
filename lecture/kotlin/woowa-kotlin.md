# 코프링 매우 알은 체하기

아래 테크 세미나 내용을 요약한 내용입니다.
- [어디 가서 코프링 매우 알은 체하기!](https://www.youtube.com/watch?v=ewBri47JWII)
- [고품격 Kotlin 개발: 테스트 코드를 우아하게 작성하는 방법](https://www.youtube.com/watch?v=PqA6zbhBVZc)

Kopring Sample Repository

- [woowacourse/service-apply](https://github.com/woowacourse/service-apply)

# Kotlin?

코틀린은 멀티 플랫폼 언어

<p align="center" width="100%">
    <img src="../../.gitbook/assets/kotlin/kotlin-1.png" width="40%">
    <img src="../../.gitbook/assets/kotlin/kotlin-2.png" width="40%">
</p>

**코딩 컨벤션**

- [Coding conventions](https://kotlinlang.org/docs/coding-conventions.html)
- [Kotlin 스타일 가이드](https://developer.android.com/kotlin/style-guide?hl=ko)

**커밋 메세지**

- [Commit Message Conventions](https://gist.github.com/stephenparish/9941e89d80e2bc58a153)

## ktlint

코드의 컨벤션 규약

- ktlint는 `Kotlin Coding Convention`과 `Android Kotlin Style Guide`를 기본으로 따르고 있다.

```kotlin
plugins {
    ...
    id("org.jlleitschuh.gradle.ktlint") version "10.3.0" // Kotlin 코드 스타일을 자동으로 검사하고 포맷팅하는 도구   
}
```

ktlint: 코틀린 style, convention 가이드 적용

- ⭐️ IntelliJ IDEA formatter를 ktlint에 맞게 설정(해당 프로젝트만):
    - `$ ./gradlew ktlintApplyToIdea`
- IntelliJ 사용 모든 프로젝트에 formatter 적용(모든 IDEA 프로젝트에):
    - `$ ./gradlew ktlintApplyToIdeaGlobally`
- 수동으로 ktlint를 이용하여 스타일 체크: `$ ./gradlew clean ktlintCheck`
    - ktlInt Check: `Tasks  → verification → ktlintCheck`
- ⭐️ Git hook을 통해 ktlint 설정: 커밋 전에 ktlintCheck 테스트 실행
    
    ```bash
    $ mkdir .git/hooks
    $ ./gradlew addKtlintCheckGitPreCommitHook
    ```
    

# Kotlin/JVM

## **Basic**

<figure><img src="../../.gitbook/assets/kotlin/kotlin-jvm.png" alt=""><figcaption></figcaption></figure>

코틀린은 안전하게 코딩하도록 의식하지 않아도 안전하게 코딩되도록 지원

- @NotNull, @Nullable, final 자동 적용
- 안전하게 코딩하고 싶지 않을 경우 의식적으로 코딩

---

## 코틀린 컴파일

코틀린 코드만 사용할 경우

<figure><img src="../../.gitbook/assets/kotlin/kotlin-compile-1.png" alt=""><figcaption></figcaption></figure>

코틀린, 자바 코드를 함께 사용할 경우

- 롬복에서 생성된 자바 코드는 코틀린 코드에서 접근 불가

<figure><img src="../../.gitbook/assets/kotlin/kotlin-compile-2.png" alt=""><figcaption></figcaption></figure>

---

## **Item 1. 코틀린 표준 라이브러리**

**코틀린 표준 라이브러리를 익히고 사용하기**

> 코틀린에서 자바와 관련된 import를 최대한 제거하려고 노력하자
> 

표준 라이브러리를 사용하면 그 코드를 작성한 전문가의 지식과 여러분보다 앞서 사용한 다른 프로그래머들의 경험을 활용할 수 있다.

- AS-IS
    
    ```kotlin
    import java.util.Random
    import java.util.concurrent.ThreadLocalRandom
    
    Random.nextInt()
    ThreadLocalRandom.current().nextInt()
    ```
    
- TO-BE
    
    ```kotlin
    import kotlin.random.Random
    
    Random.nextInt() // thread safe
    ```
    

코틀린은 읽기 전용 컬렉션과 변경 가능한 컬렉션을 구별해 제공

- 인터페이스를 만족하는 실제 컬렉션이 반환
- 따라서 플랫폼별 컬렉션을 사용 가능

---

## **Item 2. 자바로 역컴파일**

**자바로 역컴파일하는 습관 들이기**

> 코틀린 숙련도를 높이는 가장 좋은 방법은 코틀린으로 작성한 코드가 자바로 어떻게 표현되는지 확인하기
> 
- 역컴파일로 예기치 않은 코드 생성을 방지
- 기존 자바 라이브러리와 프레임워크를 사용하며 문제가 발생할 때 빠르게 확인 가능

Kotlin to Java

- Tool > Kotlin > Show Kotlin Bytecode > Decompile

Java to Kotlin

- Convert Java File to Kotlin File

---

## **Item 3. 롬복대신 데이터 클래스**

롬복 대신 데이터 클래스 사용하기

- 데이터 클래스를 사용하면 컴파일러가 `equals()`, `hashCode()`, `toString()`, `copy()` 등을 자동 생성
    - 주 생성자의 매개변수를 기준으로 생성하고, 주 생성자에는 하나 이상의 매개변수가 있어야 함
    - 매개변수는 val 또는 var로 표시
- 데이터 클래스에 property 를 선언하는 순간 해당 property 는 field, Getter, Setter, 생성자 파라미터 역할
    
    ```kotlin
    val person = Person(name = "Aaron", age = 29)
    val person2 = person.copy(age = 25)
    
    data class Person(val name: String, val age: Int)
    ```

<p align="center" width="100%">
    <img src="../../.gitbook/assets/kotlin/data-class-1.png" width="40%">
    <img src="../../.gitbook/assets/kotlin/data-class-2.png" width="40%">
</p>

---

## plugins

```groovy
plugins {
    val kotlinVersion = "1.6.21"
    id("org.springframework.boot") version "2.3.3.RELEASE" // 스프링 부트를 사용하기 위한 플러그인
    id("io.spring.dependency-management") version "1.0.10.RELEASE" // 스프링 부트 프로젝트에서 의존성 관리를 쉽게 할 수 있도록 지원하는 플러그인
    kotlin("jvm") version kotlinVersion // Kotlin JVM을 사용하는 프로젝트를 위한 플러그인
    kotlin("plugin.spring") version kotlinVersion
    kotlin("plugin.jpa") version kotlinVersion
    id("org.jlleitschuh.gradle.ktlint") version "10.3.0" // Kotlin 코드 스타일을 자동으로 검사하고 포맷팅하는 도구
    id("org.asciidoctor.jvm.convert") version "3.3.2" // Asciidoctor를 사용해서 문서를 생성하고 변환할 수 있도록 지원하는 플러그인
    id("org.flywaydb.flyway") version "7.12.0" // Flyway를 사용하여 데이터베이스 마이그레이션을 자동으로 관리할 수 있게 해주는 플러그인
}
```

---

# Spring Boot

## **final 클래스**

- `@SpringBootApplication`은 `@Configuration`을 포함하고, 스프링은 기본적으로 CGLIB을 사용해서 `@Configuration` 클래스에 대한 프록시를 생성
- 하지만, final 클래스는 상속이나 오버라이드가 불가하므로 프록시 생성이 불가
- 상속을 허용하고 오버라이드를 허용하려면 `open` 변경자 추가 필요
- Spring Framework 5.2 부터는 `@Configuration` 의 proxyBeanMethod 옵션을 통해 프록시 생성 비활성화 가능

## **All-open 컴파일러 플러그인**

- 코틀린은 다양한 컴파일러 플러그인을 제공
- `all-open` 컴파일러 플러그인은 지정한 애너테이션이 있는 클래스와 모든 멤버에 open 변경자를 추가
- 스프링을 사용할 경우 `all-open` 컴파일러 플러그인을 래핑한 `kotlin-spring` 컴파일러 플러그인을 사용 가능
    - `@Component`, `@Transactional`, `@Async` 등이 기본적으로 지정
- File > Project Structure > Project Settings > Modules > Kotlin > Compiler Plugins 에서 지정된 애너테이션 확인 가능

```groovy
plugins {
    kotlin('plugins.spring') version "1.5.21"
}

allOpen {
    annotation("com.my.Annotation")
}
```

---

## **Item 4. 지연 초기화**

필드 주입이 필요하면 지연 초기화를 사용하자

> 코틀린에서 `lateinit` 변경자를 붙이면 프로퍼티를 나중에 초기화할 수 있다.
나중에 초기화하는 프로퍼티는 항상 var
> 

```kotlin
@Autowired
private lateinit var objectMapper: ObjectMapper
```

---

## jackson-module-kotlin

- **jackson은 기본적으로 역직렬화 과정을 위해 매개변수가 없는 생성자가 필요하지만**
    - 코틀린에서 매개변수가 없는 생성자를 만들기 위해 모든 매개변수에 기본 인자가 필요
- `jackson-module-kotlin`은 매개변수가 없는 생성자가 없더라도 직렬화와 역직렬화를 지원
    - 코틀린은 매개변수가 없는 생성자를 만들기 위해 생성자의 모든 매개변수에 기본 인자가 필요
        
        ```kotlin
        // Unit Test의 경우 ObjectMapper 직접 호출 필요
        val mapper1 = jacksonObjectMapper()
        val mapper2 = ObjectMapper().registerKotlinModule()
        ```
        
        ```
        dependencies {
            implementation("com.fasterxml.jackson.module:jackson-module-kotlin")
        }
        ```
        

---

## Kotlin Annotation

<figure><img src="../../.gitbook/assets/kotlin/annotation.png" alt=""><figcaption></figcaption></figure>

데이터 클래스에 property를 선언하는 순간 `field`, `Getter`, `Setter`, `생성자 파라미터` 역할 수행

- `@param.JsonProperty` : parameter에 사용될 이름
- `@get.JsonProperty` : getter에 사용될 이름

---

## **Item 5. 변경 가능성 제한**

변경 가능성을 제한하자

> 코틀린 클래스와 멤버가 final인 것처럼 일단 val로 선언하고 필요 시 var로 변경하자.
> 
- 생성자 바인딩을 사용하려면 `@EnableConfigurationProperties`
    - 또는 `@ConfigurationPropertiesScan`  사용
    
    ```kotlin
    @ConfigurationProperties("application")
    @ConstructorBinding
    data class ApplicationProperties(val url: String)
    
    @ConfigurationPropertiesScan
    @SpringBootApplication
    class Application
    ```
    
- private property and backing property
    - 공개 API 와 구현 세부 사항 프로퍼티로 나눌 경우
    - private 프로퍼티 이름의 접두사로 밑줄을 사용(backing property)
- JVM에서는 기본 getter, setter가 있는 private 프로퍼티에 대해 함수 호출 오버헤드를 방지하도록 최적화
    
    ```kotlin
    @OneToMany(cascade = [CascadeType.PERSIST, CascadeType.MERGE], orphanRemoval = true)
    @JoinColumn(name = "session_id", nullable = false)
    private val _students: MutableSet<Student> = students.toMutableSet() //backing property
    val student: Set<Student> // public
        get() = _students
    ```
    

---

# Persistence

## No-arg 컴파일러 플러그인

- JPA에서 엔티티 클래스를 생성하려면 ***매개변수가 없는 생성자가 필요***
- `no-arg` 컴파일러 플러그인은 지정한 애너테이션이 있는 클래스에 ***매개변수가 없는 생성자를 추가***
    - JPA, Kotlin에서 직접 호출할 수 없지만 리플랙션을 사용하여 호출 가능
- `kotlin-spring Compiler Plugin`과 마찬가지로 JPA를 사용하는 경우 no-arg 컴파일러 플러그인을 래핑한 `kotlin-jpa Compiler Plugin` 사용 가능
    - JPA, Kotlin 사용 시 자동으로 `kotlin-jpa Compiler Plugin`  추가
    - `@Entity`, `@Embeddable`, `@MappedSuperclass` 가 기본적으로 지정

```groovy
plugins {
    kotlin('plugins.spring') version "1.5.21"
    kotlin('plugins.jpa') version "1.5.21"
}

allOpen { // JPA 지연로딩 적용을 위해 프록시 생성 목적으로 설정
    annotation("javax.persistence.Entity")
    annotation("javax.persistence.MappedSuperclass")
}
```

---

## **Item 6. 엔티티**

엔티티에 데이터 클래스 사용을 피하자.

> 양방향 연관 관계의 경우 `toString()`, `hashcode()` 로 무한 순환 참조가 발생
> 

---

## **Item 7. 사용자 지정 getter**

사용자 지정 getter를 사용하자.

> JPA 에 의해 인스턴스화 될 때, 초기화 블록이 호출되지 않으므로, 영속화하지 않는 필드는 사용자 지정 getter를 사용하자.
> 

```kotlin
/**
 * AS-IS: 초기화된 프로퍼티
 * 영속화하지 않을 목적이었지만, JPA 에 의해 인스턴스화 될 때
 * null을 허용하지 않았음에도 불구하고, 초기화 블록이 호출되지 않으므로 null이 들어가게 된다.
 */
@Transient
val fixed: Boolean = startDate.until(endDate).year < 1

---

/**
 * TO-BE: 사용자 지정 getter
 * 따라서, 사용자 지정 getter 를 정의해서 프로퍼티에 접근할 때마다 호출되도록 하자.
 * 뒷받침하는 필드(backing property)가 존재하지 않으므로 AccessType.FIELD 라도 @Transient 불필요
 */
val fixed: Boolean
  get() = startDate.until(endDate).year < 1 // 일종의 메서드로 생각하자
```

---

## **Item 8. Null 타입 제거**

Null이 될 수 있는 타입은 빠르게 제거하자.

> Null이 될 수 있는 타입을 사용하면 Null 검사나 !! 연산자가 필요하다.
> 
- 엔티티 클래스의 id를 0 또는 빈 문자열로 초기화해서 Null이 될 수 있는 타입을 제거하자.
- Optional 보다 Nullable 한 타입을 사용해서 불필요한 java import 를 줄이자.
    
    ```kotlin
    interface ArticleRepository : CrudRepository<Article, Long> {
        fun findBySlug(slug: String): Article? // nullable Article
        fun findAllByOrderByAddedAtDesc(): Iterable<Atricle>
    }
    
    interface UserRepository : CrudRepository<User, Long> {
        fun findByLogin(login: String): User?
    }
    ```
    
- 확장 함수를 사용해 반복되는 Null 검사를 제거할 수 있다.
    
    ```kotlin
    fun MemberRepository.getById(id: Long): Member {
        if (id == 0L) {
            return Member.SINGLE
        }
        return findByIdOrNull(id) ?: throw NoSuchElementException("존재하지 않는 아이디 입니다. id: $id")
    }
    
    interface MemberRepository : JpaRepository<Member, Long>
    ```
    

# Test

## 단위 테스트

Junit, Mockito 같은 Java 라이브러리로 테스트하므로 Kotlin 다운 코드 작성이 어려움

- 예외 테스트는 JUnit5의 `assertThrows` 및 `assertDoesNotThrow` 같은 Kotlin 함수를 사용하면 간결

```kotlin
// Java 스타일
@DisplayName("회원의 비밀번호와 다를 경우 예외가 발생한다")
@Test
fun authenticate {
    val user = createUser
    assertThatExceptionOfType(UnidentifiedUserException::class.java)
        .isThrownBy { user.authenticate(WRONG_PASSWORD) }
}

// Kotlin 스타일
@Test
fun `회원의 비밀번호와 다를 경우 예외가 발생한다`() {
    val user = createUser
    assertThrows<UnidentifiedUserException> {
        user.authenticate(WRONG_PASSWORD)
    }
}

---

// Kotest
"회원의 비밀번호와 다를 경우 예외가 발생한다" {
    val user = createUser()
    shouldThrow<UnidentifiedUserException> { user.authenticate(WRONG_PASSWORD) }
}
```

---

### 테스트 팩토리

- 테스트 픽스처를 반환하는 `팩토리 함수`를 만들 수 있다.
    - 테스트 픽스처: 테스트를 위한 전제 조건
- Kotlin 기본 인자를 사용하면 `빌더 패턴`처럼 다양한 경우를 처리 가능
- `기본 인자`와 `이름 붙인 인자`를 적절히 사용하면 테스트하려는 관심사를 드러내는 데 사용 가능

```kotlin
createMission(submittable = true) // 과제 제출물을 제출할 수 있는 과제
createMission(submittable = false) // 과제 제출물을 제출할 수 없는 과제
createMission(title = "과제1", evaluationId = 1L) // 특정 평가에 대한 과제
createMission(hidden = false) // 숨기지 않은 과제

// 관련 없는 인자에는 합리적인 기본값을 사용
fun createMission(
    title: String = MISSION_TITLE,
    description: String = MISSION_DESCRIPTION,
    evaluationId: Long = 1L,
    startDateTime: LocalDateTime = START_DATE_TIME,
    endDateTime: LocalDateTime = END_DATE_TIME,
    submittable: Boolean = true,
    hidden: Boolean = false,
    id: Long = 0L
): Mission {
    return Mission(title, description, evaluationId, startDateTime, endDateTime, submittable, hidden, id)
}
```

---

### 테스트 확장 함수

확장함수를 사용하여 값을 더 쉽게 표현

```kotlin
private val Int.ms: Long get() = milliseconds.inWholeMilliseconds

"초당 1회로 제한할 경우 1초 간격 요청은 성공한다" {
    val limiter = RateLimiter(1)
    listOf(0.ms, 1000.ms, 2000.ms).forAll {
        shouldNotThrowAny { limiter.acquire(it) }
    }
}

"초당 1회로 제한할 경우 초당 2번 요청하면 실패한다" {
    val limiter = RateLimiter(1).apply {
        acquire(0.ms)
    }
    listOf(100.ms, 500.ms, 900.ms).forAll {
        shouldThrow<ExceededRequestException> { limiter.acquire(it) }
    }
}
```

확장함수로 검증하고자 하는 부분에 가독성 향상

```kotlin
private fun ApplicantAndFormResponse.hasKeywordInNameOrEmail(keyword: String): Boolean {
    return name.contains(keyword) || email.contains(keyword)
}

...

When("특정 키워드로 특정 모집에 지원한 지원 정보를 조회하면") {
    val actual = applicantService.findAllByRecruitmentIdAndKeyword(recruitmentId, keyword)

    Then("지원 정보 및 부정행위 여부를 확인할 수 있다") {
        actual shouldHaveSize 2
        actual.filter { it.hasKeywordInNameOrEmail(keyword) } shouldHaveSize 2 // 확장함수 적용
        actual[0].isCheater.shouldBeTrue()
        actual[1].isCheater.shouldBeFalse()
    }
}
```

---

### Kotest 어설션

**👎🏻 테스트 어설션**

일부 어설션이 실패하더라도 테스트가 즉시 중지되지 않고, 끝까지 검증

- 어떤 프로퍼티 때문에 실패했는지 바로 알아채기 어려움

```kotlin
val excepted = Speaker("Aaron", "Park", 30)
val actual = Speaker("Aaron", "Kim", 28)
assertAll (
    { assertThal(actual.firstName).isEqualTo(expexted.firstName) },
    { assertThal(actual.lastName).isEqualTo(expexted.lastName) },    
    { assertThal(actual.age).isEqualTo(expexted.age) },
)
```

인스턴스를 데이터 클래스로 만들고 비교하는 것을 권장

```kotlin
val excepted = Speaker("Aaron", "Park", 30)
val actual = Speaker("Aaron", "Kim", 28)
assertThal(actual).isEqualTo(expexted)
```

주로 JUnit의 Assertions 대신 읽기 쉬운 AssertJ의 Assertions를 사용

**👍🏼 Kotest 어설션**

- Kotest 어설션은 간결하고 기존 JUnit5와 혼용 가능
- 스마트 캐스트 같은 Kotlin 기능을 지원
    - 한 번 스마트 캐스팅 되었다면 null을 다루지 않아도 된다.

```kotlin
val actual = userRepository.findByEmail("aaron@google.com")
actual.sholudNotBeNull() // 스마트 캐스트
actual.name shouldBe "박지훈"
```

**Kotlin에서 Kotest가 가장 많이 사용**

- 다양한 스타일의 테스트 레이아웃을 제공
    - 그 중에서도 StringSpec을 사용하면 애너테이션을 사용하지 않아도 된다.

```kotlin
// JUnit 5
class AuthenticationCodeTest {
    @Test
    fun `인증 코드를 생성한다`() {
        val authenticationCode = AuthenticationCode(EMAIL)
        assertAll (
            { assertThal(actual.firstName).isEqualTo(expexted.firstName) },
            { assertThal(actual.lastName).isEqualTo(expexted.lastName) },    
            { assertThal(actual.age).isEqualTo(expexted.age) },
        )
    }
    ...
}

---

// Kotest
class AuthenticationCodeTest : StringSpec({
    "인증 코드를 생성한다" {
        val authenticationCode = AuthenticationCode(EMAIL)
        assertSoftly(authenticationCode) {
            code.shouldNotBeNull()
            authenticated.shouldBeFalse()
            createdDateTime.shouldNotBeNull()
        }
    }

    "코드를 인증한다" {
        val authenticationCode = AuthenticationCode(EMAIL, VALID_CODE)
        authenticationCode.authenticate(VALID_CODE)
        authenticationCode.authenticated.shouldBeTrue()
    }
		...
})
```

## 모의 테스트

### MockK

👎🏻 Mockito

- Mockito final 클래스와 final 메서드는 모의 불가
- Kotlin 확장 함수는 Java의 정적 메서드이며 Mockito는 이를 스텁 불가
- 최상위 함수는 특정 클래스에 속하지 않기 때문에 스텁 불가

👍🏼 MockK

- DSL 기반 Kotlin 모의 라이브러리
- 코드 기반, 애너테이션 기반 등 대부분 Mockito와 동일

```kotlin
val recruitmentRepository = mockk<RecruitmentRepository>()
val recruitmentItemRepository = mockk<RecruitmentItemRepository>()

every { recruitmentRepository.save(any()) } returns recruitment
every { recruitmentItemRepository.findByRecruitmentIdOrderByPosition(any()) } returns emptyList()
every { recruitmentItemRepository.deleteAll(any()) } just Runs

slot<MethodParameter>().also { slot ->
    every { it.supportsParameter(capture(slot)) } answers {
        slot.captured.hasParameterAnnotation(LoginUser::class.java)
    }
}
```

**✅ MockK를 이용한 연쇄 스터빙**

MockK로 정적 함수를 모의하지 않고 확장 함수를 스텁하여 내부의 멤버 함수를 스텁하는 방법

- 도구가 기능을 제공한다고 해서 항상 사용해야 하는 것은 아니다
- 가짜 객체를 사용하는 것도 좋은 방법

```kotlin
every { recruitmentRepository.getById(any()) } returns createRecruitment(id = recruitmentId)

fun RecruitmentRepository.getById(id: Long): Recruitment = findByIdOrNull(id)
    ?: throw NoSuchElementException("모집이 존재하지 않습니다. id: $id")

// CrudRepositoryExtensions
fun <T, ID> CrudRepository<T, ID>.findByIdOrNull(id: ID): T? =
    findById(id).orElse(null)
```

👎🏻 Junit 5 + Mockito

<figure><img src="../../.gitbook/assets/kotlin/test-1.png" alt=""><figcaption></figcaption></figure>


👍🏼 Kotest + MockK

<figure><img src="../../.gitbook/assets/kotlin/test-2.png" alt=""><figcaption></figcaption></figure>


---

### Kotest를 이용한 모의 테스트

BDD를 지원하는 BehaviorSpec, DescribeSpec이 존재

- 테스트가 하나의 문서로서 더욱 풍부
- 중첩 테스트는 테스트 수명 주기를 이해하는 것이 중요

Kotest 수명주기

- Given, When, Then 을 이루는 모든 것을 beforeSpec, afterSpec으로 listen
- Given, When, Then 만을 본다면 Container 라고 부르고
    - beforeContainer, afterContainer 메서드로 수명주기 관리 가능
- Then 은 beforeEach, afterEach 메서드로 수명주기 관리 가능

```kotlin
Given("특정 모집에 대한 임시 지원서를 작성한 지원자가 있고 모집 항목이 있는 경우") {
  ...
  When("미제출 항목이 있는 상태에서 지원서를 최종 제출하면") {
     ..
     Then("예외가 발생한다") {
				...
      }
  }

  When("항목이 비어 있는 상태에서 지원서를 최종 제출하면") {
			...
      Then("예외가 발생한다") {
				...
      }
  }
}

Given("특정 회원이 특정 모집에 대해 작성한 임시 지원서가 있는 경우") {
		...
    When("해당 지원서를 조회하면") {
				...
        Then("지원서를 확인할 수 있다") {
					...
        }
    }
}
```

---

### Kotest의 격리 모드

Kotest는 `SingleInstance`, `InstancePerLeaf`, `InstancePerTest` 세 가지 값이 존재

- 기본은 `SingleInstance` 이며 테스트 상황에 맞게 격리 모드를 선택
- ex. 테스트 클래스 전체에 모의 객체를 만드는 데 비용이 많이 든다면 `SingleInstance`를 선택하고, clearMocks()을 호출하는 것이 더 나을 수 있다.

## 통합 테스트

Spring 5.2부터 `@TestConstructor` 사용 시 생성자를 통한 주입이 가능

- Kotest는 Spring 통합 테스트를 지원하기 위해 `SpringExtenstion`을 제공
- 별도의 애너테이션 없이 생성자 주입이 가능하며, `SpringExtenstion`을 통해서 트랜잭션 롤백도 가능

### 인수 테스트

사용자 관점에서 기능이 올바르게 작동하는지 확인하기 위한 테스트

- 일반적으로 사용자 스토리에 따라 Given-When-Then 스타일로 작성
- 연관 관계가 복잡할수록 테스트 픽스처 생성이 어려워짐

**인수 테스트를 위한 DSL**

- 연관 관계를 도메인에 특화된 언어로 표현하고, 필요한 데이터가 생성되도록 할 수 있다.
- 코드를 처음 접하는 사람들의 도메인 학습에 많은 도움이 된다.

<figure><img src="../../.gitbook/assets/kotlin/dsl.png" alt=""><figcaption></figcaption></figure>

## 부록

### Kover

- IntelliJ, JaCoCo 에이전트를 사용하는 Kotlin 코드 커버리지 도구
- Kotlin이 생성한 바이트코드로 측정하며 인라인 함수와 같이 JaCoCo로 측정할 수 없는 영역도 측정