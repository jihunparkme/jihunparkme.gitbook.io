# LangGraph 응용 LLM Workflow

# 가상환경 설정

## uv

[uv](https://github.com/astral-sh/uv)를 활용하는 방법

**uv 설치**

```shell
$ brew install uv
```

**신규 프로젝트 초기화**

```shell
$ uv init --python 3.12
```

`pyproject.toml`이 생성되며, 이후 패키지 추가 시 `uv.lock`도 함께 관리된다.

```shell
# 패키지 추가
$ uv add langgraph langchain-openai
```

**기존 프로젝트 (저장소 clone 후)**

```shell
$ cd your-repository

# pyproject.toml / uv.lock 기반으로 .venv 가상환경에 패키지 일괄 설치
$ uv sync
```

**가상환경 활성화**

```shell
$ source .venv/bin/activate
```

## pyenv

[pyenv](https://github.com/pyenv/pyenv)와 [pyenv-virtualenv](https://github.com/pyenv/pyenv-virtualenv) 플러그인을 활용하는 방법

**pyenv 및 pyenv-virtualenv 설치**

```shell
$ brew install pyenv pyenv-virtualenv
```

shell 설정 파일(`.zshrc` 또는 `.bashrc`)에 아래 내용을 추가한다.

```shell
eval "$(pyenv init -)"
eval "$(pyenv virtualenv-init -)"
```

**Python 버전 설치 및 가상환경 생성**

```shell
# Python 버전 설치
$ pyenv install 3.12.2

# 가상환경 생성
$ pyenv virtualenv 3.12.2 inflearn-langgraph-agent
```

**프로젝트 디렉토리에 가상환경 지정**

```shell
$ cd your-repository

# 해당 디렉토리 진입 시 자동으로 가상환경 활성화
$ pyenv local inflearn-langgraph-agent
```

`pyenv local` 설정 후 해당 디렉토리에 진입하면 `pyenv-virtualenv`가 자동으로 가상환경을 활성화한다.

```shell
# 패키지 설치
$ pip install langgraph langchain-openai
```

# LangChain vs LangGraph

> ℹ️ [Sample Code](https://github.com/jihunparkme/inflearn-langgraph-agent/blob/main/2.1%20LangChain%20vs%20LangGraph%20(feat.%20LangGraph%20%EA%B0%9C%EB%85%90%20%EC%84%A4%EB%AA%85).ipynb)

||LangChain|LangGraph|
|---|---|---|
|특징|순차적인 워크플로우로 구성|Conditional Edge를 활용하여 효율적인 워크플로우 구성|
|예시|항상 Rewrite를 호출하여 질문을 수정|필요할 때만 Rewrite를 호출하여 LLM 호출 횟수를 줄임|

## LangChain

[LangChain](https://python.langchain.com/)은 LLM 기반 애플리케이션 개발을 위한 프레임워크로, LangGraph와 함께 LLM 호출, 프롬프트 관리, 체인 구성 등을 담당한다.

✅ **패키지 설치**

```shell
$ pip install python-dotenv langchain-google-genai
```

✅ **환경 변수 설정**

`.env` 파일에 Google API 키를 설정한다.

```
GOOGLE_API_KEY=your-google-api-key
```

> Google AI Studio([aistudio.google.com](https://aistudio.google.com))에서 API 키를 발급받을 수 있다.

✅ **기본 사용 예시**

```python
from dotenv import load_dotenv
from langchain_google_genai import ChatGoogleGenerativeAI

load_dotenv()  # .env 파일에서 환경 변수 로드

query = '인프런에는 어떤 강의가 있나요?'
llm = ChatGoogleGenerativeAI(model="gemini-2.5-flash")

response = llm.invoke(query)
print(response.content)  # AIMessage에서 텍스트 추출
```

`llm.invoke()`는 `AIMessage` 객체를 반환하며, `.content` 속성으로 응답 텍스트를 추출한다.

## LangGraph

[LangGraph](https://docs.langchain.com/oss/python/langgraph/overview)는 상태를 중심으로 LLM 워크플로우를 그래프로 표현하는 프레임워크다. 단순한 체인보다 분기, 반복, 조건부 흐름, 멀티 에이전트 구성이 필요할 때 강점을 가진다.

**LangGraph의 주요 개념**

|개념|정의|예시|
|---|---|---|
|State|그래프 실행 중 공유되는 상태|메시지, 검색 문서, 사용자 질문, 중간 판단 결과|
|Node|상태를 입력받아 상태를 갱신하는 작업 단위|Retrieve, Generate, Rewrite 등|
|Edge|항상 다음 노드로 이동하는 연결|START -> generate|
|Conditional Edge|조건에 따라 다음 노드를 선택하는 연결|검색 결과가 충분하면 Generate, 아니면 Rewrite|

✅ **패키지 설치**

```shell
$ pip install langgraph langchain-core
```

### LangGraph 에이전트 생성 과정

1️⃣. **State 선언**

`TypedDict`와 `Annotated`를 사용해 그래프 전체에서 공유할 상태를 정의한다. 메시지 목록은 `add_messages`를 통해 기존 상태에 누적된다.

```python
from typing import Annotated
from typing_extensions import TypedDict

from langgraph.graph.message import add_messages
from langchain_core.messages import AnyMessage

class AgentState(TypedDict):
    messages: list[Annotated[AnyMessage, add_messages]]
```

2️⃣. **GraphBuilder 생성**

노드와 엣지를 추가하여 그래프를 구성.

```python
from langgraph.graph import StateGraph

graph_builder = StateGraph(AgentState)
```

3️⃣. **Node 생성**

각 노드는 현재 상태를 입력받아 새로운 상태 일부를 반환한다. 아래 예시는 메시지를 받아 LLM 응답을 추가하는 가장 단순한 `generate` 노드다.

```python
def generate(state: AgentState) -> AgentState:
    messages = state['messages']
    ai_message = llm.invoke(messages)
    return {'messages': [ai_message]}

graph_builder.add_node('generate', generate)
```

4️⃣. **Edge 추가 및 그래프 컴파일**

`START`와 `END`를 연결해 가장 단순한 단일 노드 워크플로우를 만든다.

```python
from langgraph.graph import START, END

graph_builder.add_edge(START, 'generate')
graph_builder.add_edge('generate', END)

graph = graph_builder.compile()
```

5️⃣. **그래프 시각화**

Mermaid를 사용하여 그래프 구조를 시각적으로 확인.

```python
from IPython.display import display, Image

display(Image(graph.get_graph().draw_mermaid_png()))
```

7️⃣. **그래프 실행**

```python
from langchain_core.messages import HumanMessage

# 사용자의 질문을 `state`에 담아서 `invoke` 메서드를 호출하면, 그래프가 실행
initial_state = {'messages': [HumanMessage(query)]}
graph.invoke(initial_state)
```

`graph.invoke()`의 반환값은 최종 `state`이며, 여기서는 마지막 메시지가 모델의 응답이다.

이 방식으로 불필요한 LLM 호출을 줄이고, 검색 품질 평가나 도구 호출 결과에 따라 워크플로우를 유연하게 제어할 수 있다.

**LangGraph의 장점**

* 조건부 분기와 반복 로직을 명시적으로 표현할 수 있음
* 상태 기반으로 복잡한 멀티 스텝 워크플로우를 안정적으로 구성할 수 있음
* 멀티 에이전트, RAG, 검증-재시도 패턴에 적합함
* 필요할 때만 다음 노드를 실행해 비용과 지연 시간을 줄일 수 있음

> ⚠️
>
> 단순히 한두 번의 LLM 호출만 필요한 경우에는 LangChain만으로도 충분한 경우가 많다.
>
> 분기, 재시도, 상태 관리, 사람 승인 단계처럼 워크플로우 제어가 중요할 때 LangGraph를 선택하는 편이 적절하다.

## Simple Retrieval Agent

**문서를 기반으로 답변을 생성하는 LangGraph 에이전트**
- 사용자가 질문을 하면 관련 문서를 검색하고, 해당 문서를 기반으로 LLM이 답변을 생성
- 문서 처리를 통해 더 정확하고 맥락에 맞는 답변 제공

**문서 처리 및 RAG 생성**

1. **문서 로드 및 분할**
     * PDF 문서를 로드하고 페이지 단위로 분할
     * PDF의 이미지 데이터(예: 표)는 기본 `PyPDFLoader`로 처리 불가
     * OCR 도구(`py-zerox`)를 사용하여 이미지 데이터를 텍스트로 변환
2. **Markdown 및 Text 변환**
     * OCR로 변환된 데이터를 Markdown 형식으로 저장
     * Markdown 데이터를 텍스트로 변환하여 정확한 문맥 전달
     * [markdown-loader](https://github.com/peerigon/markdown-loader)
3. **벡터 스토어 생성**
     * `Chroma`를 사용하여 문서를 벡터화하고 로컬에 저장
     * `Retriever`를 통해 벡터화된 문서를 검색 가능

### LangGraph 에이전트 구조**

**워크플로우 개요**

* `retrieve`: 사용자의 질문을 받아 문서를 검색
* `check_doc_relevance`: 검색된 문서가 질문과 관련이 있는지 검증
* `generate`: 문서가 관련이 있으면 답변을 생성
* `rewrite`: 문서가 관련이 없으면 질문을 수정(rewrite)하고 다시 검색

**노드(Node) 설계**

1. **State 정의**
     * 사용자 질문, 검색된 문서(context), 생성된 답변(answer) 포함
2. **`Retriever` 노드**
     * 사용자의 질문을 기반으로 벡터 스토어에서 관련 문서 검색
     * 검색된 문서를 `context`에 저장
3. **`CheckDocRelevance` 노드**: 검색된 문서와 질문의 **관련성을 검증**
     * Relevance Prompt: 문서와 질문의 관련성을 평가하는 프롬프트.
     * 관련성이 있으면 `Generate`로 이동
     * 관련성이 없으면 `Rewrite`로 이동
       * [rag-document-relevance](https://smith.langchain.com/hub/langchain-ai/rag-document-relevance): 문서와 질문을 받아서 점수를 측정
4. **`Generator` 노드**
   * Generate Prompt: 답변 생성을 위한 프롬프트
   * 검색된 문서를 기반으로 LLM이 질문에 대한 **답변 생성**
   * LangSmith에서 제공하는 프롬프트를 활용하여 효율적인 답변 생성
     * [rlm/rag-prompt](https://smith.langchain.com/hub/rlm/rag-prompt)
     * [teddynote/rag-prompt-korean](https://smith.langchain.com/hub/teddynote/rag-prompt-korean)
   * LLM 호출 시 사용자 질문과 검색된 문서를 함께 전달
5. **`Rewrite` 노드**
    * **질문을 수정**하여 검색 결과를 개선
    * Rewrite Prompt: 질문을 수정하기 위한 프롬프트
6. **그래프 빌더 생성 및 연결**
   * Node: `Start` → `Retrieve` → `Generate` → `End`
   * Edge: 노드 간의 순차적 연결

**LangGraph 에이전트 시나리오**

1. **질문**: "연봉 5천만 원의 소득세는 얼마인가?"
     * **Retrieve**: 관련 문서(소득세율표 등) 검색
     * **Generate**: 검색된 문서를 기반으로 답변 생성
2. **문서 검색 실패 시**
      * 검색된 문서가 부족하거나 부정확한 경우, 질문을 재작성하여 다시 검색
      * `Conditional Edge`를 활용하여 효율적인 워크플로우 구현

## Self Rag

> [Self-Reflective RAG with LangGraph](https://www.langchain.com/blog/agentic-rag-with-langgraph)
> - [examples/rag/langgraph_self_rag.ipynb](https://github.com/langchain-ai/langgraph/blob/main/examples/rag/langgraph_self_rag.ipynb?ref=blog.langchain.com)


