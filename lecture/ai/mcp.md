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