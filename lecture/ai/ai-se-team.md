# AI, 24시간 일하는 시스템 엔지니어

# AI 코드 리뷰어

1️⃣. **Claude + GitHub Actions**

콜르드 세션 내 Github App 추가

```bash
/install-github-app 
```

2️⃣. **CodeRabbit**

[coderabbitai](https://github.com/marketplace/coderabbitai)

세부 동작 제어

```yaml
# .coderabbit.yaml
reviews:
  summarize: true
  review_simple_changes: false
  path_filters:
    - "!**/*.md"
    - "!**/*.txt"
    - "!**/package-lock.json"
  auto_approve:
    enabled: false
chat:
  enabled: true
language: korean
```

# 🏗️ '인프라를 코드로' 자동 관리하기: Infra as a code

> [Terraform](https://developer.hashicorp.com/terraform)
>
> AWS 인프라를 코드로 자동 배포할 수 있는 도구

**Terraform**
- AWS 설정을 코드로 한 번만 작성하면, 같은 환경을 계속해서 찍어낼 수 있음
- 클릭할 필요 없이 한 줄의 명령어로 모든 설정이 자동으로 적용
- 필요한 부분만 수정해서 쉽게 변경 가능

인프라 구축 과정
1. **IAM**(Identity and Access Management)를 통해 AWS 엑세스 키 생성
2. **AWS CLI**(AWS Command Line Interface)에 엑세스 키 설정
   - [Install](https://brew.sh/)
3. **AWS CLI**에 AWS 계정과 연결
   ```bash
   $ aws configure
   
    `AWS Access Key ID` 입력 (IAM User 생성 시 받은 값)
    `AWS Secret Access Key` 입력 (IAM User 생성 시 받은 값)
    `Default region name` 입력 (`ap-northeast-2` → 서울 리전)
    `Default output format` 입력 (그냥 Enter)

   $ aws ec2 describe-instances # ec2 인스턴스를 모두 출력
   ```

# 👁️ '시스템의 눈': Sentry MCP 연동

> [Sentry](https://sentry.io/welcome/)
>
> 에러 트래킹'과 ‘성능 모니터링'에 특화된 도구

Sentry MCP

```bash
claude mcp add --transport http sentry https://mcp.sentry.dev/mcp
```

# 🧠 AI '두뇌'를 들여다보기: Langfuse 활용 LLM 성능 및 품질 모니터링

> [Langfuse](https://langfuse.com/)
>
>  사용자의 첫 질문부터 AI의 최종 답변까지 이어지는 모든 작업의 흐름을 하나로 묶어 기록하는 도구

**Trace가 기록하는 것:**

- 사용자가 입력한 원본 프롬프트
- 우리가 코드에서 가공한 최종 프롬프트 템플릿
- LLM을 호출하는 데 걸린 시간 (Latency)
- 사용된 입/출력 토큰의 양
- **API 호출에 들어간 정확한 비용 ($)**
- 최종적으로 생성된 답변
- 사용자의 피드백 점수 (좋아요/싫어요)

우리가 AI의 모든 행동을 속속들이 분석하고 최적화할 수 있도록 지원