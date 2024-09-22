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
