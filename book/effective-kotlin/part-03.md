# CHAPTER 3. 효율성

# 7장. 비용 줄이기

비용이 크게 들어가지는 않지만 프로그램을 효율적으로 만들 수 있는 최적화 방법

## Item 45. 불필요한 객체 생성 피하기

불필요한 객체 생성을 피하는 것이 최적화의 관점에서 좋다.

.

👉🏻 **객체 생성 비용은 항상 클까?**

- 어떤 객체를 wrap 하면, 크게 세 가지 비용이 발생
  - 객체는 더 많은 용량을 차지
  - 요소가 캡슐화되어 있다면, 접근에 추가적인 함수 호출이 필요
  - 객체는 생성되어야 함

.

👉🏻 **객체 선언**
- 매 순간 객체를 생성하지 않고, 객체를 재사용하는 간단한 방법은 객체 선언을 사용하는 것(싱글톤)

.

👉🏻 **캐시를 활용하는 팩토리 함수**
- 팩토리 함수는 캐시를 가질 수 있다.
- 실제 stdlib의 emptyList는 이를 활용해서 구현

    ```kotlin
    fun <T> List<T> emptyList() {
        return EMPTY_LIST;
    }
    ```
- 모든 순수 함수는 캐싱을 활용할 수 있는데, 이를 메모이제이션이라고 부름
- 참고로, 메모리가 필요할 때 가비지 컬렉터가 자동으로 메모리를 해제해 주는 `SoftReference`를 사용하면 좋다.
- 캐시는 언제나 메모리와 성능의 트레이드 오프가 발생하므로, 캐시를 잘 설계하는 것은 쉽지 않다.

.

👉🏻 **무거운 객체를 외부 스코프로 보내기**
- 함수가 한 파일에 다른 함수와 함께 있을 때, 함수를 사용하지 않는다면 지연 초기화(lazy initialization)를 적용하여 프로퍼티를 지연되게 만들어서 무거운 클래스 사용을 유용하게 만들 수 있다.

.

👉🏻 **지연 초기화**

```kotlin
class A {
    val b by lazy { B() }
    val c by lazy { C() }
    val d by lazy { D() }

    //...
}
```

- 이처럼 지연되게 만들면, 첫 번째 호출 때 응답 시간이 굉장히 길것이다.
- 그래서 백엔드 애플리케이션에서 좋지 않을 수 있다.
- 또한, 지연되게 만들면 성능 테스트가 복잡해지는 문제가 있다.
- 따라서, 지연 초기화는 상황에 맞게 사용하자.

.

👉🏻 **기본 자료형 사용하기**

|코틀린의 자료형|자바의 자료형|
|---|---|
|Int|int|
|Int?|Integer|
|List<Int>|List<Integer>|

- 이를 알면 랩한 자료형 대신 기본 자료형을 사용하게 코드를 최적화 할 수 있다.
- 따라서, 굉장히 큰 컬렉션을 처리할 때 차이를 확인할 수 있다.
- 결과적으로, 코드와 라이브러리의 성능이 굉장히 중요한 부분에서만 이를 적용하자.

📖 **정리**

> 이러한 최적화에 큰 변경이 필요하거나, 다른 코드에 문제를 일으킬 수 있다면 최적화를 미루는 것도 방법이다.

## Item 46. 함수 타입 파라미터를 갖는 함수에 inline 한정자 붙이기

`inline` 한정자의 역할은 컴파일 시점에 '함수를 호출하는 부분'을 '함수의 본문'으로 대체하는 것이다.

```kotlin
repeat(10) {
    print(it)
}

// 컴파일
for (index in 0 until 10) {
    print(index)
}
```

`inline` 한정자를 사용할 때 장점
- 타입 아규먼트에 `reified` 한정자를 붙여 사용 가능
- 함수 타입 파라미터를 가진 함수가 훨씬 빠르게 동작
- 비지연 리턴 사용 가능

.

👉🏻 **타입 아규먼트를 reified로 사용할 수 있다**
- 단순 호출이 본문으로 대체되므로, reified 한정자를 지정하면, 타입 파라미터를 사용한 부분이 타입 아규먼트로 대체

```kotlin
inline fun <reified T> printTypeName() {
    print(T::class.simpleName)
}

// 사용
printTypeName<Int>()
printTypeName<Char>()
printTypeName<String>()

// 컴파일
print(Int::class.simpleName)
print(Char::class.simpleName)
print(String::class.simpleName)
```

