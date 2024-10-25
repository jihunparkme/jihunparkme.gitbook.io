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