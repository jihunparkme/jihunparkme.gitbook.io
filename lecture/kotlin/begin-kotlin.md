# Kotlin 알아가기

# 코틀린?

- 타입 추론을 지원하는 `정적 타입 지정 언어`
    - 소스 코드의 정확성과 성능을 보장
    - 소스코드를 간결하게 유지
- `객체지향`과 `함수형 프로그래밍` 스타일을 모두 지원
    - 일급 시민 함수를 사용해 수준 높은 추상화가 가능
    - 불변 값 지원을 통해 다중 스레드 애플리케이션 개발과 테스트를더 쉽게 가능
- 서버 애플리케이션 개발에 활용 가능
    - 기존 자바 프레임워크를 완벽하게 지원
    - HTML 생성기나 영속화등의 일반적인 작업을 위한 새로운 도구를 제공
- 실용적이며 안전하고, 간결하며 좋은 상호 운용성
    - NPE 같이 흔히 발생하는 오류를 방지
    - 읽기 쉽고 간결한 코드를 지원
    - 자바와 아무런 제약 없이 통합

## **코틀린의 컴파일러(Kotlin Compiler)**

> 코틀린 컴파일러는 JVM에서 실행될 수 있는 **바이트코드가 포함된 클래스 파일을 생성**

<figure><img src="../../.gitbook/assets/kotlin/kotlin-compiler.png" alt=""><figcaption></figcaption></figure>

- 코틀린은 자바 컴파일러가 아닌 코틀린 컴파일러에 의해 컴파일되므로 자바를 사용할 때와 다른 부분이 존재
    - ex. 코틀린에서는 모든 예외를 언체크 예외로 인식

---

# 기초

```kotlin
fun main(args: Array<String>) {
		println("Hello, world!")
}
```

- 함수 선언 → `fun`
- 파라미터 이름 뒤에 타입 → `args: Array<String>`
- `함수를 최상위수준에` 정의 가능
    - 클래스 안에 함수를 넣어야 할 필요가 없음
- 배열도 일반적인 클래스
- System.out.println → `println`
- 줄 끝에 세미콜론(;) 없이 작성

## 함수

```kotlin
// 본문인 함수
fun max(a: Int, b: Int): Int {
		return if (a > b) a else b
}

// 식이 본문인 함수
fun max(a: Int, b: Int): Int = if (a > b) a else b

// 반환 타입 생략
fun max(a: Int, b: Int) = if (a > b) a else b
```

## 변수

> 코틀린에서는 보통 타입 지정을 생략
> 
> 기본적으로는 모든 변수를 val 키워드를 사용해 불변 변수로 선언하고, 나중에 꼭 필요할 때에만 var로 변경

```kotlin
val answer = 42 // 타입 생략
val answer: Int = 42 // 타입 지정

// 초기화 식을 사용하지 않고 변수 선언 시 변수 타입 명시 필요
val answer: Int
answer = 42
```

**변경 가능한 변수와 변경 불가능한 변수**

- val(=value): 변경 불가능한(immutable) 참조를 저장하는 변수 (like. java final)
- var(=variable): 변경 가능한(mutable) 참조 (like. java 일반 변수)

**문자열 템플릿**

- 문자열 리터럴의 필요한 곳에 변수를 넣되 변수 앞에 $를 추가
- $ 문자를 문자열에 넣고 싶으면 println("\$x")와 같이 \를 사용해 $를 이스케이프

```kotlin
fun main(args: Array<String>) {
		val name = if (args.size > 0) args[0] else "Kotlin"
		println("Hello, $name")
}
```

## 클래스와 프로퍼티

> 코틀린의 기본 가시성은 public

```kotlin
// Java
public class Person {
		private final String name;

		public Person(String name) {
				this.name = name;
		}

		public String getName() {
				return name;
		}
}

...

// Kotlin
class Person(val name: String)
```

**코틀린 프로퍼티는 자바의 필드와 접근자 메소드를 완전히 대신**

```kotlin
class Person(
		// 읽기 전용 프로퍼티: (비공개)필드와 필드를 읽는 단순한 (공개)Getter를 생성
		val name: String, 
		// 쓸 수 있는 프로퍼티: (비공개)필드, (공개)Getter/Setter를 생성
		var isMarried: Boolean 
)
```

- **코틀린에서는 클래스 임포트와 함수 임포트에 차이가 없으며, 모든 선언을 import 키워드로 가져올 수 있음**

## enum & when

> when은 자바의 switch를 대치하되 훨씬 더 강력

**enum**

- enum 클래스 안에도 프러퍼티나 메소드 정의 가능

