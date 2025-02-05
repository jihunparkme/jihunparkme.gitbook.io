# CHAPTER 1. 좋은 코드

# 1장. 안정성

## Item 1. 가변성을 제한하라

`var`(쓸 수 있는 프로퍼티)를 사용하거나, `mutable` 객체를 사용하면 상태를 가질 수 있다.

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

✅  **코틀린에서 가변성 제한하기**

- 읽기 전용 프로퍼티(val)
- 가변 컬렉션과 읽기 전용 컬렉션 구분하기
- 데이터 클래스의 copy

**👉🏻 읽기 전용 프로퍼티(val)**

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

**👉🏻 가변 컬렉션과 읽기 전용 컬렉션 구분하기**

- mutable이 붙은 인터페이스는 대응되는 읽기 전용 인터페이스를 상속 받아서, 변경을 위한 메서드를 추가
  - 읽기 전용
    - Iterable
    - Collection
    - Set
    - List
  - 읽고 쓸 수 있는 컬렉션
    - MutableIterable
    - MutableCollection
    - MutableSet
    - MutableList
- 읽기 전용에서 mutable로 변경해야 한다면, 복제를 통해 새로운 mutable 컬렉션을 만드는 `list.toMutableList`를 활용
    
    ```kotlin
    val list = ListOf(1, 2, 3)
    val mutableList = list.toMutableList()
    mutableList.add(4)
    ```

**👉🏻 데이터 클래스의 copy**

- immutable 객체를 사용할 때의 장점
  - 한 번 정의된 상태가 유지되므로 코드 이해가 쉬움
  - immutable 객체는 공유했을 때도 충돌이 따로 이루어지지 않으므로, 병렬처리를 안전하게 가능
  - immutable 객체에 대한 참조는 변경되지 않으므로, 쉽게 캐싱 가능
  - immutable 객체는 방어적 복사본을 만들 필요가 없음
  - immutable 객체는 다른 객체(mutable, immutable)를 만들 때 활용하기 좋음
  - immutable 객체는 set 또는 map 키로 사용 가능
- 데이터 모델 클래스를 만들어 immutable 객체로 만드는 것이 많은 장점을 가지므로, 기본적으로 이렇게 만드는 것을 권장

    ```kotlin
    data class User {
        val name: String,
        val surname: String
    }

    var user = User("Hello", "Kotlin")
    user = user.copy(surname = "World")
    ```

.

✅ **다른 종류의 변경 가능 지점**

```kotlin
val list1: MutableList<Int> = mutableListOf()
list1 += 1 // list1.plusAssign(1) 로 변경

var list2: List<Int> = listOf()
list2 += 1 // list2 = list2.plus(1) 로 변경
```

- 첫 번째 코드는 구체적인 리스트 구현 내부에 변경 가능 지점이 존재 (멀티스레드 처리에 위험)
- 두 번째 코드는 프로퍼티 자체가 변경 가능 지점 (멀티스레드 처리에 안정성)

mutable 리스트 대신 mutable 프로퍼티를 사용하는 형태는 사용자 정의 setter 를 활용해 변경 추적이 가능

```kotlin
var names by Delegates.observable(listOf<String>()) { _, old, new -> 
    println("Names changed from $old to $new)
}

names += "Hello" // Names changed from [] to [Hello]
```

- mutable 프로퍼티에 읽기 전용 컬렉션을 넣어 사용하는 것이 쉽다.
  - 이렇게 하면 여러 객체를 변경하는 여러 메서드 대신 setter 를 사용하면 되고, 이를 private 로 만들 수 있다.

```kotlin
var announcements = listOf<Announcement>()
    private set
```

- 최악의 방식은 프로퍼티와 컬렉션 모두 변경 가능한 지점으로 만드는 것

```kotlin
var list3 = mutableListOf<Int>() // XXX!
```

**⚠️ 상태를 변경할 수 있는 불필요한 방법은 만들지 말자 !**

.

✅ **변경 가능 지점 노출하지 말기**

- 첫 번째는 리턴이 되는 mutable 객체를 복제하는 것 (방어적 복제 / defensive copying)

    ```kotlin
    class UserHolder {
        private val user: MutableUser()

        fun get(): MutableUser {
            return user.copy()
        }
    }
    ```

- 컬렉션은 객체를 읽기 전용 슈퍼타입으로 업캐스트하여 가변성을 제한 가능

    ```kotlin
    data class User(val name: String)

    class UserRepository {
        private val storeUsers: MutableMap<Int, String> = mutableMapOf()

        fun loadAll(): Map<Int, String> {
            return storeUsers
        }
    }
    ```

📖 **정리**

