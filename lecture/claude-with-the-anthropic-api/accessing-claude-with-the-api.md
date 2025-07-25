# Anthropic overview

## Overview of Claude models

<figure><img src="../../.gitbook/assets/claude-with-the-anthropic-api/models.png" alt=""><figcaption></figcaption></figure>

세 모델 모두 Claude의 핵심 기능을 공유
- 텍스트 생성, 코딩, 이미지 분석 및 기타 여러 작업을 처리
- 실제적 차이는 최적화 방식
  - 지능 중점
  - 속도와 비용 효율성
  - 지능과 속도 사이의 균형

### Opus

Cluade의 가장 뛰어난 모델
- 가장 높은 수준의 지능을 제공
- 몇 시간 동안 지속될 수 있는 복잡한 프로젝트를 독립적으로 수행
- 추론을 지원(간단한 작업에 빠른 응답, 더 복잡한 작업을 위해 시간을 할애)
- 단점: 중간 정도의 지연 시간과 더 높은 비용
  - 매우 높은 지능을 얻는 대신, 요청마다 약간의 추가 시간과 비용이 부과

### Sonnet

Cluade의 스위트 스팟 모델
- 대부분 실용적인 사용 사례에 매우 유용하게 만드는 지능, 속도 및 비용의 좋은 균형
- 강력한 코딩 능력과 빠른 텍스트 생성

### Haiku

Cluade에서 가장 빠른 모델
- 응답 시간이 매우 중요한 애플리케이션을 위해 특별히 제작
- Opus와 Sonnet이 가진 추론 기능을 지원하지 않음
- 대신, 속도와 비용 효율성에 최적화
- 실시간 상호 작용이 필요한 사용자 대면 앱에 매우 좋은 선택

<figure><img src="../../.gitbook/assets/claude-with-the-anthropic-api/pick-model.png" alt=""><figcaption></figcaption></figure>

# Accessing Claude with the API

## Accessing the API

사용자가 채팅 인터페이스에서 "보내기"를 클릭하는 순간부터 Claude의 응답이 화면에 나타날 때까지 어떤 일이 일어나는지의 과정

**5단계 요청 흐름**

Claude와의 모든 상호작용은 **서버에 대한 요청**, **Anthropic API에 대한 요청**, **모델 처리**, **서버에 대한 응답**, **클라이언트에 대한 응답**이라는 5가지 단계로 구성된 예측 가능한 패턴을 따릅니다.


### 서버에 대한 요청

<figure><img src="../../.gitbook/assets/claude-with-the-anthropic-api/api-step-1.png" alt=""><figcaption></figcaption></figure>

**서버가 필요한 이유**  
클라이언트 측 코드에서 Anthropic API에 직접 요청을 보내면 안 되므로 서버가 필요
- API 요청에는 인증을 위한 비밀 API 키가 필요
- 클라이언트 코드에서 이 키를 노출하면 심각한 보안 취약점이 발생
- 누구든지 키를 추출하고 허가 없이 요청 가능

대신, 서버에 요청을 보내고, 해당 서버는 안전하게 저장된 키를 사용하여 Anthropic API와 통신

### Anthropic API에 대한 요청

### 모델 처리

### 서버에 대한 응답

### 클라이언트에 대한 응답










**서버가 필요한 이유**
- 클라이언트 측 코드에서 Anthropic API에 직접 요청을 보내면 안 됩니다. 이유는 다음과 같습니다.

API 요청에는 인증을 위한 비밀 API 키가 필요합니다.
클라이언트 코드에서 이 키를 노출하면 심각한 보안 취약점이 발생합니다.
누구든지 키를 추출하고 허가 없이 요청할 수 있습니다.
대신, 귀하의 웹이나 모바일 앱은 귀하의 서버에 요청을 보내고, 해당 서버는 안전하게 저장된 키를 사용하여 Anthropic API와 통신합니다.