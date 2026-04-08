# RAG & Vector Database

# Before we get started

## Retrieval Augmented Generation(RAG)

> LLM Application 개발에서 중요한 개념

- 단어를 그대로 해석하면 '검색', '증강', '생성'

ℹ️ **Retrieval**

- 언어모델이 가지고 있지 않은 **데이터를를 가져오는 것**
- 보안이 걸려있는 사내자료를 가져오는 역할을 할 수 있음
- LLM 모델들은 최신 정보를 가지고 있지 않음
  - GPT4는 23년 12월까지 정보를 가지고 있음

ℹ️ **Augmented**
- AR/VR에 사용되는 것과 같은 단어(**마치 사실인 것 처럼**)
- Retrieval 된 데이터를 LLM에게 주면서, “마치 이 정보를 아는 것 처럼” 생성하라고 지시

ℹ️ **Generation**
- 생성

> Retrieval: "내가 가져온 데이터를 제공할테니"
> 
> Augmented: "이 정보를 아는 것처럼"
> 
> Generation: "답변을 생성해라"

🙋🏼‍♂️ 우리가 해야할 일은?
- 답변 생성은 LLM의 역할
- 우리는 **데이터를 잘 저장**하고, **데이터를 잘 가져와서**, LLM에게 **잘 전달**하는 역할
- Augmented 된 Context를 기반으로 LLM에게 답변을 생성하게 하는 것
  - 우리가 가져온 데이터를 프롬프트의 어느 위치에 넣는지가 답변 생성에 굉장히 큰 영향

[Build a RAG agent with LangChain](https://docs.langchain.com/oss/python/langchain/rag#overview)

# Vector Database & Embedding Model

ℹ️ **Vector**
- 관련성 파악을 위해 `vector`를 활용
  - 단어 또는 문장의 유사도를 파악해서 관련성을 측정
- Vector 생성 방법
  - Embedding 모델을 활용해서 vector 생성
  - [Embedding Proejctor](https://projector.tensorflow.org/)

ℹ️ **Vector Database**
- Embedding 모델을 활용해 생성된 **vector(+metadata) 저장**
  - 문서 이름, 페이지 번호 등을 같이 저장하면 LLM이 생성하는 답변 퀄리티가 상승
- Vector 대상으로 **유사도 검색** 실시
  - 사용자 질문과 가장 비슷한 문서 가져오기 (Retrieval)
  - 가져온 문서를 prompt를 통해 LLM에 제공 (Augmented)
  - LLM은 prompt를 활용해 답변 생성 (Generation)
- 문서 전체를 활용하면 속도도 느리고, 토큰 수 초과로 답변 생성이 안될 수 있으므로 **문서를 나눠서 저장**하는게 필요

📚 주요 용어
- `Vector` : 텍스트를 수학적으로 변경한 상태 
- `Embedding` : 텍스트를 Vector로 변경하는 방법
- `Vector Database` : Embedding을 통해 생성된 Vector를 저장하는 데이터베이스
- 유사도 검색: 두 개의 Vector가 얼마나 "가까운지"를 계산하는 방법
- [Vector Similarity Explained](https://www.pinecone.io/learn/vector-similarity/)

> 참고. 한국말 인베딩을 사용할 경우 [업스테이지 임베딩](https://console.upstage.ai/docs/capabilities/embed) 추천

# LangChain 활용 RAG 구성

## 환경설정

```python
# 가상환경 생성
$ pyenv virtualenv 3.11 llm-application

# 추가된 가상환경 확인
pyenv virtualenvs

# 디렉토리에 가상환경 적용
$ pyenv local llm-application
```

## LLM 답변 생성

```python
###
# 패키지 설치
%pip install -q python-dotenv langchain-openai langchain-google-genai

###
# 환경변수 불러오기
from dotenv import load_dotenv

load_dotenv()

###
# LLM 답변 생성 (ChatGoogleGenerativeAI 사용)
from langchain_google_genai import ChatGoogleGenerativeAI

llm = ChatGoogleGenerativeAI(
    model="gemini-2.5-flash"
)

# LLM 답변 생성 (ChatOpenAI 사용)
from langchain_openai import ChatOpenAI

llm = ChatOpenAI(model_name='gpt-4o-mini')

###
# 답변 환인
ai_message = llm.invoke("인프런에 어떤 강의가 있나요?")

ai_message.content
```

[source code](https://github.com/jihunparkme/study-ai/blob/main/3-langchain-rag/1_langchain_llm_test.ipynb)

참고.
- [Google integrations](https://docs.langchain.com/oss/python/integrations/providers/google)
  - [ChatGoogleGenerativeAI](https://docs.langchain.com/oss/python/integrations/chat/google_generative_ai)
- [OpenAI integrations](https://docs.langchain.com/oss/python/integrations/providers/openai)
  - [ChatOpenAI](https://docs.langchain.com/oss/python/integrations/chat/openai)





<https://dandyrilla.github.io/2024-06-05/pyenv-virtualenv/>

