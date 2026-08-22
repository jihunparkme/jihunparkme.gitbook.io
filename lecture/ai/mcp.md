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
- **1. 저장소(Repository) 규칙:** 우리 프로젝트의 공식 코드가 저장된 '중앙 본부'가 어디인지 명시합니다.
    - GitHub 저장소: dingcodingco/ai-native-blog
    - 마스터 설계도에 해당하는 최종 브랜치: `main`
- **2. 브랜치(Branch) 전략:** '마스터 설계도(`main`)'를 직접 건드리는 위험한 행동을 막기 위한 규칙입니다. 모든 작업은 반드시 개인 작업 공간인 '투사지(feature 브랜치)'에서 진행해야 합니다.
    - 모든 기능 개발 브랜치는 `feature/[이슈번호]-[간단-설명]` 형식으로 만듭니다.
- **3. 커밋(Commit) 메시지 규칙:** 우리가 작업한 내용을 '작업 일지'에 어떻게 기록할지에 대한 규칙입니다.
    - 모든 커밋 메시지는 'Conventional Commits'라는 표준 양식을 따릅니다. (예: `feat: Add component`)
    - 작업 일지 본문에는, 어떤 GitHub 이슈와 관련된 작업인지 `Closes #[이슈번호]` 형식으로 반드시 명시해야 합니다.
- **4. 풀 리퀘스트(Pull Request) 규칙:** 내 작업물(브랜치)을 '마스터 설계도'에 합쳐달라고 요청하는 '공식 제안서'에 대한 규칙입니다.
    - 모든 코드는 동료의 검토(코드 리뷰)를 받는 PR 과정을 통해서만 main 브랜치에 합쳐질 수 있습니다
```

- [Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/)