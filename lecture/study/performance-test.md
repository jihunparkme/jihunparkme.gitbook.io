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

### 네트워크

네트워크는 성능 테스트에서 중요한 요소
- 네트워크는 물리적인 거리의 제약을 많이 받음(라우터에서의 연산도 필요)

API 내에서 외부 서버(데이터 베이스, 외부 API) 호출 시 네트워크 통신이 발생
- 데이터베이스 같은 경우 캐시를 활용하거나 DB 응답이 빨라지도록 개선 가능

### 데이터베이스

| 자원 | 사용 |  |
| --- | --- | --- | 
| CPU  | 사용자가 원하는 데이터인지 확인 |
| RAM  | 데이터를 CPU가 연산할 수 있게 준비 혹은 캐시 |
| DISK | 데이터들이 저장 | 

데이터베이스 지연시간이 길어지는 상황
- 짧은 시간동안 많은 요청이 들어오는 경우 (처리량 증가)
- 많은 데이터 중 필요한 데이터를 찾을 경우
  - 인덱스로 개선 가능
- 한번에 많은 데이터를 응답으로 줘야 할 경우
- 락이 너무 자주 걸리는 경우

### 스레드 풀과 커넥션 풀

기본 설정값이 있다보니 성능 테스트에서 튜닝이 필요한 자원
- 성능 테스트에서 가장 먼저 문제가 나타나는 요소

**Thread Pool**
- 서버가 요청을 받아서 처리하는 자원
- 톰캣의 경우 기본적으로 200개의 스레드가 생성
- 스레드들은 미리 생성되어 요청을 처리한 후 반납
- 지연 시간이 길어질 경우
  - 모든 스레드는 사용중인 상태가 되고, 일부 요청은 큐에 저장되지만 큐도 모두 찰 경우 이후 요청은 버려지게 된다.
  - 스레드 양을 무작정 늘리게 되면 적은 양의 스레드보다 처리량이 낮아질 수 있다.
  - 이 경우 로드 밸런싱을 통해 여러 서버가 트래픽을 받도록 하거나 비동기로 요청을 처리하는 개선이 필요

**Connectino Pool**
- 서버와 데이터베이스가 데이터를 주고 받기 위한 자원
- 스레드 풀과 동일한 문제 존재

> 스레드 풀과 커넥션 풀은 여러 번의 성능 테스트를 통해 적절한 숫자를 찾아 설정하자.

### 성능 테스트의 방향성

(1) 한 건씩 요청을 보내서 **지연시간**이 어느정도 나오는지 테스트

(2) 처리량을 높이면서 현재 인프라 구성에서 **지연시간이 치솟는 지점** 찾기

(3) 어떤 부분이 병목이 되는건지 가설을 세워보고 **서버 자원 모니터링, 로그 등을 통해 병목 지점 탐색**

(4) 병목을 해결할 수 있는 방법 적용

## Artillery

