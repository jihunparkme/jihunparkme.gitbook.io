# MCP

MCP(Model Context Protocol)는 앤트로픽이 개발한 오픈소스 AI 프로토콜
- AI 모델과 외부 도구 및 데이터 소스 간의 표준화된 연결을 통해 AI가 컨텍스트를 이해하고 복잡한 작업을 수행하도록 지원
- MCP는 AI 에이전트와 다양한 서비스를 연결하는 과정을 간소화하여, 사용자가 AI 기능을 더욱 쉽게 활용하고 확장할 수 있게 도움
- Claude와 외부 도구(브라우저, 검색 엔진, Jira 등)를 연결해주는 '다리' 역할을 하는 로컬 또는 원격 서버

## 개발자를 위한 필수 MCP

🔎 [Context7](https://context7.com/): 최신 정보를 탐색하는 '리서처'
- 웹 검색 및 최신 기술 문서를 찾아주는 검색 전문 에이전트

👀 [Browser MCP](https://browsermcp.io/): 웹과 상호작용하는 '눈과 손'
- Playwright와 같은 브라우저 자동화 도구를 제어하여, Claude가 직접 웹 브라우저를 조작

👨🏼 [Jira MCP](https://github.com/sooperset/mcp-atlassian): 똑똑한 '프로젝트 관리자'
- Claude를 여러분 팀의 Jira 프로젝트와 직접 연결

⚠️ [Observability MCPs](https://github.com/sandraschi/observability-mcp) (Datadog, Sentry): 24시간 대기하는 '응급 구조대원'
- Claude를 Datadog이나 Sentry 같은 실시간 모니터링 및 에러 트래킹 서비스와 연결

# gh CLI 협업 자동화

> [gh CLI](https://cli.github.com/)

**claude.md**

```markdown
# 개인 블로그 프로젝트 Git 워크플로우 규칙

## 1. Repository
- **GitHub Repository:** `[여러분의-GitHub-ID]/my-personal-blog`
- **Main Branch:** `main`

## 2. Branching Strategy
- 모든 기능 개발은 `feature/[이슈번호]-[간단-설명-kebab-case]` 형식의 브랜치에서 진행한다.
- 이슈 번호가 없는 간단한 수정은 `fix/[간단-설명]` 또는 `chore/[간단-설명]` 브랜치를 사용한다.

## 3. Commit Message Convention
- 모든 커밋 메시지는 **Conventional Commits** 명세를 따른다.
- (예: `feat: Add author profile component`, `fix: Correct typo in footer`)
- 커밋 본문에는 변경 이유를 명확히 서술하고, 관련된 GitHub 이슈를 `Closes #[이슈번호]` 형식으로 반드시 포함한다.

## 4. Pull Request (PR) Process
- 모든 코드는 `main` 브랜치로 직접 푸시할 수 없으며, 반드시 PR을 통해 코드 리뷰를 받아야 한다.
- PR 제목은 커밋 메시지와 동일한 형식을 따른다.
- PR 본문은 `.github/PULL_REQUEST_TEMPLATE.md` 템플릿을 사용한다.
```

- [Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/)

**PULL_REQUEST_TEMPLATE.md**

<details>
<summary>.github/PULL_REQUEST_TEMPLATE.md</summary>

    ```markdown
    ## 📝 변경사항

    <!-- 이 PR에서 변경된 내용을 간단히 설명해주세요 -->

    ### 주요 변경사항
    - [ ] 새로운 블로그 포스트 추가
    - [ ] 기존 포스트 수정
    - [ ] UI/UX 개선
    - [ ] 성능 최적화
    - [ ] 버그 수정
    - [ ] 기타

    ### 상세 설명
    <!-- 변경사항에 대한 자세한 설명을 작성해주세요 -->

    ## 🔍 변경된 파일

    <!-- 변경된 파일 목록을 작성해주세요 -->
    - `app/blog/posts/` - 새로운 블로그 포스트
    - `app/components/` - 컴포넌트 수정
    - `app/` - 기타 파일 수정

    ## 🧪 테스트

    <!-- 테스트 방법을 설명해주세요 -->
    - [ ] 로컬에서 개발 서버 실행 확인
    - [ ] 블로그 포스트 렌더링 확인
    - [ ] 반응형 디자인 확인
    - [ ] SEO 메타데이터 확인

    ### 테스트 방법
    ```bash
    # 개발 서버 실행
    pnpm dev

    # 빌드 테스트
    pnpm build

    # 린트 검사
    pnpm lint
    ```

    ## 📸 스크린샷 (선택사항)

    <!-- UI 변경사항이 있는 경우 스크린샷을 첨부해주세요 -->

    ## ✅ 체크리스트

    - [ ] 코드가 프로젝트의 코딩 스타일을 따릅니다
    - [ ] 자체적으로 코드를 검토했습니다
    - [ ] 코드에 대한 주석을 작성했습니다 (특히 이해하기 어려운 부분)
    - [ ] 해당 변경사항에 대한 문서를 업데이트했습니다
    - [ ] 새로운 기능에 대한 테스트를 추가했습니다
    - [ ] 모든 테스트가 통과합니다

    ## 🚀 배포 관련

    - [ ] 이 변경사항이 프로덕션에 배포되어도 안전합니다
    - [ ] 환경 변수 변경이 필요한 경우 문서화했습니다

    ## 📚 관련 이슈

    <!-- 관련된 이슈가 있다면 링크해주세요 -->
    Closes #(이슈 번호)

    ## 💡 추가 정보

    <!-- 추가로 전달하고 싶은 정보가 있다면 작성해주세요 -->

    ---

    **리뷰어 참고사항:**
    - 블로그 포스트의 경우 마크다운 문법과 내용의 품질을 확인해주세요
    - SEO 최적화가 필요한지 검토해주세요
    - 모바일 반응형이 제대로 작동하는지 확인해주세요
    ```
</details>

# 🦸 AI '어벤져스 팀' 구성하기

> Sub-agents를 활용한 병렬 작업

```bash
/agents
```

**'리팩토링 전문가' 서브에이전트**

```text
너는 10년 경력의 '클린 코드' 전문가야. 너의 유일한 임무는 React 컴포넌트 파일의 코드를 리팩토링하는 것이야. 

작업 절차:
1. 지정된 파일을 읽고 분석
2. SOLID 원칙을 적용하여 구조 개선
3. 변수명과 함수명을 더 명확하게 변경
4. 불필요한 중복 코드 제거
5. 원본 파일을 개선된 코드로 덮어쓰기
6. 완료되면 'Refactoring complete.' 메시지만 출력
```

**'테스트 전문가' 서브에이전트**

```text
너는 React Testing Library 전문 QA 엔지니어야. 

작업 절차:
1. 지정된 컴포넌트 파일을 분석
2. 모든 props와 엣지 케이스를 커버하는 완벽한 단위 테스트 작성
3. 테스트 파일을 .test.tsx 확장자로 생성
4. 완료되면 'Test file created.' 메시지만 출력
```

**'문서 작성 전문가' 서브에이전트**

```text
너는 전문 테크니컬 라이터야. 

작업 절차:
1. 컴포넌트 파일을 읽고 분석
2. 컴포넌트의 목적과 기능 파악
3. 각 props의 타입과 설명 정리
4. 명확한 사용 예제 작성
5. 마크다운 형식으로 문서 생성
6. 완료되면 'Documentation created.' 메시지만 출력
```

명시적 호출 또는 자동 위임으로 서브 에이전트들에게 작업을 지시

> 더 능동적인 서브 에이전트 사용을 장려하려면 description 필드에 “use PROACTIVELY” 또는 “MUST BE USED”와 같은 문구를 포함

<details>
<summary>🌱 Kopring 기반 서브 에이전트 프롬프트</summary>

**1. 리팩토링 전문가**

```text
너는 10년 경력의 Kotlin & Spring Boot 백엔드 아키텍트야. 너의 유일한 임무는 지정된 Kotlin 파일(Controller/Service/Repository/Domain)의 코드를 리팩토링하는 것이야.

작업 절차:
1. 지정된 파일과 관련 의존 클래스를 읽고 분석
2. SOLID 원칙과 Kotlin idiomatic 스타일(data class, sealed class, scope function, null safety) 적용
3. 서비스 로직과 트랜잭션 경계(@Transactional), 계층 책임(Controller-Service-Repository) 분리 재검토
4. 불필요한 nullable 타입, !! 연산자, 중복 로직 제거
5. 기존 시그니처(public API)는 유지하거나 변경 시 호출부까지 함께 수정
6. 원본 파일을 개선된 코드로 덮어쓰기
7. 완료되면 'Refactoring complete.' 메시지만 출력

제약: 비즈니스 로직 변경 금지, 테스트 코드는 별도 에이전트에 위임, 리팩토링 사유는 커밋 메시지가 아닌 코드 주석 최소화로 표현.

참고: 더 능동적인 서브 에이전트 사용을 장려하기 위해 description 필드에 “use PROACTIVELY” 또는 “MUST BE USED”와 같은 문구를 포함.
```

**2. 테스트 전문가**

```text
너는 Kotlin & Spring Boot 전문 QA 엔지니어야. JUnit5, MockK, SpringMockK, AssertJ, @SpringBootTest / @WebMvcTest / @DataJpaTest에 능숙해.

작업 절차:
1. 지정된 클래스(Service/Controller/Repository)를 분석
2. 계층에 맞는 테스트 전략 선택 (Controller→@WebMvcTest+MockMvc, Service→단위 테스트+MockK, Repository→@DataJpaTest)
3. 정상 케이스, 예외 케이스, 경계값(null, empty, 동시성 필요 시 표시)까지 커버
4. Given-When-Then 패턴으로 테스트 메서드명 작성 (예: `should_ReturnUser_when_ValidIdProvided`)
5. 테스트 파일을 `src/test/kotlin/.../XxxTest.kt`로 생성
6. 완료되면 'Test file created.' 메시지만 출력

제약: 프로덕션 코드 수정 금지, 외부 API/DB는 반드시 mock 처리.

참고: 더 능동적인 서브 에이전트 사용을 장려하기 위해 description 필드에 “use PROACTIVELY” 또는 “MUST BE USED”와 같은 문구를 포함.
```

**3. 자료 검색 전문가**

```text
너는 기술 리서처야. 너의 임무는 웹에서 최신 기술 문서, 공식 레퍼런스, 베스트 프랙티스를 검색하고 요약하는 것이야.

작업 절차:
1. 질문에서 핵심 키워드와 검색 범위(공식 문서/커뮤니티/버전별 차이) 파악
2. 공식 문서(Kotlin, Spring 공식 사이트 등)를 최우선 출처로 검색
3. 검색 결과를 신뢰도 순으로 정리, 출처 URL 반드시 명시
4. 상충되는 정보가 있으면 버전/조건별 차이를 명확히 구분
5. 최종 결과는 '요약 → 근거(출처 포함) → 실무 적용 시 주의사항' 순서로 보고
6. 코드 수정이나 파일 생성은 하지 않음

제약: 추측성 정보 금지, 출처 없는 주장 금지, 3개 이상 출처 교차 검증 권장.

참고: 더 능동적인 서브 에이전트 사용을 장려하기 위해 description 필드에 “use PROACTIVELY” 또는 “MUST BE USED”와 같은 문구를 포함.
```

**4. 개발 전문가**

```text
너는 Kotlin & Spring Boot 실무 백엔드 개발자야. 요구사항에 따라 실제 동작하는 기능을 구현하는 것이 임무야.

작업 절차:
1. 요구사항과 기존 프로젝트 구조(패키지 컨벤션, 계층 구조)를 파악
2. 필요한 Entity/DTO/Repository/Service/Controller를 계층 규칙에 맞게 설계
3. Kotlin idiomatic하게 구현 (data class, val 우선, null safety, coroutine은 명시된 경우만)
4. 예외는 커스텀 예외 + @ExceptionHandler(or @RestControllerAdvice)로 처리
5. 기존 코드 스타일(네이밍, 어노테이션 순서, 패키지 규칙)을 그대로 따름
6. 구현 후 컴파일/빌드 가능 여부 확인
7. 완료되면 변경 파일 목록과 'Implementation complete.' 메시지 출력

제약: 테스트 작성은 테스트 전문가에게 위임, 임의로 기존 API 시그니처 변경 금지.

참고: 더 능동적인 서브 에이전트 사용을 장려하기 위해 description 필드에 “use PROACTIVELY” 또는 “MUST BE USED”와 같은 문구를 포함.
```

**6. API 설계 전문가**

```text
너는 REST API 설계 전문가야. Spring Boot 기반 API의 엔드포인트/DTO 설계를 담당해.

작업 절차:
1. 요구사항에서 리소스와 액션 파악
2. RESTful 규칙에 맞는 URL, HTTP Method, 상태 코드 설계
3. 요청/응답 DTO 스키마 정의 (버전 관리, validation 어노테이션 포함)
4. 에러 응답 포맷 표준화 제안
5. OpenAPI(Swagger) 주석 또는 스펙 초안 작성
6. 실제 구현은 하지 않고 설계 문서만 산출

참고: 더 능동적인 서브 에이전트 사용을 장려하기 위해 description 필드에 “use PROACTIVELY” 또는 “MUST BE USED”와 같은 문구를 포함.
```

**7. DB/쿼리 최적화 전문가**

```text
너는 JPA/QueryDSL 기반 DB 성능 최적화 전문가야.

작업 절차:
1. 지정된 Repository/Service의 쿼리 패턴 분석
2. N+1, 불필요한 fetch join, 인덱스 미사용 여부 파악
3. 실행 계획 기반 개선안 제시 (fetch join, batch size, 인덱스 추가 등)
4. 개선 코드와 함께 예상 성능 개선 근거 설명
5. 스키마 변경이 필요하면 마이그레이션 스크립트 초안 제공

참고: 더 능동적인 서브 에이전트 사용을 장려하기 위해 description 필드에 “use PROACTIVELY” 또는 “MUST BE USED”와 같은 문구를 포함.
```

**8. 보안 리뷰 전문가**

```text
너는 Spring Security 및 애플리케이션 보안 전문가야. 읽기 전용으로 취약점만 찾아.

작업 절차:
1. 인증/인가 로직, 입력 검증, 시크릿 하드코딩 여부 점검
2. SQL Injection, 권한 우회, CSRF/CORS 설정 오류 등 확인
3. 발견된 이슈를 심각도/확신도와 함께 표로 보고
4. 코드 수정은 요청 시에만 수행

참고: 더 능동적인 서브 에이전트 사용을 장려하기 위해 description 필드에 “use PROACTIVELY” 또는 “MUST BE USED”와 같은 문구를 포함.
```
</details>

# 🕹️ 여러 AI 동시 관리하기

> [claude-squad](https://github.com/smtg-ai/claude-squad)
>
> 여러 명의 독립적인 AI 어시스턴트를 동시에 고용하여, 각자 격리된 작업 공간에서 서로 다른 임무를 병렬로 수행하게 하고, 우리는 중앙 관제실에서 이 모든 상황을 한눈에 지휘
>
> - tmux (터미널 세션 관리자)
> - git worktree (Git 작업 공간 관리자)

**claude-squad의 작동 원리**

✅ **[Tmux](https://github.com/tmux/tmux/wiki)**
- 터미널 세션을 관리하고 여러 개의 창과 패널을 활용해 작업 효율을 극대화하는 도구

✅ **[Git Worktree](https://git-scm.com/docs/git-worktree)**
- 하나의 Git 저장소에 여러 개의 작업 디렉토리를 만들어 여러 브랜치를 동시에 작업할 수 있게 해주는 기능

> [orca](https://github.com/stablyai/orca)
>
> Git Worktree를 통해 여러 에이전트를 독립 실행하는 IDE급 기능을 가진 도구

| 구분                  | Orca (`stablyai/orca`)                                                                                                                          | Claude Squad (`smtg-ai/claude-squad`)                                                                                        |
| :-------------------- | :---------------------------------------------------------------------------------------------------------------------------------------------- | :--------------------------------------------------------------------------------------------------------------------------- |
| **기본 정체성**       | AI 에이전트 통합 관리를 위한 **GUI 데스크톱 앱(ADE)**                                                                                           | 터미널 기반의 **CLI / TUI 관리 도구**                                                                                        |
| **Git Worktree 격리** | 각 에이전트를 Worktree 기반 독립 폴더에서 시각적으로 분리 실행                                                                                  | Worktree 기반 백그라운드 세션 격리 관리                                                                                      |

# 🧪 Playwright E2E 테스트 자동화

> [playwright-mcp](https://github.com/microsoft/playwright-mcp)

```bash
# Playwright 설치
pnpm add -D @playwright/test playwright
    
# MCP 등록
claude mcp add playwright npx -- @playwright/mcp@latest
    
# 권한 부여
# 필요 시 `.claude/settings.local.json` 파일의 `allowTools`에 `"mcp__playwright__*"`를 추가하여 자동 실행 허용
```

## 1단계: 테스트 시나리오 '설계'하기 (자연어)

<details>
<summary>테스트하고 싶은 사용자의 여정을 명확한 자연어로 설명</summary>

```
테스트 시나리오 1: 블로그 홈페이지 네비게이션
1. 홈페이지(/)로 이동한다
2. 네비게이션 바가 표시되는지 확인한다
3. 블로그 포스트 목록이 표시되는지 확인한다
4. Footer가 정상적으로 렌더링되는지 확인한다

테스트 시나리오 2: 블로그 포스트 읽기
1. /blog 페이지로 이동한다
2. 첫 번째 블로그 포스트를 클릭한다
3. 포스트 제목이 표시되는지 확인한다
4. 작성자 프로필이 표시되는지 확인한다
5. 포스트 내용이 정상적으로 렌더링되는지 확인한다
```
</details>

## 2단계: 테스트 케이스 '자동 생성'하기

<details>
<summary>Playwright 테스트 코드를 작성하도록 지시</summary>

```
[Prompt]


"우리 Next.js 블로그의 E2E 테스트를 만들어줘.
다음 시나리오를 테스트하는 Playwright 코드를 작성해줘. 이 과정에서 playwright mcp 를 활용해서 올바른 방식인지 검토해 

1. 홈페이지 접속 및 기본 요소 확인
2. 블로그 목록 페이지에서 포스트 확인
3. 개별 블로그 포스트 페이지 접근 및 콘텐츠 확인

tests/blog-navigation.spec.ts 파일로 저장해줘."
```
</details>


## 3단계: AI가 직접 '테스트 실행'하기

<details>
<summary>AI가 자기가 만든 코드를 직접 실행</summary>

```
[Prompt]
"아주 좋아. 이제 방금 네가 만든 테스트를 playwright MCP를 사용해서 직접 실행해줘.
테스트 결과를 나에게 보고해줘. 만약 테스트가 실패한다면, 실패한 지점의 스크린샷과 함께 에러 메시지를 알려줘."
```
</details>

# 🕸️ Puppeteer 데이터 수집 자동화

> [Puppeteer](https://pptr.dev/)
>
> 스크레이핑 전문 에이전트

```bash
claude mcp add puppeteer npx @modelcontextprotocol/server-puppeteer
```

## Browser MCP

> [browsermcp](https://browsermcp.io/)
> 
> AI가 사용자의 실제 웹 브라우저 탭을 직접 제어할 수 있게 해주는 도구
> - Chrome 확장 프로그램을 통해 보고 있는 브라우저 탭에 연결
> - 기존에 로그인된 세션이나 쿠키를 그대로 활용할 수 있어, 로그인이나 복잡한 인증 절차가 필요한 작업에 유용

```
claude mcp add browsermcp -- npx @browsermcp/mcp@latest
```

**크롬 확장 프로그램 설치**

- [Browser MCP](https://chromewebstore.google.com/detail/browser-mcp-automate-your/bjfgambnhccakkhmkepdoekmckoijdlc?pli=1&authuser=0)

| 특징                | Browser MCP                 | Puppeteer MCP            |
| :------------------ | :-------------------------- | :----------------------- |
| **연결 방식**       | 브라우저 확장 프로그램 필요 | 자동 브라우저 실행       |
| **스크린샷**        | 성공 ✅                      | 성공 ✅                   |
| **DOM 접근**        | 제한적 (페이지 크기 문제)   | 자유로운 JavaScript 실행 |
| **사용 편의성**     | 수동 연결 필요              | 자동화 우수              |
| **로그인 상태**     | 유지 가능                   | 별도의 로그인 로직 필요  |
| **주요 활용**       | 개인 작업, 디버깅           | 대량 자동화, E2E 테스트  |
| **백그라운드 실행** | ❌                           | ✅                        |

# 🚀 AI '드림팀': SuperClaude

> [SuperClaude](https://github.com/SuperClaude-Org/SuperClaude_Framework)
>
> Claude Code 위에 설치되어, AI의 '행동 방식' 자체를 더 똑똑하고 체계적으로 만들어주는 '메타-프로그래밍 설정 프레임워크'

**`SuperClaude`가 제공하는 4가지 핵심 요소:**

1. **Commands (`/sc:*`):** `/sc:brainstorm`, `/sc:implement` 등, 개발의 전체 라이프사이클을 커버하는 **22개의 체계적인 명령어**를 제공합니다.
2. **Agents (`@agent-*`):** `@agent-security`, `@agent-frontend` 등, 특정 전문 지식을 가진 **14명의 AI 전문가 에이전트**를 즉시 호출할 수 있습니다.
3. **Modes (행동 모드):** AI가 처한 상황에 맞게 **'행동 방식' 자체를 전환**시킵니다. 예를 들어, `Brainstorming` 모드에서는 질문을 유도하고, `Token-Efficiency` 모드에서는 컨텍스트를 아껴 쓰는 방식으로 작동합니다.
4. **MCP Servers:** `Context7`(최신 문서 검색), `Playwright`(브라우저 테스트) 등, 사전 구성된 **6개의 강력한 MCP 서버**를 통합하여 제공합니다.

Install

```bash
# Install pipx if not present
python3 -m pip install --user pipx
python3 -m pipx ensurepath

# Install SuperClaude
pipx install SuperClaude

# Run the installer
SuperClaude install
```