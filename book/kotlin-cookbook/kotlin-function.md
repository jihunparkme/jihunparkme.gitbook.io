- 컬렉션
- 시퀀스
- 영역 함수
- 코틀린 대리자

# 컬렉션

## 배열 다루기

> arrayOf 함수를 이용해 배열을 만들고 
> 
> Array 클래스에 들어 있는 속성과 메소드를 이용해 배열에 들어 있는 값을 다룰 수 있다.

코틀린은 배열을 생성하는 `arrayOf` 라는 이름의 간단한 팩토리 메소드를 제공

```kotlin
val string = arrayOf("this", "is", "an", "array", "of", "strings")
```

`arrayOfNulls` 팩토리 메소드를 사용해 널로만 채워진 배열을 생성 가능

```kotlin
val nullStringArray = arrayOfNulls<String>(5)
```

Array 클래스에는 public 생성자가 하나만 존재하고, 이 생성자는 아래 두 인자를 받는다.
- Int 타입의 size
- init, 즉 (Int) -> T 타입의 람다

```kotlin
val squares = Array(5) { i -> (i * i).toString() }
```

.

코틀린에는 오토박싱, 언박싱 비용을 방지할 수 있는 기본 타입을 나타내는 클래스가 있다.
- booleanArrayOf
- byteArrayOf
- shortArrayOf
- charArrayOf
- intArrayOf
- longArrayOf
- floatArrayOf
- ShortArray

{% hint style="info" %}

**코틀린의 타입**

코틀린에는 명시적인 기본 타입은 없지만 값이 널 허용 값인 경우 생성된 바이트코드는 Integer와 Double 같은 자바 래퍼 클래스를 사용하고

널 비허용 값인 경우 생성된 바이트코드는 int와 double 같은 기본 타입을 사용

{% endhint %}

배열의 확장 메소드

👉🏻 **배열의 적법한 인덱스 값 획득하기**

```kotlin
@Test
fun `valid indices`() {
    val strings = arrayOf("this", "is", "an", "array", "of", "strings")
    val indices = strings.indices
    assertThat(indices, contains(0, 1, 2, 3, 4, 5))
}
```

.

일반적으로 배열을 순회할 때 표준 `for-in` 루프를 사용하지만 배열의 인덱스 값도 같이 사용하고 싶다면, `withIndex` 함수를 사용하자

```kotlin
fun <T> Array<out T>.withIndex(): Iterable<IndexedValue<T>>

data class IndexedValue<out T>(public val index: Int, public val value: T)
```

👉🏻 **withindex를 사용해 배열 값이 접근하기**

```kotlin
@Test
fun `withIndex returns IndexValues`() {
    val strings = arrayOf("this", "is", "an", "array", "of", "strings")
    /**
        * Index 0 maps to this
        * Index 1 maps to is
        * Index 2 maps to an
        * Index 3 maps to array
        * Index 4 maps to of
        * Index 5 maps to strings
        *
        */
    for ((index, value ) in strings.withIndex()) {
        println("Index $index maps to $value") // withIndex 호출
        assertTrue(index in 0..5) // 각각의 인덱스와 값에 접근
    }
}
```

---

## 컬렉션 생성하기

> List, Set, Map은
>
> listOf, setOf, mapOf 처럼 **변경 불가능한 컬렉션**을 생성하기 위해 만들어진 함수나
>
> mutableListOf, mutableSetOf, mutableMapOf 처럼 **변경 가능한 컬렉션**을 생성하기 위해 고안된 함수 중 하나를 사용

{% hint style="info" %}

**asList 구현**

asList 구현은 읽기 전용 리스트를 리턴하는 자바의 Arrays.asList에 위임

{% endhint %}

👉🏻 **불변 List, Set, Map 생성하기**

```kotlin
val numList = listOf(3, 1, 4, 1, 5, 9) // 불변 리스트 생성
val numSet = setOf(3, 1, 4, 1, 5, 9) // 불변 세트 생성(중복 포함 X)
val map = mapOf(1 to "one", 2 to "two", 3 to "three") // 불변 맵 생성
```

기본적으로 코틀린 컬렉션은 불변이다.
- 컬렉션은 원소를 추가하거나 제거하는 메소드를 지원하지 않는다.
- 컬렉션을 변경하는 메소드는 팩토리 메소드에서 제공하는 가변 인터페이스에 들어 있다.
  - mutableListOf
  - mutableSetOf
  - mutableMapOf

👉🏻 **가변 List, Set, Map 생성하기**

```kotlin
val numList = mutableListOf(3, 1, 4, 1, 5, 9)
val numSet = mutableSetOf(3, 1, 4, 1, 5, 9)
val map = mutableMapOf(1 to "one", 2 to "two", 3 to "three")
```

👉🏻 **List, Set, Map 인터페이스를 직접 구현한 클래스의 인스턴스 생성**

```kotlin
@Test
fun `instantiating a linked list`() {
    val list = LinkedList<Int>()
    list.add(3) // addLast 의 별칭
    list.add(1)
    list.addLast(999)
    list[2] = 4 // 배열 타입 접근은 get or set 호출
    list.addAll(listOf(1, 5, 9, 2, 6, 5))
    
    assertThat(list, contains(3, 1, 4, 1, 5, 9, 2, 6, 5))
}
```

---

## 컬렉션에서 읽기 전용 뷰 생성하기

> toList, toSet, toMap 메소드를 사용해 새로운 읽기 전용 컬렉션을 생헝하자

👉🏻 **List 타입의 레퍼런스를 리턴하는 toList 메소드 호출**

```kotlin
@Test
fun `toList on mutableList makes a readOnly new list`() {
    val mutableNums = mutableListOf(3, 1, 4, 1, 5, 9)
    val readOnlyNumList: List<Int> = mutableNums.toList()
    
    assertEquals(mutableNums, readOnlyNumList)
    assertNotSame(mutableNums, readOnlyNumList)
}
```

