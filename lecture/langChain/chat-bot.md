# ChatBot

[streamlit](https://streamlit.io/)을 활용한 ChatBot 구현하기

# streamlit

[streamlit api reference](https://docs.streamlit.io/develop/api-reference/text)를 보면서 쉽게 UI를 그릴 수 있는 장점

## Install Streamlit

[Install Streamlit](https://docs.streamlit.io/get-started/installation)

```shell
 $ mkdir streamlit
 $ cd streamlit
 $ pyenv virtualenv 3.11 streamlit
 $ pyenv local streamlit

 $ pip install streamlit
 $ streamlit hello
```

## Write User Message

**코드 작성**
- Streamlit은 React 기반으로 돌아가고, 자체적으로 최적화가 되어 DOM을 비교한 후 변경된 사항들만 UI를 그리도록 되어 있습니다.

```python
import streamlit as st

st.set_page_config(page_title="소득세 챗봇", page_icon="🤖")
st.title("🤖 소득세 챗봇")
st.caption("소득세에 관련된 모든것을 답해드립니다!")

# Session State 적용
# https://docs.streamlit.io/develop/api-reference/caching-and-state/st.session_state
if 'message_list' not in st.session_state:
    st.session_state.message_list = []

# Session State 에 저장된 메시지들을 화면에 렌더링
for message in st.session_state.message_list:
    with st.chat_message(message["role"]):
        st.write(message["content"])

if user_question := st.chat_input(placeholder="소득세에 관련된 궁금한 내용들을 말씀해주세요!"):
    with st.chat_message("user"):
        st.write(user_question)
    # Session State에 메시지 저장
    st.session_state.message_list.append({"role": "user", "content": user_question})
```

**streamlit 실행**

```shell
streamlit run chat.py 
```