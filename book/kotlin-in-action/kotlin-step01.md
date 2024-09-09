# Kotlin step 01

# 람다 프로그래밍

<details>
<summary>📒 람다 프로그래밍 요약</summary>

- 람다를 사용하여 코드 조각을 다른 함수에게 인자로 넘기기
    
    ```kotlin
    /**
     * Performing operation on: 5
     * Result: 10
     */
    fun performOperation(num: Int, operation: (Int) -> Unit) {
        println("Performing operation on: $num")
        operation(num)
    }
    
    // 코드 조각을 performOperation 함수의 인자로 넘기기
    // 람다가 함수 인자인 경우 괄호 밖으로 람다를 빼내기
    performOperation(5) { 
    	// // 람다 인자가 하나뿐인 경우 인자 이름을 지정하지 않고 디폴트 이름(it)으로 사용 가능
    	println("Result: ${it * 2}") 
    }
    ```
    
    - 람다 안에서 바깥 함수의 변수 읽기/쓰기
    
    ```kotlin
    var result = 0
    val add: (Int) -> Unit = { result += it }
    add(5)
    assertEquals(5, result)
    ```
    
    - 메소드, 생성자, 프로퍼티 이름 앞에 `::`을 붙이면 각각에 대한 참조 생성 가능
        - 참조를 람다 대신 다른 함수에게 넘길 수 있음
    
    ```kotlin
    fun printMessage(message: String): String {
        return message
    }
    
    // 메소드, 생성자, 프로퍼티 참조
    val ref = ::printMessage
    assertEquals("Hello", ref("Hello"))
    ```
    
    - 컬렉션 함수 활용으로 컬렉션에 대한 연산을 직접 원소를 이터레이션 하지 않고 수행
    
    ```kotlin
    val numbers = listOf(1, 2, 3, 4, 5)
    val evenNumbers = numbers.filter { it % 2 == 0 }
    val doubledNumbers = numbers.map { it * 2 }
    val allEven = numbers.all { it % 2 == 0 }
    val anyEven = numbers.any { it % 2 == 0 }
    
    assertEquals(listOf(2, 4), evenNumbers)
    assertEquals(listOf(2, 4, 6, 8, 10), doubledNumbers)
    assertFalse(allEven)
    assertTrue(anyEven)
    ```
    
    - 시퀀스를 사용하면 중간 결과를 담는 컬렉션을 생성하지 않고도 컬렉션에 대한 여러 연산을 조합 가능
    
    ```kotlin
    val numbers = listOf(1, 2, 3, 4, 5)
    val result = numbers.asSequence()
        .filter { it % 2 == 0 }
        .map { it * 2 }
        .toList()
    
    assertEquals(listOf(4, 8), result)
    ```
    
    - 함수형 인터페이스(추상 메소드가 단 하나뿐인 SAM 인터페이스)를 인자로 받는 자바 함수를 호출할 경우 람다를 함수형 인터페이스 인자 대신 넘길 수 있다.
    
    ```kotlin
    // Runnable 인터페이스를 인자로 받는 자바 메소드 호출
    val thread = Thread { println("Running in a thread") }
    thread.start()
    ```
    
    - 수신 객체 지정 람다를 사용하면 람다 안에서 미리 정해둔 수신 객체의 메소드를 직접 호출 가능
    
    ```kotlin
    fun buildString(): String {
        return StringBuilder().apply {
            append("Hello, ")
            append("World!")
        }.toString()
    }
    
    assertEquals("Hello, World!", buildString())
    ```
    
    - 표준 라이브러리의 `with` 함수를 사용하면 어떤 객체에 대한 참조를 반복해서 언급하지 않으면서 그 객체의 메소드를 호출 가능
        - `apply`를 사용하면 어떤 객체라도 빌더 스타일의 API를 사용해 생성하고 초기화 가능
    
    ```kotlin
    val result = with(StringBuilder()) {
          append("Hello, ")
          append("World!")
          toString()
      }
      assertEquals("Hello, World!", result)
    
      val result2 = StringBuilder().apply {
          append("Hello, ")
          append("World!")
      }.toString()
      assertEquals("Hello, World!", result2)
    ```
</details>

## **람다 식과 멤버 참조**

**코드 블록을 함수 인자로 넘기기**

- 함수를 직접 다른 함수에 전달할 수 있다.
- 람다 식을 사용하면 코드가 더욱 더 간결해진다.

## **람다와 컬렉션**

> 함수나 프로퍼티를 반환하는 역할을 수행하는 람다는 멤버 참조로 대치할 수 있다.

```kotlin
data class Person(val name: String, val age: Int)

/* Java */
fun findTheOldest(people: List<Person>): Person? {
    var maxAge = 0
    var theOldest: Person? = null
    for (person in people) {
        if (person.age > maxAge) {
            maxAge = person.age
            theOldest = person
        }
    }
    return theOldest
}

@Test
fun `람다와 컬렉션`() {
    val javaPeople = listOf(Person("Alice", 29), Person("Bob", 31))
    assertEquals(Person("Bob", 31), findTheOldest(javaPeople))

    val kotlinPeople = listOf(Person("Alice", 29), Person("Bob", 31))
    assertEquals(Person("Bob", 31), kotlinPeople.maxByOrNull { it.age })
}
```

