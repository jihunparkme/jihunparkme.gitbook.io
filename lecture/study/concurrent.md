# Concurrency issues

# Race Condition

[Race condition](https://en.wikipedia.org/wiki/Race_condition)

경쟁상태는 두 개 이상의 스레드가 공유 데이터에 액세스할 수 있고, 동시에 변경을 하려고 할 때 발생하는 문제입니다.

```java
@Transactional
public void decreases(Long id, Long quantity) {
    final Stock stock = stockRepository.findById(id).orElseThrow();
    stock.decrease(quantity);

    stockRepository.saveAndFlush(stock);
}
```

우리는 아래 테스트에서 0의 결과를 예상하지만, `동시에 들어오는 요청들이 갱신 전 값을 읽고 수정`하면서 실제 갱신이 누락되는 현상이 발생하게 됩니다.

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

대부분의 운영 서비스는 2대 이상의 서버로 운영되기 때문에 Java Synchronized 는 거의 사용되지 않는 방법입니다.

## DataBase Lock

Mysql

### Pessimistic Lock

- 비관적 락
- 실제로 **데이터에 Lock** 을 걸어서 정합성을 맞추는 방법
- Row or Table 단위로 Locking
- Exclusive Lock 을 걸게 되며, 다른 트랜잭션에서는 **Lock 해제 전에 데이터를 가져갈 수 없음**
- **데드락**이 걸릴 수 있기 때문에 주의 필요


```java
public interface StockRepository extends JpaRepository<Stock, Long> {

    @Lock(LockModeType.PESSIMISTIC_WRITE)
    @Query("select s from Stock s where s.id = :id")
    Stock findByIdWithPessimisticLock(@Param("id") Long id);
}

..

@Service
@RequiredArgsConstructor
public class PessimisticLockStockService {

    private final StockRepository stockRepository;

    @Transactional
    public void decrease(Long id, Long quantity) {
        final Stock stock = stockRepository.findByIdWithPessimisticLock(id);

        stock.decrease(quantity);

        stockRepository.save(stock);
    }
}
```

장점.
- 충돌이 빈번하게 일어난다면 `Optimistic Lock` 보 성능이 좋을 수 있다.
- 락을 통해 업데이터를 제어하므로 데이터 정합성이 보장된다.

단점.
- 별도의 락을 잡기 때문에 성능 감소가 있을 수 있다.

### Optimistic Lock

- 낙관적 락
- 실제로 Lock 을 이용하지 않고 **버전을 이용**함으로써 정합성을 맞추는 방법
- 먼저 데이터를 읽은 후 update 수행 시, 현재 내가 읽은 버전이 맞는지 확인하며 업데이트 수행
- 내가 읽은 버전에서 수정사항이 생겼을 경우 애플리케이션에서 다시 읽은 후에 작업을 수행

### Named Lock

- 이름을 가진 [Metadata Locking](https://dev.mysql.com/doc/refman/8.0/en/metadata-locking.html)
- 이름을 가진 Lock 획득 후, 해제할 때까지 다른 세션은 이 Lock 을 획득할 수 없도록 함
- Transaction 종료 시 Lock 이 자동으로 해제되지 않는 점 주의
- 별도의 명령어로 해제를 수행해 주거나 선점 시간이 끝나야 해제
- 비관적 락과 유사하지만 로우나 테이블이 아닌 메타데이터의 락킹을 하는 방식


Reference.

- [Locking Functions](https://dev.mysql.com/doc/refman/8.0/en/locking-functions.html)

## Finish

**Java Synchronized**
- @Transactional 적용 불가
- 2대 이상의 서버로 운영될 경우 적용 불가

## Reference

[재고시스템으로 알아보는 동시성이슈 해결방법](https://www.inflearn.com/course/%EB%8F%99%EC%8B%9C%EC%84%B1%EC%9D%B4%EC%8A%88-%EC%9E%AC%EA%B3%A0%EC%8B%9C%EC%8A%A4%ED%85%9C/dashboard)

[Repository](https://github.com/jihunparkme/Study-project-spring-java/tree/main/stock-concurrency)