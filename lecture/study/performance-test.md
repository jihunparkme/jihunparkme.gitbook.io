# Performance Test

[백엔드 애플리케이션 성능 테스트하기](https://www.inflearn.com/course/%EB%B0%B1%EC%97%94%EB%93%9C-%EC%95%A0%ED%94%8C%EB%A6%AC%EC%BC%80%EC%9D%B4%EC%85%98-%EC%84%B1%EB%8A%A5-%ED%85%8C%EC%8A%A4%ED%8A%B8/dashboard) 강의를 요약한 내용입니다.

## Intro

성능 테스트 툴을 활용하여 성능 테스트 결과를 반복하면서 **예상되는 사용자 수와 그에 따른 요청 수를 계산**할 수 있습니다.
- 이러한 계산을 통해 **얼마나 많은 수의 서버가 필요한지 테스트**

성능 테스트가 필요한 상황
- 비효율적으로 동작하는 **애플리케이션 로직 개선** (GC ..)
- 데이터베이스 같은 저장소에 대한 **성능 개선** (Index, DeadLock ..)
- 시스템 **설계 개선** (비동기적인 구조, Circuit Breaker ..)

참고.
- 트래픽: 네트워크를 통해 흐르는 데이터
- 트래픽이 많고 적음의 기준은 상대적
- 성능 테스트 = 부하 테스트 = 스트레스 테스트

## 배경지식

### 지연시간과 처리량

**지연시간**(`Latency`)
- 클라이언트가 요청을 보낸 후 응답을 받기까지 걸린 시간
- 주로 ms, s 단위

```text
어떤 요청이 처리되어서 사용자에게 응답이 되기까지 1초가 걸렸다면 이 요청의 지연시간은 1초
```

**처리량**(`Throughput`)
- 단위 시간동안 몇건의 요청을 처리할 수 있는지 (데이터의 전송량)
- 주로 TPS(Transaction Per Second) 단위

```text
어떤 API가 1초 동안 1,000개의 API 요청이 들어올 때 1,000의 요청을 문제없이 처리해낼 수 있다면
이 API의 처리량은 1,000 TPS
```

참고. **대역폭**(`Bandwidth`)
- 네트워크에서 유의미하게 사용될 수 있는 개념
  - 백엔드 애플리케이션에서는 여러 요소들의 간섭으로 정확한 대역폭 측정이 어려움
- 네트워크가 단위시간 동안 전송할 수 있는 최대한의 처리량
- 해당 네트워크를 최대한으로 활용했을 때 얼마나 빠른 속도로 많은 데이터를 보낼 수 있는가
- 처리량을 계속 올리면서 테스트하다 보면 대역폭에 가까워질 수 있음

.

**성능 테스트 목표**
- 성능 테스트 목표는 예상되는 **서비스 이용자의 숫자**와 사용자들이 **수용할 수 있는 지연시간**
- 성능을 측정 시 `처리량`과 `지연시간`을 모두 측정
- 사용자들이 불편함을 느끼지 않도록 처리하려면 어느 정도의 지연시간 미만으로 처리가 되어야 하는지의 식
- 성능 테스트에는 많은 변수가 있으므로 95~99% 수준의 목표를 적용

```text
초당 3000개의 요청(처리량)이 들어올 때 99%의 요청이
100ms 미만으로 처리(지연시간)되어야 함
```

> 요청을 초당 1개만 처리하면 지연시간이 낮게 나오겠지만,
>
> 처리량을 늘리면 지연시간도 함께 늘어날 가능성이 높다.

### 운영체제와 서버 자원

**운영체제**(OS, Operating System)
- 서버의 물리적인 자원을 관리하며 프로세스라는 단위로 애플리케이션을 실행

| 자원    | 내용    | 사용 | |
| ------- | ------- | --- | --- |
| CPU | 프로세스 실행 | 계산 작업, 이미지, 영상 인코딩 | 암호화, 해시화 |
| RAM | 프로세스 = 메모리(Stack, Heap..) + 코드 | CPU와 함께 많이 사용 | 인스턴스 대량 생성, 캐싱, 컬렉션 객체 등 |
| DISK | Program | 파일 입출력, 로그 대량 발생 | DB 데이터 대량 입출력 |

`CPU`
- **프로세스**가 가진 **코드**에 따라 실행시키면서 **메모리**에 있는 내용을 읽거나 쓰기

`RAM`
- **프로세스**: 현재 실행 중인 프로그램 (**메모리** + **코드**)
  - **메모리**는 **코드**에 따라 수동적으로 사용되는 형태
- **페이징**: 메모리 내용 중 일부를 디스크의 가상 메모리에 저장했다가 불러오는 과정

`DISK`
- **프로그램**: 디스크에 저장되어 있는 실행될 수 있는 파일이나 소프트웨어 집합(exe, jar ..)
- **프로그램**을 실행시키면 **프로그램**은 메모리 위로 올라와서 실행(=**프로세스**)

지연시간, 처리량과의 관계
- (1) 클라이언트 요청 증가
- (2) 처리량 증가
- (3) 서버 자원 사용량 증가
- (4) 프로세스, 스레드 간 자원 사용을 위한 대기 시간 증가
  - 경쟁적인 상태 -> 처리량이 기대 이상으로 감소
- (5) 지연 시간 증가
- (6) 미처리 요청 증가
- (7) 요청 중 일부 실패

> 프로세스 실행을 위해 CPU, Memoery, Disk가 서로 상호작용
> 
> 성능 테스트 시 서버가 가지고 있는 자원 중 사용률이 높아지는 자원이 있다면 해당 자원의 사용률을 낮추는 방법을 고민해 보자.
>
> 효과적으로 성능 테스트를 하고 성능을 개선하려면 서버 자원에 대한 모니터링도 중요.