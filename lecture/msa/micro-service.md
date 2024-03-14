# Micro Service

## Micro Service

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