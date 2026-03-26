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

<https://github.com/jihunparkme/this-and-that-py/blob/main/llm/langchain-basic-01.ipynb>

### 랭체인 스타일로 프롬프트 작성하는 방법







## Reference

- [LangChain Docs](https://docs.langchain.com/oss/python/deepagents/overview)
- [pyenv, virtualenv 설치 및 사용법](https://dandyrilla.github.io/2024-06-05/pyenv-virtualenv/)
- [OpenAI integrations](https://docs.langchain.com/oss/python/integrations/providers/openai)