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

- Brave News 추출 (Brave Search)
- 추출된 정보를 JSON 형태로 변환 (Code)
- Loop Over Items
- 뉴스 감정 분석 (Basic LLM Chain)
- 결과를 시트에 추가 (Sheets)

<figure><img src="../.gitbook/assets/ai-agent/news-sample-result.png" alt=""><figcaption></figcaption></figure>

## News Agent (Naver HTTP Request)

### 👉🏻 HTTP Request

웹에서 데이터를 가져오거나 보내는 가장 기본적인 노드
- 특정 URL로 HTTP 요청(GET, POST, PUT, DELETE 등)을 보내고, 그 응답을 받아 워크플로우에 통합
- API와의 상호작용은 대부분 이 노드를 통해 이루어짐

### 👉🏻 HTML

HTML 문서에서 특정 데이터를 추출하는 데 사용
- 웹 스크래핑(Web Scraping)의 핵심 노드
- CSS 셀렉터(Selector)를 이용해 원하는 태그나 클래스, 아이디를 가진 요소를 선택하고 그 안의 텍스트나 속성 값을 추출

### 👉🏻 Filter

데이터를 특정 조건에 따라 걸러내는 노드
- 입력 데이터가 설정된 조건을 만족하는지 확인하여, 조건을 만족하는 데이터만 다음 노드로 전달
- 조건은 등호, 부등호, 포함 여부, 정규식 등 다양하게 설정

### 👉🏻 Notion

Notion 데이터베이스 또는 페이지와 상호작용하는 노드
- Notion 계정과 연결하여 데이터베이스에 새로운 항목을 추가하거나, 기존 페이지의 내용을 업데이트하거나, 특정 데이터를 검색하는 등의 작업을 수행

**Notion Credential**

[notion integration](https://developers.notion.com/docs/create-a-notion-integration) → 
View my integrations → New API integration → Notion 옵션 → 연결 → API integration 연결

### Result

<figure><img src="../.gitbook/assets/ai-agent/naver-news.png" alt=""><figcaption></figcaption></figure>

- 기사 목록 요청 (HTTP Request)
- Loop Over Items
- 기사 제목 & URL 추출 (HTML)
- 추출된 정보를 JSON 형태로 변환 (Code)
- Loop Over Items
- 기사 URL 요청 (HTTP Request)
- 기사 내용 추출 (HTML)
- 연관성 파악 (Basic LLM Chain)
- 연관성이 있는지 필터링 (Filter)
- 뉴스 감정 분석 (Basic LLM Chain)
- 노션 데이터베이스에 추가 (Notion)

<figure><img src="../.gitbook/assets/ai-agent/naver-news-result.png" alt=""><figcaption></figcaption></figure>

## n8n을 활용한 사내 QnA Bot

### 데이터 저장

#### 👉🏻 Get rows in sheet

Google Sheets, Excel 등 스프레드시트에서 특정 행들을 읽어오는 노드
- 필터링 조건을 설정하여 특정 행만 선택적으로 가져올 수 있으며, 가져온 데이터는 워크플로우의 다음 노드로 전달되어 가공되거나 활용
- ex) 파일 링크 목록 가져오기

#### 👉🏻 Download file

특정 URL에서 파일을 다운로드하여 워크플로우로 가져오는 노드
- 다운로드된 파일은 다음 노드에서 처리할 수 있는 형태로 변환
- ex) 추출된 파일 링크를 읽고 해당 파일을 다운로드

#### 👉🏻 Pinecone Vector Store

