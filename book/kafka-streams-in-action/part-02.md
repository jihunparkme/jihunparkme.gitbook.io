# Part 2. 카프카 스트림즈 개발

# 카프카 스트림즈 개발

👉🏻 **스트림 프로세서 API**
- 카프카 스트림즈 애플리케이션을 신속하게 만들 수 있게 해주는 `고수준 API`
- 핵심은 키/값 쌍 레코드 스트림을 나타내는 `KStream 객체`
- 대부분의 메소드는 KStream 객체 레퍼런스를 반환해 플루언트 인터페이스(fluent interface) 스타일의 프로그래밍
  - 메소드 호출의 반환값이 원래 메소드를 호출한 인스턴스와 같음
- [Fluent Interface](https://martinfowler.com/bliki/FluentInterface.html)