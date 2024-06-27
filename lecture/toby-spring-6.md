# Toby Spring Boot

토비님의 [토비의 스프링 6 - 이해와 원리](https://www.inflearn.com/course/%ED%86%A0%EB%B9%84%EC%9D%98-%EC%8A%A4%ED%94%84%EB%A7%816-%EC%9D%B4%ED%95%B4%EC%99%80-%EC%9B%90%EB%A6%AC/dashboard) 강의를 요약한 내용입니다.

[Github](https://github.com/tobyspringboot/hellospring)

[MY Project](https://github.com/jihunparkme/inflearn-toby-spring-6)

# Intro

JDK
- [Amazon corretto](https://aws.amazon.com/corretto/)
- [Eclipse Temurin](https://adoptium.net/en-GB/temurin/releases/)
- [SDK Man](https://sdkman.io/)
    ```bash
    $ sdk # sdk man 터미널 열기
    > sdk list java # java 관련 모든 설치 가능한 버전 확인
    > sdk list java | grep installed # 설치된 버전만 확인
    > sdk install java [jdk name] # 자바 설치
    > sdk use java [jdk name] # 폴더에서 해당 버전 사용
    > sdk env init # 프로젝트 폴더의 사용 JDK 저장
    > sdk env # 프로젝트 폴더의 사용 JDK 복구
    ```

API
- [httpie](https://httpie.io/)

# 오브젝트와 의존관계

## 오브젝트

**오브젝트**
- 프로그램을 실행하면 만들어져서 움직이는 것(=객체)
- 오브젝트는 일을 하며 우리가 원하는 기능을 수행
- 자바의 오브젝트는 클래스의 인스턴스 또는 배열

**클래스**
- 우리가 작성하는 코드
- 오브젝트를 만들어내기 위해 필요한 상세한 정보(설계도, 프로토타입, 모델 ..)

**인스턴스**
- 추상적인 것(ex. 클래스)에 대한 실체

## 의존관계

**의존관계**
- 반드시 **두 개 이상의 대상이 존재**
  - 하나가 다른 하나를 **사용**, **호출**, **생성**, **인스턴스화**, **전송** 등을 할 때 의존관계에 있다고 이야기
- 클래스 사이에 의존관계가 있을 때 의존 대상이 변경되면 이를 사용하는 클래스의 코드도 영향을 받는다.
- 오브젝트 사이에 의존관계는 실행되는 런타임에 의존관계가 만들어지고 실행 기능에 영향을 받지만, 클래스 레벨의 의존관계와는 다를 수 있다.