> ⚠️ 가변성 제한을 위해 코틀린이 제공하는 다양한 도구를 활용해 가변 지점을 제한하며 코드를 작성하자
>
> 이 때 활용할 수 있는 몇 가지 규칙들
>
> 1. var 보다 val 사용하기
> 2. mutable 프로퍼티보다는 immutable 프로퍼티 사용하기
> 3. mutable 객체와 클래스보다 immutable 객체와 클래스 사용하기
> 4. 변경이 필요한 대상을 만들어야 한다면, immutable 데이터 클래스로 만들고 copy 활용하기
> 5. 컬렉션에 상태를 저장해야 한다면, mutable 컬렉션보다 읽기 전용 컬렉션을 사용하기
> 6. 변이 지점을 적절하게 설계하고, 불필요한 변이 지점은 만들지 않기
> 7. mutable 객체를 외부에 노출하지 않기

효율성 때문에 immutable 객체보다 mutable 객체를 사용하는 것이 좋을 때가 있다.
- 추가로 immutable 사용 시 멀티스레드 때에 더 많은 주의가 필요

## Item 2. 변수의 스코프를 최소화하라

상태를 정의할 때는 변수와 프로퍼티의 스코프를 최소화하는 것이 좋다.

- 프로퍼티보다는 지역 변수를 사용하기
- 최대한 좁은 스코프를 갖게 변수를 사용하기

✅ 스코프를 좁게 만드는 것이 좋은 이유는 굉장히 많지만, 가장 중요한 이유는 프로그램을 추적하고 관리하기 쉽기 때문

📖 **정리**

> - 변수의 스코프는 좁게 만들어 활용하는 것이 좋다.
> 
> - var 보다는 val를 사용하는 것이 좋다.

## Item 3. 최대한 플랫폼 타입을 사용하지 말라

`플랫폼 타입`: 다른 프로그래밍 언어에서 전달되어 nullable 인지 아닌지 알 수 없는 타입
- 플랫폼 타입은 String! 처럼 타입 이름 뒤에 `!` 기호를 붙여 표기

```kotlin
val repo = UserRepo()
val user1 = repo.user // user1 타입 : User!
val user2: User = repo.user // user1 타입 : User
val user3: User? = repo.user // user1 타입 : User?
```

