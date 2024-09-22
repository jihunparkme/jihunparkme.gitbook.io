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

