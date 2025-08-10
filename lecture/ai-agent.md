# AI agent

[회사에서 바로 쓰는 업무자동화 AI 에이전트 (w. n8n, LangGraph)](https://www.inflearn.com/course/%ED%9A%8C%EC%82%AC%EC%97%90%EC%84%9C-%EB%B0%94%EB%A1%9C%EC%93%B0%EB%8A%94-%EC%97%85%EB%AC%B4%EC%9E%90%EB%8F%99%ED%99%94-ai%EC%97%90%EC%9D%B4%EC%A0%84%ED%8A%B8) 강의를 요약한 내용입니다.

## n8n

[n8n](https://n8n.io/)

> Flexible AI workflow automation
>
> 다양한 웹 서비스, 앱, API들을 연결하여 자동화 워크플로우를 만드는 오픈소스 통합 플랫폼

n8n 컨테이너 구동

```bash
docker volume create n8n_data

docker run -d -it --rm --name n8n -p 5678:5678 -v n8n_data:/home/node/.n8n docker.n8n.io/n8nio/n8n
```

## Email Agent

### Gmail Actions

> On message received

**Gmail API 활성화**

- [GCP console](https://cloud.google.com/cloud-console) → API 및 서비스 → API 및 서비스 사용 설정 → Gmail API
- 사용자 인증 정보 만들기 → OAuth 클라이언트 ID 만들기 

Gmail API로 이용할 수 있는 다양한 action, trigger 기능을 연동 가능

<figure><img src="../.gitbook/assets/ai-agent/gmail.png" alt=""><figcaption></figcaption></figure>

### Basic LLM Chain

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

## News Agent

#### 👉🏻 **Schedule Trigger**

#### 👉🏻 **Brave Search**

#### 👉🏻 **Code**

#### 👉🏻 **Loop Over Items**

#### 👉🏻 **Basic LLM Chain**

**Prompt**

```text
"=Categorize below news article by reading the description into \"positive,\" \"negative,\", \"neutral\":\n\nNews Article:\nTitle: {{ $json.title }}\nDescription: {{ $json.description }}\nURL: {{ $json.url }}",
```

**Chat Messages**

```text
"Your job is to analyze the sentiment of a news article from a user every morning and report to the executives and share with my colleagues\n\nAfter analyzing the sentiment, use the Google Sheet Tool provided to you to insert the analysis into the spreadsheet for other colleagues to see"
```

#### 👉🏻 **Append row in sheet**

시트에 헤더 추가

**Google Sheets API**

- [GCP console](https://cloud.google.com/cloud-console) → API 및 서비스 → API 및 서비스 사용 설정 → Gmail API
- 사용자 인증 정보 만들기 → OAuth 클라이언트 ID 만들기 

#### Result

<figure><img src="../.gitbook/assets/ai-agent/news-sample.png" alt=""><figcaption></figcaption></figure>