.

👉🏻 **함수 타입 파라미터를 가진 함수가 훨씬 빠르게 동작한다**
- 모든 함수는 inline 한정자를 붙이면 조금 더 빠르게 동작한다.
  - 함수 호출과 리턴을 위해 점프하는 과정과 백스택을 추적하는 과정이 없기 때문
  - 그래서 표준 라이브러리에 있는 간단한 함수들에는 대부분 inline 한정자 사용

.

👉🏻 **비지역적 리턴을 사용할 수 있다**
- 함수 리터럴이 컴파일될 때, 함수가 객체로 래핑되어서 문제가 발생할 수 있다.
  - 함수가 다른 클래스에 위치하므로 return을 사용해서 main으로 돌아올 수 없기 때문
  - 함수가 main 함수 내부에 박히기 때문에 인라인 함수는 이런 제한이 없다.

```kotlin
fun  main() {
    repeat(10) {
        print(it)
        return
    }
}
```

.

👉🏻 **inline 한정자의 비용**
- inline 한정자는 굉장히 유용한 한정자지만, 모든 곳에 사용 불가하다.
  - 대표적인 예로, 인라인 함수는 재귀적으로 동작이 불가
  - 재귀적으로 사용하면, 무한하게 대체되는 문제가 발생
- 또한, 인라인 함수는 더 많은 가시성 제한을 가진 요소를 사용할 수 없다.
  - public 인라인 함수 내부에서는 private과 internal 가시성을 가진 함수와 프로퍼티를 사용할 수 없다.
- inline 한정자를 남용하면, 코드의 크기가 쉽게 커진다.
  - 서로 호출하는 인라인 함수가 많아지면, 코드가 기하급수적으로 증가하므로 위험하다.

.

👉🏻 **crossinline과 noinline**
- 함수를 인라인으로 만들고 싶지만, 어떤 이유로 일부 함수 타입 파라미터는 inline으로 받고 싶지 않을 경우, 아래 한정자를 사용할 수 있다.
  - `crossinline`:
    - 아규먼트로 인라인 함수를 받지만, 비지역적 리턴을 하는 함수는 받을 수 없게 만든다.
    - 인라인으로 만들지 않은 다른 람다 표현식과 조합해서 사용할 때 문제가 발생하는 경우 활용
  - `noinline`:
    - 아규먼트로 인라인 함수를 받을 수 없게 만든다.
    - 인라인 함수가 아닌 함수를 아규먼트로 사용하고 싶을 때 활용

```kotlin
inline fun requestNewToken(
    hasToken: Boolean,
    crossinline onRefresh: ()->Unit,
    noinline onGernerate: ()-Unit
) {
    if (hasToken) {
    	httpCall("get-token", onGenerate)
        // 인라인이 아닌 함수를 아규먼트로 전달하려면 noinline을 사용
    } else {
    	httpCall("refresh-token") {
            onRefresh()
            // Non-local 리턴이 허용되지 않는 컨텍스트에서 
            // inline 함수를 사용하고 싶다면 crossinline을 사용
            onGerenate()
        }
    }
}

fun httpCall(url: String, callback: ()->Unit) {
	/* ... */
}
```

📖 **정리**

> 인라인 함수가 사용되는 주요 사례
> - print 함수처럼 매우 많이 사용되는 경우
> - 타입 아규먼트로 reified 타입을 전달받는 경우
> - 함수 타입 파라미터를 갖는 톱레벨 함수를 정의해야 하는 경우
>   - ex. 컬렉션 헬퍼 함수(map, filter, flatMap, joinToString..)
>   - 스코프 함수(also, apply, let...)
>   - 톱레벨 유틸리티 함수(repeat, run, with..)

## Item 47. 인라인 클래스의 사용을 고려하라

하나의 값을 보유하는 객체도 inline으로 만들 수 있다.
- 기본 생성자 프로퍼티가 하나인 클래스 앞에 inline을 붙이면, 해당 객체를 사용하는 위치가 모두 해당 프로퍼티로 교체

```kotlin
inline class Name(private val value: String) {
    // ..
}
```

이러한 inline 클래스는 타입만 맞다면, 다음과 같이 그냥 값을 곧바로 집어 넣는 것도 허용

```kotlin
val name: Name = Name("Marcin")

//컴파일
val name: String = "Marcin"
```

inline 클래스의 메서드는 모두 정적 메서드로 만들어 진다.