👉🏻 **독립된 객체를 생성하는데 독립된 객체의 내용은 원본과 같지만 더 이상 같은 객체를 나타내지는 않는다.**

```kotlin
@Test
fun `modify mutable list does not change read-only list`() {
    val mutableNums = mutableListOf(3, 1, 4, 1, 5, 9)
    val readOnly: List<Int> = mutableNums.toList()
    mutableNums.add(2)

    assertThat(readOnly, not(contains(2)))
}
```

👉🏻 **내용이 같은 읽기 전용 뷰를 생성하고 싶다면, List 타입의 레퍼런스에 가변 리스트를 할당하자.**


```kotlin
@Test
fun `read-only view of a mutable list`() {
    val mutableNums = mutableListOf(3, 1, 4, 1, 5, 9)
    // List 타입의 레퍼런스에 가변 리스트 할당
    val readOnlySameList: List<Int> = mutableNums
    assertEquals(mutableNums, readOnlySameList)
    assertSame(mutableNums, readOnlySameList)

    mutableNums.add(2)
    assertEquals(mutableNums, readOnlySameList)
    assertSame(mutableNums, readOnlySameList) // 여전히 같은 기저 객체
}
```

---

## 컬렉션에서 맵 만들기

> 키 리스트가 있을 때 각각의 키와 생성한 값을 연관시켜 맵을 만들려면
>
> `associateWith` 함수에 각 키에 대해 실행되는 람다를 제공해 사용하자.

👉🏻 **associateWith로 값 생성하기**

```kotlin
@Test
fun `associateWith`() {
    val keys = 'a'..'f'
    val map = keys.associateWith { it.toString().repeat(5).capitalize() }
    println(map)
    // {a=Aaaaa, b=Bbbbb, c=Ccccc, d=Ddddd, e=Eeeee, f=Fffff}
}
```

---

## 컬렉션이 빈 경우 기본값 리턴하기

> 컬렉션이나 문자열이 비어 있는 경우 ifEmpty, ifBlank 함수를 사용해 기본값을 리턴하자.

👉🏻 **기본값 테스트**

```kotlin
data class Product(
    val name: String,
    var price: Double,
    var onSale: Boolean = false,
)

// 판매 중인 상품 이름 얻기
fun sameOfProductsOnSale(products: List<Product>) =
    products.filter { it.onSale }
        .map { it.name }
        .joinToString(separator = ", ")

// 빈 컬렉션에 기본 리스트 제공
fun onSaleProductsIfEmptyCollection(products: List<Product>) =
    products.filter { it.onSale }
        .map { it.name }
        .ifEmpty { listOf("none") }
        .joinToString(separator = ", ")

// 빈 문자열에 기본 문자열 제공
fun onSaleProductsIfEmptyString(products: List<Product>) =
    products.filter { it.onSale }
        .joinToString(separator = ", ") { it.name }
        .ifEmpty { "none" }

private val overthruster = Product("Oscillation Overthruster", 1_000_000.0)
private val fluxcapacitor = Product("Flux Capacitor", 299_999.95, onSale = true)
private val tpsReportCoverSheet = Product("TPS Report Cover Sheet", 0.25)

@Test
fun `asis_productsNotOnSale`() {
    val sameOfProductsOnSale = sameOfProductsOnSale(listOf(overthruster, tpsReportCoverSheet))
    assertEquals("", sameOfProductsOnSale) // 판매 중인 상품이 없는 경우 빈 컬렉션을 리턴하고 빈 문자열로 반환
}

@Test
fun productsOnSale() {
    val products = listOf(overthruster, fluxcapacitor, tpsReportCoverSheet)

    assertAll("On sale products",
        { assertEquals("Flux Capacitor", onSaleProductsIfEmptyCollection(products)) },
        { assertEquals("Flux Capacitor", onSaleProductsIfEmptyString(products)) })
}

@Test
fun productsNotOnSale() {
    val products = listOf(overthruster, tpsReportCoverSheet)

    assertAll("No products on sale",
        { assertEquals("none", onSaleProductsIfEmptyCollection(products)) },
        { assertEquals("none", onSaleProductsIfEmptyString(products)) })
}
```

{% hint style="info" %}

코틀린도 Optional\<T\>를 지원하지만 ifEmpty 함수를 사용해 특정한 값을 리턴하는 방법이 더 사용하기 쉽다.

{% endhint %}

---

## 주어진 범위로 값 제한하기

> 값이 주어졌을 때, 주어진 값이 특정 번위 안에 들면 해당 값을 리턴하고, 
> 
> 그렇지 않다면 범위의 최솟값 또는 최댓값을 리턴하려면
>
> kotlin.ranges의 `coerceIn` 함수를 범위 인자 또는 구체적인 최솟값, 최댓값과 함께 사용하자.

👉🏻 **coerceIn 함수는 값이 범위 안에 있으면 해당 값을 리턴하고 그렇지 않다면 범위의 경계 값을 리턴**

```kotlin
@Test
fun `coerceIn given a range`() {
    val range = 3..8
    assertEquals(5, 5.coerceIn(range))
    assertEquals(range.start, 1.coerceIn(range))
    assertEquals(range.endInclusive, 9.coerceIn(range))
}
```

👉🏻 **원하는 최대/최솟값이 있다면 범위를 생성하지 않아도 된다.**

```kotlin
@Test
fun `coerceIn given a range`() {
    val min = 2
    val max = 6
    assertEquals(5, 5.coerceIn(min, max))
    assertEquals(min, 1.coerceIn(min, max))
    assertEquals(max, 9.coerceIn(min, max))
}
```

---

## 리스트 구조 분해하기

> 최대 5개의 원소를 가진 그룹에 리스트를 할당하기

👉🏻 **리스트의 원소를 구조 분해하기**

```kotlin
val list = listOf("a", "b", "c", "d", "e", "f", "g")
val (a, b, c, d, e) = list
println("$a $b $c $d $e")
```

