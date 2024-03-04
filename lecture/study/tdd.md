# TDD

[실전! 스프링부트 상품-주문 API 개발로 알아보는 TDD](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81%EB%B6%80%ED%8A%B8-%EC%8B%A4%EC%A0%84-%EC%83%81%ED%92%88%EC%A3%BC%EB%AC%B8-tdd/dashboard) 강의를 요약한 내용입니다.

## 상품 등록 기능 구현하기

### POJO

**Java POJO Test**

(1) 필요한 서비스 호출
```java
productService.addProduct(request);
```
- 필요한 동작에 필요한 기능들만으로 구성 가능

(2) 서비스 호출에 필요한 Request 클래스 생성
```java
final AddProductRequest request = new AddProductRequest(name, price, discountPolicy);
```
- Request 클래스는 `record` 클래스로 사용 가능
- `Assert` 를 활용해서 생성자 파라미터 검증

(3) 서비스 클래스 생성
- 도메인 클래스 생성

(4) 어댑터 생성

(5) 레파지토리 생성
- 테스트에서는 데이터를 인메모리(Map)로 관리

> Then -> Given -> When 순서로 적성해 보기
> 
> [ProductService addProduct test](https://github.com/jihunparkme/Study-project-spring-java/commit/cd9618b3796bc23676d8b139b32c96a9d488beb5)

.

**Move Inner class for test to Upper level**

테스트 코드에 작성된 inner class 들을 main class 로 이동
- `F6` 단축키 활용

> [Move Inner class for test to Upper leve](https://github.com/jihunparkme/Study-project-spring-java/commit/77f7c48116c8a6cef995c5cb606eef8a696ac227)

### 스프링부트 테스트로 전환하기

순수 자바로 구현된 서비스를 스프링 빈으로 등록하고 스프링 부트 테스트로 동작하도록 전환
- service, adapter, repository 클래스에 @Component 선언
- 기존 테스트에 @SpringBootTest 선언 및 서비스는 @Autowired 로 주입 

> [스프링부트 테스트로 전환하기](https://github.com/jihunparkme/Study-project-spring-java/commit/6760bf24931426bd05d612de3249df806f8f9bf8)

### API 테스트로 전환하기

API 테스트를 위해 io.rest-assured:rest-assured 사용하기
- rest-assured 는 격리가 잘 안 되는 문제가 존재
  - 데이터가 캐싱이 되어 table sequence 가 꼬여 다른 테스트가 깨질 수 있음(ex. table sequence)
  - 격리 방법은 [3월 우아한테크세미나 / 우아한ATDD](https://www.youtube.com/watch?v=ITVpmjM4mUE) 참고

> [API 테스트로 전환하기](https://github.com/jihunparkme/Study-project-spring-java/commit/23cb68c22a17024770c53ac2f5a9acaf0f5d3f37)
>
> [in-memory to jpa](https://github.com/jihunparkme/Study-project-spring-java/commit/2ae8051aedca8205f47e1910e89c59331eefce2c)
>
> [rest-assured 테스트 격리 코드 추가](https://github.com/jihunparkme/Study-project-spring-java/commit/45c3b6e33810b7495f46addcc8f8a2177119a54d)

### JPA 적용하기

기존 in-memory 형태의 데이터 저장소를 jpa 로 적용

> [JPA 적용하기](https://github.com/jihunparkme/Study-project-spring-java/commit/6161a4ca2f625ea544926294c5834abfd71265db)

## 상품 조회 기능 구현하기

상품 등록 구현으로 이미 기반이 잡혀 있으므로 POJO 부터 테스트를 시작하지 않고 @SpringBootTest 로 시작

> [상품 조회 기능 구현하기](https://github.com/jihunparkme/Study-project-spring-java/commit/31059fc6e7451c8ac337c4eb94b8ceb703a79d0f)
>
> [상품 조회 기능 API 테스트로 전환](https://github.com/jihunparkme/Study-project-spring-java/commit/398a9fb06745cdcb458abf32ba72e7fb6d52b69c)

## 상품 수정 기능 구현하기

### POJO

상품 등록과 동일한 방식으로 진행.

(1) 필요한 서비스 호출
```java
productService.updateProduct(productId, request);
```

(2) 서비스 호출에 필요한 Request record 클래스 생성
- Assert 를 활용해서 생성자 파라미터 검증
```java
final UpdateProductRequest request = new UpdateProductRequest(name, price, discountPolicy);
```

(3) 서비스에 상품 수정 메서드 구현

(4) 상품 수정을 위한 도메인 메서드는 테스트 필수
- Product.update

(4) StubClass 또는 Mockito 를 활용하여 테스트

> [상품 수정 기능 구현 및 Stub 클래스를 활용한 테스트](https://github.com/jihunparkme/Study-project-spring-java/commit/ef37e180a0072560a38fc04ff32eb0998ae86019)
> 
> [Use Mockito instead of StubClass](https://github.com/jihunparkme/Study-project-spring-java/commit/7361bfa68d980c14b289b30973b36f54c3f59218)

### 스프링부트 테스트로 전환하기

> [POJO Test to SpringBoot Test](https://github.com/jihunparkme/Study-project-spring-java/commit/e52d9694562c230a78a26dea4dbe001dd5f57335)

### API 테스트로 전환하기

> [API 테스트로 전환하기](https://github.com/jihunparkme/Study-project-spring-java/commit/8990a65831422b3d8e9650ca514fcfe8281eb7b6)

## 상품 주문 기능 구현하기

**POJO**

> [POJO 상품 주문 기능 구현하기](https://github.com/jihunparkme/Study-project-spring-java/commit/c697d1fb5dc9ba60cc164ef688f29889bd5419d7)
> 
> [Move Inner class for test to Upper leve](https://github.com/jihunparkme/Study-project-spring-java/commit/7db7c50cb4866be71d7036a6a1dd101331537f26)

**스프링부트 테스트로 전환하기**

> [스프링부트 테스트로 전환하기](https://github.com/jihunparkme/Study-project-spring-java/commit/d6f496f566e7bd354e0f60132cb7f08ddb8ad429)

**API 테스트로 전환하기**

> [API 테스트로 전환하기](https://github.com/jihunparkme/Study-project-spring-java/commit/1a5ce7ab653f2c05e3469bca2bfd58f33559a7e4)

**JPA 적용**

> [JPA 적용하기](https://github.com/jihunparkme/Study-project-spring-java/commit/abe8faed1b62c9f7bfa1e64452c7f8a5becde571)

## 주문 결제 기능 구현하기

**POJO**

> [POJO 주문 결제 기능 구현하기](https://github.com/jihunparkme/Study-project-spring-java/commit/dcaf15ad648516d4c4f06ff4011b34dac73eb9ed)

**스프링부트 테스트로 전환하기**

> [스프링부트 테스트로 전환하기](https://github.com/jihunparkme/Study-project-spring-java/commit/3f30f767abb4c9c6d949c51239c71a3ae2e98f82)

**API 테스트로 전환하기**

> [API 테스트로 전환하기](https://github.com/jihunparkme/Study-project-spring-java/commit/001dc06edaafe5750b614a6629038026e5f696cf)

**JPA 적용**

> [JPA 적용하기](https://github.com/jihunparkme/Study-project-spring-java/commit/3423b8c9b6a1041b2aa3e1177f6b3816b77e09d8)

## 상세 패키지 구조 만들기

> [상세 패키지 구조 만들기](https://github.com/jihunparkme/Study-project-spring-java/commit/f15572fb55d486ef6172a7dd6d9ad65ccc5091ac)

## Point.

- `final` keyword
- `record` class
- `Assert` in constructor
- `rest-assured` api test
- `var` type
- HttpStatus
  - 200 OK
  - 201 CREATE