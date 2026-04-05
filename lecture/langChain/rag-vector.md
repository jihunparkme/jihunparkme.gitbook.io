# RAG & Vector Database

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

## Vector Database & Embedding Model

ℹ️ **Vector**
- 관련성 파악을 위해 `vector`를 활용
  - 단어 또는 문장의 유사도를 파악해서 관련성을 측정
- Vector 생성 방법
  - Embedding 모델을 활용해서 vector 생성
  - [Embedding Proejctor](https://projector.tensorflow.org/)

ℹ️ **Vector Database**
