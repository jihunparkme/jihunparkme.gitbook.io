# LangGraph 응용 LLM Workflow

## 가상환경 설정

### uv

[uv](https://github.com/astral-sh/uv)를 활용하는 방법

```shell
$ brew install uv

$ cd your-repository

$ uv sync
```

명령어 하나로 `.venv` 가상환경에 필요한 패키지들이 모두 설치

### pyenv

[pyenv](https://github.com/pyenv/pyenv)를 활용하는 방법

```shell
$ pyenv virtualenv 3.12.2 inflearn-langgraph-agent

$ pyenv local inflearn-langgraph-agent
```

## LangChain