코틀린 표준 라이브러리의 List 클래스에 N이 1부터 5까지인 componentN 이라는 확장 함수가 정의되어 가능한 동작

```kotlin
package kotlin.collections

@kotlin.internal.InlineOnly
public inline operator fun <T> List<T>.component1(): T {
    return get(0)
}

@kotlin.internal.InlineOnly
public inline operator fun <T> List<T>.component2(): T {
    return get(1)
}

@kotlin.internal.InlineOnly
public inline operator fun <T> List<T>.component3(): T {
    return get(2)
}

@kotlin.internal.InlineOnly
public inline operator fun <T> List<T>.component4(): T {
    return get(3)
}

@kotlin.internal.InlineOnly
public inline operator fun <T> List<T>.component5(): T {
    return get(4)
}
```

데이터 클래스는 정의된 모든 속성 관련 component 메소드를 자동으로 추가
- 데이터 클래스가 아닌 클래스를 정의하면 필요한 component 메소드를 직접 정의 가능

---

## 다수의 속성으로 정렬하기

> `sortedWith`, `comparedBy` 함수로 다수 속성으로 정렬을 해보자.

👉🏻 **연이은 속성으로 골프 선수 정렬하기**
- `compareBy` 함수는 Comparator 를 생성하고,
  - Comparator 속성을 추출하는 선택자 목록을 제공
  - 차례차례 정렬에 사용되는 Comparator 생성
- `sortedWith` 함수는 Comparator 를 인자로 받는다.

```kotlin
val sorted = golfers.sortedWith(
    compareBy({ it.score }, { it.last }, { it.first })
)
sorted.forEach { println(it) }
/**
    * Golfer(score=68, first=Bubba, last=Watson)
    * Golfer(score=68, first=Tom, last=Watson)
    * Golfer(score=68, first=Ty, last=Webb)
    * Golfer(score=70, first=Jack, last=Nicklaus)
    * Golfer(score=70, first=Tiger, last=Woods)
    */
```

{% hint style="info" %}

sortBy, sortWith 함수는 자신의 원소를 제자리에서 정렬하므로 변경 가능 컬렉션을 요구

{% endhint %}

👉🏻 **비교 후 새로운 비교를 적용하는 thenBy 함수**
- 위 테스트와 동일한 결과를 보임

```kotlin
@Test
fun `comparator 연쇄`() {
    val comparator = compareBy<Golfer>(Golfer::score)
        .thenBy(Golfer::last)
        .thenBy(Golfer::first)

    golfers.sortedWith(comparator)
        .forEach(::println)
}
```

---

## 사용자 정의 이터레이터 정의하기

> next, hasNext 함수를 모두 구현한 iterator 를 리턴하는 연산자 함수 정의하기

**kotlin.collections Iterator Interface**

```kotlin
public interface Iterator<out T> {
    public operator fun next(): T
    public operator fun hasNext(): Boolean
}
```

👉🏻 **확장 함수로 iterator 정의하기**

```kotlin
data class Player(val name: String)

class Team(
    val name: String,
    val players: MutableList<Player> = mutableListOf(),
) {
    fun addPlayers(vararg people: Player) =
        players.addAll(people)
    //...
}

@Test
fun `확장함수 활용하기`() {
    val team = Team("Warriors")
    team.addPlayers(Player("Curry"), Player("Thompson"), Player("Durant"), Player("Green"), Player("Cousins"))

    operator fun Team.iterator() : Iterator<Player> = players.iterator()
    for (player in team) {
        println(player)
    }
}
```

👉🏻 **iterator 구현**

```kotlin
@Test
fun `iterator 구현하기`() {
    val team = Team("Warriors")
    team.addPlayers(Player("Curry"), Player("Thompson"), Player("Durant"), Player("Green"), Player("Cousins"))

    assertEquals("Curry, Thompson, Durant, Green, Cousins",
        team.map { it.name }.joinToString())
}
```

---

## 타입으로 컬렉션을 필터링하기

> `filterIsInstance` or `filterIsInstanceTo` 확장 함수로 
>
> 여러 타입이 섞여 있는 컬렉션에서 특정 타입의 원소로만 구성된 새 컬렉션을 생성할 수 있다.

코틀린 컬렉션은 특정 불리언 조건을 만족하는 원소를 필터링하기 위해 사용되는 Predicate를 인자로 받는 `filter` 확장 함수를 포함한다.

```kotlin
@Test
fun `타입으로 컬렉션을 필터링하기`() {
    val list = listOf("a", LocalDate.now(), 3, 1, 4, "b")
    val strings = list.filter { it is String}

    /**
        * 필터링은 동작하지만 string 변수의 추론 타입은 List<Any> 이므로
        * List<Any>의 개별 원소를 string 타입으로 영리한 타입 변환하지 않는다.
        */
    for (s in strings) {
        println(s.length) // 컴파일 에러
    }
}
```

👉🏻 **위와 같은 상황에서 filterIsInstance 함수를 사용**
- filterIsInstance 함순은 구체적인 타입을 사용

```kotlin
/**
 * Returns a list containing all elements that are instances of specified type parameter R.
 */
public inline fun <reified R> Iterable<*>.filterIsInstance(): List<@kotlin.internal.NoInfer R> {
    return filterIsInstanceTo(ArrayList<R>())
}

...

@Test
fun `구체적인 타입 사용하기_filterIsInstance`() {
    val list = listOf("a", LocalDate.now(), 3, 1, 4, "b")

    val all = list.filterIsInstance<Any>()
    val strings = list.filterIsInstance<String>()
    val ints = list.filterIsInstance<Int>()
    val dates = list.filterIsInstance(LocalDate::class.java)

    assertThat(all, `is`(list))
    assertThat(strings, containsInAnyOrder("a", "b"))
    assertThat(ints, containsInAnyOrder(1, 3, 4))
    assertThat(dates, contains(LocalDate.now()))
}

@Test
fun `구체적인 타입을 사용해 제공된 리스트 채우기_filterIsInstanceTo`() {
    val list = listOf("a", LocalDate.now(), 3, 1, 4, "b")

    val all = list.filterIsInstanceTo(mutableListOf<Any>())
    val strings = list.filterIsInstanceTo(mutableListOf<String>())
    val ints = list.filterIsInstanceTo(mutableListOf<Int>())
    val dates = list.filterIsInstanceTo(mutableListOf<LocalDate>())

    assertThat(all, `is`(list))
    assertThat(strings, containsInAnyOrder("a", "b"))
    assertThat(ints, containsInAnyOrder(1, 3, 4))
    assertThat(dates, contains(LocalDate.now()))
}
```

