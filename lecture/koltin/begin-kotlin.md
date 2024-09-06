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
> 

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
기본적으로는 모든 변수를 val 키워드를 사용해 불변 변수로 선언하고, 나중에 꼭 필요할 때에만 var로 변경
> 

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
> 

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
> 

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
> 

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
`!in`을 사용하면 어떤 값이 범위에 속하지 않는지 검사
> 

```kotlin
fun recognize(c: Char) = when (c) {
    in '0'..'9' -> "It's a digit!"
    in 'a'..'z', in 'A'..'Z' -> "It's a letter!"
    else -> "I don't know…"
}
```

## 예외처리

> 함수는 정상적으로 종료할 수 있지만 오류가 발생하면 예외를 던질 수 있음
단, 함수가 던질 수 있는 예외를 선언하지 않아도 된다
> 

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