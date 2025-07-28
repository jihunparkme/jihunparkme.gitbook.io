# Prompt evaluation

## Prompt evaluation

클로드와 함께 일할 때 좋은 프롬프트를 작성하는 것은 시작에 불과합니다. 신뢰할 수 있는 AI 애플리케이션을 구축하려면 `prompt engineering`과 `prompt evaluation`라는 두 가지 중요한 개념을 이해해야 합니다. 

- prompt engineering:
  - 더 나은 프롬프트를 작성하는 기술을 제공
- prompt evaluation
  - 이러한 프롬프트가 실제로 얼마나 잘 작동하는지 측정하는 데 도움

## Prompt Engineering vs Prompt Evaluation

<figure><img src="../../.gitbook/assets/claude-with-the-anthropic-api/prompt-evaluation.png" alt=""><figcaption></figcaption></figure>

`prompt engineering`은 효과적인 프롬프트를 만드는 도구로, 다음과 같은 기술이 포함
- 멀티샷 프롬프트
- XML 태그를 사용한 구조화
- 다른 많은 모범 사례

이 기술들은 클로드가 당신이 무엇을 요구하고 어떻게 반응하기를 원하는지 정확히 이해하는 데 도움

`prompt evaluation`는 다른 접근 방식을 취합니다. 프롬프트를 작성하는 방법에 초점을 맞추는 대신 자동화된 테스트를 통해 프롬프트의 효과를 측정
- 예상된 답변에 대한 테스트
- 동일한 프롬프트의 다양한 버전 비교
- 오류가 있는지 출력 검토

.

**Three Paths After Writing a Prompt**

프롬프트를 작성한 후에는 일반적으로 다음에 수행할 작업에 세 가지 옵션이 있습니다:

<figure><img src="../../.gitbook/assets/claude-with-the-anthropic-api/prompt-evaluation-1.png" alt=""><figcaption></figcaption></figure>

- **Option 1**: 프롬프트를 한 번 테스트한 후 충분히 좋다고 판단. 이는 사용자가 예상치 못한 입력을 제공하면 생산에 차질이 생길 위험이 큼.
- **Option 2**: 프롬프트를 몇 번 테스트하고 코너 케이스 한두 개를 처리하도록 조정. 옵션 1보다는 낫지만 사용자가 고려하지 않은 매우 예상치 못한 결과를 제공하는 경우가 많음.
- **Option 3**: 평가 파이프라인을 통해 프롬프트를 실행하여 점수를 매긴 다음 객관적인 지표에 따라 프롬프트를 반복. 이 접근 방식은 더 많은 작업과 비용이 필요하지만 프롬프트의 신뢰성에 대한 신뢰도가 훨씬 높음

.

**Why Most Engineers Fall Into Testing Traps**

옵션 1과 2는 모든 엔지니어가 흔히 접하는 함정. 진지한 애플리케이션에 대한 프롬프트를 작성하고 충분히 테스트하지 않는 것이 당연. 우리는 실제 사용자가 얼마나 많은 엣지 케이스를 마주하게 될지 과소평가하는 경향이 있음.

실제로 프롬프트를 프로덕션에 배포하면 사용자가 예상치 못한 방식으로 프롬프트와 상호 작용할 수 있습니다. 제한된 테스트 중에 견고한 프롬프트처럼 보였던 것은 다양한 실제 입력에 직면하면 빠르게 무너질 수 있습니다.

.

**The Evaluation-First Approach**

옵션 3은 신속한 개발을 위한 보다 체계적인 접근 방식. 평가 파이프라인을 통해 프롬프트를 실행하면 다양한 테스트 사례에서 **프롬프트의 성능에 대한 객관적인 지표**를 획득. 이 데이터 기반 접근 방식을 사용하면 다음과 같은 이점이 존재:
- 생산 문제가 되기 전에 약점 파악하기
- 다양한 프롬프트 버전을 객관적으로 비교하기
- 측정 가능한 개선 사항을 바탕으로 자신 있게 반복
- 더 신뢰할 수 있는 AI 애플리케이션 구축

