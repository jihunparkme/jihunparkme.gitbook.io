# Domain Driven Design

## DDD & Micro Service

**마이크로서비스 설계에서 도메인 주도 설계 활용**

<figure><img src="../../.gitbook/assets/micro-service/ddd-design.png" alt=""><figcaption></figcaption></figure>

`전략적 설계`를 이용한 마이크로서비스 `식별`
- 마이크로서비스을 어떤 기준으로 분리할 것인가?
  - Bounded	Context, Ubiquitous Language
- 서비스 간은 어떻게 연계할 것인가?
  - Context	Mapping
- 어떤 이벤트에 의해 마이크로서비스는 서로 반응하는가?
  - Domain Event

> 참고 도서. 
> 
> [도메인 주도 설계 핵심](https://www.yes24.com/Product/Goods/48577718)

`전술적 설계`와 미이크로서비스 `내부 설계`
- 마이크로서비스 내부는 어떻게 설계 할 것인가?
  - Domain Model

> 참고 도서. 
> 
> [도메인 주도 개발 시작하기](https://www.yes24.com/Product/Goods/108431347)
>
> [도메인 주도 설계 첫걸음](https://www.yes24.com/Product/Goods/109708596)
>
> [도메인 주도 설계 철저 입문](https://www.yes24.com/Product/Goods/93384475)

## 도메인 주도 설계

구축해야 하는 소프트웨어와 시스템을 위해서는 문제를 이해
- `Problem`: 그 조직의 비즈니스 전략과 소프트웨어를 통해서 얻고자 하는 가치
- `Business Domain`: 기업의 주요 활동 영역, 회사가 제공하는 서비스
- `Sub domain`: 비즈니스 활동의 세분화된 영역, 제공하는 서비스 단위
  - ex. 회원, 고객, 상품, 주문, 배송..

> Problem -> Business -> Domain -> Sub Domain
>
> 시스템에서 사용하는 모든 개념들이 비즈니스 도메인을 표현해야 한다.

.

**Sub domain**

핵심(Core)
- 회사만의 차별성. 복잡성이 높지만 경쟁력 제공
- 핵심 인재 할당과 진보된 엔지니어링 기술 적용
- ex) 우버의 손님 매칭 서비스, 구글의 검색 순위 알고리즘 ..

일반(Generic)
- 모든 회사가 같은 방식으로 수행하는 비지니스 활동
- 복잡하고 구현하기 어려우나 주로 오픈 소스를 사용하고, 경쟁력을 제공하지는 않음
- ex) 인증, 권한 부여 ..

지원(Supporting)	
- 회사 비지니스 지원 활동
- 간단한 기능으로 경쟁우위 제공하지 않음
- ex) CRUD, ETL