전문 벡터 데이터베이스에 데이터를 저장하거나 검색
- LLM 애플리케이션에서 방대한 양의 비정형 데이터를 효율적으로 검색하고 관리하는 데 사용
- [create API Key](https://app.pinecone.io/)

**Gemini**
- Pinecone Vector Store 노드와 함께 사용되는 Gemini 노드는 Google의 Gemini AI 모델을 사용하여 텍스트 데이터를 벡터로 변환하는 역할을
- 벡터화된 데이터는 Pinecone 데이터베이스에 저장되어 유사도 검색에 사용

**Default Data Loader**
- 다양한 소스(웹사이트, PDF, CSV 파일 등)에서 데이터를 불러와서 처리 가능한 형태로 변환하는 역할
- 이 노드는 특히 LLM 워크플로우에서 외부 데이터를 모델에 입력하기 전에 전처리하는 데 사용
- Recursive Character Text Splitter
  - 길고 복잡한 텍스트를 LLM이 한 번에 처리하기 적합한 크기의 '덩어리(chunk)'로 나누는 역할
  - 특정 문자(예: '\n\n', '\n', '.' 등)를 기준으로 텍스트를 재귀적으로 분할하여, 의미 있는 단위가 손상되지 않도록 함

<figure><img src="../.gitbook/assets/ai-agent/QnA-bot.png" alt=""><figcaption></figcaption></figure>

### Bot using LLM Chain

#### 👉🏻 When Chat message received
- 텔레그램, 슬랙 등 특정 메신저 채널에서 새로운 메시지가 수신되었을 때 워크플로우를 자동으로 시작하는 트리거 노드
- 채팅 기반의 자동화 워크플로우를 구축하는 데 필수적

#### 👉🏻 Basic LLM Chain
- 복잡한 문서나 데이터 소스를 바탕으로 질문에 대한 답변을 생성하는 고급 노드
- 사용자의 질문을 입력으로 받아, 연결된 데이터 소스에서 관련 정보를 찾아내고, 이를 바탕으로 LLM이 자연어 답변을 생성
- RAG(Retrieval Augmented Generation) 기술을 구현하는 데 사용

**Vector Store Retriever**
- 질문과 관련된 정보를 벡터 데이터베이스에서 효율적으로 검색하는 역할
- 사용자의 질문을 벡터로 변환한 후, 이 벡터와 유사한 의미를 가진 기존 문서 벡터들을 데이터베이스에서 탐색
- 이 노드를 통해 LLM은 방대한 데이터 속에서 필요한 정보만 '검색'

**Pinecone Vector Store**
- 벡터 데이터베이스의 한 종류로, Vector Store Retriever 노드와 함께 사용
- 이곳에 저장된 문서 벡터들을 기반으로 검색
- LLM이 학습하지 않은 최신 데이터나 전문 문서에 대한 질의응답 기능을 구현할 때 주로 활용

**Embeddings Google Gemini**
- 텍스트 데이터를 구글의 Gemini AI 모델을 사용하여 벡터로 변환하는 역할
- 질문과 문서 모두 이 노드를 거쳐 벡터화되며, 이 벡터는 Vector Store Retriever 노드를 통해 Pinecone 데이터베이스에 저장되거나 검색에 사용

<figure><img src="../.gitbook/assets/ai-agent/QnA-bot-ai-chain.png" alt=""><figcaption></figcaption></figure>

### Bot using AI Agent

#### 👉🏻 AI Agent

LLM을 이용해 복잡한 작업을 스스로 계획하고 실행하게 하는 고급 노드
- 단순히 텍스트를 생성하는 것을 넘어, 여러 도구를 조합하고 순차적으로 실행하여 목표를 달성

System Message

```text
You are a helpful assistant. Use the tools that are available to you in order to answer the user's question
```

**Pinecone Vector Store tool**
- AI Agent가 Pinecone 벡터 데이터베이스를 '도구'로 사용
- AI Agent는 이 도구를 이용해 데이터베이스에서 특정 정보를 검색하거나, 새로운 데이터를 저장하는 등의 작업을 워크플로우 내에서 수행

tool Description

```text
The documents within this knowledge base contains information about company's policy such as HR, IT Support, and so on
```

<figure><img src="../.gitbook/assets/ai-agent/QnA-bot-ai-agent.png" alt=""><figcaption></figcaption></figure>

## Text-to-SQL

**Text-to-SQL**
- 자연어(영어나 한국어)를 SQL로 변환하는 기술
- AI를 통한 데이터베이스 접근

#### 👉🏻 When chat message received

#### 👉🏻 테이블 이름 가져오기

**Postgres Execute a SQL query**

```sql
SELECT 
    col.table_name,
    COALESCE(obj_description(c.oid), 'No description available') AS table_description,
    string_agg(col.column_name, ', ' ORDER BY col.ordinal_position) AS all_columns
FROM information_schema.columns col
LEFT JOIN pg_class c ON c.relname = col.table_name
LEFT JOIN pg_namespace n ON n.oid = c.relnamespace
WHERE col.table_schema = 'public'
AND n.nspname = 'public'
GROUP BY col.table_name, c.oid
ORDER BY col.table_name;
```

#### 👉🏻 테이블 정보 합치기

**Code**

```javascript
let schema = '';

for (const item of $input.all()) {
  schema += `tableName: ${item.json.table_name}
tableDescription: ${item.json.table_description}
columnList: ${item.json.all_columns}`;
}

return [{schema}];
```

#### 👉🏻 테이블 리스트 추출

**Basic LLM Chain**

Prompt

```text
{{ $('When chat message received').first().json.chatInput }}
```

System Message

```text
TABLE DESCRIPTION:
{{ $json.schema }}


By looking at the table description above, which contains information about the name of tables and their relative descriptions along with the list of columns, return the list of names of the tables that you need to access in order to retrieve data related to the user's question
```

structured Output Parser

```json
{
	"type": "object",
	"properties": {
		"tables": {
			"type": "array",
			"table": {
				"type": "string"
			}
		}
	}
}
```

#### 👉🏻 스키마 조회

**Postgres Execute a SQL query**

```sql
SELECT
    'CREATE TABLE ' || table_name || ' (' || E'\n' ||
    -- 각 컬럼에 대한 정보를 조합합니다.
    string_agg(
        '    ' || column_name || ' ' || 
        CASE 
            WHEN data_type = 'character varying' THEN 'VARCHAR(' || character_maximum_length || ')'
            WHEN data_type = 'integer' AND column_default LIKE 'nextval%' THEN 'SERIAL PRIMARY KEY'
            WHEN data_type = 'numeric' THEN 'DECIMAL(' || numeric_precision || ',' || numeric_scale || ')'
            WHEN data_type = 'timestamp without time zone' THEN 'TIMESTAMP'
            WHEN data_type = 'boolean' THEN 'BOOLEAN'
            ELSE UPPER(data_type)
        END ||
        CASE WHEN is_nullable = 'NO' AND column_default NOT LIKE 'nextval%' THEN ' NOT NULL' ELSE '' END ||
        CASE WHEN column_default IS NOT NULL AND column_default NOT LIKE 'nextval%' 
             THEN ' DEFAULT ' || column_default ELSE '' END,
        E',\n' ORDER BY ordinal_position
    ) || E'\n' || ');' || E'\n\n' AS create_statement
FROM 
    information_schema.columns 
WHERE 
    table_schema = 'public'
AND TABLE_NAME IN ({{  $json.output.tables.map((tableName) => `'${tableName}'`) }})
GROUP BY 
    table_name
ORDER BY 
    table_name;
```

#### 👉🏻 스키마 정보 합치기

**Code**

```javascript
let schema = ''
for (const item of $input.all()) {
  schema += `${item.json.create_statement}\n\n`
}

return [{schema}]
```

#### 👉🏻 쿼리 생성

**Basic LLM Chain**

Prompt

```text
{{ $('When chat message received').first().json.chatInput }}
```

System message

```text
DATABASE SCHEMA:
{{ $json.schema }}

Looking at the database schema above, convert a user's question into a SQL query to fetch data from the database. return the SQL query only 
```

Structured Output Parser

```json
{
	"type": "object",
	"properties": {
		"query": {
			"type": "string"
		}
	}
}
```

#### 👉🏻 쿼리 실행

**Postgres Execute a SQL query**

```text
{{ $json.output.query }}
```

#### 👉🏻 쿼리 결과 다듬기

**Basic LLM Chain**

Prompt

```text
QUERY RESULT:
{{ JSON.stringify($json) }}


Original Question:  
{{ $('When chat message received').first().json.chatInput }}
```

System Message

```text
Look at the query result and the user's question and return a user friendly message
```

<figure><img src="../.gitbook/assets/ai-agent/text-to-sql-2.png" alt=""><figcaption></figcaption></figure>

## MCP

[Introducing the Model Context Protocol](https://www.anthropic.com/news/model-context-protocol)

[MCP Doc.](https://modelcontextprotocol.io/docs/getting-started/intro)

### MCP 용어

**Protocol** 
- 통신 계약서
- 웹 개발의 HTTP 통신과 유사
- HTTP = Hypertext Transfer Protocol (웹 통신 계약)
- MCP = 모델과 컨텍스트 간의 통신 계약

**Model**
- LLM 모델을 의미
- GPT, Anthropic의 Claude, Gemini 등
- 한국의 Upstage Solar 모델 등 포함

**Context**
- LLM에 원하는 결과를 얻기 위해 잘 전달해야 하는 정보
- MCP를 통해 프롬프트와 툴 관리 가능
- 한국어로는 "AI에게 컨텍스트를 전달하는 프로토콜"

**MCP의 장점**

프로토콜의 범용성
- HTTP 통신처럼 서버 언어(Java, Python, Node)에 관계없이 통신 가능
- 다양한 모델 지원 가능

USB-C 포트 비유
- 공식 문서에서 "AI 애플리케이션을 위한 USB-C 포트"로 표현
- 어댑터를 통해 모든 MCP 서버와 모든 MCP 클라이언트 연결 가능

### Code Review Agent with MCP

[github-mcp-server](https://github.com/github/github-mcp-server)

[Slack MCP Server](https://github.com/modelcontextprotocol/servers-archived/tree/main/src/slack)

ref. [MCP를 활용한 코드리뷰 에이전트 생성하기](https://github.com/jasonkang14/inflearn-agent-use-cases-lecture/blob/main/24.%20MCP%EB%A5%BC%20%ED%99%9C%EC%9A%A9%ED%95%9C%20%EC%BD%94%EB%93%9C%EB%A6%AC%EB%B7%B0%20%EC%97%90%EC%9D%B4%EC%A0%84%ED%8A%B8%20%EC%83%9D%EC%84%B1%ED%95%98%EA%B8%B0.ipynb)

```python
##########################################
# 환경변수 불러오기
from dotenv import load_dotenv

load_dotenv()

##########################################
# MCP Client 세팅
import os
from langchain_mcp_adapters.client import MultiServerMCPClient

github_pat = os.getenv("GITHUB_PAT")
slack_bot_token = os.getenv("SLACK_BOT_TOKEN")
slack_team_id = os.getenv("SLACK_TEAM_ID")
slack_channel_ids = os.getenv("SLACK_CHANNEL_IDS")

mcp_client = MultiServerMCPClient({
    "github": {
      "command": "docker",
      "args": [
        "run",
        "-i",
        "--rm",
        "-e",
        "GITHUB_TOOLSETS",
        "-e",
        "GITHUB_PERSONAL_ACCESS_TOKEN",
        "ghcr.io/github/github-mcp-server"
      ],
      "env": {
        "GITHUB_TOOLSETS": "context,pull_requests",
        "GITHUB_PERSONAL_ACCESS_TOKEN": github_pat
      },
      "transport": "stdio"
    },
     "slack": {
      "command": "docker",
      "args": [
        "run",
        "-i",
        "--rm",
        "-e",
        "SLACK_BOT_TOKEN",
        "-e",
        "SLACK_TEAM_ID",
        "-e",
        "SLACK_CHANNEL_IDS",
        "mcp/slack"
      ],
      "env": {
        "SLACK_BOT_TOKEN": slack_bot_token,
        "SLACK_TEAM_ID": slack_team_id,
        "SLACK_CHANNEL_IDS": slack_channel_ids
      },
      "transport": "stdio"
    }
})

##########################################
# MCP Client Tool 확인
tool_list = await mcp_client.get_tools()

##########################################
# create react agent 활용
from langgraph.prebuilt import create_react_agent

agent = create_react_agent(
    model="openai:gpt-4.1",
    tools=tool_list,
    prompt="Use the tools provided to you to answer the user's question"
)

##########################################
# 스트림한 결과를 보기 위한 헬퍼 함수
async def process_stream(stream_generator):
    results = []
    try:
        async for chunk in stream_generator:

            key = list(chunk.keys())[0]
            if key == 'agent':
                # Agent 메시지의 내용을 가져옴. 메세지가 비어있는 경우 어떤 도구를 어떻게 호출할지 정보를 가져옴
                content = chunk['agent']['messages'][0].content if chunk['agent']['messages'][0].content != '' else chunk['agent']['messages'][0].additional_kwargs
                print(f"'agent': '{content}'")
            
            elif key == 'tools':
                # 도구 메시지의 내용을 가져옴
                for tool_msg in chunk['tools']['messages']:
                    print(f"'tools': '{tool_msg.content}'")
            
            results.append(chunk)
        return results
    except Exception as e:
        print(f"Error processing stream: {e}")
        return results

##########################################
# 프롬프로 생성
from langchain_core.messages import HumanMessage

human_message = """깃헙의 Pull Request를 확인하고 코드 리뷰를 작성해주세요. 
PR의 코드를 리뷰한 후에, 아래 항목을 확인해주세요;
1. 코드가 개선되었는지
2. 예측하지 못한 side effect가 있는지
3. 보안상 문제가 될 수 있는 부분이 없는지

위 내용을 확인해서 PR에 코멘트로 남겨주세요.
그리고 코멘트를 남긴 후에, 슬랙 채널에도 메세지를 전송해서 엔지니어에게 알려주세요
채널: <@C092QEJH6LB>
엔지니어: <@U092VN0DHBP>

PR URL:https://github.com/jasonkang14/sat-reading-client/pull/3 """

stream_generator = agent.astream({"messages": [HumanMessage(human_message)]}, stream_mode="updates")

##########################################
# 결과 확인
all_chunks = await process_stream(stream_generator)

if all_chunks:
    final_result = all_chunks[-1]
    print("\nFinal result:", final_result)
```

## Video Summary Agent

[OpenAI Whisper를 활용한 영상 요약](https://github.com/jasonkang14/inflearn-agent-use-cases-lecture/blob/main/25.%20OpenAI%20Whisper%EB%A5%BC%20%ED%99%9C%EC%9A%A9%ED%95%9C%20%EC%98%81%EC%83%81%20%EC%9A%94%EC%95%BD.ipynb)

## Reference

[uv Libeary](https://github.com/astral-sh/uv)
- An extremely fast Python package and project manager, written in Rust.

[zerox](https://github.com/getomni-ai/zerox)
- A dead simple way of OCR-ing a document for AI ingestion. Documents are meant to be a visual representation after all. With weird layouts, tables, charts, etc. The vision models just make sense!

Lecture Materials
- [inflearn-agent-use-cases-lecture](https://github.com/jasonkang14/inflearn-agent-use-cases-lecture)
- [agent-use-cases-with-n8n-and-langgraph](https://www.kangsium.com/agent-use-cases-with-n8n-and-langgraph)