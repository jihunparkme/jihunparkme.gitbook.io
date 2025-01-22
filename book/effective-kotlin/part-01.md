# CHAPTER 1. 좋은 코드

# 1장. 안정성

## Item 1. 가변성을 제한하라

> `var`(쓸 수 있는 프로퍼티)를 사용하거나, `mutable` 객체를 사용하면 상태를 가질 수 있다.

```kotlin
var a = 10
var list: MutableList<Int> = mutableListOf()
```

시간의 변화에 따라서 변하는 요소를 표현할 수 있다는 것은 유용하지만, 상태를 적절하게 관리하는 것은 생각보다 어렵다.
- 프로그램을 이해하고 디버그하기 힘듦
- 가변성이 있으면, 코드의 실행을 추론하기 어려움
- 멀티스레드 프로그램일 경우 적절한 동기화가 필요
- 테스트하기 어려움
- 상태 변경이 일어날 때, 변경을 다른 부분에 알려야 하는 경우가 발생

.

**코틀린에서 가변성 제한하기**

- 읽기 전용 프로퍼티(val)
- 가변 컬렉션과 읽기 전용 컬렉션 구분하기
- 데이터 클래스의 copy

👉🏻 읽기 전용 프로퍼티(val)

- 마치 값 처럼 동작
- 일반적인 방법으로는 값이 변하지 않음(읽고 쓸 수 있는 프로퍼티는 var로 생성)
- 읽기 전용 프로퍼티가 mutable 객체를 담고 있다면 내부적으로 변할 수는 있음

    ```kotlin
    val list = mutableListOf(1, 2, 3)
    ```

- 읽기 전용 프로퍼티는 다른 프로퍼티를 활용하는 사용자 정의 게터로도 정의 가능

    ```kotlin
    var name: String = "Hello"
    var surname: String = "World"
    // var 프로퍼티를 사용하는 val 프로퍼티는 var 프로퍼티가 변할 때 변경 가능
    val fullName
        get() = "$name $surname"
    ```

- var는 getter, setter 모두 제공하지만, val은 변경 불가능하므로 getter만 제공
  - val을 var로 오버라이드 가능

    ```kotlin
    interface Element {
        var active: Boolean
    }

    class ActualElement: Element {
        override var active: Boolean = false
    }
    ```

- 만일 완전히 변경할 필요가 없다면 final 프로퍼티 사용하기

👉🏻 가변 컬렉션과 읽기 전용 컬렉션 구분하기

👉🏻 데이터 클래스의 copy








75