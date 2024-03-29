# Micro Service

**Monolith System**
- 애플리케이션이 `한 덩어리`로 구성
- `단일 프로세스` 실행
- 한꺼번에 수정, 배포 필요 -> 서비스 중단
- 하나가 실패하면 모두 실패
- 모노리스를 클라우드 인프라에서 활용 시 스케일 아웃 대상은 `모노리스 전체`
  - 확장성, 탄력성 보장이 가능하나 비용이 비효율적

```bash
├── Client
│   ├── Server1
│   ├── Server2
│   └── Server3
│      └── DataBase
└── 
```

**Micro Service**
- 애플리케이션이 `여러 개의 서비스 조각`으로 구성
- 서비스는 각기 `독립적인 기능`을 제공
- 서비스가 사용하는 저장소는 `다른 서비스와 완벽히 격리`
  - 독립적으로 수정, 별도 배포, 확장 가능
- 하나의 서비스 실패는 전체 실패가 아닌 `부분적인 실패`를 의미

```bash
├── Client
│   ├── Servive A
│   │  └── DataBase A
│   │
│   ├── Servive B
│   │  └── DataBase B
│   │
│   ├── Servive C-1
│   ├── Servive C-2
│   └── Servive C-3
│      └── DataBase C
└── 
```

...

**마이크로서비스의 정의**

- `Public Interface`, `데이터 캡슐화`
- 확장 시, 특정 기능별 `독립적으로 확장` 가능
- 특정 서비스의 변경 시 `해당 서비스만` 빌드, 배포
- `독립적`으로 서로 다른 언어로 개발 가능

```bash
├── Client
│   ├── Java Service
│   │  └── Oracle
│   │
│   ├── Node.JS Service
│   │  └── MariaDB
│   │
│   ├── C# Service
│   │  └── MySQL
│   │
│ 
```

- 여러 개의 작은 `서비스 집합으`로 개발하는 접근 방법
- 각 서비스는 `개별 프로세스`에서 실행
- HTTP 자원 `API` 같은 가벼운 수단을 사용하여 `통신`
- 서비스는 `Biz 기능 단위`로 구성
  - 중앙 집중적인 관리 최소화
- 각 서비스는 서로 다른 언어, 데이터, 저장 기술 사용
- 참고. SOA(Service-Oriented Architecture)는 저장소를 공유하는 형태

# 아키텍처 정의

**Outer Architecture 정의**

