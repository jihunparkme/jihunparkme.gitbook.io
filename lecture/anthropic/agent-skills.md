# Introduction to agent skills

[Introduction to agent skills](https://anthropic.skilljar.com/introduction-to-agent-skills/434525)

# What are skills?

🔑 **Key takeaways**

- 스킬은 클로드 코드가 작업을 더욱 정확하게 처리하기 위해 찾아 사용할 수 있는 **지침 폴더**
  - 각 스킬은 이름과 설명이 포함된 `SKILL.md` 파일에 저장
- 클로드는 설명을 활용하여 요청에 맞는 **스킬을 매칭**
  - 사용자가 클로드에게 무언가를 요청하면, 클로드는 사용자의 요청을 사용 가능한 스킬 설명과 비교하여 일치하는 스킬을 활성화
- **개인 스킬**은 한 곳(`~/.claude/skills`)에 저장되어 모든 프로젝트에서 활용 가능
  - 프로젝트 관련 스킬은 `.claude/skills` 저장소에 저장되며, 해당 저장소를 복제하는 모든 사용자와 공유
- **스킬은 필요에 따라 로드**
  - `CLAUDE.md` 파일(모든 대화에 로드)이나 슬래시 명령어(명시적으로 호출)와 달리, 클로드가 상황을 인식하면 스킬이 자동으로 활성화
- 만약 당신이 클로드에게 같은 내용을 반복해서 설명하고 있다면, 그건 당신이 언젠가 글로 써볼 만한 기술일 것이다.

스킬은 클로드에게 어떤 작업을 한 번 수행하는 방법을 가르쳐주는 마크다운 파일입니다. 클로드는 그 지식을 필요할 때마다 자동으로 적용합니다.

## What Skills Are

`스킬`은 클로드 코드가 작업을 더욱 정확하게 처리하기 위해 찾아 사용할 수 있는 **지침 및 리소스 폴더**입니다. 각 스킬은 `SKILL.md`파일 내에 저장되며, 파일 이름 앞부분에 설명이 있습니다.

`설명`은 **클로드가 해당 스킬을 사용할지 여부를 결정하는 기준**입니다. 클로드에게 PR 검토를 요청하면, 클로드는 요청을 사용 가능한 스킬 설명과 비교하여 적절한 설명을 찾습니다. 클로드는 요청을 읽고 모든 스킬 설명과 비교한 후, 일치하는 스킬을 활성화합니다

스킬의 프런트매터

```
---
name: pr-review
description: Reviews pull requests for code quality. Use when reviewing PRs or checking code changes.
---
```

서문 아래에는 실제 지침, 즉 검토 체크리스트, 서식 설정 또는 클로드에게 필요한 모든 정보를 작성합니다.

## Where Skills Live

스킬은 특정 작업에 적용되는 전문 지식에 가장 효과적

- 팀에서 준수하는 코드 검토 표준
- 선호하는 커밋 메시지 형식
- 귀사 브랜드 가이드라인
- 특정 유형의 문서에 대한 문서 템플릿
- 특정 프레임워크용 디버깅 체크리스트

간단한 원칙: 만약 클로드에게 같은 내용을 반복해서 설명하고 있다면, 그것은 스킬로 쓰여져야 할 가능성이 높다.

## Skills vs. CLAUDE.md vs. Slash Commands

클로드 코드는 행동을 사용자 지정하는 여러 가지 방법이 있습니다. 스킬은 자동적이고 작업에 특화되어 있기 때문에 독특합니다. 두 기술을 비교하는 방법은 다음과 같습니다.

- `CLAUDE.md` 파일은 모든 대화에 로드됩니다. 클로드가 항상 TypeScript의 엄격한 모드를 사용하도록 하려면 `CLAUDE.md`로 이동합니다.
- `스킬`은 요청과 일치할 때 필요에 따라 로드됩니다. 클로드는 처음에 이름과 설명만 로드하므로 전체 컨텍스트 창을 채우지 않습니다. 디버깅할 때는 PR 리뷰 체크리스트가 컨텍스트에 있을 필요가 없으며, 실제로 리뷰를 요청할 때 로드됩니다.
- `Slash commands`은 명시적으로 입력해야 합니다. 스킬은 그렇지 않습니다. 클로드는 상황을 인식할 때 이를 적용합니다.

클로드가 사용자의 요청에 맞는 스킬을 찾으면 터미널에 해당 스킬이 표시됩니다.

## When to Use Skills

기술은 특정 작업에 적용되는 전문 지식에 가장 효과적입니다.

- 팀에서 준수하는 코드 검토 표준
- 선호하는 커밋 메시지 형식
- 브랜드 가이드라인
- 특정 유형의 문서에 대한 문서 템플릿
- 특정 프레임워크용 디버깅 체크리스트

간단한 원칙은 이렇습니다. 만약 클로드에게 같은 내용을 반복해서 설명하고 있다면, 그것은 쓰여질 스킬일 가능성이 높습니다.

🎬

# Creating your first skill

## Creating a Skill

클로드가 일관된 형식으로 PR 설명 작성법을 익힐 수 있도록 개인 스킬을 만들어 보자. 이 스킬은 개인 스킬 목록에 저장되어 모든 프로젝트에서 활용할 수 있습니다.

먼저, skills 폴더 안에 스킬용 디렉토리를 생성하세요. 디렉토리 이름은 스킬 이름과 일치해야 합니다.

```shell
mkdir -p ~/.claude/skills/pr-description
```

그다음 해당 디렉토리 안에 `SKILL.md` 파일을 생성합니다. 파일은 대시(-)로 구분된 두 부분으로 구성됩니다.

```md
---
name: pr-description
description: Writes pull request descriptions. Use when creating a PR, writing a PR, or when the user asks to summarize changes for a pull request.
---

When writing a PR description:

1. Run `git diff main...HEAD` to see all changes on this branch
2. Write a description following this format:

## What
One sentence explaining what this PR does.

## Why
Brief context on why this change is needed

## Changes
- Bullet points of specific changes made
- Group related changes together
- Mention any files deleted or renamed
```

이름은 스킬을 식별하는 데 사용됩니다. 설명은 클로드에게 언제 스킬을 사용해야 하는지 알려주는데, 이것이 매칭 기준입니다. 두 번째 대시(-) 쌍 이후의 내용은 스킬이 활성화될 때 클로드가 따르는 지침입니다.