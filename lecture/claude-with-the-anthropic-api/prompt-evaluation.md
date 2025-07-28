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