# First Come First Served Event

[실습으로 배우는 선착순 이벤트 시스템](https://www.inflearn.com/course/%EC%84%A0%EC%B0%A9%EC%88%9C-%EC%9D%B4%EB%B2%A4%ED%8A%B8-%EC%8B%9C%EC%8A%A4%ED%85%9C-%EC%8B%A4%EC%8A%B5/dashboard) 강의를 듣고 요약한 내용입니다.

# Intro

선착순 이벤트 진행 시 발생할 수 있는 문제점
- 쿠폰이 개수보다 더 많이 발급되었을 경우
- 이벤트 페이지 접속이 안 될 경우
- 이벤트랑 상관없는 페이지도 느려질 경우

문제 해결
- 트래픽이 몰렸을 때 대처할 방법
- Redis 를 활용하여 쿠폰 발급 개수 보장하는 방법
- Kafka 를 활용하여 다른 페이지들에 대한 영향도를 줄이는 방법