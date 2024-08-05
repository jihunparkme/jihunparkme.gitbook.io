# Multi-Thread And Concurrency

영한님의 [김영한의 실전 자바 - 고급 1편, 멀티스레드와 동시성](https://www.inflearn.com/course/%EA%B9%80%EC%98%81%ED%95%9C%EC%9D%98-%EC%8B%A4%EC%A0%84-%EC%9E%90%EB%B0%94-%EA%B3%A0%EA%B8%89-1/dashboard) 강의를 요약한 내용입니다.

## Process And Thread

### 멀티태스킹과 멀티프로세싱

**프로그램 실행**

> 프로그램을 구성하는 코드를 순서대로 CPU에서 연산(실행)하는 일

- CPU 코어가 하나일 경우, 한 번에 하나의 프로그램 코드만 실행
- 이를 해결하기 위해 하나의 CPU 코어로 여러 프로그램을 동시에 실행하는 `Multitasking` 기술 등장

#### ℹ️ Multitasking

> 하나의 컴퓨터 시스템이 동시에 여러 작업을 수행하는 능력
  - 운영체제가 스케줄링을 수행하고, CPU를 최대한 사용하면서 작업이 골고루 수행될 수
있게 최적화
- 현대의 CPU는 초당 수십억 번 이상의 연산을 수행
  - CPU가 매우 빠르게 두 프로그램의 코드를 번갈아 수행한다면, 사람이 느낄 때 두 프로그램이 동시에 실행되는 것처럼 느껴짐
  - 시분할(Time Sharing) 기법: 프로그램의 실행 시간을 분할해서 마치 동시에 실행되는 것 처럼 하는 기법

#### ℹ️ Multiprocessing

> 컴퓨터 시스템에서 둘 이상의 프로세서(CPU Core)를 사용하여 여러 작업을 동시에 처리하는 기술

- 하나의 CPU 안에 보통 2개 이상의 코어
- 멀티프로세싱 시스템은 하나의 CPU 코어만을 사용하는 시스템보다 동시에 더 많은 작업
을 처리

**Multiprocessing VS. Multitasking**

|Multiprocessing|Multitasking|
|---|---|
|여러 CPU(여러 CPU 코어)를 사용하여 동시에 여러 작업을 수행하는 것을 의미|단일 CPU(단일 CPU 코어)가 여러 작업을 동시에 수행하는 것처럼 보이게 하는 것|
|하드웨어 기반으로 성능을 향상|소프트웨어 기반으로 CPU 시간을 분할하여 각 작업에 할당|
|다중 코어 프로세서를 사용하는 현대 컴퓨터 시스템| 현대 운영 체제에서 여러 애플리케이션이 동시에 실행되는 환경|

...

### 프로세스와 스레드

#### ℹ️ Proccess

> 운영체제 안에서 실행중인 프로그램

- 프로세스는 실행 중인 프로그램의 인스턴스
  - 각 프로세스는 독립적인 메모리 공간을 갖고 있으며, 운영체제에서 별도의 작업 단위로 분리해서 관리
  - 프로세스가 서로의 메모리에 직접 접근/간섭할 수 없다
- 자바 언어로 비유를 하자면 클래스는 프로그램이고, 인스턴스는 프로세스

**프로세스의 메모리 구성**
- Code Section: 실행할 프로그램의 코드가 저장되는 부분
- Data Section: 전역 변수 및 정적 변수가 저장되는 부분
- Heap: 동적으로 할당되는 메모리 영역
- Stack: 메서드(함수) 호출 시 생성되는 지역 변수와 반환 주소가 저장되는 영역(스레드에 포함)

#### ℹ️ Thread

> 프로세스 내에서 실행되는 작업의 단위

- 프로세스는 하나 이상의 스레드를 반드시 포함
  - 한 프로세스 내에서 여러 스레드가 존재할 수 있으며, 이들은 프로세스가 제공하는 동일한 메모리 공간을 공유(단일 스레드, 멀티 스레드)
  - 프로세스보다 단순하므로 생성 및 관리가 단순하고 가볍
- 하나의 프로그램도 그 안에서 동시에 여러 작업이 필요하므로 멀티스레드가 필요

**메모리 구성**
- 공유 메모리: 같은 프로세스의 코드 섹션, 데이터 섹션, 힙(메모리)은 프로세스 안의 모든 스레드가 공유
- 개별 스택: 각 스레드는 자신의 스택을 보유

`프로세스`는 실행 환경과 자원을 제공하는 컨테이너 역할, `스레드`는 CPU를 사용해서 코드를 하나하나 실행하는 역할

...

### 스레드와 스케줄링

**단일 코어 스케줄링**
- 운영체제는 내부에 스케줄링 큐를 가지고 있고, 각 스레드는 스케줄링 큐에서 대기
- 각 스레드는 번갈아가면서 코드를 실행

**멀티 코어 스케줄링**
- CPU 코어가 2개 이상이면 한 번에 더 많은 스레드를 물리적으로 동시에 실행

**프로세스, 스레드와 스케줄링**
- 멀티태스킹과 스케줄링
  - `멀티태스킹`은 동시에 여러 작업을 수행하는 것을 의미
  - 이를 위해 운영체제는 `스케줄링`이라는 기법을 사용
  - `스케줄링`은 CPU 시간을 여러 작업에 나누어 배분하는 방법
- 프로세스와 스레드
  - `프로세스`는 실행 중인 프로그램의 인스턴스
    - 각 프로세스는 독립적인 메모리 공간을 가지며, 운영체제에서 독립된 실행 단위로 취급
  - `스레드`는 프로세스 내에서 실행되는 작은 단위
    - 여러 스레드는 하나의 프로세스 내에서 자원을 공유하며, 프로세스의 코드, 데이터, 시스템 자원등을 공유
    - 실제로 CPU에 의해 실행되는 단위는 스레드
- 프로세스의 역할
  - `프로세스`는 실행 환경(컨테이너 역할)을 제공
    - 메모리 공간, 파일 핸들, 시스템 자원(네트워크 연결) 등이 포함
  - 프로세스 자체는 운영체제의 스케줄러에 의해 직접 실행되지 않으며, **프로세스 내의 스레드가 실행**
    - 1개의 프로세스 안에 하나의 스레드만 실행되는 경우도 있고, 여러 스레드가 실행되는 경우도 존재

...

### 컨텍스트 스위칭

**컨텍스트 스위칭(context switching)**
- 스레드A를 멈추는 시점에 CPU에서 사용하던 값들을 메모리에 저장해두어야 한다. 
- 그리고 이후에 스레드A를 다시 실행할 때 이 값들을 CPU에 다시 불러오는 과정

**멀티스레드는 대부분 효율적이지만, 컨텍스트 스위칭 과정이 필요하므로 항상 효율적인 것은 아니다**
- 현재 작업하는 문맥이 변하기 때문에 컨텍스트(현재 작업하는 문맥) 스위칭
  - 컨텍스트 스위칭 과정에서 이전에 실행 중인 값을 메모리에 잠깐 저장하고,
  - 이후에 다시 실행하는 시점에 저장한 값을 CPU에 다시 불러와야 한다.
- 컨텍스트 스위칭 과정에는 약간의 비용이 발생
  - 연산 시간 + 컨텍스트 스위칭 시간
- 실제로 컨텍스트 스위칭에 걸리는 시간은 아주 짧지만 스레드가 매우 많다면 이 비용이 커질 수 있음

**CPU와 스레드의 관계**

```text
CPU 4개 / 스레드 2개
스레드의 숫자가 너무 적으면 모든 CPU를 100% 다 활용할 수 없지만, 
스레드가 몇 개 없으므로 컨텍스트 스위칭 비용이 줄어든다.

CPU 4개 / 스레드 100개
스레드의 숫자가 너무 많으면 CPU를 100% 다 활용할 수 있지만 컨텍스트 스위칭 비용이 늘어난다.

CPU 4개 / 스레드 4개
스레드의 숫자를 CPU의 숫자에 맞춘다면 CPU를 100% 활용할 수 있고, 컨텍스트 스위칭 비용도 자주 발생하지 않기 때문에 최적의 상태가 된다. 
이상적으로는 CPU 코어 수 + 1개 정도로 스레드를 맞추면 특정 스레드가 잠시 대기할 때 남은 스레드를 활용할 수 있다.
```

**CPU 바운드 작업 vs I/O 바운드 작업**

각 스레드가 하는 작업은 크게 2가지로 구분

- `CPU-bound tasks`
  - CPU의 연산 능력을 많이 요구하는 작업을 의미
  - 계산, 데이터 처리, 알고리즘 실행 등 CPU의 처리 속도가 작업 완료 시간을 결정
  - ex. 복잡한 수학 연산, 데이터 분석, 비디오 인코딩, 과학적 시뮬레이션 등.
- `I/O-bound tasks`
  - 디스크, 네트워크, 파일 시스템 등과 같은 입출력(I/O) 작업을 많이 요구하는 작업
  - I/O 작업이 완료될 때까지 대기 시간이 많이 발생하며, CPU는 상대적으로 유휴(대기) 상태에 있는 경우가 많음(스레드가 CPU를 사용하지 않고 I/O 작업이 완료될 때 까지 대기)
  - ex. 데이터베이스 쿼리 처리, 파일 읽기/쓰기, 네트워크 통신, 사용자 입력 처리 등.

**스레드 설정**

스레드의 숫자는 CPU-바운드 작업이 많은지, I/O-바운드 작업이 많은지에 따라 다르게 설정이 필요

- `CPU-bound tasks`: CPU 코어 수 + 1개
  - CPU를 거의 100% 사용하는 작업이므로 스레드를 CPU 숫자에 최적화
- `I/O-bound tasks`: CPU 코어 수 보다 많은 스레드를 생성(CPU를 최대한 사용할 수 있는 숫자까지 스레드 생성)
  - CPU를 많이 사용하지 않으므로 성능 테스트를 통해 CPU를 최대한 활용하는 숫자까지 스레드 생성
  - 단, 너무 많은 스레드를 생성하면 컨텍스트 스위칭 비용도 함께 증가하므로 적절한 성능 테스트 필요

> 웹 애플리케이션 서버라도 상황에 따라 CPU 바운드 작업이 많을 수 있다. 
> 
> 이 경우 CPU-바운드 작업에 최적화된 CPU 숫자를 고려

...

## Thread creation and execution

### 자바 메모리 구조

<figure><img src="../../.gitbook/assets/java-adv/java-memory.png" alt=""><figcaption></figcaption></figure>

**메서드 영역(Method Area)**
- 메서드 영역은 프로그램을 실행하는데 필요한 공통 데이터를 관리하고, 프로그램의 모든 영역에서 공유
  - 클래스 정보: 클래스의 실행 코드(바이트 코드), 필드, 메서드와 생성자 코드등 모든 실행 코드 존재
  - static 영역: static 변수들을 보관
  - 런타임 상수 풀: 프로그램을 실행하는데 필요한 공통 리터럴 상수를 보관

**스택 영역(Stack Area)**
- 자바 실행 시, 하나의 실행 스택이 생성되고, 각 스택 프레임은 지역 변수, 중간 연산 결과, 메서드 호출 정보 등을 포함
  - 스택 프레임: 스택 영역에 쌓이는 네모 박스가 하나의 스택 프레임. 메서드를 호출할 때 마다 하나의 스택 프레임이 쌓이고, 메서드가 종료되면 해당 스택 프레임이 제거
- 스택 영역은 더 정확히 각 스레드별로 하나의 실행 스택이 생성. 따라서 스레드 수 만큼 스택이 생성

**힙 영역(Heap Area)**
- 객체(인스턴스)와 배열이 생성되는 영역이다
- 가비지 컬렉션(GC)이 이루어지는 주요 영역이며, 더 이상 참조되지 않는 객체는 GC에 의해 제거

...

### extends Thread 

Thread 클래스를 상속 받거나 Runnable 인터페이스를 구현하여 스레드를 생성

[thread start](https://github.com/jihunparkme/inflearn-java-adv1/commit/31e406b4aadea4f26bccb23ab966e4dbec8acea9)

**Thread.start()**
- 스레드 실행 메서드
- HelloThread 스레드가 별도의 스레드에서 run() 메서드를 실행
- main 스레드는 start() 메서드를 통해 Thread-n 스레드에게 실행을 지시
  - main 스레드가 run() 을 호출할 경우 별도의 스레드가 아닌 main 스레드에서 직접 실행

> 스레드는 순서와 실행 기간을 모두 보장하지 않는다.

...

### Daemon Thread

스레드는 User Thread, Daemon Thread 2가지 종류로 구분

**User Thread(non-daemon Thread)**
- 프로그램의 주요 작업을 수행
- 작업이 완료될 때까지 실행
- 모든 user thread가 종료되면 JVM도 종료

**Daemon Thread**
- 백그라운드에서 보조적인 작업을 수행
- 모든 user thread가 종료되면 daemon thread는 자동으로 종료

[Daemon Thread](https://github.com/jihunparkme/inflearn-java-adv1/commit/405718ae09ebcca5a0d1cfd847ded0a704539c60)

...

### implements Runnable

실무에서는 보통 Runnable을 구현하는 방식으로 사용
- 실행 결과는 기존과 동일
- 스레드와 해당 스레드가 실행할 작업이 서로 분리되어 있다는 차이
  - 스레드 객체를 생성할 때, 실행할 작업을 생성자로 전달

[Runnable Thread](https://github.com/jihunparkme/inflearn-java-adv1/commit/4ae155fbcf9f167f2a78ae78ea6d58080385392a)

**Thread 클래스 상속 방식**

`장점`
- 간단한 구현: Thread 클래스를 상속받아 run() 메서드만 재정의

`단점`
- 상속의 제한: 자바는 단일 상속만을 허용
- 유연성 부족: 인터페이스를 사용하는 방법에 비해 유연성 감소

**Runnable 인터페이스를 구현 방식**

`장점`
- 상속의 자유로움: 다른 클래스를 상속받아도 문제없이 구현 가능
- 코드의 분리: 스레드와 실행할 작업을 분리하여 코드의 가독성을 향상
- 여러 스레드가 동일한 Runnable 객체를 공유할 수 있어 자원 관리에 효율적

`단점`
- 코드가 약간 복잡: Runnable 객체를 생성하고 이를 Thread 에 전달하는 과정이 추가

> Runnable 인터페이스를 구현하는 방식을 사용하자. 
> 
> 스레드와 실행할 작업을 명확히 분리하고, 인터페이스를 사용하므로 Thread 클래스를 직접 상속하는 방식보다 더 유연하고 유지보수 하기 쉬운 코드를 만들 수 있다.

...

## 스레드 제어와 생명 주기

### 스레드의 기본 정보

Thread 클래스는 스레드를 생성하고 관리하는 기능을 제공

[스레드 기본 정보](https://github.com/jihunparkme/inflearn-java-adv1/commit/a9f10d6a2300c16db2c14105da715663586943a6)

```java
/** main Thread */
Thread mainThread = Thread.currentThread();
/**
 * Thread[#1,main,5,main]
 * [threadId, threadName, threadPriority, threadGroup]
 */
log("mainThread = " + mainThread);
/**
 * 1
 * 스레드의 고유 식별자를 반환하는 메서드(각 스레드에 대해 유일한 ID) -> ID는 스레드가 생성될 때 할당되며, 직접 지정할 수 없다
 */
log("mainThread.threadId() = " + mainThread.threadId());
/**
 * main
 * 스레드의 이름을 반환하는 메서드
 * 스레드 ID는 중복되지 않지만, 스레드 이름은 중복될 수 있다.
 */
log("mainThread.getName() = " + mainThread.getName());
/**
 * 5
 * 스레드의 우선순위를 반환하는 메서드
 * 우선순위는 1(가장 낮음)에서 10(가장 높음)까지의 값으로 설정할 수 있으며, 기본값은 5
 * setPriority() 메서드를 사용해서 우선순위를 변경 가능
 * 우선순위는 스레드 스케줄러가 어떤 스레드를 우선 실행할지 결정하는 데 사용. 하지만 실제 실행 순서는 JVM 구현과 운영체제에 따라 달라질 수 있다.
 */
log("mainThread.getPriority() = " + mainThread.getPriority());
/**
 * java.lang.ThreadGroup[name=main,maxpri=10]
 * 스레드가 속한 스레드 그룹을 반환하는 메서드
 * 스레드 그룹은 스레드를 그룹화하여 관리할 수 있는 기능을 제공
 * 기본적으로 모든 스레드는 부모 스레드와 동일한 스레드 그룹에 속함
 * 스레드 그룹은 여러 스레드를 하나의 그룹으로 묶어서 특정 작업(예: 일괄 종료, 우선순위 설정 등)을 수행 가능
 * 
 * 부모 스레드(Parent Thread)
 * 새로운 스레드를 생성하는 스레드를 의미
 * 스레드는 기본적으로 다른 스레드에 의해 생성. 이러한 생성 관계에서 새로 생성된 스레드는 생성한 스레드를 부모로 간주
 */
log("mainThread.getThreadGroup() = " + mainThread.getThreadGroup());
/**
 * RUNNABLE
 * 스레드의 현재 상태를 반환하는 메서드
 * 반환되는 값은 Thread.State 열거형에 정의된 상수 중 하나
 * - NEW: 스레드가 아직 시작되지 않은 상태
 * - RUNNABLE: 스레드가 실행 중이거나 실행될 준비가 된 상태
 * - WAITING: 스레드가 다른 스레드의 특정 작업이 완료되기를 기다리는 상태
 * - TIMED_WAITING: 일정 시간 동안 기다리는 상태
 * - BLOCKED: 스레드가 동기화 락을 기다리는 상태이
 * - TERMINATED: 스레드가 실행을 마친 상태
 */
log("mainThread.getState() = " + mainThread.getState());
```

...

### 스레드의 생명 주기

**스레드는 생성하고 시작하고, 종료되는 생명주기를 가진다.**

<figure><img src="../../.gitbook/assets/java-adv/thread-state.png" alt=""><figcaption></figcaption></figure>

**ℹ️ New (새로운 상태)**
- 스레드가 생성되고 아직 시작되지 않은 상태
- Thread 객체가 생성되었지만 start() 메서드가 호출되지 않은 상태
- `Thread thread = new Thread(runnable);`

**ℹ️ Runnable (실행 가능 상태)**
- 스레드가 실행될 준비가 되어 CPU에서 실행될 수 있는 상태
  - `thread.start()` 메서드가 호출되면 스레드는 Runnable 상태로 전이
- Runnable 상태에 있는 모든 스레드가 동시에 실행되는 것은 아니다.
  - 운영체제의 스케줄러가 각 스레드에 CPU 시간을 할당하여 실행하기 때문에, 
  - Runnable 상태에 있는 스레드는 스케줄러의 실행 대기열에 포함되어 있다가 차례로 CPU에서 실행
- 운영체제 스케줄러의 실행 대기열에 있든, CPU에서 실제 실행되고 있든 모두 RUNNABLE 상태
  - 자바에서 둘을 구분해서 확인할 수 없어서 보통 실행 상태라고 부름

**ℹ️ Blocked (차단 상태)**
- 스레드가 다른 스레드에 의해 동기화 락을 얻기 위해 기다리는 상태
  - synchronized 블록에 진입하기 위해 락을 얻어야 하는 경우

**ℹ️ Waiting (대기 상태)**
- 스레드가 다른 스레드의 특정 작업이 완료되기를 무기한 기다리는 상태
  - `wait()`, `join()` 메서드가 호출될 때 Waiting 상태로 전이
- 다른 스레드가 `notify()` or `notifyAll()` 메서드를 호출하거나, `join()` 이 완료될 때까지 대기

**ℹ️ Timed Waiting (시간 제한 대기 상태)**
- 스레드가 특정 시간 동안 다른 스레드의 작업이 완료되기를 기다리는 상태
  - `sleep(long millis)`, `wait(long timeout)`, `join(long millis)` 메서드가 호출될 때 Timed Waiting 상태로 전이
- 주어진 시간이 경과하거나 다른 스레드가 해당 스레드를 깨우면 이 상태에서 벗어남

**ℹ️ Terminated (종료 상태)**
- 스레드의 실행이 완료된 상태
  - 스레드가 정상적으로 종료되거나, 예외가 발생하여 종료된 경우 Terminated 상태로 전이
- 스레드는 한 번 종료되면 다시 시작할 수 없음

```java
public static void main(String[] args) throws InterruptedException {
    Thread thread = new Thread(new MyRunnable(), "myThread");
    log("myThread.state1 = " + thread.getState()); // (1) NEW

    log("myThread.start()");
    thread.start();
    Thread.sleep(1000);

    log("myThread.state3 = " + thread.getState()); // (3) TIMED_WAITING (Thread.sleep())
    Thread.sleep(4000);

    log("myThread.state5 = " + thread.getState()); // (5) TERMINATED
    log("end");
}

static class MyRunnable implements Runnable {
    public void run() {
        try {
            log("start");
            log("myThread.state2 = " + Thread.currentThread().getState()); // (2) RUNNABLE

            log("sleep() start");
            Thread.sleep(3000);
            log("sleep() end");

            log("myThread.state4 = " + Thread.currentThread().getState()); // (4) RUNNABLE
            log("end");
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }
    }
}
```

**참고. 자바에서 메서드 재정의 시, 재정의 메서드가 지켜야할 예외와 관련된 규칙**
- 체크 예외
  - 부모 메서드가 체크 예외를 던지지 않는 경우, 재정의된 자식 메서드도 체크 예외를 던질 수 없음
  - 자식 메서드는 부모 메서드가 던질 수 있는 체크 예외의 하위 타입만 던질 수 있음
- 언체크(런타임) 예외
  - 예외 처리를 강제하지 않으므로 상관없이 던질 수 있음

> run() 메서드는 체크 예외를 던질 수 없도록 강제하여 프로그램이 비정상 종료되는 상황을 방지
> 
> 멀티 스레딩 환경에서 예외 처리를 강제함으로써 스레드의 안정성과 일관성을 유지
>
> 최근에는 체크 예외보다는 언체크 예외를 선호

...

### Join

[join이 필요한 상황](https://github.com/jihunparkme/inflearn-java-adv1/commit/8e8510bb33ad9056e902588d24a9c71b7fe186eb)
- 기대와 다르게 두 결과 모두 0이 나온다.

[join 사용](https://github.com/jihunparkme/inflearn-java-adv1/commit/2c21b02a50957803c6a4fb4d5415f2560045203d)

**Waiting (대기 상태)**
- 스레드가 다른 스레드의 특정 작업이 완료되기를 무기한 기다리는 상태
- `join()`을 호출하는 스레드는 대상 스레드가 `TERMINATED` 상태가 될 때 까지 대기
  - 대상 스레드가 `TERMINATED` 상태가 되면 호출 스레드는 다시 `RUNNABLE` 상태가 되면서 다음 코드를 수행

> 특정 스레드가 완료될 때 까지 기다려야 하는 상황이라면 join() 을 사용하자.

다른 스레드의 작업을 일정 시간 동안만 기다릴 경우
- `join()` : 호출 스레드는 대상 스레드가 완료될 때 까지 무한정 대기
- `join(ms)` : 호출 스레드는 특정 시간 만큼만 대기
  - 호출 스레드는 지정한 시간이 지나면 다시 RUNNABLE 상태가 되면서 다음 코드를 수행

...

### Interrupt

> 인터럽트를 사용하면 `WAITING`, `TIMED_WAITING` 같은 대기 상태의 스레드를 직접 깨워서, 
> 
> 작동하는 `RUNNABLE` 상태로 만들 수 있다.

스레드가 인터럽트 상태일 때
- InterruptedException 을 던지는 메서드(ex. Thread.sleep())를 호출하거나
- 이미 위 메서드를 호출하고 대기중이라면 InterruptedException 발생

`Thread.currentThread().isInterrupted()`
- 스레드의 인터럽트 상태를 단순히 확인

`Thread.interrupted()`
- 인터럽트를 직접 체크해서 사용할 경우 
  - 스레드가 인터럽트 상태일 경우, `true 반환` 후 해당 스레드의 인터럽트 `상태를 false 로 변경`
  - 스레드가 인터럽트 상태가 아닐 경우, `false 반환` 후 해당 스레드의 인터럽트 상태를 변경하지 않음

[thread.interrupted()](https://github.com/jihunparkme/inflearn-java-adv1/commit/548ac91a4139c64305c55db64f1640c33cb708d9)

> 인터럽트 예외가 발생하고, 스레드의 인터럽트 상태를 정상(false)으로 돌리지 않으면, 이후에도 계속 인터럽트가 발생
>
> 그러므로, 자바에서 인터럽트 예외가 발생하거나 인터럽트의 목적을 달성하면, 스레드의 인터럽트 상태를 다시 정상(false)으로 돌린다.
> - InterruptedException
> - Thread.interrupted()

**프린터 예제**
- [시작]()
  - 종료 입력 시 바로 반응하지 않는 문제
- [인터럽트 도입]()
- [인터럽트 개선]()










## Section

### Sub Section

#### ℹ️

...

### Sub Section

---

## Section