```kotlin
inline class Name(private val value: String) {
    //...
    fun greet() {
        print("Hello, I am $value")
    }
}

val name: Name = Name("Marcin")
name.greet()

//컴파일
val name: String = "Marcin"
Name.`greet-impl`(name)
```

.

👉🏻 **타입 오용으로 발생하는 문제를 막을 때**

```kotlin
// AS-IS
@Entity(tableName = "grades")
class Grades(
    @ColumnInfo(name = "studentId")
    val studentId: Int,
    @ColumnInfo(name = "teacherId")
    val teacherId: Int,
    @ColumnInfo(name = "schoolId")
    val schoolId: Int
    //...
)

// TO-BE
inline class StudentId(val id: Int)
inline class TeacherId(val id: Int)
inline class SchoolId(val id: Int)

@Entity(tableName = "grades")
class Grades(
    @ColumnInfo(name = "studentId")
    val studentId: StudentId,
    @ColumnInfo(name = "teacherId")
    val teacherId: TeacherId,
    @ColumnInfo(name = "schoolId")
    val schoolId: SchoolId
    //...
)
```

Int 자료형의 값을 inline 클래스를 활용해 래핑
- ID를 사용하는 것이 굉장히 안전해지며, 컴파일할 때 타입이 Int로 대체되므로 코드를 바꿔도 별도의 문제가 발생하지 않는다.

.

👉🏻 **인라인 클래스와 인터페이스**
- 인라인 클래스도 다른 클래스와 마찬가지로 인터페이스를 구현할 수 있다.
- 하지만, 인터페이스를 구현하는 인라인 클래스는 아무런 의미가 없다.

.

👉🏻 **typealias**
- `typealias`를 사용하면 타입에 새로운 이름을 붙여줄 수 있다.
- 길고 반복적으로 사용해야 할 때 많이 유용

```kotlin
typealias ClickListener = 
    (view: View, event: Event) -> Unit

class View {
    fun addClickListener(listener: ClickListener) {}
    fun removeClickListener(listener: ClickListener) {}
    //...
}
```

하지만 typealias는 안전하지 않다.
- 단위 등을 표현하려면, 파라미터 이름 또는 클래스를 사용하자
- 이름은 비용이 적게 들고, 클래스는 안전하다
- 인라인 클래스를 사용하면, 비용과 안전이라는 두 마리 토끼를 모두 잡을 수 있다.

```kotlin
typealias Seconds = Int
typealias Millis = Int

fun getTime(): Millis = 10
fun setUpTimer(time: Seconds) {}

fun main() {
    val seconds: Seconds = 10
    val millis: Millis = seconds // 컴파일 오류가 발생하지 않는다.
    
    setUpTimer(getTime())
}
```

📖 **정리**

> 인라인 클래스를 사용하면 성능적인 오버헤드 없이 타입을 래핑할 수 있다.
>
> 인라인 클래스는 타입 시스템을 통해 실수로 코드를 잘못 작성하는 것을 막아주므로, 코드의 안정성을 향상시켜 준다.
>
> 의미가 명확하지 않은 타입, 특히 여러 측정 단위들을 함께 사용하는 경우 인라인 클래스를 꼭 활용하자.

## Item 48. 더 이상 사용하지 않는 객체의 레퍼런스를 제거하라

객체에 대한 레퍼런스를 다른 곳에 저장할 때는 메모리 누수가 발생할 가능성을 언제나 염두에 두어야 한다.

간단하게 객체를 더 이상 사용하지 않을 때, 그 레퍼런스에 null을 설정하자.

```kotlin
fun pop(): Any? {
    if (size == 0) throw EmptyStackException()
    val elem = elements[--size]
    elements[size] = null
    return elem
}
```

# 8장. 효율적인 컬렉션 처리

백엔드 애플리케이션 개발, 데이터 분석 등의 영역에서는 컬렉션 처리 최적화만 잘 해도 프로그램의 전체적인 성능이 향상된다.

## Item 49. 하나 이상의 처리 단계를 가진 경우에는 시퀀스를 사용하라

Sequence는 지연(lazy) 처리된다.
- 따라서, 시퀀스 처리 함수들을 사용하면, 데코레이터 패턴으로 꾸며진 새로운 시퀀스가 리턴
- 최종적인 계산은 `toList` 또는 `count` 등의 최종 연산이 이루어질 때 수행
- 반면, `Iterable`은 처리 함수를 사용할 때마다 연산이 이루어져 `List`가 생성