이 접근 방식은 시간과 테스트 인프라에 더 많은 선불 투자가 필요하지만, 최종 애플리케이션의 신뢰성과 견고성에 대한 배당금을 지급. 목표는 사용자가 문제를 접한 후가 아니라 개발 중에 문제를 파악하는 것.

## A typical eval workflow

일반적인 프롬프트 평가 워크플로우는 객관적인 측정을 통해 프롬프트를 체계적으로 개선하는 데 도움이 되는 다섯 가지 주요 단계를 따름
- 이러한 워크플로우를 구성하는 방법은 다양하며 다양한 오픈 소스 및 유료 도구를 사용할 수 있지만, 핵심 프로세스를 이해하면 작은 규모로 시작하고 필요에 따라 확장 가능

<figure><img src="../../.gitbook/assets/claude-with-the-anthropic-api/A-typical-eval-workflow-1.png" alt=""><figcaption></figcaption></figure>

.

**Step 1: Draft a Prompt**

개선할 초기 프롬프트를 작성하는 것부터 시작. 이 예제에서는 간단한 프롬프트를 사용:

```python
prompt = f"""
Please answer the user's question:

{question}
"""
```

<figure><img src="../../.gitbook/assets/claude-with-the-anthropic-api/A-typical-eval-workflow-2.png" alt=""><figcaption></figcaption></figure>

이 기본 프롬프트는 테스트 및 개선의 기준이 될 것입니다.

.

**Step 2: Create an Eval Dataset**

평가 데이터셋에는 프롬프트가 프로덕션에서 처리할 질문 또는 요청 유형을 나타내는 샘플 입력이 포함. 데이터셋에는 프롬프트 템플릿에 보간될 질문이 포함되어야 합니다.

<figure><img src="../../.gitbook/assets/claude-with-the-anthropic-api/A-typical-eval-workflow-3.png" alt=""><figcaption></figcaption></figure>

예를 들어, 데이터셋에는 세 가지 질문이 포함되어 있습니다:
- 2+2는 무엇인가요?
- 오트밀은 어떻게 만드나요?
- 달은 얼마나 멀리 있나요?

실제 평가에서는 수십, 수백, 심지어 수천 개의 레코드가 있을 수 있습니다. 이러한 데이터 세트를 수작업으로 조립하거나 클로드를 사용하여 생성할 수 있습니다.

.

**Step 3: Feed Through Claude**

데이터셋에서 각 질문을 가져와 프롬프트 템플릿과 병합하여 완전한 프롬프트를 만듭니다. 그런 다음 각 질문을 클로드에게 보내 응답을 얻습니다.

<figure><img src="../../.gitbook/assets/claude-with-the-anthropic-api/A-typical-eval-workflow-4.png" alt=""><figcaption></figcaption></figure>

예를 들어, 첫 번째 질문은 다음과 같습니다:

```python
Please answer the user's question:
What's 2+2?
```

클로드는 수학 문제에 대해 "2 + 2 = 4"로 대답하고, 두 번째 질문에는 오트밀 요리 지침을 제공하며, 세 번째 질문에는 달까지의 거리를 제공할 수 있습니다.

.

**Step 4: Feed Through a Grader**

채점자는 원래 질문과 클로드의 답변을 모두 검토하여 클로드의 답변의 품질을 평가합니다. 이 단계는 일반적으로 1에서 10까지의 척도로 객관적인 점수를 제공하며, 10점은 완벽한 답변을 나타내고 점수가 낮을수록 개선의 여지가 있음을 나타냅니다.

<figure><img src="../../.gitbook/assets/claude-with-the-anthropic-api/A-typical-eval-workflow-5.png" alt=""><figcaption></figcaption></figure>

예시에서, 채점자는 다음을 할당할 수 있습니다:
- 수학 문제: 10 (완벽한 답)
- 오트밀 질문: 4 (개선 필요)
- 문 질문: 9 (매우 좋은 답변)

모든 질문의 평균 점수는 객관적인 측정을 제공합니다: (10 + 4 + 9) ÷ 3 = 7.66