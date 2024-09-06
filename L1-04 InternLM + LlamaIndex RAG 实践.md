# InternLM + LlamaIndex RAG 实践

本文将分为以下几个部分来介绍，如何使用 LlamaIndex 来部署 InternLM2 1.8B（以 InternStudio 的环境为例）

- 前置知识
- 环境、模型准备
- LlamaIndex HuggingFaceLLM
- LlamaIndex RAG

## 1. 前置知识

正式介绍检索增强生成（Retrieval Augmented Generation，RAG）技术以前，大家不妨想想为什么会出现这样一个技术。 给模型注入新知识的方式，可以简单分为两种方式，一种是内部的，即更新模型的权重，另一个就是外部的方式，给模型注入格外的上下文或者说外部信息，不改变它的的权重。 第一种方式，改变了模型的权重即进行模型训练，这是一件代价比较大的事情，大语言模型具体的训练过程，可以参考[InternLM2](https://arxiv.org/abs/2403.17297)技术报告。第二种方式，并不改变模型的权重，只是给模型引入格外的信息。类比人类编程的过程，第一种方式相当于你记住了某个函数的用法，第二种方式相当于你阅读函数文档然后短暂的记住了某个函数的用法。
![RAG工作原理](https://imgsur.quicklydating.com/gh/nlospc/imgsur@main/img/Pasted%20image%2020240905144945.png)


### RAG 效果比对

如图所示，由于`xtuner`是一款比较新的框架， `InternLM2-Chat-1.8B` 训练数据库中并没有收录到它的相关信息。左图中问答均未给出准确的答案。右图未对 `InternLM2-Chat-1.8B` 进行任何增训的情况下，通过 RAG 技术实现的新增知识问答。
![使用RAG前后对比](https://imgsur.quicklydating.com/gh/nlospc/imgsur@main/img/20240905173253.png)
## 2. 环境准备
以下步骤之间参见[闯关手册和任务详细描述](https://github.com/InternLM/Tutorial/tree/879439ab9615265d54efc2d0495ab8ecdbb16317/docs/L1/LlamaIndex) 我不过多阐述
### 2.1 配置基础环境

### 2.2 安装Llamaindex和相关的包

### 2.3 下载Sentence Transformer 模型

### 2.4 下载NLTK 相关资源

## 3. LlamaIndex HuggingFaceLLM
运行以下指令，把 `InternLM2 1.8B` 软连接出来

```shell
cd ~/model
ln -s /root/share/new_models/Shanghai_AI_Laboratory/internlm2-chat-1_8b/ ./
```

运行以下指令，新建一个python文件

```shell
cd ~/llamaindex_demo
touch llamaindex_internlm.py
```

打开llamaindex_internlm.py 贴入以下代码
这里直接把问题写在了脚本里，为了在控制台看输出
我调整了一下问题内容，问了一个新出的文本格式 **RapidLayoutRecover**

```python
from llama_index.llms.huggingface import HuggingFaceLLM
from llama_index.core.llms import ChatMessage
llm = HuggingFaceLLM(
    model_name="/root/model/internlm2-chat-1_8b",
    tokenizer_name="/root/model/internlm2-chat-1_8b",
    model_kwargs={"trust_remote_code":True},
    tokenizer_kwargs={"trust_remote_code":True}
)

rsp = llm.chat(messages=[ChatMessage(content="**xtuner**是什么？")]) //可以改成你想问的其他问题
print(rsp)
```

之后运行

```shell
conda activate llamaindex
cd ~/llamaindex_demo/
python llamaindex_internlm.py
```

很明显，模型还不知道这是个什么
![RAG之前，llamaindex的回复](https://imgsur.quicklydating.com/gh/nlospc/imgsur@main/img/20240905230920.png)
## 4. LlamaIndex RAG

安装 `LlamaIndex` 词嵌入向量依赖

```shell
conda activate llamaindex
pip install llama-index-embeddings-huggingface==0.2.0 llama-index-embeddings-instructor==0.1.3
```

编写知识库，自己建一个md文件，大概输入一下相关的介绍

```shell
cd ~/llamaindex_demo
mkdir data
cd data
vi know_about_RapidLayoutRecover.md
```

运行以下指令，新建一个python文件

```shell
cd ~/llamaindex_demo
touch llamaindex_RAG.py
```

打开`llamaindex_RAG.py`贴入以下代码

```python
from llama_index.core import VectorStoreIndex, SimpleDirectoryReader, Settings

from llama_index.embeddings.huggingface import HuggingFaceEmbedding
from llama_index.llms.huggingface import HuggingFaceLLM

#初始化一个HuggingFaceEmbedding对象，用于将文本转换为向量表示
embed_model = HuggingFaceEmbedding(
#指定了一个预训练的sentence-transformer模型的路径
    model_name="/root/model/sentence-transformer"
)
#将创建的嵌入模型赋值给全局设置的embed_model属性，
#这样在后续的索引构建过程中就会使用这个模型。
Settings.embed_model = embed_model

llm = HuggingFaceLLM(
    model_name="/root/model/internlm2-chat-1_8b",
    tokenizer_name="/root/model/internlm2-chat-1_8b",
    model_kwargs={"trust_remote_code":True},
    tokenizer_kwargs={"trust_remote_code":True}
)
#设置全局的llm属性，这样在索引查询时会使用这个模型。
Settings.llm = llm

#从指定目录读取所有文档，并加载数据到内存中
documents = SimpleDirectoryReader("/root/llamaindex_demo/data").load_data()
#创建一个VectorStoreIndex，并使用之前加载的文档来构建索引。
# 此索引将文档转换为向量，并存储这些向量以便于快速检索。
index = VectorStoreIndex.from_documents(documents)
# 创建一个查询引擎，这个引擎可以接收查询并返回相关文档的响应。
query_engine = index.as_query_engine()
response = query_engine.query("RapidLayoutRecover是什么?")#记得把问题改成自己设计的RAG内容

print(response)
```

之后运行

```shell
conda activate llamaindex
cd ~/llamaindex_demo/
python llamaindex_RAG.py
```
显示
![能够识别并提供正确答案](https://imgsur.quicklydating.com/gh/nlospc/imgsur@main/img/20240906130025.png)
## 5. LlamaIndex web

运行之前首先安装依赖

```shell
pip install streamlit==1.36.0
```

运行以下指令，新建一个python文件

```shell
cd ~/llamaindex_demo
touch app.py
```

打开`app.py`贴入以下代码

```python
import streamlit as st
from llama_index.core import VectorStoreIndex, SimpleDirectoryReader, Settings
from llama_index.embeddings.huggingface import HuggingFaceEmbedding
from llama_index.llms.huggingface import HuggingFaceLLM

st.set_page_config(page_title="llama_index_demo", page_icon="🦜🔗")
st.title("llama_index_demo")

# 初始化模型
@st.cache_resource
def init_models():
    embed_model = HuggingFaceEmbedding(
        model_name="/root/model/sentence-transformer"
    )
    Settings.embed_model = embed_model

    llm = HuggingFaceLLM(
        model_name="/root/model/internlm2-chat-1_8b",
        tokenizer_name="/root/model/internlm2-chat-1_8b",
        model_kwargs={"trust_remote_code": True},
        tokenizer_kwargs={"trust_remote_code": True}
    )
    Settings.llm = llm

    documents = SimpleDirectoryReader("/root/llamaindex_demo/data").load_data()
    index = VectorStoreIndex.from_documents(documents)
    query_engine = index.as_query_engine()

    return query_engine

# 检查是否需要初始化模型
if 'query_engine' not in st.session_state:
    st.session_state['query_engine'] = init_models()

def greet2(question):
    response = st.session_state['query_engine'].query(question)
    return response

      
# Store LLM generated responses
if "messages" not in st.session_state.keys():
    st.session_state.messages = [{"role": "assistant", "content": "你好，我是你的助手，有什么我可以帮助你的吗？"}]    

    # Display or clear chat messages
for message in st.session_state.messages:
    with st.chat_message(message["role"]):
        st.write(message["content"])

def clear_chat_history():
    st.session_state.messages = [{"role": "assistant", "content": "你好，我是你的助手，有什么我可以帮助你的吗？"}]

st.sidebar.button('Clear Chat History', on_click=clear_chat_history)

# Function for generating LLaMA2 response
def generate_llama_index_response(prompt_input):
    return greet2(prompt_input)

# User-provided prompt
if prompt := st.chat_input():
    st.session_state.messages.append({"role": "user", "content": prompt})
    with st.chat_message("user"):
        st.write(prompt)

# Gegenerate_llama_index_response last message is not from assistant
if st.session_state.messages[-1]["role"] != "assistant":
    with st.chat_message("assistant"):
        with st.spinner("Thinking..."):
            response = generate_llama_index_response(prompt)
            placeholder = st.empty()
            placeholder.markdown(response)
    message = {"role": "assistant", "content": response}
    st.session_state.messages.append(message)
```

之后运行

```shell
streamlit run app.py
```

然后在命令行点击，红框里的url。
