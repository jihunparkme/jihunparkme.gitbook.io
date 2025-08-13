# AI agent

[회사에서 바로 쓰는 업무자동화 AI 에이전트 (w. n8n, LangGraph)](https://www.inflearn.com/course/%ED%9A%8C%EC%82%AC%EC%97%90%EC%84%9C-%EB%B0%94%EB%A1%9C%EC%93%B0%EB%8A%94-%EC%97%85%EB%AC%B4%EC%9E%90%EB%8F%99%ED%99%94-ai%EC%97%90%EC%9D%B4%EC%A0%84%ED%8A%B8) 강의를 요약한 내용입니다.

## n8n

[n8n](https://n8n.io/)

> Flexible AI workflow automation
>
> 다양한 웹 서비스, 앱, API들을 연결하여 자동화 워크플로우를 만드는 오픈소스 통합 플랫폼

n8n은 워크플로우를 구성하는 노드(Node)들로 이루어져 있으며, 각 노드는 특정 작업을 수행합니다.

👉🏻 **n8n 컨테이너 구동**

```bash
docker volume create n8n_data

docker run -d -it --rm --name n8n -p 5678:5678 -v n8n_data:/home/node/.n8n docker.n8n.io/n8nio/n8n
```

## Email Agent

### 👉🏻 Gmail Actions

**Gmail API 사용하기**

- [GCP console](https://cloud.google.com/cloud-console) → API 및 서비스 → API 및 서비스 사용 설정 → Gmail API -> 활성화
- 사용자 인증 정보 만들기 → OAuth 클라이언트 ID 만들기 

Gmail API로 이용할 수 있는 다양한 action, trigger 기능을 연동 가능

<figure><img src="../.gitbook/assets/ai-agent/gmail.png" alt=""><figcaption></figcaption></figure>

### 👉🏻 Basic LLM Chain

#### 답장 여부 판단

**모델 연결 → Prompt →**

```text
"=아래 이메일 내용을 보고, 답장이 필요한 이메일인지 판단해주세요. 만약 당신의 오판으로 제가 이메일에 답장을 하지 못하게되면 업무상 차질이 생길 수 있으니 주의해주세요.\n\n1. 업무 협업 메일에는 가급적이면 답장을 해야합니다. \n2. 마케팅이나 뉴스레터의 경우에는 답장을 할 필요가 없습니다.\n\n이메일 내용:\n {{ $json.text }}",
```

**Chat Messages**

```text
"당신의 업무는 이메일에 답장을 해야하는지 말아야하는지 결정하는 것입니다. 답장이 필요하다면 true, 답장이 필요 없다면 false를 리턴해주세요"
```

**Require Specific Output Format**

```json
{
	"type": "object",
	"properties": {
		"need_reply": {
			"type": "boolean"
		}
	}
}
```

#### 이메일 답장 작성

**Prompt**

```text
"=아래 이메일을 보고 답장을 작성해주세요\n\n최근 메일:\n{{ $('Gmail Trigger').item.json.text }}\n\n이메일 히스토리:\n{{ $json.messages.filter(item => item.snippet) }}",
```

**Require Specific Output Format**

```json
{
	"type": "object",
	"properties": {
		"title": {
			"type": "string"
		},
        "content": {
			"type": "string"
		}
	}
}
```

<figure><img src="../.gitbook/assets/ai-agent/sample-1.png" alt=""><figcaption></figcaption></figure>

#### 이메일 작성 봇

**System Messages**

```text
"당신은 이메일 작성 도우미입니다. 사용자의 질문을 바탕으로, 사용자에게 필요한 이메일을 작성해주세요 \n\n당신이 이메일을 작성하는데 필요한 모든 정보를 얻을 때까지 사용자에게 질문을 한 후에, 필요한 정보를 모두 얻고 나서 이메일을 작성해주세요"
```

## News Agent (Brave Search API)

### 👉🏻 **Schedule Trigger**

워크플로우를 특정 시간에 자동으로 시작시키는 노드
- 지정된 시간 간격(예: 매일 오전 9시, 매주 월요일, 15분마다 등)에 맞춰 다음 노드로 데이터를 전달하며 워크플로우를 실행
- America/New_York Timezone 이므로 한국 시간대 계산이 필요

### 👉🏻 **Brave Search**

Brave Search 엔진을 이용해 웹 검색을 수행하고, 그 결과를 n8n 워크플로우로 가져오는 노드
- 검색 결과를 JSON 형태로 받아와서 다른 노드에서 활용
- [Brave Search API](https://brave.com/search/api/)

### 👉🏻 **Code**

워크플로우 내에서 JavaScript 코드를 직접 실행할 수 있는 노드
- n8n의 기본 노드만으로는 해결하기 어려운 복잡한 로직을 구현하거나, 데이터를 특정 형태로 가공해야 할 때 유용
- n8n의 모든 노드에서 전달받은 데이터를 코드 노드에서 처리 가능
	
	```javascript
	return $input.first().json.results
	```

### 👉🏻 **Loop Over Items**

워크플로우가 배열(리스트) 형태의 데이터 아이템들을 하나씩 순회하며 반복 작업을 수행하도록 하는 노드
- 각 아이템에 대해 동일한 일련의 작업을 적용 가능
- Batch Size 지정

### 👉🏻 **Basic LLM Chain**

LangChainJS 라이브러리를 기반으로 LLM과 상호작용하는 노드
- 복잡한 코딩 없이도 프롬프트 템플릿과 LLM을 연결하여 텍스트를 생성하거나, 질문-답변 기능을 워크플로우에 통합 가능

**Prompt**

```text
"Categorize below news article by reading the description into \"positive,\" \"negative,\", \"neutral\":\n\nNews Article:\nTitle: {{ $json.title }}\nDescription: {{ $json.description }}\nURL: {{ $json.url }}",
```

**Chat Messages**

```text
"Your job is to analyze the sentiment of a news article from a user every morning and report to the executives and share with my colleagues\n\nAfter analyzing the sentiment, use the Google Sheet Tool provided to you to insert the analysis into the spreadsheet for other colleagues to see"
```

**Require Specific Output Format**

```json
{
	"type": "object",
	"properties": {
		"sentiment": {
			"type": "string"
		},
      "title": {
			"type": "string"
		},
      "description": {
			"type": "string"
		},
      "url": {
			"type": "string"
		}
	}
}
```

### 👉🏻 **Append row in sheet**

Google Sheets, Excel 등 스프레드시트 서비스에 새로운 행을 추가하는 노드
- 워크플로우에서 처리된 데이터를 최종적으로 스프레드시트에 정리할 때 사용

**Google Drive & Google Sheets API**

- [GCP console](https://cloud.google.com/cloud-console) → API 및 서비스 → API 및 서비스 사용 설정
  - Google Drive API
  - Google Sheets API
- 사용자 인증 정보 만들기 → OAuth 클라이언트 ID 만들기 (기존 ID 사용 가능)
- 시트에 헤더 추가 → Document & Sheet 선택 → 헤더 정보 입력

<figure><img src="../.gitbook/assets/ai-agent/sheets.png" alt=""><figcaption></figcaption></figure>

### Result

<figure><img src="../.gitbook/assets/ai-agent/news-sample.png" alt=""><figcaption></figcaption></figure>

## News Agent (Naver HTTP Request)

### 👉🏻 HTTP Request

### 👉🏻 HTML

### 👉🏻 Filter

### 👉🏻 Notion

**Notion Credential**

[notion integration](https://developers.notion.com/docs/create-a-notion-integration) → 
View my integrations → New API integration → Notion 옵션 → 연결 → API integration 연결

### Result

<figure><img src="../.gitbook/assets/ai-agent/naver-news.png" alt=""><figcaption></figcaption></figure>




