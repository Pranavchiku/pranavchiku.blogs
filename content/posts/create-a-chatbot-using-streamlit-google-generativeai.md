+++
title = 'Create a Chatbot Using Streamlit Google Generativeai'
date = 2024-04-21T22:13:26+05:30
draft = false
+++

## Introduction

Hey, glad to see you here, in this blog I'll be directing you to create a small chatbot using streamlit and google-generative-ai, so without wasting much of time, let's get started.

## Create virtual environment

To keep dependencies isolated, I prefer to create a virtual environment using the command given below:

```
python3 -m venv llm
```

Now activate the environment using:

```
# On Windows
llm\Scripts\activate
# On macOS and Linux
source llm/bin/activate
```

Created environment can be deactivated using `deactivate` command.

## Setup google api key

Now, to leverage power of available llms, you need to generate an api key, this can be done at [gemini-api](https://ai.google.dev/gemini-api/docs/api-key).

Once, this is done, set it as an env variable:

```
export GOOGLE_API_KEY='<your-api-key-here>'
```

## Install requirements

The following requirements need to be installed, present in `requirements.txt` file.

> requirements.txt

```
google-generativeai
streamlit
streamlit-chat
```

Execute the following command:

```
pip install -r requirements.txt
```

## Create streamlit application

Create a file name `streamlit_app.py`, import the required dependencies as shown below

> streamlit_app.py

```python
import os
import streamlit as st
import google.generativeai as palm
from streamlit_chat import message
```

Now, add title to the app and fetch API_KEY along with available google palm models

```python
st.title('🦜🔗 LLM Demo')

API_KEY = os.environ.get('GOOGLE_API_KEY')

models = [model.name[7:] for model in palm.list_models()]
```

Further down, we'll create a sidebar that will have a radio button to select model of our choice from available options and a button to clear context.

```python
# Sidebar - let user choose model and let user clear the current conversation
st.sidebar.title("Sidebar")
model_name = st.sidebar.radio("Choose a model:", models)
clear_button = st.sidebar.button("Clear Conversation", key="clear")
```

Now, we need to intialise a few state variables that will hold, generated and user messages, this can be done via:

```python
if 'generated' not in st.session_state:
    st.session_state['generated'] = []
if 'past' not in st.session_state:
    st.session_state['past'] = []
if 'messages' not in st.session_state:
    st.session_state['messages'] = [
        {"role": "system", "content": "You are a helpful assistant."}
    ]
```

Now, add provision to reset everything:

```python
# reset everything
if clear_button:
    st.session_state['generated'] = []
    st.session_state['past'] = []
    st.session_state['messages'] = [
        {"role": "system", "content": "You are a helpful assistant."}
    ]
```

Add a helper function to generate response from desired google-palm

```python
def generate_response(input_text):
    completion=palm.generate_text(
        model=model_id,
        prompt=input_text,
        temperature=0.99,
        max_output_tokens=800,
    )
    # st.info(completion.result)
    st.session_state['messages'].append({"role": "assistant", "content": completion.result})
    return completion.result
```

Setup a container for chat history

```python
# container for chat history
response_container = st.container()
# container for text box
container = st.container()
```

And inside this container, create a form to take input and a submit button to generate response, also save user input and generated response in session state variables.

```python
with container:
    with st.form(key='my_form', clear_on_submit=True):
        user_input = st.text_area("You:", key='input', height=100)
        submit_button = st.form_submit_button(label='Send')

    if submit_button and user_input:
        output = generate_response(user_input)
        st.session_state['past'].append(user_input)
        st.session_state['generated'].append(output)
```

Now, if there are any generated messages, render it on screen:

```python
if st.session_state['generated']:
    with response_container:
        for i in range(len(st.session_state['generated'])):
            # message(st.session_state["past"][i], is_user=True, key=str(i) + '_user')
            # message(st.session_state["generated"][i], key=str(i))
            st.chat_message("user", avatar="👨‍💻").write(st.session_state["past"][i])
            st.chat_message("user", avatar="🤖").write(st.session_state["generated"][i])
```

That's it, now run the application using and enjoy conversation :)

```
streamlit run streamlit_app.py
```