```kotlin
enum class Color(
        val r: Int, val g: Int, val b: Int
) {
    RED(255, 0, 0), ORANGE(255, 165, 0),
    YELLOW(255, 255, 0), GREEN(0, 255, 0), BLUE(0, 0, 255),
    INDIGO(75, 0, 130), VIOLET(238, 130, 238);
		
		// 메소드를 정의하는 경우 enum 상수 목록과 메소드 정의 사이에 세미콜론 필요
    fun rgb() = (r * 256 + g) * 256 + b
}
```

**when 으로 enum 다루기**

```kotlin
// 자바와 달리 각 분기 끝에 break 불필요
fun getMnemonic(color: Color) =
  when (color) {
      Color.RED -> "Richard"
      Color.ORANGE -> "Of"
      Color.YELLOW -> "York"
      Color.GREEN -> "Gave"
      Color.BLUE -> "Battle"
      Color.INDIGO -> "In"
      Color.VIOLET -> "Vain"
}

// 갹체를 함께 사용 가능
fun mix(c1: Color, c2: Color) =
		when (setOf(c1, c2)) {
		    setOf(RED, YELLOW) -> ORANGE
		    setOf(YELLOW, BLUE) -> GREEN
		    setOf(BLUE, VIOLET) -> INDIGO
		    else -> throw Exception("Dirty color")
		}
```

**스마트 캐스트: 타입 검사와 타입 캐스트를 조합**

> 코틀린에서는 is를 사용해 변수 타입을 검사 (like. java instanceof)

```kotlin
fun eval(e: Expr): Int {
    if (e is Num) { // **타입 검사와 타입 캐스트**
        val n = e as Num
        return n.value
    }
    if (e is Sum) { // **타입 검사와 타입 캐스트**
        return eval(e.right) + eval(e.left)
    }
    throw IllegalArgumentException("Unknown expression")
}
```

**다중 if → when**

```kotlin
fun eval(e: Expr): Int =
    if (e is Num) {
        e.value
    } else if (e is Sum) {
        eval(e.right) + eval(e.left)
    } else {
        throw IllegalArgumentException("Unknown expression")
    }
    
...

fun eval(e: Expr): Int =
    when (e) {
        is Num ->
            e.value
        is Sum ->
            eval(e.right) + eval(e.left)
        else ->
            throw IllegalArgumentException("Unknown expression")
    }
```

## Iteration (while & loop)

**수에 대한 이터레이션**

```kotlin
// 100부터 거꾸로 세되 짝수만으로 게임을 진행
fun main(args: Array<String>) {
    for (i in 100 downTo 1 step 2) {
        print(fizzBuzz(i))
    }
}
```

**in으로 컬렉션이나 범위의 원소 검사**

> `in`으로 어떤 값이 범위에 속하는지 검사
> 
> `!in`을 사용하면 어떤 값이 범위에 속하지 않는지 검사

```kotlin
fun recognize(c: Char) = when (c) {
    in '0'..'9' -> "It's a digit!"
    in 'a'..'z', in 'A'..'Z' -> "It's a letter!"
    else -> "I don't know…"
}
```

## 예외처리

> 함수는 정상적으로 종료할 수 있지만 오류가 발생하면 예외를 던질 수 있음
> 
> 단, 함수가 던질 수 있는 예외를 선언하지 않아도 된다

```kotlin
// 조건이 참이면 number 값이 초기화되고, 거짓이면 초기화되지 않고 throw 호출
val number = try {
    Integer.parseInt(reader.readLine())
} catch (e: NumberFormatException) {
    return // 예외가 발생한 경우 catch 블록 다음의 코드는 실행되지 않음 
}

println("number") // 실행 X
```

**try 식으로 사용**

- try 키워드는 if, when 과 마찬가지로 식이다.
- 따라서 try의 값을 변수에 대입 가능

---

# **함수 정의와 호출**

## 컬렉션 만들기

```kotlin
val set = hashSetOf(1, 7, 53)
val list = arrayListOf(1, 7, 53)
val map = hashMapOf(1 to "one", 7 to "seven", 53 to "fifty-three")

...

fun main(args: Array<String>) {
    val strings = listOf("first", "second", "fourteenth")
    println(strings.last()) // 리스트의 마지막 원소
    val numbers = setOf(1, 14, 2)
    println(numbers.max()) // 컬렉션에서 최댓값
}
```

## **확장 함수와 확장 프로퍼티**

> **메소드를 다른 클래스에 추가**

**`확장 함수`**

- 어떤 클래스의 멤버 메소드인 것처럼 호출할 수 있지만 그 클래스의 밖에 선언된 함수
- 확장 함수를 만들려면 추가하려는 함수 이름 앞에 그 함수가 확장할 클래스의 이름을 덧붙이자
- **확장 함수는 오버라이드할 수 없음**

