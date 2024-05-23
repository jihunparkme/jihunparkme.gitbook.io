# First Come First Served Event

[실습으로 배우는 선착순 이벤트 시스템](https://www.inflearn.com/course/%EC%84%A0%EC%B0%A9%EC%88%9C-%EC%9D%B4%EB%B2%A4%ED%8A%B8-%EC%8B%9C%EC%8A%A4%ED%85%9C-%EC%8B%A4%EC%8A%B5/dashboard) 강의를 듣고 요약한 내용입니다.

# Intro

요구사항
- 선착순 100명에게 할인쿠폰을 제공하는 이벤트
- 선착순 100명에게만 지급되어야한다.
- 101개 이상이 지급되면 안된다.
- 순간적으로 몰리는 트래픽을 버틸 수 있어야한다.

선착순 이벤트 진행 시 발생할 수 있는 문제점
- 쿠폰이 개수보다 더 많이 발급되었을 경우
- 이벤트 페이지 접속이 안 될 경우
- 이벤트랑 상관없는 페이지도 느려질 경우

문제 해결
- 트래픽이 몰렸을 때 대처할 방법
- Redis 를 활용하여 쿠폰 발급 개수 보장하는 방법
- Kafka 를 활용하여 다른 페이지들에 대한 영향도를 줄이는 방법

```java
/** Service **/
public void apply(Long userId) {
    final long count = couponRepository.count();

    if (count > 100) {
        return;
    }

    couponRepository.save(new Coupon(userId));
}

...

/** Test **/
@Test
void apply_only_once() {
    applyService.apply(1L);

    final long count = couponRepository.count();

    assertThat(count).isEqualTo(1);
}
```