[Cloud Infra 정의](https://cloud.google.com/learn/paas-vs-iaas-vs-saas)
- `IaaS`([Infrastructure as a Service](https://cloud.google.com/learn/what-is-iaas?hl=ko)): 인터넷을 통해 확장성이 뛰어난 컴퓨팅 리소스를 서비스로서 제공하는 주문형 가용성 서비스
  - ex. AWS EC2, AWS S3
  
  ```text
  원하는 집의 모습과 방 개수를 알려주면 도급업자는 지시에 따라 집을 짓습니다. 
  IaaS도 마찬가지로 하드웨어를 대여하여 애플리케이션을 실행하지만 OS, 런타임, 확장 및 모든 데이터를 관리할 책임은 본인에게 있습니다.
  ```
- `PaaS`([Platform as a Service](https://cloud.google.com/learn/what-is-paas?hl=ko)): 클라우드 컴퓨팅 서비스 모델로서 앱을 개발, 배포, 실행, 관리할 수 있는 유연하고 확장 가능한 클라우드 플랫폼을 제공
  - ex. AWS Lambda

  ```text
  거실 가구를 고르는 일이 번거롭게 느껴진다면 가구가 비치된 집을 대여할 수 있습니다. 
  PaaS를 사용하면 자체 코드를 가져와 배포할 수 있지만 서버 관리 및 수직 확장은 클라우드 제공업체가 맡습니다
  ```
- `CaaS`(Containers as a Service): 컨테이너를 사용하여 애플리케이션을 개발하고 배포하는 데 필요한 모든 하드웨어 및 소프트웨어 리소스를 제공하고 관리
  - ex. AWS ECS, EKS

  ```text
  집을 구매할 경우 뒤따르는 유지관리가 부담스러운 경우 대여를 선택할 수 있습니다. 
  기본 설비는 포함되어 있고 가구를 직접 들여놓고 공간을 꾸밉니다. 
  컨테이너를 사용하면 컨테이너화된 애플리케이션을 도입할 수 있으므로 기본 운영체제에 대해 걱정할 필요 없이 규모와 런타임을 제어할 수 있습니다
  ```

Platform 구성요소 , MSA 패턴
- 라우팅, 로드 밸런싱, 인증/인가, 로깅, 트레이싱, 모니터링

DevOps Infra 정의
- 형상관리, 빌드, 배포(CI/CD)

.

**Inner Architecture 정의**

Front End
- 기술 stack 정의
- 내부 구조 정의

Back End
- 기술 stack, 프레임워크
- 내부 구조 정의

통신
- 서비스 간
- 레거시 연계

.

**설계와 구현**

```text
(1) DDD 설계를 통한 마이크로서비스 식별
• 도메인: 업무 영역, 제공 서비스
• 구성원 역량, 운영 조직 구조
• 서비스 변경/배포 빈도
• 사용량: 트랜잭션 빈도
• 데이터베이스 주제 영역, 오너쉽

(2) 백엔드 설계
• 도메인 모델 설계
• API 설계
• 데이터 모델 설계

(3) 프론트엔드 설계
• UI 흐름
• UI 레이아웃 정의

(4) 테스트
• 단위
• API
• EtoE

(5) 배포
• CI/CD
```

- 시스템 복잡성으로 `로컬 복잡성`과 `글로벌 복잡성`이 있는데, 너무 많지도 적지도 않는 적절한 수준으로 분리가 필요하다.
  - 로컬 복잡성: 각각의 개별 마이크로서비스의 복잡성
  - 글로벌 복잡성: 전체 시스템의 복잡성, 서비스 간의 상호작용과 의존성

# 설계

## REST API

**REST 구성**
- REST(Representational State Transfer): 자원의 정보를 주고 받는 구조
  - HTTP 프로토콜과 JSON 포맷을 사용한 구조
- `자원`(Resource) : URI로 표현 (http://service/apis)
- `행위`(Verb): HTTP Method (POST, GET, PUT, DELETE)
- `표현`(Representations): HTTP POST, http://service/apis
  - {"apis":{"name":"sample"}}

**자원**
- 간결하고 직관적인 기준 URL 유지
- 자원(Resource) 별로 두 가지 형식의 기준 URL 사용
  - GET /dogs (목록 조회)
  - GET /dogs/1 (1번 개체 조회)
  - POST /dogs (개체 생성)
  - PUT /dogs/1 (1번 개체 수정)
  - DELETE /dogs/1 (1번 개체 삭제)
- 동사 보다는 복수 명사 사용
  - 권장 방법: GET /dogs, POST /dogs/{puppy}/owner/{terry}
  - 비권장하는 방법: /getDogs, /setDogs

**행위**
- POST: 리소스 생성
- GET: 리소스 조회
- PUT: 리소스 수정
- DELETE: 리소스 삭제

**HTTP 상태 코드**
- 200-level: 성공
- 400-level: 잘못된 요청
- 500-level: 서버 에러

...

**REST API 디자인 가이드**
- REST	API	설계는 아키텍처 스타일이지 표준은 아니므로 유연하게 적용
- API 목표는 개발자의 생산성과 성공을 극대화 (실용주의)
- [Manage APIs with unmatched scale, security, and performance](http://www.apigee.com)
- [Web API Design](https://pages.apigee.com/rs/apigee/images/api-design-ebook-2012-03.pdf)

**멱등성(Idempotent)**
- 반복적으로 호출하더라도 상태가 계속 유지되고, 상태의 변화가 없는 것
- 생성 요청의 POST는 멱등성을 따르지 않음
- GET, DELETE는 멱등성을 따르게 설계 가능
-  Vesioning
   - /v1/products
   - /v2/products
- [Richardson Maturity Model](https://martinfowler.com/articles/richardsonMaturityModel.html)

# EventStorming

> UML Tool: https://lucid.co/
>
> EventStorming Tool: https://miro.com/ 

이벤트 스토밍 진행 순서
- 요구사항 분석
- 도메인 이벤트, 핫스팟 도출
- 커맨드 및 외부시스템 도출
- 어그리거트 도출
- 바운디드 컨텍스트 식별
- 컨텍스트 매핑 정의

**업무 흐름 예시**
- 사내 도서 대여 시스템

<figure><img src="../../.gitbook/assets/micro-service/event-storming-1.png" alt=""><figcaption></figcaption></figure>

**이벤트 스토밍 결과 예시**
- 노란섹: 어그리거트
- 파란색: 커맨드
- 주황색: 도메인 이벤트

<figure><img src="../../.gitbook/assets/micro-service/event-storming-2.png" alt=""><figcaption></figcaption></figure>

**컨텍스트 매핑 결과 예시**

<figure><img src="../../.gitbook/assets/micro-service/event-storming-3.png" alt=""><figcaption></figcaption></figure>

**도메인 모델링을 위한 이벤트 스토밍 결과 예시**
- 이벤트 스토밍을 통해 도출된 
  - 어그리거트가 도메인 모델 요소
  - 커맨드가 컨트롤러 역할
  - 도메인 이벤트는 도메인 모델에 포함

<figure><img src="../../.gitbook/assets/micro-service/event-storming-4.png" alt=""><figcaption></figcaption></figure>

# Domain Modeling

**대여 도메인 모델 예시**

<figure><img src="../../.gitbook/assets/micro-service/domain-modeling-1.png" alt=""><figcaption></figcaption></figure>

**회원 도메인 모델 예시**

<figure><img src="../../.gitbook/assets/micro-service/domain-modeling-2.png" alt=""><figcaption></figcaption></figure>

**도서 도메인 모델 예시**

<figure><img src="../../.gitbook/assets/micro-service/domain-modeling-3.png" alt=""><figcaption></figcaption></figure>

**BEST 도서 도메인 모델 예시**

<figure><img src="../../.gitbook/assets/micro-service/domain-modeling-4.png" alt=""><figcaption></figcaption></figure>

# Heuristics

경험에 기반한 규칙

바운디드 컨텍스트
- 크기로 경계를 구분하지 않는다.
- 바운디드 컨텍스트를 식별할 때는 넓은 경계로 시작
- 나중에 도메인 지식이 쌓이게 되면 좀 더 작은 경계로 분리

**비즈니스 로직 구현 패턴과 아키텍처 패턴 결정**

- 간단한 비즈니스 로직(지원, 일반): `Transaction Script`, `Active Record`
  - 단순한 자료 구조: `Transaction Script`
  - 복잡한 자료구조: `Active Record`
- 복잡한 비즈니스 로직(핵심): Domain Model
  - 금전 또는 통화의 트랜잭션 추적, 일관된 감사 로그, 동작에 따른 심층적 분석 요구 -> Event Sourcing

<figure><img src="../../.gitbook/assets/micro-service/heuristics-1.png" alt=""><figcaption></figcaption></figure>

> [Book review: Learning Domain-Driven Design by Vlad Khononov](https://tonisoueid.medium.com/book-review-learning-domain-driven-design-by-vlad-khononov-c7473afa5ba)

**테스트 전략 결정**

<figure><img src="../../.gitbook/assets/micro-service/heuristics-2.png" alt=""><figcaption></figcaption></figure>