---

## 범위를 수열로 만들기

> 정수 또는 문자로 구성되어 있지 않는 수열의 범위를 순회하려면 사용자 정의 수열을 생성하자.

사용자 정의 수열은 IntProgression, LongProgression, CharProgression 처럼 Iterable 인터페이스를 구현해야 한다.

👉🏻 **LocalDate Progression**

```kotlin
class LocalDateProgression(
    override val start: LocalDate,
    override val endInclusive: LocalDate,
    val step: Long = 1
) : Iterable<LocalDate>, ClosedRange<LocalDate> {

    override fun iterator(): Iterator<LocalDate> =
        LocalDateProgressionIterator(start, endInclusive, step)

    infix fun step(days: Long) =
        LocalDateProgression(start, endInclusive, days)
}

internal class LocalDateProgressionIterator(
    start: LocalDate,
    val endInclusive: LocalDate,
    val step: Long
) : Iterator<LocalDate> {

    private var current = start

    override fun hasNext() = current <= endInclusive

    override fun next(): LocalDate {
        val next = current
        current = current.plusDays(step)
        return next
    }
}

operator fun LocalDate.rangeTo(other: LocalDate) = LocalDateProgression(this, other)
```

👉🏻 **LocalData 수열 테스트**

```kotlin
@Test
fun `use LocalDate as a progression`() {
    val startDate = LocalDate.now()
    val endDate = startDate.plusDays(5)

    val dateRange = startDate..endDate // forEachIndexed 범위 생성
    println(dateRange.joinToString(", ")) // 2024-10-25, 2024-10-26, 2024-10-27, 2024-10-28, 2024-10-29, 2024-10-30
    dateRange.forEachIndexed { index, locaDate ->
        assertEquals(locaDate, startDate.plusDays(index.toLong()))
    }

    val dateList=  dateRange.map { it.toString() }
    assertEquals(6, dateList.size)
}

@Test
fun `use LocalDate as a progression with a step`() {
    val startDate = LocalDate.now()
    val endDate = startDate.plusDays(5)

    val dateRange = startDate..endDate step 2  // forEachIndexed 범위 생성
    println(dateRange.joinToString(", ")) // 2024-10-25, 2024-10-27, 2024-10-29
    dateRange.forEachIndexed { index, locaDate ->
        assertEquals(locaDate, startDate.plusDays(index.toLong() * 2))
    }

    val dateList=  dateRange.map { it.toString() }
    assertEquals(3, dateList.size)
}
```

# 시퀀스

{% hint style="info" %}

**시퀀스**

데이터 처리를 위해 시퀀스를 사용하면, 각각의 원소는 자신의 다음 원소가 처리되지 전에 전체 파이프라인을 완료한다.

지연 처리 방식은 데이터가 많거나 first 같은 쇼트 서킷 연산의 경우 도움이 되고, 원하는 값을 찾았을 때 시퀀스를 종료할 수 있게 도와준다.

{% endhint %}

## 지연 시퀀스 사용하기

> 특정 조건을 만족시키는 데 필요한 최소량의 데이터만 처리하고 싶다면 
>
> 코틀린 시퀀스를 `쇼트 서킷 함수`와 함께 사용하자

쇼트 서킷: 특정 조건에 다다를 때까지 오직 필요한 데이터만을 처리하는 방식

👉🏻 **filter 함수 사용하기**

```kotlin
/**
 * AS-IS
 */
(100 until 200).map { it * 2 } // 100개의 계산
    .filter { it % 3 == 0 } // 100개의 계산
    .first() 

/**
 * TO-BE
 */ 
(100 until 200).map { it * 2 } // 100개의 계산
    .first { it % 3 == 0 } // 3개의 계산
```

👉🏻 **시퀀스 사용하기**
- 시퀀스가 비었다면 first 함수는 예외를 던지므로 firstOrNull을 사용하자

```kotlin
(100 until 200).asSequence() // 범위를 시퀀스로 변경(6개의 연산만 수행)
    .map { println("doubling $it"); it * 2 }
    .filter { println("filtering $it"); it % 3 == 0 }
    .firstOrNull()
```

{% hint style="info" %}

**시퀀스 API**

시퀀스 API는 컬렉션에 들어 있는 함수와 똑같은 함수를 가지고 있지만, 시퀀스에 대한 연산은 중간 연산과 최종 연산이라는 범주로 나뉜다.

- map, filter 같은 중간 연산은 새로운 시퀀스를 리턴
- first, toList 같은 최종 연산은 시퀀스가 아닌 다른 것을 리턴

{% endhint %}

---

## 시퀀스 생성하기

> - 이미 원소가 있다면 `sequenceOf`
> - Iterable이 있다면 `asSequence`
> - 그외의 경우는 시퀀스 생성기를 사용핵서
>
> 시퀀스를 생성하자

👉🏻 **원소나, Iterable이 존재하는 경우 시퀀스 생성**

```kotlin
// Sequence<Int> 생성
val numSequence1 = sequenceOf(3, 1, 4, 1, 5, 9)
val numSequence2 = listOf(3, 1, 4, 1, 5, 9).asSequence()
```

