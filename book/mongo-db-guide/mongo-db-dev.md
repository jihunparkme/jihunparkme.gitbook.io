# PART II. 몽고DB 개발

# 인덱싱

## 인덱싱 소개

> 인덱스를 사용하지 않는 쿼리를 컬렉션 스캔이라 하며,
>
> 서버가 쿼리 결과를 찾으려면 '전체 내용을 살펴봐야 함'을 의미

1️⃣ 인덱스 생성

> 인덱스를 만들려면 `createIndex` 컬렉션 메서드를 사용

```sql
db.users.createIndex({"username" : 1})
```

- 다른 셸에서 db.currentOp()를 실행하거나 mongod의 로그를 확인해 인덱스 구축의 진행률 체크 가능

인덱스는 쿼리 시간에 놀라운 차이를 만든다.
- 단점이라면, 인덱싱된 필드를 변경하는 쓰기(삽입, 갱신, 삭제) 작업은 오래 걸린다.
  - 데이터가 변경될 때마다 도큐먼트뿐 아니라 모든 인덱스를 갱신 필요
- 어떤 필드가 인덱싱하기에 적합한지 신중이 파악이 필요

{% hint style="info" %}

**몽고DB 인덱스**

전형적인 관계형 데이터베이스의 인덱스와 거의 동일하게 작동

{% endhint %}

인덱스를 생성할 필드를 선택하려면, 자주 쓰는 쿼리와 빨리 수행해야 하는 쿼리를 조사해 공통적인 키 셋을 찾아보자.

.

2️⃣ 복합 인덱스

> 두 개 이상의 필드로 구성된 인덱스
>
> 쿼리에서 정렬 방향이 여러 개이거나 검색 조건에 여러 개의 키가 있을 때 유용

몽고DB가 실행하는 쿼리 종류에 따라 인덱스를 사용하는 방법이 다른데 가장 많이 사용하는 세 가지 방법을 보자.

👉🏻 **단일 값을 찾는 동등 쿼리**

```sql
db.users.find({"age": 21}).sort({"username": -1})
```

👉🏻 **범용 쿼리**

```sql
db.users.find({"age": {"$gte": 31, "$lte": 30}})
```

👉🏻 **정렬이 포함된 다중값 쿼리**
- 결과를 반환하기 전에 메모리에서 정렬 작업을 수행하여 정렬 시간이 소요
- 속도는 검색 조건과 일치하는 결과가 얼마나 많은지에 따라 다름

```sql
db.users.find({"age": {"$gte": 31, "$lte": 30}}).sort({"username": 1})
```

{% hint style="info" %}

복합 인덱스를 구성할 때는 정렬 키를 첫 번째에 놓으면 좋다.

{% endhint %}

.

4️⃣ 복합 인덱스 사용

- 읽기, 쓰기를 가능한 효율적으로 수행하도록 인덱스를 설계하자.
- 인덱스의 선택성을 고려하자.
- 커서 hint 메서드를 사용하면 모양이나 이름을 지정함으로써 사용할 인덱스 지정 가능
- `planCacheSetFilter` 함수를 인덱스 필터와 함께 사용하면, 쿼리 옵티마이저가 인덱스 필터에 지정된 인덱스만 고려하도록 제안 가능

모든 데이터셋에 해당되지는 않지만, 일반적으로 동등 필터를 사용할 필드가 다중값 필터를 사용할 필드보다 앞에 오도록 복합 인덱스를 설계해야 한다.
- 복합 인덱스 설계 시 인덱스를 사용할 공통 쿼리 패턴의 동등 필터, 다중값 필터, 정렬 구성 요소를 처리하는 방법을 알아야 한다.
- 인덱스를 더 잘 설계하면 인메모리 정렬을 피할 수 있다.

```sql
db.students.createIndex({class_id:1, student_id:1})
```

{% hint style="info" %}

**복합 인덱스 설계 시**

- 동등 필터에 대한 키를 맨 앞에 표시하자.
- 정렬에 사용되는 키는 다중값 필드 앞에 표시하자.
- 다중값 필터에 대한 키는 마지막에 표시하자.

지침에 따라 복합 인덱스 설계 후, 인덱스가 지원하는 쿼리 패턴의 범위를 실제 워크로드에서 테스트하자.

{% endhint %}

**키 방향 선택하기**

- 복합 정렬을 서로 다른 방향으로 최적화하려면 방향이 맞는 인덱스를 사용해야 한다.
- 인덱스 방향은 다중 조건에 따라 정렬할 때만 문제가 된다.
- 단일 키로 정렬하면 몽고DB는 인덱스를 쉽게 역순으로 읽을 수 있다.

**커버트 쿼리 사용하기**