시퀀스 처리의 지연 처리는 다음과 같은 장점이 있다.
- 자연스러운 처리 순서를 유지
- 최소한만 연산
- 무한 시퀀스 형태로 사용 가능
- 각각의 단계에서 컬렉션을 만들어 내지 않음

.

👉🏻 **순서의 중요성**
- 시퀀스 처리는 요소 하나하나에 지정한 연산을 한꺼번에 적용
- 반면, 이터러블은 요소 전체를 대상으로 연산을 차근차근 적용

```kotlin
sequenceOf(1, 2, 3)
	.filter { print("F$it, "); it % 2 == 1 }
	.map { print("M$it, "); it * 2 }
	.forEach { print("E$it, ") }
//F1, M1, E2, F2, F3, M3, E6, 

println()

listOf(1, 2, 3)
	.filter { print("F$it, "); it % 2 == 1 }
	.map { print("M$it, "); it * 2 }
	.forEach { print("E$it, ") }
//F1, F2, F3, M1, M3, E2, E6,
```

.

👉🏻 **최소 연산**
- 중간 처리 단계를 모든 요소에 적용할 필요가 없는 경우 시퀀스를 사용하는 것이 좋다.
- `find`처럼 처리를 적용하고 싶은 요소를 선택하는 연산으로는 first, take, any, all, none, indexOf 가 존재

```kotlin
(1..10).asSequence()
     .filter { print("F$it, "); it % 2 == 1 }
     .map { print("M$it, "); it * 2 }
     .find { it > 5 }
// F1, M1, F2, F3, M3,

(1..10)
     .filter { print("F$it, "); it % 2 == 1 }
     .map { print("M$it, "); it * 2 }
     .find { it > 5 }
// F1, F2, F3, F4, F5, F6, F7, F8, F9, F10, M1, M3, M5, M7, M9,
```

.

👉🏻 **무한 시퀀스**
- 시퀀스는 실제 최종 연산이 일어나기 전까지 컬렉션에 어떠한 처리도 하지 않음
- 따라서, 무한 시퀀스를 만들고, 필요한 부분까지만 값을 추출하는 것도 가능
- 무한 시퀀스를 만드는 일반적인 방법은 `generateSequence` 또는 `sequence`를 사용하는 것

```kotlin
generateSequence(1) { it + 1 }
    .map { it * 2 }
    .take(10)
    .forEach { print("$it, ") } 
// 2, 4, 6, 8, 10, 12, 14, 16, 18, 20, 
```

.

👉🏻 **각각의 단계에서 컬렉션을 만들어 내지 않음**
- 표준 컬렉션 처리 함수는 각각의 단계에서 새로운 컬렉션을 만들어 낸다.
  - 일반적으로 대부분 List
- 각각의 단계에서 만들어진 결과를 활용하거나 저장할 수 있다는 것은 컬렉션의 장점이지만, 각 단계에서 결과가 만들어 지면서 공간을 차지하는 비용이 든다는 큰 단점이 존재

```kotlin
numbers
    .filter { it % 10 == 0 } // 컬렉션 하나
    .map { it * 2 } // 컬렉션 하나
    .sum()
// 전체적으로 2개의 컬렉션 생성

numbers
    .asSequence()
    .filter { it % 10 == 0 }
    .map { it * 2 }
    .sum()
// 컬렉션이 생성되지 않음
```

.

👉🏻 **시퀀스가 빠르지 않은 경우**
- 컬렉션 전체를 기반으로 처리해야 하는 연산은 시퀀스를 사용해도 빨라지지 않는다.
  - 유일한 예시로 코틀린 stdlib의 sorted

.

👉🏻 **자바 스트림의 경우**
- 자바의 스트림과 코틀린의 시퀀스는 세 가지 큰 차이점이 존재
  - 코틀린의 시퀀스가 더 많은 처리 함수를 가짐
  - 자바 스트림은 병렬 함수를 사용해서 병렬 모드로 실행 가능
  - 코틀린의 시퀀스는 코틀린/JVM, 코틀린/JS, 코틀린/Native 등의 일반적인 모듈에서 모두 사용 가능

📖 **정리**

