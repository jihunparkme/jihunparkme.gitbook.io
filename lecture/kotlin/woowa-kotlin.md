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
    
