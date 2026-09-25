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