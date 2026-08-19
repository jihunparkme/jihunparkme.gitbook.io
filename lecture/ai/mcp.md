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