**generateSequence**

```kotlin
public fun <T : Any> generateSequence(seed: T?, nextFunction: (T) -> T?): Sequence<T> =
    if (seed == null)
        EmptySequence
    else
        GeneratorSequence({ seed }, nextFunction)
```

👉🏻 **무한의 원소를 갖는 시퀀스 생성하기**
- 다음 소수를 찾기 위해 얼마나 많은 수를 확인해야 하는지 알 수 없는 이러한 경우 시퀀스를 사용하기 좋은 이유

```kotlin
@Test
fun `test`() {
    fun Int.isPrime() =
        // 2인지 확인 후 2가 아니면 2부터 해당 수의 제곱근 값을 반올림한 수까지 범위로 생성
        this == 2 || (2..ceil(sqrt(this.toDouble())).toInt())
            // 주어진 수를 이 범위의 각각의 수로 나누어 정확히 떨어지는 수를 범위에서 찾을 수 없다면
            .none { divisor -> this % divisor == 0 }

    /**
    * 무한대의 정수를 생성하고 첫 번째 소수를 찾을 때까지 생성한 정수를 하나씩 평가
    */
    fun nextPrime(num: Int) =
        generateSequence(num + 1) { it + 1} // 주어진 수보다 1 큰 수에서 시작하고 1 증가 반복
            .first(Int::isPrime) // 첫 소수 값을 리턴

    assertEquals(10007, nextPrime(9973))
}
```

---

## 무한 시퀀스 다루기

> 무한대의 원소를 갖는 시퀀스의 일부분이 필요하다면,
>
> 널을 리턴하는 시퀀스 생성기를 사용하거나, 시퀀스 확장 함수 중 takeWhile 같은 함수를 사용하자.

```kotlin
@Test
fun `처음 N개의 소수 찾기`() {
    fun Int.isPrime() =
        this == 2 || (2..ceil(sqrt(this.toDouble())).toInt())
            .none { divisor -> this % divisor == 0 }

    fun nextPrime(num: Int) =
        generateSequence(num + 1) { it + 1}
            .first(Int::isPrime)

    /** 처음 N개의 소수 찾기 */
    fun firstNPrimes(count: Int) =
        generateSequence(2, ::nextPrime) // 2부터 시작하는 소수의 무한 시퀀스
            .take(count) // 요청한 수만큼만 원소를 가져오는 중간 연산
            .toList() // 최종 연산

    /** 주어진 수보다 작은 모든 소수 Ver.1 (널을 리턴하는 시퀀스 생성기) */
    fun primesLessThanV1(max: Int): List<Int> =
        generateSequence(2) { n -> if (n < max) nextPrime(n) else null }
            .toList()
            .dropLast(1)

    /** 주어진 수보다 작은 모든 소수 Ver.2 (takeWhile 사용) */
    fun primesLessThanV2(max: Int): List<Int> =
        generateSequence(2, ::nextPrime)
            .takeWhile { it < max }
            .toList()

    assertEquals(listOf(2, 3, 5, 7, 11), firstNPrimes(5))
    assertEquals(listOf(2, 3, 5, 7, 11, 13, 17, 19, 23, 29, 31, 37, 41, 43, 47, 53, 59, 61, 67, 71, 73, 79, 83, 89, 97), primesLessThanV1(100))
    assertEquals(listOf(2, 3, 5, 7, 11, 13, 17, 19, 23, 29, 31, 37, 41, 43, 47, 53, 59, 61, 67, 71, 73, 79, 83, 89, 97), primesLessThanV2(100))
}
```

---

## 시퀀스에서 yield하기

> 구간을 지정해 시퀀스에서 값을 생성하려면
>
> yield 중단 함수와 함께 sequence 함수를 사용하자

sequence 함수는 주어진 블록에서 평가되는 시퀀스를 생성
- 이 블록은 인자 없는 람다 함수이며 void를 리턴하고 평가 후 SequenceScope 타입을 받는다.

**SequenceScope**

```kotlin
public abstract class SequenceScope<in T> internal constructor() {
    public abstract suspend fun yield(value: T)

    public abstract suspend fun yieldAll(iterator: Iterator<T>)
    public suspend fun yieldAll(elements: Iterable<T>)
    public suspend fun yieldAll(sequence: Sequence<T>)
}
```

yield 함수는 이터레이터에 값을 제공하고 다음 값을 요청할 때까지 값 생성을 중단

👉🏻 **시퀀스 연산에서 값 추출하기**
- yield는 중단 함수이고 코루틴과도 잘 동작
- 코루틴에 값을 제공한 후 다음 값을 요청할 때까지 해당 코루틴을 중단시킬 수 있다.

```kotlin
@Test
fun `first 10 Fibonacci numbers form sequence`() {
    fun fibonacciSequence() = sequence {
        var terms = Pair(0, 1)

        while (true) {
            yield(terms.first)
            terms = terms.second to terms.first + terms.second
        }
    }

    val fibs = fibonacciSequence()
        .take(10)
        .toList()

    assertEquals(listOf(0, 1, 1, 2, 3, 5, 8, 13, 21, 34), fibs)
}
```

# 영역 함수

> let, run, apply, also

## apply로 객체 생성 후 초기화

> 객체 사용 전 생성자 인자만으로 할 수 없는 초기화 작업이 필요할 경우 apply 함수를 사용하자.

`apply` 함수는 this를 인자로 전달하고 this를 리턴하는 확장 함수다.
- 모든 제네릭 타입 `T`에 존재하는 확장 함수
- 명시된 블록을 수신자인 `this와` 함께 호출하고 해당 블록이 완료되면 `this` 리턴

```kotlin
public inline fun <T> T.apply(block: T.() -> Unit): T {
    contract {
        callsInPlace(block, InvocationKind.EXACTLY_ONCE)
    }
    block()
    return this
}
```