```kotlin
package strings
// 문자열의 마지막 문자를 돌려주는 확장 메소드
fun String.lastChar(): Char = this.get(this.length - 1)

...

// String = 클래스 이름 : 수신 객체 타입(receiver type)
// "Kotlin" = 확장 함수가 호출되는 대상 : 수신 객체(receiver object)
println("Kotlin".lastChar())
```

**임포트와 확장함수**

- 확장 함수를 사용하기 위해서는 그 함수를 다른 클래스나 함수와 마찬가지로 임포트 필요

```kotlin
import strings.lastChar // 명시적으로 사용
import strings.* // * 사용 가능
import strings.lastChar as last // as 키워드를 사용 가능
```

**함수를 호출하기 쉽게 만들기**

```kotlin
// AS-IS
fun <T> joinToString(
        collection: Collection<T>,
        separator: String = ";", // 디폴트 파라미터
        prefix: String = "(",
        postfix: String = ")"
): String {

    val result = StringBuilder(prefix)

    for ((index, element) in collection.withIndex()) {
        if (index > 0) result.append(separator)
        result.append(element)
    }

    result.append(postfix)
    return result.toString()
}

val list = listOf(1, 2, 3)
println(joinToString(list, "; ", "(", ")"))
println(joinToString(collection = list, separator = ";", prefix = "(", postfix = ")"))
println(joinToString(list))
println(joinToString(list, "; "))

...

// TO-BE : 확장 함수로 유틸리티 함수 정의
fun <T> Collection<T>.joinToString(
        separator: String = ", ",
        prefix: String = "",
        postfix: String = ""
): String {
    val result = StringBuilder(prefix)

    for ((index, element) in this.withIndex()) {
        if (index > 0) result.append(separator)
        result.append(element)
    }

    result.append(postfix)
    return result.toString()
}

fun main(args: Array<String>) {
    val list = arrayListOf(1, 2, 3)
    println(list.joinToString(" "))
}

val list = listOf(1, 2, 3)
println(list.joinToString2("; ", "(", ")"))
println(list.joinToString2(separator = ";", prefix = "(", postfix = ")"))
println(list.joinToString2())
println(list.joinToString2("; "))

```

**`확장 프로퍼티`**

- 기존 클래스 객체에 대한 프로퍼티 형식의 구문으로 사용할 수 있는 API를 추가
- 상태를 저장할 적절한 방법이 없으므로 아무 상태도 가질 수 없음
- 계산한 값을 담을 장소가 전혀 없으므로 초기화 코드도 사용 불가

```kotlin
val String.lastChar: Char
		get() = get(length -1)

var StringBuilder.lastChar: Char
    get() = get(length - 1) // 뒷받침하는 필드가 없어서 최소한 게터는 꼭 정의
    set(value: Char) {
        this.setCharAt(length - 1, value)
    }

@Test
fun `확장 프로퍼티`() {
    println("Kotlin".lastChar) // n
    
    val sb = StringBuilder("Kotlin?")
    sb.lastChar = '!'
    println(sb) // Kotlin!
}
```

## **가변 인자 함수**

> 메소드를 호출할 때 원하는 개수만큼 값을 인자로 넘기면 자바 컴파일러가 배열에 그 값들을 넣어주는 기능
- 자바는 타입 뒤에 ...를 붙이지만, 코틀린은 파라미터 앞에 `varag` 변경자를 붙인다.

```kotlin
public fun <T> listOf(vararg elements: T): List<T> = 
		if (elements.size > 0) elements.asList() else emptyList()

fun main(args: Array<String>) {
    val list = listOf("one", "two", "eight")
}
```

**이미 배열에 들어있는 원소를 가변 길이 인자로 넘길 때**

- 자바에서는 배열을 그냥 넘기면 되지만,
- 코틀린에서는 배열을 명시적으로 풀어서 `배열의 각 원소가 인자로 전달`되게 해야 한다.
- 기술적으로는 `스프레드(spread) 연산자`가 이러한 작업을 수행

```kotlin
fun main(args: Array<String>) {
    val list = listOf("args: ", *args)
}
```

## **값의 쌍 다루기 (중위호출, 구조 분해 선언)**

> 맵을 만들려면 `mapOf` 함수를 사용

```kotlin
val map = mapOf(1 to "one", 7 to "seven", 53 to "fifty-three")
```

- `중위 호출`이라는 특별한 방식으로 `to`라는 일반 메소드를 호출
- 중위 호출 시에는 수신 객체와 유일한 메소드 인자 사이에 메소드 이름을 넣는다

```kotlin
1.to("one") // "to" 메소드를 일반적인 방식으로 호출함
1 to "one" // "to" 메소드를 중위 호출 방식으로 호출함
```

**메소드를 중위 호출에 사용하도록 허용하고 싶을 경우 `infix` 변경자를 메소드 선언 앞에 추가**