## **람다 현재 영역에 있는 변수에 접근**

```kotlin
// 메시지의 목록을 받아 모든 메시지에 똑같은 접두사를 붙여서 출력
fun printMessagesWithPrefix(messages: Collection<String>, prefix: String) {
    messages.forEach {
        println("$prefix $it")
    }
}
```

코틀린은 자바와 달리 람다 밖 함수에 있는 final이 아닌 변수에 접근할 수 있고, 그 변수를 변경할 수도 있다.

```kotlin
fun printProblemCounts(responses: Collection<String>) {
    var clientErrors = 0
    var serverErrors = 0
    responses.forEach {
        if (it.startsWith("4")) {
            clientErrors++ // 람다 밖에 있는 변수를 변경
        } else if (it.startsWith("5")) {
            serverErrors++
        }
    }
    println("$clientErrors client errors, $serverErrors server errors")
}
```

단, 람다를 이벤트 핸들러나 다른 비동기적으로 실행되는 코드로 활용하는 경우 

- 함수 호출이 끝난 다음에 로컬 변수가 변경될 수도 있다.

```kotlin
// 핸들러는 tryToCountButtonClicks가 clicks를 반환한 다음에 호출
fun tryToCountButtonClicks(button: Button) : Int {
		var clicks = 0
		button.onClick { clicks++ }
		return clicks
}
```

---

# **컬렉션 함수형 API**

## **filter & map**

> `filter` 함수는 컬렉션에서 원치 않는 원소를 제거
- 하지만 filter는 원소를 변환은 불가
- 원소를 변환하려면 map 함수를 사용해야 한다.
 
> `map` 함수는 주어진 람다를 컬렉션의 각 원소에 적용한 결과를 모아서 새 컬렉션을 생성

```kotlin
@Test
fun `filter and map`() {
    val people1 = listOf(Person("Alice", 29), Person("Bob", 31))
    assertEquals(listOf(Person("Bob", 31)), people1.filter { it.age > 30 })

    val people2 = listOf(Person("Alice", 29), Person("Bob", 31))
    assertEquals(listOf("Alice", "Bob"), people2.map { it.name })

    val numbers = mapOf(0 to "zero", 1 to "one")
    assertEquals(mapOf(0 to "ZERO", 1 to "ONE"), numbers.mapValues { it.value.uppercase() })
}
```

## **all, any, count, find**

> 컬렉션의 모든 원소가 어떤 조건을 만족하는지 판단하는 연산(ex. all, any ..)

```kotlin
@Test
fun `all any count find`() {
    val canBeInClub27 = { p: Person -> p.age <= 27 }
    val people1 = listOf(Person("Alice", 27), Person("Bob", 31))
    // Returns true if all elements match the given predicate.
    assertFalse(people1.all(canBeInClub27))

    val people2 = listOf(Person("Alice", 27), Person("Bob", 31))
    // Returns true if at least one element matches the given predicate.
    assertTrue(people2.any(canBeInClub27))

    val people3 = listOf(Person("Alice", 27), Person("Bob", 31))
    // Returns the number of elements matching the given predicate.
    assertEquals(1, people3.count(canBeInClub27))

    val people4 = listOf(Person("Alice", 27), Person("Bob", 31))
    // Returns the first element matching the given predicate, or null if no such element was found.
    assertEquals(Person("Alice", 27), people4.find(canBeInClub27))
}
```

## **flatMap & flatten**

> `flatMap` 함수는 먼저 인자로 주어진 람다를 컬렉션의 모든 객체에 적용하고
> 
> 람다를 적용한 결과 얻어지는 여러 리스트를 한 리스트로 모은다.

```kotlin
@Test
// Returns a single list of all elements yielded from results of transform function being invoked on each element of original collection.
fun `flatMap`() {
    val strings = listOf("abc", "def")
    assertEquals(listOf('a', 'b', 'c', 'd', 'e', 'f'), strings.flatMap { it.toList() })

    val books = listOf(
        Book("Thursday Next", listOf("Jasper Fforde")),
        Book("Mort", listOf("Terry Pratchett")),
        Book("Good Omens", listOf("Terry Pratchett", "Neil Gaiman"))
    )
    assertEquals(setOf("Jasper Fforde", "Terry Pratchett", "Neil Gaiman"), books.flatMap { it.authors }.toSet())
}

@Test
// Returns a single list of all elements from all collections in the given collection.
fun `flatten`() {
    val lists = listOf(
        listOf(1, 2, 3),
        listOf(4, 5, 6),
        listOf(7, 8, 9)
    )

    assertEquals(listOf(1, 2, 3, 4, 5, 6, 7, 8, 9), lists.flatten())
}
```