👉🏻 **apply를 사용하여 단 하나의 구문으로, 저장해야 할 인스턴스를 받아 새로운 키로 한번에 갱신**
- apply 블록은 이미 인스턴스화된 객체의 추가 설정을 위해 사용하는 가장 일반적인 방법

```kotlin
@Repository
class JdbcOfficerDAO(private val jdbcTemplate: JdbcTemplate) {
    private val insertOfficer = SimpleJdbcInsert(jdbcTemplate)
            .withTableName("OFFICERS")
            .usingGeneratedKeyColumns("id")

    // Officer의 id 속성은 apply 블록 안에서 갱신된 다음
    // Officer 인스턴스가 리턴
    fun save(officer: Officer) =
        officer.apply {
            // excuteAndReturnKey 메소드는 컬럼 이름과 같으로 이뤄진 맵을 인자로 받아
            // 데이터베이스에서 생성된 기본 키 값을 리턴
            id = insertOfficer.excuteAndReturnKey(
                    mapOf("rank" to rank,
                            "first_name" to first,
                            "last_name" to last))
            // ... 추가적인 초기화
        }
        
    // ...
}
```

---

## 부수 효과를 위한 also

> 부수 효과를 생성하는 동작을 수행하려면 `also` 함수를 사용하자.

also 함수는 코틀린 표준 라이브러리에 있는 확장 함수
- 모든 제네릭 타입 T에 추가되고
- block 인자 호출 수 자기 자신을 리턴

```kotlin
@kotlin.internal.InlineOnly
@SinceKotlin("1.1")
public inline fun <T> T.also(block: (T) -> Unit): T {
    contract {
        callsInPlace(block, InvocationKind.EXACTLY_ONCE)
    }
    block(this)
    return this
}
```

👉🏻 **also는 일반적으로 객체에 함수 호출을 연쇄시키기 위해 사용**

```kotlin
val book = createBook()
    .also { println(it) }
    .also { Logger.getAnonymousLogger().info(it.toString()) }
```

👉🏻 **서비스 테스트 작성 시 부수 효과로 로그 기록**

```kotlin
class Site(
    val name: String,
    val latitude: Double,
    val longitude: Double,
)

...

@Test
fun `lat,lng of Boston, MA`() = service.getLatLng("Boston", "MA")
    .also { logger.info(it.toString())} // 부수 효과로서 로그 기록
    .run {
        assertThat(latitude, `is`(closeTo(42.36, 0.01)))
        assertThat(longgitude, `is`(closeTo(-71.06, 0.01)))
    }
```

---

## let 함수와 엘비스 연산자

> 레퍼런스가 널일경우 기본값을 리턴하도록 하려면
>
> 엘비스 연산자를 결합한 안전 호출 연산자와 함께 let 영역 함수를 사용하자.

`let` 함수는 모든 제네릭 타입 T의 확장 함수
- 컨텍스트 객체가 아닌 블록의 결과를 리턴
- 객체를 위한 map처럼 마치 컨텍스트 객체의 변형처럼 동작

```kotlin
@kotlin.internal.InlineOnly
public inline fun <T, R> T.let(block: (T) -> R): R {
    contract {
        callsInPlace(block, InvocationKind.EXACTLY_ONCE)
    }
    return block(this)
}
```

👉🏻 **문자열 대문자 변경과 특수한 입력 처리**

```kotlin
@Test
fun `let test`() {
    fun processString_asis(str: String) =
        str.let {
            when {
                it.isEmpty() -> "Empty"
                it.isBlank() -> "Blank"
                else -> it.replaceFirstChar { if (it.isLowerCase()) it.titlecase(Locale.getDefault()) else it.toString() }
            }
        }

    fun processString_tobe(str: String?) =
        str?.let { // 안전 호출 연산자와 let 을 같이 사용
            when {
                it.isEmpty() -> "Empty"
                it.isBlank() -> "Blank"
                else -> it.replaceFirstChar { if (it.isLowerCase()) it.titlecase(Locale.getDefault()) else it.toString() }
            }
        } ?: "Null" // 널인 경우 처리하는 엘비스 연산자


    assertEquals("Abcdef", processString_asis("abcdef"))
    // assertEquals("", processString_asis(null)) // Null can not be a value of a non-null type String

    assertEquals("Abcdef", processString_tobe("abcdef"))
    assertEquals("Null", processString_tobe(null))
}
```

---

## 임시 변수로 let

> 연산 결과를 임시 변수에 할당하지 않고 처리하고 싶다면,
>
> 연산에 let 호출을 연쇄하고 let에 제공된 람다 또는 함수 레퍼런스 안에서 그 결과를 처리하자

👉🏻 **임시 변수를 사용하지 않고 let 함수로 바로 처리**

```kotlin
@Test
fun `let example as-is`() {
    val numbers = mutableListOf("one", "two", "three", "four", "five")
    val resultList = numbers.map { it.length }.filter { it > 3 }
    assertEquals(listOf(5, 4, 4), resultList)
}

@Test
fun `let example to-be`() {
    val numbers = mutableListOf("one", "two", "three", "four", "five")
    numbers.map { it.length }.filter { it > 3 }.let {
        assertEquals(listOf(5, 4, 4), it)
        // ...
    }
}
```

👉🏻 **List 출력을 위해 let(or also) 사용**
- `let` 함수는 블록의 결과를 리턴
- `also` 함수는 컨텍스트 객체를 리턴
  - 출력과 같은 부수적인 효과는 also를 사용하는 것이 더 코틀린다운 사용법

```kotlin
Gson().formJson(
    URL("http://api.open-notify.org/astros.json").readText(),
    AstroResult::class.java
).people.map { it.name }.let(::println)
```

# 코틀린 대리자

## 대리자를 사용해서 합성 구현하기

> **연산을 위임할 메소드가 포함된 인터페이스**를 만들고.
>
> **클래스에서 해당 인터페이스를 구현**한 후 `by` 키워드를 사용해 **바깥쪽에 래퍼 클래스**를 만들자