```kotlin
/**
 * Creates a tuple of type [Pair] from this and [that].
 *
 * This can be useful for creating [Map] literals with less noise, for example:
 * @sample samples.collections.Maps.Instantiation.mapFromPairs
 */
public infix fun <A, B> A.to(that: B): Pair<A, B> = Pair(this, that)
```

이러한 기능을 `구조 분해 선언`이라고 부른다. 

- Pair 인스턴스 외 다른 객체에도 구조 분해 적용 가능
    - ex) key, value 두 변수를 맵의 원소를 사용해 초기화

```kotlin
for ((index, element) in collection.withIndex()) {
		println("$index: $element")
}
```

## 문자열과 정규식

**문자열 나누기**

> split 함수에 전달하는 값의 타입에 따라 정규식이나 일반 텍스트 중 어느 것으로 문자열을 분리하는지 쉽게 알 수 있다.

```kotlin
println("12.345-6.A".split("\\\\.|-".toRegex())) // 정규식을 명시적으로 만든다. 
```

간단한 경우에는 꼭 정규식을 쓸 필요가 없다.

- split 확장 함수를 오버로딩한 버전 중에는 구분 문자열을 하나 이상 인자로 받는 함수가 있다.

```kotlin
println("12.345-6.A".split(".","-")) // 여러 구분 문자열을 지정한다. 
[12, 345, 6, A]
```

**3중 따옴표 문자열**

> 역슬래시(\)를 포함한 어떤 문자도 이스케이프 불필요

```kotlin
// AS-IS
fun parsePath(path: String) {
    val directory = path.substringBeforeLast("/")
    val fullName = path.substringAfterLast("/")

    val fileName = fullName.substringBeforeLast(".")
    val extension = fullName.substringAfterLast(".")

    println("Dir: $directory, name: $fileName, ext: $extension")
    // Dir: /Users/yole/kotlin-book, name: chapter, ext: adoc
}

fun main(args: Array<String>) {
    parsePath("/Users/yole/kotlin-book/chapter.adoc")
}

...

// TO-BE
// 정규식을 사용하지 않고도 문자열을 쉽게 파싱
// 정규식이 필요할 때는 코틀린 라이브러리를 사용하면 더 편리
fun parsePath(path: String) {
    val regex = """(.+)/(.+)\\.(.+)""".toRegex()
    val matchResult = regex.matchEntire(path) // 정규식을 인자로 받은 path에 매치
    if (matchResult != null) {
		    // 그룹별로 분해한 매치 결과를 의미하는 destructured 프로퍼티를 각 변수에 대입
        val (directory, filename, extension) = matchResult.destructured
        // 구조 분해 선언은 Pair로 두 변수를 초기화할 때 썼던 구문과 동일
        println("Dir: $directory, name: $filename, ext: $extension")
    }
}
```

## 로컬 함수와 확장

> 함수에서 추출한 함수를 원 함수 내부에 중첩시킬 수 있다.
> 
> 문법적인 부가 비용을 들이지 않고도 깔끔하게 코드를 조직 가능

로컬 함수와 확장으로 코드를 다듬는 과정

```kotlin
// AS-IS
fun saveUser(user: User) {
    if (user.name.isEmpty()) {
        throw IllegalArgumentException(
            "Can't save user ${user.id}: empty Name")
    }

    if (user.address.isEmpty()) {
        throw IllegalArgumentException(
            "Can't save user ${user.id}: empty Address")
    }

    // Save user to the database
}

...

// 첫 번째 개선
fun saveUser(user: User) {
    fun validate(user: User,
                    value: String,
                    fieldName: String) {
        if (value.isEmpty()) {
            throw IllegalArgumentException(
                "Can't save user ${user.id}: empty $fieldName")
        }
    }

    validate(user, user.name, "Name")
    validate(user, user.address, "Address")
    // Save user to the database
}

...

// 두 번째 개선
fun saveUser(user: User) {
        // user 파라미터를 중복 사용하지 않는다. 
    fun validate(value: String, fieldName: String) { 
        if (value.isEmpty()) {
            throw IllegalArgumentException(
                    // 바깥 함수의 파라미터에 직접 접근할 수 있다. 
                "Can't save user ${user.id}: " +
                    "empty $fieldName")
        }
    }

    validate(user.name, "Name")
    validate(user.address, "Address")
    // Save user to the database
}

...

// TO-BE (User 클래스를 확장한 함수)
fun User.validateBeforeSave() {
    fun validate(value: String, fieldName: String) {
        if (value.isEmpty()) {
            throw IllegalArgumentException(
                "Can't save user $id: empty $fieldName")
        }
    }

    validate(name, "Name")
    validate(address, "Address")
}

fun saveUser(user: User) {
    user.validateBeforeSave()
    // Save user to the database
}
```

---