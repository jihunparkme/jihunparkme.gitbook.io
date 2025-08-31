# Prompt engineering techniques

## Prompt enginerring

프롬프트 엔지니어링은 작성한 프롬프트를 가져와 더 신뢰할 수 있고 고품질의 출력을 얻기 위해 개선하는 것입니다. 이 프로세스는 기본 프롬프트로 시작하여 **성능을 평가한 다음 엔지니어링 기법을 체계적으로 적용하여 개선**하는 반복적인 정제 과정을 포함합니다.

<figure><img src="../../.gitbook/assets/claude-with-the-anthropic-api/prompt-enginerring.png" alt=""><figcaption></figcaption></figure>

### The Iterative Improvement Process

이 접근 방식은 원하는 결과를 얻을 때까지 반복할 수 있는 명확한 주기를 따릅니다:

1. **목표 설정** - 프롬프트에서 달성하고자 하는 목표 정의

2. **초기 프롬프트 작성** - 기본 첫 시도 만들기

3. **프롬프트 평가** - 기준에 맞게 테스트합

4. **신속한 엔지니어링 기법 적용** - 성능 향상을 위해 특정 방법 사용

5. **재평가** - 변경 사항이 실제로 결과를 개선했는지 확인

성능에 만족할 때까지 마지막 두 단계(4-5)를 반복합니다. 각 반복마다 평가 점수가 눈에 띄게 향상되어야 합니다.

### Setting Up Your Evaluation Pipeline

이 과정을 시연하기 위해 운동선수를 위한 일일 식단을 생성하는 프롬프트를 만드는 실용적인 예를 들어보겠습니다. 프롬프트는 운동선수의 키, 체중, 목표 및 식이 제한을 고려한 다음 종합적인 식단을 작성해야 합니다.

<figure><img src="../../.gitbook/assets/claude-with-the-anthropic-api/setting-up-your-evaluation-pipeline.png" alt=""><figcaption></figcaption></figure>

평가 설정은 **데이터셋 생성** 및 **모델 등급을 처리**하는 `PromptEvaluator` 클래스를 사용합니다. 평가자 인스턴스를 생성할 때 `max_concurrent_tasks` 매개변수와의 동시성을 제어할 수 있습니다:

```python
evaluator = PromptEvaluator(max_concurrent_tasks=5)
```

속도 제한 오류를 방지하려면 낮은 동시성 값(like 3)부터 시작하세요. API 할당량이 더 빠른 처리를 가능하게 한다면 이 값을 늘릴 수 있습니다.

### Generating Test Data

평가 시스템은 프롬프트 요구 사항에 따라 테스트 케이스를 자동으로 생성할 수 있습니다. 프롬프트에 필요한 입력을 정의합니다:

```python
dataset = evaluator.generate_dataset(
    task_description="Write a compact, concise 1 day meal plan for a single athlete",
    prompt_inputs_spec={
        "height": "Athlete's height in cm",
        "weight": "Athlete's weight in kg", 
        "goal": "Goal of the athlete",
        "restrictions": "Dietary restrictions of the athlete"
    },
    output_file="dataset.json",
    num_cases=3
)
```

개발 중 테스트 케이스 수를 낮게 유지(2~3개)하여 반복 주기를 단축하세요. 최종 검증을 위해 이를 늘릴 수 있습니다.

### Writing Your Initial Prompt

간단하고 순진한 프롬프트로 시작하여 기준선을 설정하세요. 의도적으로 기본적인 첫 번째 시도의 예는 다음과 같습니다:

```json
def run_prompt(prompt_inputs):
    prompt = f"""
What should this person eat?

- Height: {prompt_inputs["height"]}
- Weight: {prompt_inputs["weight"]}
- Goal: {prompt_inputs["goal"]}
- Dietary restrictions: {prompt_inputs["restrictions"]}
"""
    
    messages = []
    add_user_message(messages, prompt)
    return chat(messages)
```