인덱스가 쿼리가 요구하는 값을 모두 포함하면, 쿼리가 커버드 된다고 한다.
- 실무에서는 도큐먼트로 받지 말고 항상 커버드 쿼리(covered query)를 사용하자.
- 이 방법으로 작업 셋을 훨씬 작게 만들 수 있다.
- 쿼리가 확실히 인덱스만 사용하게 하려면 "_id" 필드를 반환받지 않도록 반환받을 키를 지정해야 한다.

**암시적 인덱스**

- 여러 키에 필요한 만큼 적용 가능
- 인덱스가 N 개의 키를 가진다면 키들의 앞부분은 '공짜'인덱스가 된다.

```json
// 아래 인덱스가 있다면 
{"a" : 1, "b" : 1, "c" : 1, ... "z" : 1 } 
// 공짜 인덱스를 가짐
{"a" : 1}, {"a" : 1, "b" : 1 }, {"a" : 1, "b" : 1, "c" : 1 }  ...
```

. 

5️⃣ $ 연산자의 인덱스 사용법

**비효율적인 연산자**

- 일반적으로 부정 조건은 비효율적
- "`$ne`" 쿼리는 인덱스를 사용하긴 하지만 잘 활용하지 못 한다.

**범위**

- 복합 인덱스는 몽고DB가 다중 절 쿼리를 더 효율적으로 실행하도록 돕는다.

**OR 쿼리**

- 현재 몽고DB는 쿼리당 하나의 인덱스만 사용 가능
- 유일한 예외는 "`$or`"
  - "`$or`"는 두 개의 쿼리를 수행하고 결과를 합치므로 "`$or`"은 "`$or`"절마다 하나씩 인덱스 사용
- 가능하면 "`$or`"보다는 "`$in`"을 사용하자.
  - 일반적으로 두 번 쿼리해서 결과를 병합하면 한 번 쿼리할 때보다 훨씬 비효율적
  - "`$or`"를 사용해야 한다면 몽고DB가 두 쿼리의 결과를 조사하고 중복을 모두 제거해야 한다.

.

6️⃣ 객체 및 배열 인덱싱

**내장 도큐먼트 인덱싱하기**

- 인덱스는 일반적인 키에 생성될 떄와 동일한 방식으로 내장 도큐먼트에 키 생성 가능
- 서브 필드에 인덱스를 만들어 해당 필드를 이용하는 쿼리의 속도를 높일 수 있다.
- 서브 도큐먼트 전체를 인덱싱하면 서브 도큐먼트 전체에 쿼리할 때만 도움이 된다.

```sql
db.users.createIndex({"loc.city" : 1})
```

**배열 인덱싱하기**

- 배열에도 인덱스 생성 가능
- 인덱스를 사용하면 배열의 특정 요소를 효율적으로 탐색
- 단, 내장 도큐먼트와 달리 배열 전체를 단일 개체처럼 인덱싱 불가
  - 배열 필드 인덱싱은 배열 자체가 아니라 배열의 각 요소를 인뎅싱하기 때문

```sql
db.blog.createIndex({"comments.date" : 1})
```

**다중키 인덱스가 미치는 영향**

- 어떤 도큐먼트가 배열 필드를 인덱스 키로 가지면 인덱스는 즉시 다중키 인덱스로 표시
- 다중키 인덱스는 비다중키 인덱스보다 약간 느릴 수 있음
  - 하나의 도큐먼트를 여러 개의 인덱스 항목이 가리킬 수 있으므로 몽고DB는 결과 반환 전 중복을 제거

.

7️⃣ 인덱스 카디널리티

- 카디널리티(cardinality)는 컬렉션의 한 필드에 대해 고윳값이 얼마나 많은지를 나타냄
- "username", "email" 같은 필드는 컬렉션의 각 도큐먼트마다 유일한 값을 가지며, 이는 높은 카디널리티
- 일반적으로 필드의 카디널리티가 높을수록 인덱싱이 더욱 도움
- 적어도 복합 인덱스에서 높은 카디널리티 키를 낮은 카디널리티 키 앞에 놓자.

## explain 출력

> explain은 쿼리에 대한 많은 정보를 제공하며, 느린 쿼리를 위한 중요한 진단 도구
>
> 쿼의 explain 출력을 보면 어떤 인덱스가 어떻게 사용되는지 알 수 있다.

쿼리가 "COLLSCAN"을 사용하면 인덱스를 사용하지 않음을 알 수 있다.

```sql
db.users.find({"age" : 42}).explain('executionStats')
```

- "totalKeysExamined"는 검색한 인덱스 항목 개수
- 스캔한 도큐먼트 개수 "nscannedObjects"
- "executionTimeMillis"는 서버가 요청을 받고 응답을 보낸 시점까지 쿼리가 얼마나 빨리 실행됐는지