자바와 코틀린을 함께 사용할 때 가능한 자바에 `@Nullable`, `@NotNull` 을 붙여 사용하자.
- 현재 지원되는 어노테이션들
- [JvmAnnotationNames.kt](https://github.com/JetBrains/kotlin/blob/master/core/compiler.common.jvm/src/org/jetbrains/kotlin/load/java/JvmAnnotationNames.kt)

📖 **정리**

> - 다른 프로그래밍 언어에서 와서 nullable 여부를 알 수 없는 타입을 **플랫폼 타입**이라고 부름.
> 
> - 연결되어 있는 자바 생성자, 메서드, 필드에 nullable 여부를 지정하는 어노테이션을 활용하는 것이 좋음.

## Item 4. inferred 타입으로 리턴하지 말라

```kotlin
val DEFAULT_CAR: Car = Fiat126P()

// inferred(추론) 타입 (Fiat126P 이외의 자동차 생산 불가)
val DEFAULT_CAR = Fiat126P()
```

📖 **정리**

> - 타입을 확실하게 지정해야 하는 경우에는 명시적으로 타입을 지정해야 한다는 원칙만 갖기
>
> - 안전을 위해 외부 API 생성 시 반드시 타입을 지정하고, 지정한 타입을 특별한 이유와 확실한 확인 없이는 제거하지 않기
>
> - inferred 타입은 프로젝트가 진전될 때, 제한이 너무 많아지거나 예측하지 못한 결과를 낼 수 있다.

## Item 5. 예외를 활용해 코드에 제한을 걸어라

확실하게 어떤 형태로 동작해야 하는 코드가 있다면, 예외를 활용해 제한을 거는 것이 좋다.
- `require` block: 아규먼트 제한
- `check` block: 상태와 관련된 동작 제한
- `assert` block: 어떤 것이 true 인지 확인(테스트 모드에서만 작동)
- `return` or `throw`와 함께 활용하는 Elvis 연산자
  - 예상하지 못한 동작을 하지 않고 예외를 throw
  - 코드가 어느정도 자체적으로 검사

👉🏻 **아규먼트**

```kotlin
fun something(n: Int): Long {
    require(n >= 0) { "Cannot calculate.. of $n .." }
}
```

- 함수 정의 시 타입 시스템을 활용해서 아규먼트에 제한을 거는 코드를 많이 사용
- `require` 함수는 제한을 확인하고, 제한을 만족하지 못할 경우 `IllegalArgumentException` throw

👉🏻 **상태**

```kotlin
fun something(text: String) {
    check(isInitialized)
    ..
}

fun something2(): User {
    checkNotNull(token)
    ...
}
```

- 상태와 관련된 제한을 걸 경우 일반적으로 `check` 함수 사용
- 지정된 예측을 만족하지 못할 경우 `IllegalStateException` throw

👉🏻 **Assert 계열 함수**

```kotlin
fun something(num: Int = 1): List<T> {
    ...
    assert(ret.size == nul)
    return ret
}
```

- 테스트에서만 활성화. 단위 테스트 대신 함수에서 활용 가능
  - 코드를 자체 점검하며, 더 효율적으로 테스트 가능
  - 특정 상황이 아닌 모든 상황에 대한 테스트 가능
  - 실행 시점에 정확하게 어떻게 되는지 확인 가능
  - 실제 코드가 더 빠른 시점에 실패하도록 만듦
- 심각한 오류이고 심각한 결과를 초래할 수 있다면 `check`를 활용하자.

✅ nullability와 스마트 캐스팅

제한을 통해 타입을 비교했다면 스마트 캐스트가 작동

```kotlin
fun something(person: Person) {
    require(person.outfit is Dress)
    val dress: Dress = person.outfit
}
```

`requireNotNull`, `checkNotNull` 특수 함수를 사용하여 unpack 용도로 활용 가능

```kotlin
val email = requireNotNull(person.email)
validateEmail(email)
...
```

nullability 목적으로, 오른쪽에 `throw` 또는 `return`을 두고 `Elvis` 연산자 활용 가능

```kotlin
val email: String = person.email ?: return
..
```

null일 때 여러 처리가 필요한 경우에도 `return/throw`, `run` 함수를 조합

```kotlin
val email: String = person.email ?: run {
    log("...")
    return
}
```

📖 **정리**

> 제한의 이점
> 
>  - 제한을 더 쉽게 확인
>
> - 애플리케이션을 더 안정적으로 방어
>
> - 코드를 잘못 쓰는 상황 예방
>
> - 스마트 캐스팅 활용
>
> 제한 메커니즘
>
> - `require`: 아규먼트와 관련된 예측을 정의할 때 사용하는 범용적인 방법
>
> - `check`: 상태와 관련된 예측을 정의할 때 사용하는 범용적인 방법
>
> - `assert`: 테스트 모드에서 데스트할 때 사용하는 범용적인 방법
> 
> - `return`, `throw`, `Elvis` 연산자 사용

## Item 6. 사용자 정의 오류보다는 표준 오류를 사용하라

```kotlin
inline fun <reified T> String.readObject(): T {
    //...
    if (incorrectSign) {
        throw JsonParsingException()
    }
    //...
    return result
}
```

직접 오류를 정의하는 것보다 최대한 표준 라이브러리의 오류를 사용하는 것이 좋다.
- 다른 사람들이 API를 더 쉽게 배우고 이해
- IllegalArgumentException
- IllegalStateException
- IndexOutOfBoundException
- ConcurrentModificationException: 동시 수정 금지에도 발생 시
- UnsupportedOperationException: 사용하려는 메서드가 현재 객체에서 사용 불가
- NoSuchElementException: 사용하려는 요소가 존재하지 않음

## Item 7. 결과 부족이 발생할 경우 null, Failure 사용하라

함수가 원하는 결과를 만들어 낼 수 없을 때 처리하는 메커니즘
- 예외 throw
- null 또는 실패를 나타내는 sealed 클래스(일반적으로 Failure)

👉🏻 **두 메커니즘의 중요한 차이점**

1️⃣ **예외**
- 예외는 정보를 전달하는 방법으로 사용해서는 안 된다.
  - 예외는 잘못된 특별한 상황을 나타내야 하며, 처리되어야 한다.
  - 예외는 예외적인 상황이 발생했을 때 사용하는 것을 권장

2️⃣ **null과 Failure**
- 충분이 예측할 수 있는 범위의 오류는 `null`, `Failure를` 사용하고
- 예측하기 어려운 예외적인 범위의 오류는 `예외`를 throw해서 처리하는 것을 권장
- null을 처리해야 한다면 사용자는 `safe call` 또는 `Elvis` 연산자 같은 다양한 널 안정성 기능 활용 가능
  - 일반적으로 `getOrNull` 또는 `Elvis` 연산자 사용

```kotlin
// null 사용
inline fun <reified T> String.readObjectOrNull(): T? {
    //...
    if (incorrectSign) {
        return null
    }
    //...
    return result
}

// Failure 사용
inline fun <reified T> String.readObject(): Result<T> {
    //...
    if (incorrectSign) {
        return Failure(JsonParsingException())
    }
    //...
    return Success(result)
}

sealed class Result<out T>
class Success<out T>(val result: T): Result<T>()
class Failure(val throwable: Throwable): REsult<Nothing>()
class JsonParsingException: Exception()
```

ℹ️ Result같은 공용체를 리턴하기로 했다면, when 표현식을 사용해서 처리 가능
- try-catch 블록보다 효율적이며 사용하기 쉽고 더 명확

```kotlin
val person = userText.readObjectOrNull<Person>()
val age = when(person) {
    is Success -> person.age
    is Failure -> -1
}
```

ℹ️ `null` 값과 `sealed result` 클래스의 차이점
- 추가적인 정보를 전달해야 한다면 `sealed result`
- 그렇지 않으면 null 사용

📖 **정리**

> - 개발자는 항상 자신이 요소를 안전하게 추출할 것이라고 생각한다. 따라서, nullable을 리턴해서는 안 된다.
>
> - 개발자에게 null이 발생할 수 있다는 경고를 주려면, getOrNull 등을 사용해서 무엇이 리턴되는지 예측할 수 있게 하자.


138