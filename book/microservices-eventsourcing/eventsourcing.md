# CHAPTER 3. 이벤트 소싱 I

> 데이터의 상태 변화 과정을 기록해야 하는 요구사항에서 이벤트 소싱을 적용할 수 있다.

## 감사와 이력

> 변경 사항을 기록하기 위해 이전 상태를 분리해서 기록해 둔 후 필요할 때 현제 상태와 비교하기 위한 방법

### 단일 테이블과 시퀀스

첫 번째 방법으로 TB_EMPLOYEE 테이블 식별자인 EMP_NO 외에 일련변호를 추가하고, 변경해야 할 때 기존 레코드를 유지하고 일련번호를 증가시켜 새로운 레코드를 추가하기.
- 일련번호 대신 시간으로 사용 가능

<figure><img src="../../.gitbook/assets/microservices-eventsourcing/t3-3.png" alt=""><figcaption></figcaption></figure>

1. 현재 상태의 마지막 일련번호 조회
2. 일련번호 값 증가
3. 파라미터 employee에 새로운 일련번호 할당
4. employeeDao를 사용해 최종 상태를 새로운 레코드로 추가

### 상태 테이블과 이력 테이블

두 번째 방법으로 현재 상태를 가진 TB_EMPLOYEE 테이블과 이력을 기록하는 TB_EMPLOYEE_HISTORY 테이블로 분리
- 기록 테이블은 현재 상태를 가진 테이블과 동일한 컬럼을 구성하고
- 단일 테이블-시퀀스 방법과 동일하게 상태 변화 순서를 구별하기 위해 일련번호를 추가

<figure><img src="../../.gitbook/assets/microservices-eventsourcing/t3-4-5.png" alt=""><figcaption></figcaption></figure>

1. 직원 정보와 직원 정보 이력에서 일련번호 조회
2. 이력 일련번호 증가
3. 저장 단계에서 변경 이전 직원 상태를 이력 테이블에 추가
4. 직원의 새로운 상태를 테이블에 저장

### 변경 값

세 번째 방법은 변경한 속성 이름과 값의 목록만 기록하기
- 전체 복사본이 아닌 변경한 속성만 선택해 기록하면 변경 속성을 찾기 위해 모든 속성의 값을 비교할 필요도 없고 데이터베이스도 효과적으로 사용 가능

<figure><img src="../../.gitbook/assets/microservices-eventsourcing/t3-7.png" alt=""><figcaption></figcaption></figure>

1. 서비스는 애그리게이트가 제공하는 메소드를 호출
2. 영향을 받은 속성 목록을 반환
3. 리포지토리가 제공하는 삽입 메소드에 전달

## 도메인 이벤트

## 이벤트 소싱

## 이벤트 소싱 구현

## 마이크로서비스 모듈

## 이벤트 소싱과 단위 테스트

## 요약