코틀린에서 `by` 키워드는 포함된 객체에 있는 모든 public 함수를 이 객체를 담고 있는 컨테이너를 통해 노출할 수 있다.
- 컨테이너에 노출하려면 포함된 객체의 public 메소드의 인터페이스를 생성해야 한다.
- 생성자에서 객체를 인스턴스화하고 모든 public 함수를 위임하도록 정의할 수 있다.
- 오직 포함된 객체의 public 함수만 노출

```kotlin
interface Dialable { // 연산을 위임할 메소드가 포함된 인터페이스
    fun dial(number: String): String
}

class Phone : Dialable {
    override fun dial(number: String) = "Dialing $number..."
}

interface Snappable { // 연산을 위임할 메소드가 포함된 인터페이스
    fun takePicture(): String
}

class Camera : Snappable {
    override fun takePicture() = "Taking picture..."
}

class SmartPhone(
    // 생성자에서 객체를 인스턴스화하고 모든 public 함수를 위임하도록 정의
    private val phone: Dialable = Phone(),
    private val camera: Snappable = Camera()
) : Dialable by phone, Snappable by camera 
// by 키워드를 사용해서 위임
// 내부적으로 SmartPhone 클래스는 위임된 속성을 인터페이스 타입으로 정의

private val smartPhone: SmartPhone = SmartPhone() // SmartPhone 인스턴스화

@Test
fun `by keyword test`() {
    // Dialing delegates to internal phone
    assertEquals("Dialing 555-1234...",
        smartPhone.dial("555-1234")) // 위임 함수 호출

    // Taking picture delegates to internal camera
    assertEquals("Taking picture...",
        smartPhone.takePicture()) // 위임 함수 호출
}
```

---

## lazy 대리자 사용하기

> 어떤 속성이 필요할 때까지 해당 속성의 초기화를 지연시키고 싶다면 `lazy` 대리자를 사용하자

코틀린은 어떤 속성의 **획득자**와 **설정자**가 대리자라고 불리는 다른 객체에서 구현되어 있다는 것을 암시하기 위해 속성에 `by` 키워드를 사용
- 코틀린 표준 라이브러리에 다수의 대리자 함수가 있는데,
- 그 중 가장 인기 있는 함수는 `lazy`

👉🏻 **처음 접근까지 속성의 초기화를 대기**
- 첫 호출은 lazy가 받은 람다를 실행하고 그 다음 변수에 저장될 값을 리턴
- 내부적으로 이 값을 캐시하는 Lazy 타입의 `ultimateAnswer$delegate`라는 특별한 속성을 생성

```kotlin
@Test
fun `lazy test`() {
    val ultimateAnswer: Int by lazy {
        println("computing the answer")
        42
    }

    assertEquals(42, ultimateAnswer) // 여기서 초기화
    assertEquals(42, ultimateAnswer)
}
```

`lazy`는 `LazyThreadSafetyMode` 타입의 이넘도 인자로 받는다.
- **SYNCHRONIZED**
  - 오직 하나의 스레드만 Lazy 인스턴스를 초기화할 수 있게 락 사용
- **PUBLICATION**
  - 초기화 함수가 여러 번 호출될 수 있지만 첫 번째 리턴값만 사용
- **NONE**
  - 락이 사용되지 않음

{% hint style="info" %}

**LazyThreadSafetyMode**

lazy의 lock 인자를 제공하면 값 계산 시 이 객체가 대리자를 동기화

lock 인자가 없을 경우 대리자는 자신 스스로 동기화

{% endhint %}

---

## 값이 널이 될 수 없게 만들기

> notNull 함수를 이용해 값이 설정되지 않았다면 예외를 던지는 대리자를 제공하자.

속성 초기화를 지연시키는 한 가지 방법은 속성에 처음 접근하기 전에 속성이 사용되면 예외를 던지는 대리자를 제공하는 `notNull` 함수를 사용하는 것이다.

👉🏻 **속성에 값이 제공되기 전 접근을 시도하면 IllegalStateException**

```kotlin
var shouldNotBeBull: String by Delegates.notNull()

@Test
fun `uninitialized value throws exception`() {
    assertThrows<IllegalStateException> { shouldNotBeBull }
}

@Test
fun `initialize value then retrieve it`() {
    shouldNotBeBull = "Hello, World!"
    assertDoesNotThrow { shouldNotBeBull }
    assertEquals("Hello, World!", shouldNotBeBull)
}
```

---

## observable, vetoable 대리자

> 변경 감지에는 `observable` 함수를
>
> 변경 적용 여부를 결정할 때는 `vetoable` 함수와 람다를 사용하자.

```kotlin
var watched: Int by Delegates.observable(1) { prop, old, new ->
    // 변수의 값이 변경될 때마다 메시지 출력
    println("${prop.name} changed from $old to $new")
}

var checked: Int by Delegates.vetoable(0) { prop, old, new ->
    // 오직 양수 값으로만 변경 가능
    println("Trying to change ${prop.name} from $old to $new")
    new >= 0
}

@Test
fun `watched variable prints old and new values`() {
    assertEquals(1, watched)
    watched *= 2 // watched changed from 1 to 2
    assertEquals(2, watched)
    watched *= 2 // watched changed from 2 to 4
    assertEquals(4, watched)
}

@Test
fun `veto values less than zero`() {
    assertAll(
        { assertEquals(0, checked) },
        { checked = 42; assertEquals(42, checked) }, // Trying to change checked from 0 to 42
        { checked = -1; assertEquals(42, checked) }, // Trying to change checked from 42 to -1
        { checked = 17; assertEquals(17, checked) } // Trying to change checked from 42 to 17
    )
}
```

{% hint style="info" %}

observable, vetoable 함수 구현은 개발자가 대리자를 작성할 때 참고할 만한 좋은 패턴이다.

