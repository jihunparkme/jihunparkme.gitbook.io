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

### 오브젝트
- 프로그램을 실행하면 만들어져서 움직이는 것(=객체)
- 오브젝트는 일을 하며 우리가 원하는 기능을 수행
- 자바의 오브젝트는 클래스의 인스턴스 또는 배열

**클래스**
- 우리가 작성하는 코드
- 오브젝트를 만들어내기 위해 필요한 상세한 정보(설계도, 프로토타입, 모델 ..)

**인스턴스**
- 추상적인 것(ex. 클래스)에 대한 실체

.

### 의존관계
- 반드시 **두 개 이상의 대상이 존재**
  - 하나가 다른 하나를 **사용**, **호출**, **생성**, **인스턴스화**, **전송** 등을 할 때 의존관계에 있다고 이야기
- 클래스 사이에 의존관계가 있을 때 의존 대상이 변경되면 이를 사용하는 클래스의 코드도 영향을 받는다.
- 오브젝트 사이에 의존관계는 실행되는 런타임에 의존관계가 만들어지고 실행 기능에 영향을 받지만, 클래스 레벨의 의존관계와는 다를 수 있다.

.

### 관심사의 분리(Separation of Concerns)
- 관심사는 **동일한 이유로 변경되는 코드의 집합**
- API로 정보를 가져와서 JSON을 오브젝트에 매핑하는 관심과 응답 객체를 준비하는 로직은 관심이 다르다.
- 변경의 이유와 시점을 살펴보고 이를 분리하자.

[commit](https://github.com/jihunparkme/inflearn-toby-spring-6/commit/2c818d5bfb4a966432fa8e643e3235355d1dbeb9)

.

### 상속을 통한 확장
- 객체의 변경 없이 정보를 가져오는 방법을 확장하게 만들려면 **상속**을 이용할 수 있다.
- 어떻게 객체를 준비할 것이가와 어떻게 정보를 가져올 것인가라는 **관심사가 클래스 레벨에서 분리**된다.
- 정보를 담은 오브젝트인 ExRate를 생성하는 **책임을 서브 클래스에게 위임**하는 방식이다.
  - 객체지향 디자인 패턴에 나오는 **팩토리 메소드 패턴을 이용**한다.
- 하지만, 자바는 **다중 상속을 허용하지 않**으므로 다른 관심사를 분리할 경우 확장을 이용하기 어렵다.
  - 또한 상위 클래스의 변경에 따라 하위 클래스를 모두 변경해야 하므로 **상속을 통한 확장은 관심사를 분리하기에 한계**가 있다.

[commit](https://github.com/jihunparkme/inflearn-toby-spring-6/commit/184f5ed5d1eb4c84b2e1e08273fe18223398d7f4)

.

### 클래스의 분리
- **관심사에 따라 클래스를 분리**해서 각각 독립적으로 구성할 수 있다.
  - 결과적으로 클래스 레벨에 사용 의존관계가 만들어지기 때문에 **강한 코드 수준의 결합**이 생긴다. 
  - 실제로 사용할 클래스가 변경되면 이를 이용하는 쪽의 코드도 따라서 변경이 되어야 한다.
- 상속을 통한 확장처럼 **유연한 변경도 불가능**해진다.
- 상속한 것이 아니기 때문에 사용하는 클래스의 메소드 이름과 구조도 제각각일 수 있다. 
  - 그래서 클래스가 변경되면 많은 코드가 따라서 변경되어야 한다.
- 클래스가 다르다는 것을 제외하면 관심사의 분리가 잘 된 방법이 아니다.

[commit](https://github.com/jihunparkme/inflearn-toby-spring-6/commit/639866635f4865340f6ddfb0f869849fa40aafb1)