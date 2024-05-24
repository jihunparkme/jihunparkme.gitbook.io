# Concurrency issues

[재고시스템으로 알아보는 동시성이슈 해결방법](https://www.inflearn.com/course/%EB%8F%99%EC%8B%9C%EC%84%B1%EC%9D%B4%EC%8A%88-%EC%9E%AC%EA%B3%A0%EC%8B%9C%EC%8A%A4%ED%85%9C/dashboard) 강의를 듣고 요약한 내용입니다.

# Race Condition

[Race condition](https://en.wikipedia.org/wiki/Race_condition)

경쟁상태는 두 개 이상의 스레드가 공유 데이터에 액세스할 수 있고, 동시에 변경을 하려고 할 때 발생하는 문제입니다.

```java
@Service
@RequiredArgsConstructor
public class StockService {

    private final StockRepository stockRepository;

    public synchronized void decrease(Long id, Long quantity) {
        final Stock stock = stockRepository.findById(id).orElseThrow();
        stock.decrease(quantity);

        stockRepository.saveAndFlush(stock);
    }
}
```

우리는 아래 테스트에서 0의 결과를 예상하지만, `동시에 들어오는 요청들이 갱신 전 값을 읽고 수정`하면서 실제 갱신이 누락되는 현상이 발생하게 됩니다.

| Thread-1 | Stock      | Thread-2 |
| -------- | ---------- | -------- |
| select   | quantity:5 |          |
|          | quantity:5 | select   |
| update   | quantity:4 |          |
|          | quantity:4 | update   |

```java
@Test
void decrease_inventory_concurrent_requests() throws InterruptedException {
    int threadCount = 100;
    ExecutorService executorService = Executors.newFixedThreadPool(32);
    CountDownLatch latch = new CountDownLatch(threadCount);

    for (int i = 0; i < threadCount; i++) {
        executorService.submit(() -> {
            try {
                stockService.decrease(1L, 1L);
            } finally {
                latch.countDown();
            }
        });
    }

    latch.await();

    Stock stock = stockRepository.findById(1L).orElseThrow();

    assertEquals(0, stock.getQuantity());
}
```

## Java Synchronized

synchronized 를 메서드 선언부에 붙여주면 해당 메서드는 한 개의 스레드만 접근이 가능하게 됩니다.

```java
@Transactional
public synchronized void decreases(Long id, Long quantity) {
    final Stock stock = stockRepository.findById(id).orElseThrow();
    stock.decrease(quantity);

    stockRepository.saveAndFlush(stock);
}
```

.

⚠️ **여기서 문제가 있습니다.**

`Spring Transactional` 의 동작 방식은 아래와 같이 우리가 만든 클래스를 래핑한 클래스를 새로 만들어서 실행하게 됩니다.

 ```java
 public class TransactionService {
    private StockService stockService;

    ...

    public void decreases(Long id, Long quantity) {
        startTransaction();

        stockService.decreases(id, quantity);

        endTransaction();
    }

    ...
 }
 ```

 우리가 만든 클래스를 필드로 가지고 호출하게 되는데 `트랜잭션 종료 전에 다른 스레드가 갱신된 전 값을 읽게 되면` 결국 이전과 동일한 문제가 발생하게 됩니다.

 여기서 단순하게 Transactional Annotation 을 제거하여 문제를 해결할 수는 있습니다.

.

⚠️ **하지만, Java Synchronized 의 문제는 또 존재합니다.**

Java Synchronized 는 하나의 프로세스 안에서만 보장이 됩니다.

그러다보니 서버가 1대일 경우에는 괜찮겠지만, 서버가 2대 이상일 경우 결국 `여러 스레드에서 동시에 데이터에 접근`할 수 있게 되면서 레이스 컨디션이 발생하게 됩니다.

| Time  | Thread-1 | Stock      | Thread-2 |
| ----- | -------- | ---------- | -------- |
| 10:00 | select   | quantity:5 |          |
|       |          | quantity:5 | select   |
| 10:05 | update   | quantity:4 |          |
|       |          | quantity:4 | update   |

대부분의 운영 서비스는 2대 이상의 서버로 운영되기 때문에 Java Synchronized 는 거의 사용되지 않는 방법입니다.

## DataBase Lock

> MySQL

### Pessimistic Lock

**비관적 락.**
- 실제로 `데이터에 Lock`을 걸어서 정합성을 맞추는 방법
- `Row or Table 단위`로 Locking
- `Exclusive Lock`을 걸게 되며, 다른 트랜잭션에서는 `Lock 해제 전에 데이터를 가져갈 수 없음`
- `데드락`(두 개 이상의 작업이 서로 상대방의 작업이 끝나기 만을 기다는 상태) 주의 필요
- 충돌이 빈번하게 일어나거나 예상된다면 추천하는 방식

**장점.**
- 충돌이 빈번하게 일어난다면 `Optimistic Lock` 보다 성능이 좋을 수 있다.
- 락을 통해 업데이터를 제어하므로 **데이터 정합성이 보장**된다.

**단점.**
- 별도의 락을 잡기 때문에 **성능 감소**가 있을 수 있다.

```java
/** Repository **/
@Lock(LockModeType.PESSIMISTIC_WRITE)
@Query("select s from Stock s where s.id = :id")
Stock findByIdWithPessimisticLock(@Param("id") Long id);

...

/** Service **/
@Transactional
public void decrease(Long id, Long quantity) {
    final Stock stock = stockRepository.findByIdWithPessimisticLock(id);
    stock.decrease(quantity);
    stockRepository.save(stock);
}
```

[commit](https://github.com/jihunparkme/Study-project-spring-java/commit/69ceb1a7810835483eb33f6a8b0c88fd10d5d094)

### Optimistic Lock

**낙관적 락.**
- 실제로 Lock 을 이용하지 않고 `버전을 이용`함으로써 정합성을 맞추는 방법
- 먼저 데이터를 읽은 후 update 수행 시, 현재 내가 읽은 `버전이 맞는지 확인하며 업데이트` 수행
- 내가 읽은 버전에서 수정사항이 생겼을 경우 애플리케이션에서 다시 읽은 후에 작업을 수행
- 충돌이 빈번하게 일어나지 않을 경우 추천하는 방식

**장점.**
- 별도의 락을 잡지 않으므로 Pessimistic Lock 보다 성능상 이점이 있다.

**단점.**
- 업데이트 실패 시 재시도 로직을 개발자가 직접 작성해야 한다.

```java
/** Entity **/
@Version
private Long version;

...

/** Repository **/
@Lock(value = LockModeType.OPTIMISTIC)
@Query("select s from Stock s where s.id = :id")
Stock findByIdWithOptimisticLock(@Param("id") Long id);

...

/** Service **/
@Transactional
public void decrease(Long id, Long quantity) {
    final Stock stock = stockRepository.findByIdWithOptimisticLock(id);
    stock.decrease(quantity);
    stockRepository.save(stock);
}

...

/** Facade **/
@Component
@RequiredArgsConstructor
public class OptimisticLockStockFacade {

    private final OptimisticLockStockService optimisticLockStockService;

    public void decrease(Long id, Long quantity) throws InterruptedException {
        while (true) { // 업데이트 실패 시 재시도
            try {
                optimisticLockStockService.decrease(id, quantity);
                break;
            } catch (Exception e) {
                Thread.sleep(50);
            }
        }
    }
}
```

[commit](https://github.com/jihunparkme/Study-project-spring-java/commit/db1e69c46ab0cf3c50a3f015aecc7c0ffb1db653)

### Named Lock

- 이름을 가진 [Metadata Locking](https://dev.mysql.com/doc/refman/8.0/en/metadata-locking.html)
- `이름을 가진 Lock` 획득 후, 해제할 때까지 다른 세션은 이 Lock 을 획득할 수 없도록 함
- Transaction 종료 시 `Lock 이 자동으로 해제되지 않는 점` 주의
  - 별도의 명령어로 해제를 수행해 주거나 선점 시간이 끝나야 해제
- 비관적 락과 유사하지만 로우나 테이블이 아닌 `메타데이터의 락킹`을 하는 방식
- 커넥션 풀 부족 현상을 막기 위해 데이터 소스를 분리해서 사용할 것을 권장
- 주로 분산락 구현 시 사용

**장점.**
- Timeout을 쉽게 구현 가능

**단점.**
- 트랜잭션 종료 시 락 해제, 세션 관리 필요
- 구현 방법이 복잡해질 수 있음

```java
/** Repository **/
@Query(value = "select get_lock(:key, 3000)", nativeQuery = true)
void getLock(@Param("key") String key);

@Query(value = "select release_lock(:key)", nativeQuery = true)
void releaseLock(@Param("key") String key);

...

/** Service **/
@Transactional(propagation = Propagation.REQUIRES_NEW)
public void decrease(Long id, Long quantity) {
    final Stock stock = stockRepository.findById(id).orElseThrow();
    stock.decrease(quantity);

    stockRepository.saveAndFlush(stock);
}

...

/** Facade **/
@Component
@RequiredArgsConstructor
public class NamedLockStockFacade {

    private final LockRepository lockRepository;

    private final StockService stockService;

    @Transactional
    public void decrease(Long id, Long quantity) {
        try {
            lockRepository.getLock(id.toString());
            stockService.decrease(id, quantity);
        } finally {
            lockRepository.releaseLock(id.toString());
        }
    }
}
```

[commit](https://github.com/jihunparkme/Study-project-spring-java/commit/9ba0d029795cba455cf71e1c941b60d8a2df85cf)

> Reference.
>
> - [Locking Functions](https://dev.mysql.com/doc/refman/8.0/en/locking-functions.html)

## Redis

MySQL의 Named Lock과 유사한 방식

### Lettuce

- `setnx`(set if not exist) 명령어를 활용하여 분산락 구현
- `Spin Lock` 방식
  - 락을 획득하려는 스레드가 락을 사용할 수 있는지 반복적으로 확인하면서 락 획득을 시도하는 방식
  - 재시도 로직 개발 필요

**장점.**
- 구현이 단순
- 별도의 라이브러리가 불필요(spring data redis 사용 시 기본적으로 Lettuce 적용)

**단점.**
- Spin Lock 방식이므로 동시에 많은 스레드가 락 획득 대기 상태라면 레디스에 부하가 갈 수 있음
  - 락 획득 재시도 간 대기 시간도 필요

```bash
# key 는 1 이고, value 는 lock 인 데이터 생성
> setnx 1 lock
(integer) 1

# 이미 1 이라는 키가 있으므로 실패
> setnx 1 lock
(integer) 0

# 키 삭제
> del 1
(integer) 1
```

```java
/* Repository */
public Boolean lock(Long key) {
    return redisTemplate
            .opsForValue()
            .setIfAbsent(generateKey(key), "lock", Duration.ofMillis(3_000));
}

public Boolean unlock(Long key) {
    return redisTemplate.delete(generateKey(key));
}

...

/* Facade */
public void decrease(Long key, Long quantity) throws InterruptedException {
    while (!redisLockRepository.lock(key)) {
        Thread.sleep(100);
    }

    try {
        stockService.decrease(key, quantity);
    } finally {
        redisLockRepository.unlock(key);
    }
}
```

[commit](https://github.com/jihunparkme/Study-project-spring-java/commit/5a916ad7efe8e40d68cce113dc03e8b077c05b5d)

### Redisson

[Redisson/Spring Boot Starter](https://mvnrepository.com/artifact/org.redisson/redisson-spring-boot-starter)

- `pub-sub` 기반으로 Lock 구현 제공
  - 채널을 만들고 락을 점유중인 스레드가 락 획득을 대기중인 스레드에게 락 해제라는 메시지를 전송하면, 메시지를 전달받은 스레드가 락 획득을 시도하는 방식
- 락 획득을 위해 반복적으로 락 획득을 시도하는 Lettuce 대비 부하를 줄일 수 있음

```bash
Thread-1 ----------> Channel -------------------> Thread-2
          I'm done            Try to get a Lock
```

```bash
# ch1 채널 구독(thread-2)
> subscribe ch1
1) "subscribe"
2) "ch1"
3) (integer) 1

# 메시지 전송(thread-1)
> publish ch1 hello
(integer) 1

# thread-2 메시지 수신
...
1) "message"
2) "ch1"
3) "hello"
```

**장점.**
- pub-sub 기반 구현으로 lettuce 대비 레디스에 부하가 덜 감
- 락 획득 재시도를 기본적으로 제공

**단점.**
- 복잡한 구현
- 별도의 라이브러리 필요

```java
/* Facade */
public void decrease(Long key, Long quantity) {
    RLock lock = redissonClient.getLock(key.toString());

    try {
        boolean available = lock.tryLock(10, 1, TimeUnit.SECONDS); // (락 획득 시도 시간, 점유 시간)

        if (!available) {
            System.out.println("lock 획득 실패");
            return;
        }

        stockService.decrease(key, quantity);
    } catch (InterruptedException e) {
        throw new RuntimeException(e);
    } finally {
        lock.unlock();
    }
}
```

> 실무에서는 보통
>
> 재시도가 필요하지 않은 락은 Lettuce를 활용하고,
>
> 재시도가 필요한 경우 Redisson을 활용

[commit](https://github.com/jihunparkme/Study-project-spring-java/commit/c9ff889d5bbda6c08b1970098329bcd34d0275e0)

## Finish

**Java Synchronized**
- 한 개의 스레드만 접근 가능하도록 제한
- @Transactional 적용 불가
- 2대 이상의 서버로 운영될 경우 레이스 컨디션은 또 다시 발생

**DataBase Lock**
- Pessimistic Lock
  - 로우, 테이블 단위로 락킹
  - 락 해제 전까지 다른 스레드는 데이터를 가져갈 수 없음(데드락 주의)
  - 충돌이 빈번하게 일어날 경우 추천
- Optimistic Lock
  - 버전을 이용
  - 업데이트 실패 시 재시도
  - 충돌이 빈번하게 일어나지 않을 경우 추천
  - 업데이터 실패 시 재시도 로직 개발 필요
- Named Lock
  - Pessimistic Lock 과 유사하지만 이름들 가진 데이터에 락킹
  - 트랜잭션 종료 시 락 해제, 세션은 직접 관리 필요
  - 주로 분산락 구현 시 사용

**Redis**
- Lettuce
  - Spin Lock 방식(레디스에 부하 가능성 존재)
  - 재시도 로직 개발이 필요
  - 재시도가 필요하지 않은 경우 권장
- Redisson
  - pub-sub 기반
  - 별도의 라이브러리가 필요하고 구현이 복잡
  - 재시도가 필요한 경우 권장

**MySql vs. Redis**

|Mysql|Redis|
|---|---|
|Mysql을 사용중이라면 별도 비용없이 사용 가능|활용중인 Redis가 없다면 별도의 구축 비용과 인프라 관리 비용이 발생|
|Redis 보다 성능이 좋지 않음(어느정도의 트래픽까지는 문제없이 활용 가능)|Mysql 보다 성능이 좋음|

## Reference

[Repository](https://github.com/jihunparkme/Study-project-spring-java/tree/main/stock-concurrency)