```kotlin
package kotlin.properties

public object Delegates {
    public fun <T : Any> notNull(): ReadWriteProperty<Any?, T> = NotNullVar()

    public inline fun <T> observable(initialValue: T, crossinline onChange: (property: KProperty<*>, oldValue: T, newValue: T) -> Unit):
            ReadWriteProperty<Any?, T> =
        object : ObservableProperty<T>(initialValue) {
            override fun afterChange(property: KProperty<*>, oldValue: T, newValue: T) = onChange(property, oldValue, newValue)
        }

    public inline fun <T> vetoable(initialValue: T, crossinline onChange: (property: KProperty<*>, oldValue: T, newValue: T) -> Boolean):
            ReadWriteProperty<Any?, T> =
        object : ObservableProperty<T>(initialValue) {
            override fun beforeChange(property: KProperty<*>, oldValue: T, newValue: T): Boolean = onChange(property, oldValue, newValue)
        }

}
```

**inline and crossinline**

inline 키워드는 컴파일러가 함수만 호출하는 완전히 새로운 객체를 생성하는 것이 아닌 해당 호출 위치를 실제 소스 코드로 대체하도록 지연시키고

가끔 inline 함수는 지역 객체 또는 중첩 함수 같은 다른 컨텍스트에서 실행되어야 하는 파라미터로서 전달되는 람다.
- '로컬이 아닌' 제어 흐름은 람다 내에서는 허용되지 않는다.

ObservableProperty 확장 클래스 대신 observable 또는 vetoable 관련해 onChange 람다가 실행되기 떄문에 crossinline 제어자가 필요

{% endhint %}

---

## 대리자로서 Map 제공

> 맵을 제공해 객체를 초기화하고 싶을 경우 getValue, setValue 함수 구현을 사용하자.

👉🏻 **MutableMap을 인자로 받고 해당 맵의 키에 해당하는 값으로 클래스 속성 초기화**

```kotlin
data class Project(val map: MutableMap<String, Any?>) {
    val name: String by map
    val priority: Int by map
    var completed: Boolean by map
}

@Test
fun `use map delegate for Project`() {
    val project = Project(
        mutableMapOf(
            "name" to "Learn Kotlin",
            "priority" to 5,
            "completed" to true))

    assertAll(
        { assertEquals("Learn Kotlin", project.name) },
        { assertEquals(5, project.priority) },
        { assertTrue( project.completed) }
    )
}
```

👉🏻 **JSON 문자열로 속성을 파싱할 경우에도 사용**

```kotlin
private fun getMapFormJSON() =
    Gson().fromJson<MutableMap<String, Any?>>(
        """{ "name": "Learn Kotlin", "priority": 5, "completed": true }""",
        MutableMap::class.java)

@Test
fun `create project from map parsed from JSON string`() {
    val project = Project(getMapFormJSON())
    assertAll(
        { assertEquals("Learn Kotlin", project.name) },
        { assertEquals(5, project.priority) },
        { assertTrue( project.completed) }
    )
}
```

---

## 사용자 정의 대리자 만들기

> `ReaOnlyProperty` or `ReadWriteProperty`를 구현하는 클래스를 생성하여 직접 속성 대리자를 작성하자.

값을 획득하거나 설정하는 동작을 다른 객체에 위임할 수 있다.
- 사용자 정의 속성 대리자를 생성하려면 `ReaOnlyProperty` or `ReadWriteProperty` 인터페이스에 존재하는 함수를 제공해야 한다.

`ReaOnlyProperty`, `ReadWriteProperty` 인터페이스의 시그니처
- 대리자를 만들려고 위 인터페이스를 구현할 필요가 없다.
- 아래 코드에 보이는 시그니처와 동일한 getValue, setValue 함수만으로 충분하다.

```kotlin
public fun interface ReadOnlyProperty<in T, out V> {
    public operator fun getValue(thisRef: T, property: KProperty<*>): V
}

public interface ReadWriteProperty<in T, V> : ReadOnlyProperty<T, V> {
    public override operator fun getValue(thisRef: T, property: KProperty<*>): V
    public operator fun setValue(thisRef: T, property: KProperty<*>, value: V)
}
```

👉🏻 **간단한 대리자 예제**

```kotlin
class Delegate {
    operator fun getValue(thisRef: Any?, property: KProperty<*>): String {
        return "$thisRef, thank you for delegating '${property.name}' to me!"
    }

    operator fun setValue(thisRef: Any?, property: KProperty<*>, value: String) {
        println("$value has been assigned to '${property.name}' in $thisRef.")
    }
}

@Test
fun `delegate test`() {
    class Example {
        var p: String by Delegate()
    }

    val e  = Example()
    println(e.p) // ScopeFunction$delegate test$Example@710636b0, thank you for delegating 'p' to me!
    e.p = "NEW" // NEW has been assigned to 'p' in ScopeFunction$delegate test$Example@710636b0.
}
```

완전히 다른 예제 집합으로 그레이들 빌드 도구는 위임된 속성을 통해 컨테이너와 상호작용할 수 있게 도와주는 `Kotlin DSL`을 제공한다.
- 프로젝트 자체(org.gradle.api.Project)와 연관된 속성의 집합
- 프로젝트 전체에 사용할 수 있는 extra 속성

```groovy
val myProperty; String by project // project 속성 myProperty를 사용 가능하게 만들기
val myNullableProperty: String? by project // 널이 될 수 있는 속성을 사용 가능하게 만들기

val myNewProperty by extra("initial value") // extra 속성 myNewProperty를 만들고 초기화
val myOtherNewProperty by extra { "lazy initial value" } // 처음 접근이 일어날 때 초기화되는 속성 생성
```

{% hint style="info" %}

**속성 대리자 생성**

속성 대리자 생성 방법은 간단하지만 이미 표준 라이브러리에 들어 있거나 또는 그레이들과 같은 서드파티가 제공하는 대리자를 사용하게 될 가능성이 높다.

{% endhint %}