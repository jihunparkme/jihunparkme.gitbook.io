# LangGraph 응용 LLM Workflow

## 가상환경 설정

### uv

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

### pyenv

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

## LangChain