중요한 필드
- `isMultiKey` : 다중키 인덱스 사용 여부(true/false)
- `nReturned` : 쿼리에 의해 반환된 도큐먼트 개수
- `totalDocsExamined` : 몽고DB가 디스크 내 실제 도큐먼트를 가리키는 인덱스 포인터를 따라간 횟수
- `totalKeysExamine` : 인덱스가 사용되었다면 살펴본 인덱스 항목 개수
- `stage` : 몽고DB가 인덱스를 사용해 쿼리할 수 있었는지 여부
  - "COLSCAN": 컬렉션 스캔
  - "IXSCAN": 인덱스 스캔
- `needYields` : 쓰기 요청을 처리하도록 쿼리가 양보한 횟수
- `executionTimeMillis` : DB가 쿼리하는 데 걸린 시간
- `indexBounds` : 인덱스가 어떻게 사용됐는지 설명하며 탐색한 인덱스의 범위를 제공

## 인덱스를 생성하지 않는 이유

> 인덱스는 데이터의 일부를 조회할 때 가장 효율적이며 어떤 쿼리는 인덱스가 없는 게 더 빠르다

인덱스는 컬렉션에서 가져와야 하는 부분이 많을수록 비효율적인데, 인덱스를 하나 사용하려면 두 번의 조회를 해야 하기 때문
- 한 번은 인덱스 항목을 살펴보고, 또 한 번은 도큐먼트를 가리키는 인덱스의 포인터를 따라간다
- 반면, 컬렉션 스캔 시에는 도큐먼트만 살펴보면 된다.
- 최악의 경우(컬렉션의 모든 도큐먼트를 반환해야 할 경우) 인덱스를 사용하면 두 배나 많은 조회를 수행하며 컬렉션 스캔보다 훨씬 느리다.

## 인덱스 종류

**고유 인덱스**

- 고유 인덱스는 각 값이 인덱스에 최대 한 번 나타나도록 보장

**부분 인덱스**

- 부분 인덱스를 만들려면 "partialFilterExpression" 옵션을 포함

## 인덱스 관리

데이터베이스의 인덱스 정보는 모두 `system.indexes` 컬렉션에 저장
- 특정 컬렉션의 모든 인덱스 정보를 확인하려면 `db.c.getIndexes()`를 실행

```sql
db.studnts.getIndexes()
```

- "key"는 힌트에 사용하거나, 인덱스가 명시돼야 하는 위치에 사용할 수 있음
- 인덱스명("name")은 dropIndexes와 같은 관리적인 인덱스 작업에서 식별자로 사용

**인덱스 식별**

- 인덱스명은 기본적으로 `키명1_방향1_키명2_방향_..._키명N_방향N`
- `getLastError` 호출은 인덱스 생성의 성공 여부 혹은 실패 원인을 보여준다.

# 집계 프레임워크

## 파이프라인

> 집계 프레임워크는 파이프라인 개념을 기반으로 한다.

파이프라인은 몽고DB 컬렉션과 함께 작동
- 파이트라인은 단계로 구성되며 각 단계는 입력에 서로 다른 데이터 처리 작업을 수행하고
- 출력으로 도큐먼트를 생성해 다음 단계로 전달

## 단계 시작

`aggregate`는 집계 쿼리를 실행할 때 호출하는 메서드
- "_id" 필드는 제외하고 "name"과 "founded_year"는 포함
- **필터링을 위한 일치 단계**와 **출력을 도큐먼트당 두 개의 필드로 제한하는 선출 단계**가 존재

```sql
db.companies.aggregate([
    {$match: {founded_year: 2004}},
    {
        $project: {
            _id: 0,
            name: 1,
            founded_year: 1
        }
    }
])
```

**5단계의 파이프라인**
- 제한을 선출단계 이전에 수행하도록 구축 limit
- 파이프라인을 구축할 때 한 단계에서 다른 단계로 전달해야하는 도큐먼트 수를 반드시 제한
- 순서가 중요하다면 제한 단계 전에 정렬을 수행 sort

```sql
db.companies.aggregate([
    {$match: {founded_year: 2004}}, // 컬렉션 필터링 단계
    {$sort: {name: 1}}, // 정렬
    {$skip: 10}, // 항목 건너뛰기
    {$limit: 5} // 도큐먼트 선출
    { // 도큐먼트 모양
        $project: {
            _id: 0,
            name: 1,
        }
    }
])
```

## $project

> 중첩 필드nested field()를 승격(promoting)하는 방법