> 컬렉션과 시퀀스는 같은 처리 메서드를 지원하며, 사용하는 형태가 거의 비슷하다.
>
> 일반적으로 데이터를 컬렉션에 저장하므로, 시퀀스 처리를 하려면 시퀀스로 변환하는 과정이 필요하다. 또한, 최종적으로 컬렉션 결과를 원하는 경우가 많으므로, 시퀀스를 다시 컬렉션으로 변환하는 과정도 필요하다. 이것이 시퀀스 처리의 단점이라고 할 수 있다.
>
> 하지만 시퀀스는 lazy하게 처리되어 아래와 같은 장점이 있다.
> - 자연스러운 처리 순서를 유지
> - 최소한만 연산
> - 무한 시퀀스 형태로 사용 가능
> - 각각의 단계에서 컬렉션을 만들지 않음

## Item 50. 컬렉션 처리 단계 수를 제한하라

모든 컬렉션 처리 메서드는 비용이 많이 든다.
- 따라서, 적절한 메서드를 활용해서, 컬렉션 처리 단계 수를 적절하게 제한하는 것이 좋다.
- 어떤 메서드를 사용하는지에 따라 컬렉션 처리의 단계 수가 달라진다.

```kotlin
class Student(val name: String?)

// 작동은 한다.
fun List<Student>.getNames(): List<String> = this
    .map { it.name }
    .filter { it != null }
    .map { it!! }

// 더 좋다.
fun List<Student>.getNames(): List<String> = this
    .map { it.name }
    .filterNotNull()

// 가장 좋다.
fun List<Student>.getNames(): List<String> = this
    .mapNotNull { it.name }
```

컬렉션 처리를 어떤 형태로 줄일 수 있는지 알아두면 좋다.
- 다음 표는 두 단계 이상의 컬렉션 처리 함수를 한번에 끝내는 방법을 정리한 것이다.

![Result](https://github.com/jihunparkme/jihunparkme.gitbook.io/blob/main/.gitbook/assets/kotlin/collection-56.png?raw=true 'Result')

📖 **정리**

> 대부분의 컬렉션 처리 단계는 '전체 컬렉션에 대한 반복'과 중간 컬렉션 생성'이라는 비용이 발생한다.
>
> 이 비용은 적절한 컬렉션 처리 함수들을 활용해서 줄일 수 있다.

## Item 51. 성능이 중요한 부분에는 기본 자료형 배열을 사용하라

코틀린은 기본 자료형을 선언할 수 없지만, 최적화를 위해 내부적으로는 사용할 수 있다.
- 기본 자료형은 다음과 같은 특징이 있다.
- 가볍다. (일반적인 객체와 다르게 추가적으로 포함되는 것들이 없기 때문)
- 빠르다. (값에 접근할 때 추가 비용이 들어가지 않음)

대규모의 데이터를 처리할 때 기본 자료형을 사용하면, 상당히 큰 최적화가 이루어진다.
- 코틀린에서 사용되는 List와 Set 등의 컬렉션은 제네릭 타입
- 하지만, 성능이 중요한 코드라면 IntArray와 LongArray 등의 기본 자료형을 활용하는 배열을 사용하는 것이 좋다.

|코틀린 타입|자바 타입|
|---|---|
|Int|Int|
|List\<Int\>|List\<Integer\>|
|Array\<Int\>|Integer[]|
|IntArray|Int[]|

📖 **정리**

> 일반적으로 Array보다 List와 Set을 사용하는 것이 좋다.
>
> 하지만, 기본 자료형의 컬렉션을 굉장히 많이 보유해야 하는 경우에는 성능을 높이고, 메모리 사용량을 줄일 수 있도록 Array를 사용하는 것이 좋다.
>
> 단, 라이브러리 개발자, 게임 개발자, 고급 그래픽을 처리해야 하는 개발자들에게 도움이 될 내용이다.

## Item 52. mutable 컬렉션 사용을 고려하라

`immutable` 컬렉션보다 `mutable` 컬렉션이 좋은 점은 성능적인 측면에서 더 빠르다
- `immutable` 컬렉션에 요소를 추가하려면, 새로운 컬렉션을 만들면서 여기에 요소를 추가해야 한다.

지역 변수로 사용할 때는 `mutable` 컬렉션을 사용하는 것이 더 합리적이다.

📖 **정리**

> 가변 컬렉션은 일반적으로 추가 처리가 빠르다
>
> immutable 컬렉션은 컬렉션 변경과 관련된 처리를 더 세부적으로 조정할 수 있다.
>
> 일반적으로 지역 스코프에서는 이러한 세부적인 조정이 필요하지 않으므로, 가변 컬렉션을 사용하는 것이 좋다.
>
> 특히 utils 에서는 요소 삽입이 자주 발생할 수 있기 떄문이다.