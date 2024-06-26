# Toby Spring Boot

토비님의 [토비의 스프링 6 - 이해와 원리](https://www.inflearn.com/course/%ED%86%A0%EB%B9%84%EC%9D%98-%EC%8A%A4%ED%94%84%EB%A7%816-%EC%9D%B4%ED%95%B4%EC%99%80-%EC%9B%90%EB%A6%AC/dashboard) 강의를 요약한 내용입니다.

[Github](https://github.com/tobyspringboot/hellospring)

[MY Project]()

## Intro

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