greylock 파트너스가 참여한 펀딩 라운드를 포함하는 모든 회사를 필터링
- `$` 문자는 선출 단계에서 값 지정에 사용되며, 해당 값이 필드 경로로 해석되고 각 필드에서 선출할 값을 선택하는 데 사용

```sql
db.companies.aggregate([
    {$match: {"funding_rounds.investments.financial_org.permalink": "greylock"}},
    {
        $project: {
            _id: 0,
            name: 1,
            ipp: "$ipo.pub_year",
            valuation: "$ipo.valuation_amount",
            funders: "$funding_rounds.investments.financial_org.permalink"
        }
    }
])
```

## $unwind

입력 도큐먼트에서 배열 필드(key3)를 전개하도록 구성될 때 아래와 같은 도큐먼트를 생성
- 즉 배열 10개의 요소가 있으면 전개 단계에서는 10개의 출력 도큐먼트가 생성

```text
{ 
  key1 : "value1",
  key2 : "value2",
  key3 : ["elem1", "elem2", "elem3",]}
}

...

$unwind

{ 
  key1 : "value1",
  key2 : "value2",
  key3 : "elem1"
}
{ 
  key1 : "value1",
  key2 : "value2",
  key3 : "elem2"
}
{ 
  key1 : "value1",
  key2 : "value2",
  key3 : "elem3"
}
```

Example
- 그레이록이 한 번이라도 펀딩 라운드에 참여한 회사를 먼저 필터링
- 펀딩 라운드를 전개하고 
- 다시 필터링해 그레이록이 실제 팜여한 펀딩 라운드를 나타내는 도큐먼트만 선출 단계로 전달

```json
db.companies.aggregate([
    {$match: {"funding_rounds.investments.financial_org.permalink": "greylock"}},
    {$unwind: "$funding_rounds"},
    {$match: {"funding_rounds.investments.financial_org.permalink": "greylock"}},
    {
        $project: {
            _id: 0,
            name: 1,
            individualFunder: "$funding_rounds.investments.person.permalink",
            fundingOrganizaion: "$funding_rounds.investments.financial_org.permalink",
            amount: "$funding_rounds.raised_amount",
            year: "$funding_rounds.funded_year",
        }
    }
])
```

## 배열 표현식

`$filter` 연산자는 배열 필드와 함께 작동하도록 설계되었으며, 우리가 제공하는 옵션을 지정
- 첫 번째 옵션은(input) : 단순히 배열을 지정
- 두 번째 옵션(as)은 나머지 필터 표현식 전체에서 funding_rounds 배열에 사용할 이름 지정
  - 필터 표현식 내에서 변수를 정의
- 세 번째 옵션은 조건을 지정
  - `$$`는 작업 중인 표현식 내에서 정의된 변수를 참조하는 데 사용

```sql
db.companies.aggregate([
    {$match: {"funding_rounds.investments.financial_org.permalink": "greylock"}},
    {
        $project: {
            _id: 0,
            name: 1,
            founded_year: 1,
            rounds: {
                $filter: {
                    input: "$funding_rounds",
                    as: "round",
                    cond: {$gte: ["$$round.raised_amount", 100000000]}
                }
            }
        }
    },
    {$match: {"rounds.investments.financial_ord.permalink": "greylock"}},
])
```

{% hint style="info" %}

[집계 참조](https://www.mongodb.com/ko-kr/docs/manual/reference/aggregation/)

{% endhint %}

## 누산기

**선출 단계에서 누산기 사용**

"funding_rounds" 필드를 포함하고 funding_rounds 배열이 있지 않은 도큐먼트를 필터링

```sql
db.companies.aggregate([
    { $match: {"funding_rounds": { $exists: true, $ne: []}}},
    {
        $project: {
            _id: 0,
            name: 1,
            largest_round: { $max: "$funding_rounds.raised_amount" }
        }
    }
])
```

`$sum` 누산기를 사용해 컬렉션에 있는 각 회사의 총 모금액을 계산

```sql
db.companies.aggregate([
    { $match: {"funding_rounds": { $exists: true, $ne: []}}},
    {
        $project: {
            _id: 0,
            name: 1,
            largest_round: { $max: "$funding_rounds.raised_amount" }
        }
    }
])
```

{% hint style="info" %}

[연산자](https://www.mongodb.com/ko-kr/docs/manual/reference/operator/aggregation/)

{% endhint %}

## 그룹화

> 그룹 단계에서는 여러 도큐먼트의 값을 함께 집계하고, 집계한 값에 평균 계산과 같은 집계 작업을 수행

```sql
db.companies.aggregate([
    {
        $group: {
            _id: {foundede_year: "$founded_year"},
            average_number_of_employees: {$avg: "$number_of_employees"}
        }
    },
    {$sort: {average_number_od_employees: -1}}
])
```