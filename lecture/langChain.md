# LangChain

[한시간으로 끝내는 LangChain 기본기](https://www.inflearn.com/course/%ED%95%9C%EC%8B%9C%EA%B0%84-%EB%81%9D%EB%82%B4%EB%8A%94-%EB%9E%AD%EC%B2%B4%EC%9D%B8-%EA%B8%B0%EB%B3%B8/dashboard?cid=336213)

## LLM을 활용해서 답변 생성하기

로컬 모델로 [ollama](https://ollama.com/) 활용하기
- [models](https://ollama.com/search) 메뉴에서 모델 확인

### 1️⃣ 가상환경 만들기

```shell
$ brew install pyenv

$ pyenv install 3.11

$ pyenv virtualenv 3.11 langchain-basics

$ mkdir langchain-basics

$ cd langchain-basics

$ pyenv local langchain-basics

# 해당 경로에서 jupyter notebook 실행

$ ollama pull llama3.2:1b
```

### 2️⃣ 가상환경 안에서 ollam 실행시키기

```shell
%pip install -q langchain-ollama
```

```python
from langchain_ollama import ChatOllama

llm = ChatOllama(model="llama3.2:1b")

llm.invoke("What is the capital of France?")
```

[source code](https://github.com/jihunparkme/this-and-that-py/blob/main/llm/langchain-basic-01.ipynb)

## 랭체인 스타일로 프롬프트 작성하기

```python
from langchain_ollama import ChatOllama

llm = ChatOllama(model="llama3.2:1b")

llm.invoke(0)
```

[ChatOllama](https://reference.langchain.com/python/langchain-ollama/chat_models/ChatOllama)의 invoke 메서드 파라미터로는 `PromptValue`, `str`, or `list of BaseMessages` 타입이 들어가야 합니다.

> ValueError: Invalid input type <class 'int'>. Must be a PromptValue, str, or list of BaseMessages.

### PromptValue

[PromptTemplate](https://reference.langchain.com/python/langchain-core/prompts/prompt/PromptTemplate)의 invoke 메서드를 통해 [PromptValue](https://reference.langchain.com/python/langchain-core/prompt_values/PromptValue) 생성

```python
from langchain_ollama import ChatOllama
from langchain_core.prompts import PromptTemplate

llm = ChatOllama(model="llama3.2:1b")

prompt_template = PromptTemplate(
    template="What is the capital of {country}?",
    input_variables=["country"],
)
prompt = prompt_template.invoke({"country": "France"})

print(prompt)

llm.invoke(prompt)
```

### BaseMessages

`BaseMessage`를 상속받는 대표적인 클래스
- SystemMessage
  - LLM Application의 목적(할 일)
- HumanMessage
  - 사용자 메시지
- AIMessage
  - LLM
- ToolMessage
  - 에이전트 생성 시 사용

```python
from langchain_core.messages import HumanMessage, SystemMessage, AIMessage

# few shots > 답변 예
# 대화 이력이 있던 것처럼 LLM을 속여서 답변을 우리가 원하는 형태로 유도
# 복잡한 애플리케이션 구현 시 예시를 주는 것이 정확도를 높이는 방법
message_list = [
    SystemMessage(content="You are a helpful assistant!"),
    HumanMessage(content="What is the capital of France?"),
    AIMessage(content="The capital of France is Paris."),
    HumanMessage(content="What is the capital of Germany?"),
    AIMessage(content="The capital of France is Berlin."),
    HumanMessage(content="What is the capital of Italy?"),
    AIMessage(content="The capital of Italy is Rome."),
]

llm.invoke(message_list)
```

[Language Models are Few-Shot Learners](https://arxiv.org/pdf/2005.14165)

⚠️ 하지만, **이 방식은 LangChain스럽지 않은 방식**

### ChatPromptTemplate

`LangChain`은 `ChatPromptTemplate` 사용을 권장합니다.
- `LCEL`이라는 LangChain 구성 요소를 연결시키는 기능에서 연동이 가능하므로 확장성에서 유리합니다.

```python
from langchain_core.prompts import ChatPromptTemplate

chat_prompt_template = ChatPromptTemplate.from_messages([
    ("system", "You are a helpful assistant!"),
    ("human", "What is the capital of {country}?"),
])

chat_prompt = chat_prompt_template.invoke({"country": "France"})

print(chat_prompt)
# messages=[SystemMessage(content='You are a helpful assistant!', additional_kwargs={}, response_metadata={}), HumanMessage(content='What is the capital of France?', additional_kwargs={}, response_metadata={})]

chat_prompt.messages
# [SystemMessage(content='You are a helpful assistant!', ...),
#  HumanMessage(content='What is the capital of France?', ...]

llm.invoke(chat_prompt)
# AIMessage(content='The capital of France is Paris.', ...)
```

[source code](https://github.com/jihunparkme/this-and-that-py/blob/main/llm/prompt.ipynb)

## 답변 형식 컨트롤하기

`llm.invoke()`의 결과는 기본적으로 `AIMessage` 타입으로 응답하므로, String 타입으로 반환을 해주어야 다른 언어에서도 쉽게 파싱이 가능합니다.

### StrOutputParser

```python
from langchain_core.prompts import PromptTemplate
from langchain_core.output_parsers import StrOutputParser

prompt_template = PromptTemplate(
    template="What is the capital of {country}? Return the name of the city only",
    input_variables=["country"],
)

prompt = prompt_template.invoke({"country": "France"})

print(prompt)
# text='What is the capital of France? Return the name of the city only'

ai_message = llm.invoke(prompt_template.invoke({"country": "France"}))

output_parser = StrOutputParser()

answer = output_parser.invoke(llm.invoke(prompt_template.invoke({"country": "France"})))
```

응답 결과 확인.

```python
ai_message
# AIMessage(content='Paris.', additional_kwargs={}, response_metadata={'model': 'llama3.2:1b', 'created_at': '2026-03-29T10:27:27.973169Z', 'done': True, 'done_reason': 'stop', 'total_duration': 410026625, 'load_duration': 124092416, 'prompt_eval_count': 39, 'prompt_eval_duration': 259093083, 'eval_count': 3, 'eval_duration': 23268708, 'logprobs': None, 'model_name': 'llama3.2:1b', 'model_provider': 'ollama'}, id='lc_run--019d3922-60c6-7523-b86e-9f5474e692f3-0', tool_calls=[], invalid_tool_calls=[], usage_metadata={'input_tokens': 39, 'output_tokens': 3, 'total_tokens': 42})
```

```python
answer
# 'Paris.'
```

### JsonOutputParser

`JsonOutputParser`는 형식이 오락가락하기 때문에 사용을 권장하지 않습니다.  
대신 `pydantic` 모델을 설정하여 사용할 것을 권장합니다.

```python
from pydantic import BaseModel, Field

# PromptTemplate 대체
# 모델 설정
class CountryDetail(BaseModel):
    # 타입 설정
    capital: str = Field(description="The capital city of the country")
    population: int = Field(description="The population of the country")
    language: str = Field(description="The official language of the country")
    currency: str = Field(description="The currency used in the country")

structured_llm = llm.with_structured_output(CountryDetail)

#

country_detail_prompt = PromptTemplate(
    template="""Give following information about {country}:
    - Capital
    - Population
    - Language
    - Currency

    return it in JSON format. and return the JSON dictionary only
    """,
    input_variables=["country"],
)

json_ai_message = structured_llm.invoke(country_detail_prompt.invoke({"country": "France"}))
```







## Reference

- [LangChain Docs](https://docs.langchain.com/oss/python/deepagents/overview)
- [pyenv, virtualenv 설치 및 사용법](https://dandyrilla.github.io/2024-06-05/pyenv-virtualenv/)
- [OpenAI integrations](https://docs.langchain.com/oss/python/integrations/providers/openai)