[Get Started > Get Artillery](https://www.artillery.io/docs/get-started/get-artillery)

```bash
# install node
$ brew install node

# check node version
$ node -v

# install artillery
$ npm install -g artillery@1.7.6

# check artillery version
$ artillery --version
```

### First Artillery Test

[Create an Artillery test script](https://www.artillery.io/docs/get-started/first-test#create-an-artillery-test-script)

**기본 테스트 스크립트 작성**

```yml
config:
  target: 'http://localhost:8080'
  phases:
    # 60초동안 매 초마다 5개 요청(Throughput: 5)
    - duration: 60
      arrivalRate: 5
scenarios:
  - flow:
    - get:
        url: "/hello"
```

**성능 테스트 실행**

```bash
$ artillery run --output report.json test-config.yaml 
```

- `test-config.yaml` 파일을 사용해서 성능 테스트를 진행하고 결과로 `report.json` 파일 생성

**성능 테스트 로그**

```bash
...

Report @ 00:00:00(+0000) 2024-00-00
Elapsed time: 1 minute, 0 seconds
  Scenarios launched:  50
  Scenarios completed: 50
  Requests completed:  50
  Mean response/sec: 5
  Response time (msec):
    min: 0
    max: 4
    median: 2
    p95: 3
    p99: 4
  Codes:
    200: 50

...

All virtual users finished
Summary report @ 00:00:00(+0000) 2024-00-00
  Scenarios launched:  300
  Scenarios completed: 300
  Requests completed:  300
  Mean response/sec: 4.98
  Response time (msec):
    min: 0
    max: 100
    median: 2
    p95: 3
    p99: 5
  Scenario counts:
    0: 300 (100%)
  Codes:
    200: 300

```

- `min`, `max`, `median`은 latency를 의미
- `p95`, `p99`는 각각 95%, 99% 사용자가 어느 정도르 latency를 느끼고 있는지
- `200 Codes`는 1초마다 5개의 요청을 하므로 10초당 50개
- 최공 결과에는 1초 * 5개 * 60초 = 300개의 결과

**결과 파일(report.json) html 파일로 변환**

```bash
$ artillery report report.json --output report.html
```

### Scenario Test

[Reference > Test Scripts](https://www.artillery.io/docs/reference/test-script)

**시나리오 테스트 스크립트 작성**

```yml
config:
  target: 'http://localhost:8080'
  phases: # 성능 테스트의 요청 설정
    # 30초 동안 초당 3개의 요청
    - duration: 30
      arrivalRate: 3 # 시나리오 개수만큼 요청
      name: Warm up # 페이지 이름
    - duration: 30
      arrivalRate: 3
      rampTo: 30 # 점점 요청 횟수를 늘려서 최종 초당 30개의 요청
      name: Ramp up load
    - duration: 60
      arrivalRate: 30
      name: Sustained load
    - duration: 30
      arrivalRate: 30
      rampTo: 10 # 점점 요청 횟수를 줄여서 최종 초당 10개의 요청
      name: End of load
scenarios: # 한 명의 사용자가 어떤 순서로 요청을 할지 설정
  - name: "login and use some functions" # 1번 시나리오 이름
    flow:
      - post:
          url: "/login"
      - get:
          url: "/some-function-one"
      - get:
          url: "/some-function-two"
  - name: "just login" # 2번 시나리오 이름
    flow:
      - post:
          url: "/login"
```

**성능 테스트 로그**

```bash
# 성능 테스트 실행
$ artillery run --output secario-report.json scenario-test-config.yaml

...

All virtual users finished
Summary report @ 00:00:00(+0000) 2024-00-00
  Scenarios launched:  3025
  Scenarios completed: 3025
  Requests completed:  6115
  Mean response/sec: 40.47
  Response time (msec):
    min: 15
    max: 64
    median: 24
    p95: 36
    p99: 39
  Scenario counts:
    login and use some functions: 1545 (51.074%)
    just login: 1480 (48.926%)
  Codes:
    200: 6115


# 결과를 html 파일로 변환
$ artillery report secario-report.json --output secario-report.html
```

### Parameter Test

[payload - loading data from CSV files](https://www.artillery.io/docs/reference/test-script#payload---loading-data-from-csv-files)

**파라미터 테스트 스크립트 작성**

```yml
config:
  target: 'http://localhost:8080'
  phases:
    - duration: 30
      arrivalRate: 3
      name: Warm up
  payload:
    path: "id-password.csv" # 파일 경로
    fields: # 필드 구분
      - "id"
      - "password"
scenarios:
  - name: "just login"
    flow:
      - post: # json 파라미터
          url: "/login-with-id-password"
          json:
            id: "{{ id }}"
            password: "{{ password }}"
  - name: "find user"
    flow:
      - get: # url 파라미터
          url: "/search?query={{ id }}"
```

**성능 테스트 로그**

```bash
# 성능 테스트 실행
$ artillery run --output parameter-report.json parameter-test-config.yaml

...

All virtual users finished
Summary report @ 00:00:00(+0900) 2024-00-00
  Scenarios launched:  90
  Scenarios completed: 90
  Requests completed:  90
  Mean response/sec: 2.98
  Response time (msec):
    min: 1
    max: 56
    median: 3
    p95: 4
    p99: 36
  Scenario counts:
    just login: 46 (51.111%)
    find user: 44 (48.889%)
  Codes:
    200: 90


# 결과를 html 파일로 변환
$ artillery report parameter-report.json --output parameter-report.html
```

### High Load Test

**파라미터 테스트 스크립트 작성**

```yml
config:
  target: 'http://localhost:8080'
  phases:
    - duration: 30
      arrivalRate: 20
      name: Warm up
    - duration: 10
      arrivalRate: 20
      rampTo: 200
      name: Ramp up load
    - duration: 10
      arrivalRate: 200
      name: Sustained load
    - duration: 30
      arrivalRate: 200
      rampTo: 20
      name: End of load
scenarios:
  - name: "high load cpu"
    flow:
      - get:
          url: "/high-load-cpu"
 - name: "high load memory"
   flow:
     - get:
         url: "/high-load-memory"
```

**성능 테스트 로그**

```bash
# 성능 테스트 실행
$ artillery run --output high-load-report.json high-load-test-config.yaml

...

All virtual users finished
Summary report @ 00:00:00(+0900) 2024-00-00
  Scenarios launched:  7214
  Scenarios completed: 6767
  Requests completed:  6767
  Mean response/sec: 87.59
  Response time (msec):
    min: 2
    max: 7781
    median: 1823
    p95: 4456
    p99: 5369
  Scenario counts:
    high load cpu: 3669 (50.859%)
    high load memory: 3545 (49.141%)
  Codes:
    200: 6767
  Errors:
    ECONNRESET: 14
    ETIMEDOUT: 433



# 결과를 html 파일로 변환
$ artillery report high-load-report.json --output high-load-report.html
```

## 참고

**다른 서능 테스트 툴**
- [Apache JMeter](https://jmeter.apache.org/)
- [Locust](https://locust.io/)
- [Artillery](https://www.artillery.io/)
- [Gragana K6](https://grafana.com/docs/k6/latest/testing-guides/running-large-tests/)
- [Ngrinder](https://naver.github.io/ngrinder/)
- ..