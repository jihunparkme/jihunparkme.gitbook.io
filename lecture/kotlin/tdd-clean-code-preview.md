# TDD, Clean Code with Kotlin Preview

[TDD, Clean Code with Kotlin](https://catsbi.oopy.io/5fcea983-4c9c-4580-9876-ac5349e0e3a0) 글을 참고하여 작성한 글입니다.

# Start Kotlin
- [자바 개발자를 위한 코틀린 입문](https://www.inflearn.com/course/java-to-kotlin)
- [DIMO Kotlin 강좌](https://www.youtube.com/watch?v=8RIsukgeUVw&list=PLQdnHjXZyYadiw5aV3p6DwUdXV2bZuhlN)
- [Kotlin in Action](https://www.yes24.com/Product/Goods/55148593)
- [아토믹 코틀린](https://www.yes24.com/Product/Goods/117817486)
- [코틀린 쿡북](https://www.yes24.com/Product/Goods/90452827)
- [객체지향의 사실과 오해](https://www.yes24.com/Product/Goods/18249021)
- [오브젝트](https://www.yes24.com/Product/Goods/74219491)

**Kotlin Web Compiler Site**

<https://play.kotlinlang.org/>

## 변수와 자료형

**✅ 변수의 선언**

> `var`: 일반적으로 통용되는 변수. 언제든지 읽기 쓰기가 가능
`val`: 선언시에만 초기화 가능. 중간에 값 변경 불가
> 

```kotlin
fun main() {
    var a: Int
    a = 123
    println(a) // 123
    
    var b: Int? = null // nallable 변수
    b = null
    println(b) // null
}
```

변수의 선언 위치에 따른 이름

- `Property`: 클래스에 선언된 변수
- `Local Variable`: 이외의 Scope 내에 선언된 변수

| 코틀린은 기본 변수에서 null을 허용하지 않는다.

- 변수에 값을 할당하지 않은채로 사용하게 되면 컴파일 에러

**✅ 코틀린의 기본 자료형**

> 자바와의 호환을 위해 자바와 거의 동일
> 

```kotlin
fun main() {
    // 정수형(8진수 표기는 미지원)
    var intValue:Int = 1234 // 32비트 이내의 10진수
    var longValue:Long = 1234L // 64비트 Long타입 10진수
    var intValueByHex:Int = 0x1af // 16진수
    var intValueByBin:Int = 0b10110110 // 2진수
    
    // 실수형
    var doubleValue:Double = 123.5 // 실수의 기본
    var doubleValueWithExp:Double = 123.5e10 // 필요 시 지수 표기법 추가
    var floatValue:Float = 123.5f // 16비트 float

    // 문자형(내부적으로 문자열을 UTF-16 BE로 관리. 글자 하나가 2bytes 메모리 공간 사용)
    var charValue:Char = 'a'
    var koreanCharValue:Char = '가'
    
    // 논리형
    var booleanValue:Boolean = true
    
    // 문자열
    val stringValue = "one line string test"
    val multiLineStringValue = """multiline
    string
    test"""
}
```

지원되는 특수문자

<center><img src="../../.gitbook/assets/kotlin/datatype.png" width="50%"></center>

## 형변환과 배열

**✅ 형변환**

> 코틀린은 형변환 시 발생할 수 있는 오류를 막기 위해 ***암시적 형변환은 미지원***
> 

```kotlin
fun main() {
    // 명시적 형변환
    var a: Int = 54321
    var b: Long = a.toLong()
}
```

**✅ 배열**

> `arrayOf`, `arrayOfNulls`
> 

```kotlin
fun main() {
    // 값이 있는 배열 생성
    var intArr = arrayOf(1, 2, 3, 4, 5)
    
    // 특정 크기를 가진 비어있는 배열 생성
    var nullArr = arrayOfNulls<Int>(5)
    
    intArr[2] = 8
    println(intArr[4])
}
```

## 타입추론과 함수

**✅ 타입추론**

> 변수 함수들을 선언할 때나 연산이 이루어질 때 자료형을 코드에 명시하지 않아도 자동으로 자료형을 추론
> 
- 반드 특정한 자료형으로 지정해야하는 상황이 아니라면 대부분은 코틀린의 타입추론 기능을 이용

**✅ 함수**

> 코틀린에서 함수는 내부적으로 기능을 가진 형태지만, 
외부에서 볼 때는 파라미터를 넣는다는 점 외에는 자료형이 결정된 변수라는 개념으로 접근
> 

```kotlin
fun main() {
	  println(add(5, 6, 7))
    println(add2(5, 6, 7))
}

// 함수
fun add(a: Int, b: Int, c: Int): Int {
    return a + b + c
}

// 단일표현식(반환형 타입 추론)
fun add2(a: Int, b: Int, c: Int) = a + b + c
```

## 조건문과 비교연산자

**✅ 조건문**

`if`

```kotlin
fun main() {
    var a = 11
    if (a < 10) {
        println("a is greater than 10")
    } else {
        println("a is less than or equal to 10")
    }
}

```

`when`

- 등호나 부등호는 사용 불가

```kotlin
fun doWhen (a: Any) {
    when(a) {
        1 -> println("this is number")
        "Hello" -> println("this is string")
        is Long -> println("this is long type")
        !is String -> println("this is not String type")
        else -> println("this is else area")
    }
}

fun doWhenReturn (a: Any) {
    var result = when(a) {
        1 -> "this is number"
        "Hello" -> "this is string"
        is Long -> "this is long type"
        !is String -> "this is not String type"
        else -> "this is else area"
    }
    println(result)
}
```

**✅ 비교연산자**

- 부등호: <, ≤, >, ≥, ≠
- 등호: ==
- 자료형 확인: is, !is
    - 호환 여부를 체크하고 형변환까지 한번에 진행
    
    ```kotlin
    a is Int
    ```
    

## 반복문

다른 언어에서의 반복문과는 약간의 차이가 있다.

```kotlin
fun main() {
	for (i in 0..9) {
        print(i)
    } // 0123456789
    
    println()
    for (i in 0..9 step 3) {
        print(i)
    } // 0369
    
    println()
    for (i in 9 downTo 0) {
        print(i)
    } // 9876543210
    
    println()
    for (i in 9 downTo 0 step 3) {
        print(i)
    } // 9630
    
    println()
    for (i in 'a'..'e') {
        print(i)
    } // abcde
}
```

레이블이 달린 반복문 기준으로 반복문을 종료시켜주는 기능

- 레이블 이름과 @기호로 즉시 반복문 종료

```kotlin
loop@for (i in 1..10) {
    for (j in 1..10) {
        if (i == 1 && j == 2) break@loop
        println("i : $i, j : $j")
    }
}
```

## 클래스

```kotlin
fun main() {
	var a = Person("박보영", 1990)
    var b = Person("전정국", 1997)
    var c = Person("장원영", 2004)
    
    println("안녕하세요. ${a.birthYear}년생 ${a.name}입니다.")
    
    b.introduce()
    c.introduce()
    
    var d = Person("이루다")
    var e = Person("차은우")
    var f = Person("류수정")
}

class Person(var name:String, val birthYear:Int) { // 클래스의 속성들을 선언함과 동시에 생성자를 선언하는 방법
    /** init
     * 생성자를 통해 인스턴스가 만들어질 때 호출되는 함수
     */
    init {
        println("[init] ${this.birthYear}년생 ${this.name}님의 인스턴스가 생성되었습니다.")        
    }
    
    /** 보조 생성자
     * 보조 생성자를 만들 경우 반드시 기본 생성자를 통해 속성을 초기화
     */
    constructor(name:String) : this(name, 1997) {
        println("[constructor] 보조 생성자가 사용되었습니다.")
        
    }
    
    fun introduce() {
        println("[introduce] 안녕하세요. ${birthYear}년생 ${name}입니다.")
    }
}
```

```bash
[init] 1990년생 박보영님의 인스턴스가 생성되었습니다.
[init] 1997년생 전정국님의 인스턴스가 생성되었습니다.
[init] 2004년생 장원영님의 인스턴스가 생성되었습니다.
안녕하세요. 1990년생 박보영입니다.
[introduce] 안녕하세요. 1997년생 전정국입니다.
[introduce] 안녕하세요. 2004년생 장원영입니다.
[init] 1997년생 이루다님의 인스턴스가 생성되었습니다.
[constructor] 보조 생성자가 사용되었습니다.
[init] 1997년생 차은우님의 인스턴스가 생성되었습니다.
[constructor] 보조 생성자가 사용되었습니다.
[init] 1997년생 류수정님의 인스턴스가 생성되었습니다.
[constructor] 보조 생성자가 사용되었습니다.
```

### 상속

```kotlin
fun main() {
	var a = Animal("별이", 5, "개")
    var b = Dog("별이", 5)
    
    a.introduce()
    b.introduce()
    
    b.bark()
    
    var c = Cat("루이", 1)
    
    c.introduce()
    c.meow()
}

/** open
 * 클래스가 상속될 수 있도록 허용하는 키워드
 */
open class Animal (var name:String, var age:Int, var type:String) {
    fun introduce() {
        println("저는 ${type} ${name}이고, ${age}살 입니다.")
    }
}

/** 상속 규칙
 * 1. 서브 클래스는 수퍼 클래스에 존재하는 속성과 같은 이름의 속성을 가질 수 없다.
 * 2. 서브 클래스가 생성될 때 반드시 수퍼클래스의 생성자까지 호출되어야 한다.
 */
 class Dog (name:String, age:Int) : Animal (name, age, "개") {
     fun bark() {
         println("멍멍")
     }
 }
 
class Cat (name:String, age:Int) : Animal (name, age, "고양이") {
     fun meow() {
         println("야옹야옹")
     }
}
```

### 오버라이딩

```kotlin
fun main() {
    var t = Tiger()
    t.eat()
}

/** 
 * 상속이 가능하도록 open 된 클래스
 */
open class Animal () {
    // 수퍼 클래스에서 open 된 함수는 서브 클래스에서 override 가능
    open fun eat() {
        println("음식을 먹습니다")
    }
}

class Tiger : Animal() {
    override fun eat() {
        println("고기를 먹습니다")
    }
}
```

### 추상화

추상 클래스: 추상 함수를 포함하는 클래스

```kotlin
fun main() {
    var r = Rabbit()
    r.eat()
    r.sniff()
}

// 추상 클래스
abstract class Animal () {
    abstract fun eat() // 추상 함수
    fun sniff() {
        println("킁킁")
    }
}

class Rabbit : Animal() {
    override fun eat() {
        println("당근을 먹습니다")
    }
}
```

인터페이스: 속성, 추상함수, 일반함수 포함

- 구현부가 있는 함수 → open 함수로 간주
- 구현부가 없는 함수 → abstract 함수로 간주
- ⚠️ 여러개의 인터페이스나 클래스에서 같은 이름과 형태를 가진 함수를 구현하고 있다면,
    - 서브클래스에서는 혼선이 일어나지 않도록 반드시 오버라이딩하여 재구현 필요

```kotlin
fun main() {
    var d = Dog()
    
    d.run()
    d.eat()
}

interface Runner {
    fun run()
}

interface Eater {
    fun eat() {
        println("음식을 먹습니다")
    }
}

class Dog : Runner, Eater {
    override fun run() {
        println("우다다다 뜁니다")
    }
    
    override fun eat() {
        println("허겁지겁 먹습니다")
    }
}
```

## 프로젝트 구조

- `Project`
    - 모든 내용을 담는 큰 틀
- `Module`
    - 하나의 프로젝트는 여러개의 모듈로 이루어질 수 있다.
        - 모듈은 직접 만들 수도 있고, 필요한 기능을 미리 구현해 둔 라이브러리 모듈을 가져와 사용 가능
    - 모듈 안에는 다수의 폴더(kt, 모듈 관련 설정, 리소스 파일 등)와 파일이 존재
- `Package`
    - 소스 코드의 소속을 지정하기 위한 논리적 단위
    - 일반적으로 패키지 이름을 지을 때 회사 도메인을 거꾸로 해서 프로젝트명과 세부 기능을 붙이는 방식
        - com.youtube.aaron
        - com.youtube.aaron.base
        - com.youtube.aaron.kotlin
        - com.youtube.aaron.talk
    - 코틀린은 자바와 달리 폴더 구조와 패키지 명을 일치시키지 않아도 된다.
        - 단순히 파일 상단에 패키지만 명시해 주면 컴파일러가 알아서 처리

> 코틀린은 클래스명과 파일명이 일치하지 않아도 되며, 
> 
> 하나의 파일에 여러개의 클래스를 넣어도 알아서 컴파일 가능
- 파일이나 폴더 기준으로 구분하지 않고 파일내에 있는 `package` 키워드 기준으로 구분

## 스코프와 접근 제한자

**✅ 스코프**

> 패키지 안에 변수, 함수, 클래스는 모두 하나의 스코프에 있는 멤버
> 
> 함수, 클래스안에 또다른 변수, 함수가 존재한다면 패키지 안에 또다른 하위 스코프로 동작

<center><img src="../../.gitbook/assets/kotlin/scope.png" width="50%"></center>

**스코프에 대한 세 가지 규칙**

- (1) 스코프 외부에서는 스코프 내부의 멤버를 `참조연산자`로만 참조 가능

```kotlin
a.eat()
import com.google.aaron
import com.google.aaron.A
```

- (2) 동일 스코프 내에서는 멤버들을 `공유`할 수 있음
- (3) 하위 스코프에서는 상위 스코프의 멤버를 재정의 가능

✅ 접근 제한자

> 변수, 함수, 클래스 선언 시 맨 앞에 붙여 사용

스코프 외부에서 스코프 내부에 접근할 때 그 권한을 `개발자가 제어`할 수 있는 기능
- public
- internal
- private
- protected

**Package Scope**

| public (default) | 어떤 패키지에서도 접근 가능  |
| ---------------- | ---------------------------- |
| internal         | 같은 모듈 내에서만 접근 가능 |
| private          | 같은 파일 내에서만 접근 가능 |
| protected        | 미사용                       |

**Class Scope**

| public (default) | 클래스 외부에서 항상 접근 가능              |
| ---------------- | ------------------------------------------- |
| private          | 클래스 내부에서만 접근 가능                 |
| protected        | 클래스 자신과 상속받은 클래스에서 접근 가능 |
| internal         | 미사용                                      |

## 고차함수와 람다함수

✅ 고차함수

> 함수를 마치 클래스에서 만들어낸 `인스턴스처럼` 취급하는 방법
- 함수를 `파라미터`로 넘겨줄 수도 있고, `결과값`으로 반환받을 수도 있는 방법

코틀린에서는 모든 함수를 고차함수로 사용 가능

- `::` → 일반 함수를 고차 함수로 변경해 주는 연산자
- 함수를 파라미터로 받을 경우 타입은 함수의 `(파라미터 자료형) -> 반환형 자료형`

```kotlin
fun main() {
    b(::a) // 일반 함수를 고차 함수로 변경
}

fun a (str: String) {
    println("$str 함수 a")
}

// 함수를 파라미터로 받기. (파라미터 자료형) -> 반환형 자료형
fun b (function: (String)->Unit) {
    function("b가 호출한")
}
```

✅ 람다함수

> 람다함수는 일반함수와 달리 그 자체가 고차함수이므로 별도의 연산자 없이 변수에 담을 수 있다.

```kotlin
fun main() {
    /** 자료형 자동 추론으로 축약 사용
     * var c: (String) -> Unit = { str:String -> println("$str 함수 a")}
     * var c: (String) -> Unit = { str -> println("$str 함수 a")}
     */
    var c = { str:String -> println("$str 함수 a")}
    b(c)
}

fun b (function: (String)->Unit) {
    function("b가 호출한")
}
```

ℹ️ 고차함수와 람다함수를 사용하여 함수를 일종의 변수로 사용 가능한 편의성

- 컬렉션 조작이나 스코프 함수에도 도움