이 기본 프롬프트는 좋지 않은 결과를 초래할 가능성이 높지만, 개선을 측정할 수 있는 출발점을 제공합니다.

### Adding Evaluation Criteria

평가를 실행할 때 채점 모델이 고려해야 할 추가 기준을 지정할 수 있습니다:

```json
results = evaluator.run_evaluation(
    run_prompt_function=run_prompt,
    dataset_file="dataset.json",
    extra_criteria="""
The output should include:
- Daily caloric total
- Macronutrient breakdown  
- Meals with exact foods, portions, and timing
"""
)
```

이는 사용 사례에 중요한 특정 요구 사항에 대해 프롬프트를 평가하는 데 도움이 됩니다.

### Analyzing Results

평가를 실행하면 숫자 점수와 자세한 HTML 보고서를 모두 얻을 수 있습니다. 이 보고서는 각 점수에 대한 모델의 추론을 포함하여 각 테스트 사례가 정확히 어떻게 수행되었는지 보여줍니다.

초기 점수가 낮다고 낙담하지 마세요. 첫 번째 시도에서는 10점 만점에 2.3점이 일반적입니다. 엔지니어링 기법을 적용하면 일관된 개선을 볼 수 있습니다.

자세한 평가 보고서는 프롬프트가 실패하고 있는 부분과 개선이 필요한 부분을 정확히 이해하는 데 도움이 됩니다. 이 피드백을 사용하여 다음 반복을 안내하세요.

### Nest Steps

기준선이 설정되면 특정 프롬프트 엔지니어링 기법을 적용할 준비가 됩니다. 각 기법을 배울 때마다 평가 점수가 눈에 띄게 향상되어 기본 프롬프트가 점차 신뢰할 수 있고 고성능의 도구로 전환될 것입니다.

**신속한 엔지니어링은 반복적인 프로세스**라는 점을 기억하세요. 핵심은 한 번에 하나씩 변경하고 영향을 평가하며 효과가 있는 기술을 기반으로 구축하는 것입니다. 이러한 체계적인 접근 방식을 통해 특정 사용 사례에 가장 큰 가치를 제공하는 기술이 무엇인지 이해할 수 있습니다.

## Being clear and direct

프롬프트의 첫 번째 줄은 전체 요청에서 가장 중요한 부분입니다. 여기서 다음에 나오는 모든 것의 무대를 설정하고 올바르게 설정하면 결과가 극적으로 향상될 수 있습니다.

### Being Clear and Direct

이 중요한 첫 번째 줄을 만들 때는 `명확성(clarity)`과 `직접성(directness)`이라는 두 가지 핵심 원칙에 집중해야 합니다. 즉, 클로드가 원하는 작업에 대해 모호함의 여지를 남기지 않는 간단한 언어를 사용해야 합니다.

### Clear Communication

명확하다는 것의 의미
- 누구나 이해할 수 있는 **간단한 언어** 사용하기
- 덤불 주위를 돌지 않고 **원하는 것을 정확히** 말하
- 클로드의 과제에 대한 **간단한 설명**으로 리드하기

 모호한 말 대신 직설적인 말 사용하기
 - AS-SI: "사람들이 지붕에 태양열을 사용하는 것들에 대해 알아야 해요 - 태양열 패널이라고 부르는 것들 - 에 대해 알고 싶어요"
 - TO-BE: "태양열 패널이 어떻게 작동하는지에 대해 세 단락을 써보세요."

### Direct Instructions

"직접적"이라는 것은 요청을 구성하는 방식에 중점
- 질문이 아닌 **지침 사용**
- "Write", "Create" 또는 "Generate"과 같은 직접 작용 동사로 시작하기

지침을 사용하기
- AS-IS: "재생 에너지와 지열 에너지에 대해 읽고 있었어요. 어떤 나라에서 지열 에너지를 사용하나요?"
- TO-BE: "지열 에너지를 사용하는 세 나라를 식별하세요. 각각에 대한 발전 통계를 포함하세요."