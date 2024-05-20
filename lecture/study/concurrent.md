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

우리는 아래 테스트에서 0의 결과를 예상하지만, 동시에 들어오는 요청들이 갱신 전 값을 읽고 수정하면서 실제 갱신이 누락되는 현상이 발생하게 됩니다.

```java
@Test
void decrease_inventory_concurrent_requests() throws InterruptedException {
    int threadCount = 100;
    final ExecutorService executorService = Executors.newFixedThreadPool(32);
    final CountDownLatch latch = new CountDownLatch(threadCount);

    for (int i = 0; i < threadCount; i++) {
        executorService.submit(() -> {
            try {
                stockService.decreases(1L, 1L);
            } finally {
                latch.countDown();
            }
        });

        latch.await();
    }

    final Stock stock = stockRepository.findById(1L).orElseThrow();

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

여기서 문제가 있습니다.

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

 우리가 만든 클래스를 필드로 가지고 호출하게 되는데 트랜잭션 종료 전에 다른 스레드가 갱신된 전 값을 읽게 되면 결국 이전과 동일한 문제가 발생하게 됩니다.

 여기서 단순하게 Transactional Annotation 을 제거하여 문제를 해결할 수는 있습니다.

## Reference

[재고시스템으로 알아보는 동시성이슈 해결방법](https://www.inflearn.com/course/%EB%8F%99%EC%8B%9C%EC%84%B1%EC%9D%B4%EC%8A%88-%EC%9E%AC%EA%B3%A0%EC%8B%9C%EC%8A%A4%ED%85%9C/dashboard)

[Repository](https://github.com/jihunparkme/Study-project-spring-java/tree/main/stock-concurrency)