[TOC]

随着大语言模型（LLM）技术的快速发展，人们期望利用 LLM 解决各种复杂问题，在此背景下，构建智能体（Agent）应用受到了广泛关注。用户与 LLM 的交互可以被视为一种 **单智能体（Single-Agent）** 行为：用户通过提示词（prompt）与通用 LLM 进行对话，LLM 理解问题并提供反馈。然而，单一智能体在处理复杂任务时存在明显局限性，例如需要用户多次引导、缺乏对外部环境的感知能力、对话历史记忆有限等。

试想以下场景：在不同处理阶段调用不同的模型；当 LLM 无法完成任务时，自动查询外部知识库；或者由 LLM 自主纠正生成内容中的幻觉和错误。这些需求如何实现？ **多智能体（Multi-Agent）** 系统正是解决这类问题的有效工具。通过提示词模板为每个智能体分配角色并规范其行为，多个智能体相互协作，从而完成复杂的任务。

然而，构建多智能体应用并非易事，开发者需要面对智能体设计、通信协议、协调策略等多方面的问题。LangGraph 提供了一种以图（graph）为核心的解决方案，清晰定义了智能体之间的关系与交互规则，并通过内置的通信接口和协调策略，帮助开发者快速构建高效且可扩展的分布式智能系统。

![](1.png)

**接下来，我们将通过一个实例展示如何使用 `LangGraph` 构建一个多智能体应用，并结合 `Streamlit` 实现用户友好的前端界面。** 该应用具备以下功能：
1. 根据对话类型将请求路由到适当的处理节点。
2. 支持联网搜索，获取实时信息。
3. 根据问题和对话历史生成优化的搜索提示词。
4. 支持文件上传与处理。
5. 利用编程专用的 LLM 解决代码相关问题。
6. 基于提供的文档内容，总结生成答案。

![](2.png)

## 环境搭建与配置

项目结构如下：

```shell
.
├── .streamlit  # Streamlit 配置
│   └── config.toml
├── chains  # 智能体
│   ├── generate.py
│   ├── models.py
│   └── summary.py
├── graph   # 图结构
│   ├── graph.py
│   └── graph_state.py
├── upload_files    # 上传的文件
│   └── .keep
├── .env   # 环境变量配置
├── app.py  # Streamlit 应用
├── main.py # 命令行程序
└── requirements.txt    # 依赖
```

`requirements.txt` 中列出了程序必要的包，使用命令 `pip install -r requirements.txt` 安装依赖。

```shell
# LangChain 相关包
langchain
langchain-ollama
langchain-chroma
langchain-community
langgraph
chromadb
tavily-python
python-dotenv

# 文档处理相关包
marker-pdf
weasyprint
mammoth
openpyxl
unstructured[all-docs]
libmagic

# Streamlit 相关包
streamlit
streamlit-chat
streamlit-extras

# 文档使用GPU处理时，安装GPU版PyTorch
# use 'pip install -r requirements.txt --proxy=127.0.0.1:23474' to accelerate download speed
# --extra-index-url https://download.pytorch.org/whl/cu124
# torch==2.6.0+cu124
# torchvision==0.21.0+cu124
```

相关的环境变量配置在 `.env` 文件中，在程序中通过 `dotenv` 包读取。

```shell
TAVILY_API_KEY=tvly-dev-xxxxxx  # Tavily API 密钥
OMP_NUM_THREADS=8   # 设置线程数
```

其中 `TAVILY_API_KEY` 是 Tavily 的 API 密钥，用于网络搜索服务，需要在 https://app.tavily.com/home 注册并获取，每月有 1000 次的免费额度。

![](3.png)

## 定义智能体

在 LangChain 中，使用 **链（chain）** 来定义用户与 LLM 交互的行为，即智能体。链是一个可调用的对象，接收输入并返回输出。在 `chains` 目录下，定义了两个链：`summary.py` 和 `generate.py`，分别用于提取关键词和生成回答。

```shell
.
├── chains  # 智能体
│   ├── generate.py
│   ├── models.py
│   └── summary.py
```

### 加载模型

在定义智能体之前，需要先定义好加载模型的方法。`models.py` 文件负责根据提供的模型名称加载相应的模型。

```python
from langchain_ollama import ChatOllama, OllamaEmbeddings
from langchain_core.vectorstores import InMemoryVectorStore

def load_model(model_name: str) -> ChatOllama:
    """
    加载语言模型

    参数:
        model_name (str): 模型名称

    返回:
        ChatOllama实例，用于生成文本和回答问题
    """
    return ChatOllama(model=model_name)
    
def load_embeddings(model_name: str) -> OllamaEmbeddings:
    """
    加载嵌入模型

    参数:
        model_name (str): 模型名称

    返回:
        OllamaEmbeddings实例，用于将文本转换为向量表示
    """
    return OllamaEmbeddings(model=model_name)

def load_vector_store(model_name: str) -> InMemoryVectorStore:
    """
    创建内存向量存储

    参数:
        model_name (str): 用于生成嵌入的模型名称

    返回:
        InMemoryVectorStore实例，用于存储和检索向量化的文本
    """
    embeddings = load_embeddings(model_name)
    return InMemoryVectorStore(embeddings)
```

### 提取关键词

在 `summary.py` 文件中，定义了 `SummaryChain` 类，用于从用户问题和聊天记录中提取关键词，并生成高效的搜索查询。

```python
from langchain.prompts import ChatPromptTemplate
from chains.models import load_model

class SummaryChain:
    """
    一个用于生成搜索查询的类。
    它从用户问题和聊天记录中提取关键词，并生成高效的搜索查询。
    """
    def __init__(self, model_name):
        """
        初始化 SummaryChain 类，并加载指定的语言模型。

        参数:
            model_name (str): 要加载的语言模型的名称。
        """
        self.llm = load_model(model_name)
        self.prompt = ChatPromptTemplate.from_template(
            "You are a professional assistant specializing in extracting keywords from user questions and chat histories. Extract keywords and connect them with spaces to output a efficient and precise search query. Be careful not answer the question directly, just output the search query.\n\nHistories: {history}\n\nQuestion: {question}"
        )
        self.chain = self.prompt | self.llm

    def invoke(self, input_data):
        """
        使用提供的输入数据调用链以生成搜索查询。

        参数:
            input_data (dict): 包含 'history' 和 'question' 键的字典。

        返回:
            str: 链生成的搜索查询。
        """
        return self.chain.invoke(input_data)
```

### 生成回答

在 `generate.py` 文件中，定义了 `GenerateChain` 类，根据用户问题、聊天记录和文档内容生成回答。

```python
from langchain.prompts import ChatPromptTemplate
from chains.models import load_model

class GenerateChain:
    """
    一个用于生成问答任务响应的类。
    它使用语言模型和提示模板来处理输入数据。
    """
    def __init__(self, model_name):
        """
        初始化 GenerateChain 类，并加载指定的语言模型。

        参数:
            model_name (str): 要加载的语言模型的名称。
        """
        self.llm = load_model(model_name)
        self.prompt = ChatPromptTemplate.from_template("You are an assistant for question-answering tasks. Use the following documents or chat histories to answer the question. If the documents or chat histories is empty, answer the question based on your own knowledge. If you don't know the answer, just say that you don't know.\n\nDocuments: {documents}\n\nHistories: {history}\n\nQuestion: {question}")

        self.chain = self.prompt | self.llm

    def invoke(self, input_data):
        """
        使用提供的输入数据调用链以生成响应。

        参数:
            input_data (dict): 包含 'documents'、'history' 和 'question' 键的字典。

        返回:
            str: 链生成的响应。
        """
        return self.chain.invoke(input_data)
```

## 连接智能体

在 LangGraph 中，智能体之间的连接通过 **状态图（graph）** 来实现，使用 **状态（state）** 存储交互的信息。图由节点（node）和边（edge）组成，节点表示智能体，边表示智能体之间的关系。在 `graph` 目录下定义了两个文件：`graph.py` 和 `graph_state.py`。

```shell
.
├── graph   # 图结构
│   ├── graph.py
│   └── graph_state.py
```

### 定义图的状态

在 `graph_state.py` 文件中，定义了 `GraphState` 类，用于存储图的状态信息。

```python
from typing import Literal, Annotated, Optional
from typing_extensions import TypedDict
from langgraph.graph.message import add_messages

class GraphState(TypedDict):
    """
    定义图状态的类型字典。
    用于表示图中的状态信息。
    """
    model_name: str  # 使用的模型名称
    type: Literal["websearch", "file", "chat"]  # 操作类型，包括联网搜索、上传文件和聊天
    messages: Annotated[list, add_messages]  # 消息列表，使用add_messages注解处理消息追加
    documents: Optional[list] = []  # 文档列表，默认为空列表
```

### 定义节点方法

在 `graph.py` 文件中，定义了多个方法，表示图的结构和行为，用于处理不同类型的请求。先引入所需要的包。

```python
import os
from langchain.schema import Document
from langchain_core.runnables import RunnableConfig
from langchain_community.document_loaders import TextLoader
from langchain_community.tools.tavily_search import TavilySearchResults
from langchain_text_splitters import MarkdownHeaderTextSplitter, RecursiveCharacterTextSplitter
from langgraph.graph.state import StateGraph, CompiledStateGraph, END
from langgraph.checkpoint.memory import MemorySaver
from marker.converters.pdf import PdfConverter
from marker.models import create_model_dict
from marker.output import text_from_rendered
from graph.graph_state import GraphState
from chains.summary import SummaryChain
from chains.generate import GenerateChain
```

#### 根据指令路由

`route_question()` 方法根据 `GraphState` 类中的操作类型将请求路由到相应的处理节点。

```python
def route_question(state: GraphState) -> str:
    """
    根据操作类型路由到相应的处理节点。

    参数:
        state (GraphState): 当前图的状态

    返回:
        str: 下一个要调用的节点名称
    """

    print("--- ROUTE QUESTION ---")
    if state['type'] == 'websearch':
        print("--- ROUTE QUESTION TO EXTRACT KEYWORDS ---")
        return "extract_keywords"
    if state['type'] == 'file':
        print("--- ROUTE QUESTION TO FILE PROCESS ---")
        return "file_process"
    elif state['type'] == 'chat':
        print("--- ROUTE QUESTION TO GENERATE ---")
        return "generate"
```

当然，也可以将路由交给 LLM 决定，只需要写好相应的提示词即可，例如下面的提示词将由 LLM 决定是进行知识库查询还是网络搜索。

```python
from langchain_core.output_parsers import JsonOutputParser

prompt = ChatPromptTemplate.from_template("You are an expert at routing a user question to a vectorstore or web search. Use the vectorstore for questions on LangChain and LangGraph. You do not need to be stringent with the keywords in the question related to these topics. Otherwise, use web-search. Give a binary choice 'web_search' or 'vectorstore' based on the question. Return the a JSON with a single key 'datasource' and no premable or explaination. Question to route: {question}")
router = prompt | llm | JsonOutputParser()

source = router.invoke({"question": question})
if source['datasource'] == 'web_search':
    # TODO: route to web search
elif source['datasource'] == 'vectorstore':
    # TODO: route to vectorstore
```

#### 生成回答

`generate()` 方法根据用户问题、聊天记录和文档内容生成回答。

```python
def generate(state: GraphState) -> GraphState:
    """
    根据文档和对话历史生成答案。

    参数:
        state (GraphState): 当前图的状态

    返回:
        state (GraphState): 返回添加了LLM生成内容的新状态
    """

    print("--- GENERATE ---")
    chain = GenerateChain(state["model_name"])
    messages = state["messages"]
    state["messages"] = chain.invoke({"question": messages[-1].content, "history": messages[:-1], "documents": state["documents"]})
    return state
```

#### 文件处理

`file_process()` 方法处理上传的文件，提取文本内容并进行词嵌入（embedding），然后将向量存储至内存数据库中。`config` 是一个字典，存储 LLM 运行时的配置参数，会在调用 LLM 时传入。

```python
def file_process(state: GraphState, config: RunnableConfig) -> GraphState:
    """
    处理文件。

    参数:
        state (GraphState): 当前图的状态
        config (RunnableConfig): 可运行配置

    返回:
        state (GraphState): 返回图状态，将文档添加 config 中的向量存储
    """

    print("--- FILE PROCESS ---")
    vector_store = config["configurable"]["vectorstore"]

    for doc in state["documents"]:
        file_path: str = doc.page_content
        if os.path.exists(file_path):
            split_docs: list[Document] = None
            if file_path.endswith(".txt") or file_path.endswith(".md"):
                # 处理文本或Markdown文件
                docs = TextLoader(file_path, autodetect_encoding=True).load()
                # 文本分割
                splitter = RecursiveCharacterTextSplitter(separators=["\n\n", "\n", " ", ".", ",", "\u200B", "\uff0c", "\u3001", "\uff0e", "\u3002", ""], chunk_size=1000, chunk_overlap=100, add_start_index=True)
                split_docs = splitter.split_documents(docs)
            else: 
                # 使用 marker-pdf 处理其他文件
                converter = PdfConverter(artifact_dict=create_model_dict())
                rendered = converter(file_path)
                docs, _, _ = text_from_rendered(rendered)
                splitter = MarkdownHeaderTextSplitter([("#", "Header 1"), ("##", "Header 2"), ("###", "Header 3")], strip_headers = False)
                split_docs = splitter.split_text(docs)
            # 将处理后的文档添加到向量存储中
            vector_store.add_documents(split_docs)
    return state
```

#### 提取关键词

`extract_keywords()` 方法从用户问题和聊天记录中提取关键词，并生成高效的搜索查询。

```python
def extract_keywords(state: GraphState, config: RunnableConfig) -> GraphState:
    """
    从问题中提取关键词。

    参数:
        state (GraphState): 当前图的状态
        config (RunnableConfig): 可运行配置

    返回:
        state (GraphState): 返回添加了提取关键词的新状态
    """

    print("--- EXTRACT KEYWORDS ---")
    chain = SummaryChain(state["model_name"])
    messages = state["messages"]
    query = chain.invoke({"question": messages[-1].content, "history": messages[:-1]})
    print(query.content)

    if state["type"] == "websearch":
        # 将生成的搜索查询添加到消息列表中，下一个节点将会使用
        state["messages"] = query
    elif state["type"] == "file":
        # 使用生成的搜索查询在向量数据库中搜索
        docs = config["configurable"]["vectorstore"].max_marginal_relevance_search(query.content)
        state["documents"] = docs
    return state
```

对于“上传文件”，“提取关键词”时已经进行了查询处理，可以直接进行“生成回答”；对于“网络搜索”，“提取关键词”进行搜索后，才能进行“生成回答”。执行路径不同，还需要进行判断。

```python
def decide_to_generate(state: GraphState) -> str:
    """
    决定是进行网络搜索还是直接生成回答。

    参数:
        state (GraphState): 当前图的状态

    返回:
        str: 下一个要调用的节点名称
    """

    if state["type"] == "websearch":
        print("--- DECIDE TO WEB SEARCH ---")
        return "websearch"
    elif state["type"] == "file":
        print("--- DECIDE TO GENERATE ---")
        return "generate"
```

#### 网络搜索

`web_search()` 方法使用 Tavily API 进行网络搜索，获取实时信息。

```python
def web_search(state: GraphState) -> GraphState:
    """
    基于问题进行网络搜索。

    参数:
        state (GraphState): 当前图的状态

    返回:
        state (GraphState): 返回添加了网络搜索结果的新状态
    """

    print("--- WEB SEARCH ---")
    web_search_tool = TavilySearchResults(k=3)
    documents = state["documents"]
    try:
        docs = web_search_tool.invoke({"query": state["messages"][-1].content})
        web_results = "\n".join([d["content"] for d in docs])
        web_results = Document(page_content=web_results)
        documents.append(web_results)
        state["documents"] = documents
    except:
        pass
    return state
```

### 定义图的结构

在 LangGraph 中，图的边分为普通边和条件边。普通边表示两个节点之间的直接连接，而条件边则根据特定条件决定是否连接两个节点。在 `create_graph()` 方法中定义了图的结构：`add_node()` 方法将定义的节点方法添加到图中；`add_edge()` 方法定义了节点之间的连接关系，也就是普通边；`add_conditional_edges()` 方法定义了条件边的连接关系；`set_conditional_entry_point()` 方法定义了图的条件入口节点。

![](4.png)

```python
def create_graph() -> CompiledStateGraph:
    """
    创建并配置状态图工作流。

    返回:
        CompiledStateGraph: 编译好的状态图
    """

    workflow = StateGraph(GraphState)
    # 添加节点
    workflow.add_node("websearch", web_search)
    workflow.add_node("extract_keywords", extract_keywords)
    workflow.add_node("file_process", file_process)
    workflow.add_node("generate", generate)
    # 添加边
    workflow.set_conditional_entry_point(
        route_question,
        {
            "extract_keywords": "extract_keywords",
            "generate": "generate",
            "file_process": "file_process",
        },
    )
    workflow.add_edge("file_process", "extract_keywords")
    workflow.add_conditional_edges(
        "extract_keywords",
        decide_to_generate,
        {
            "websearch": "websearch",
            "generate": "generate",
        },
    )
    workflow.add_edge("websearch", "generate")
    workflow.add_edge("generate", END)

    # 创建图，并使用 `MemorySaver()` 在内存中保存状态
    return workflow.compile(checkpointer=MemorySaver())
```

### 运行图

最后通过 `stream_graph_updates()` 方法运行图，并流式返回结果内容。

```python
def stream_graph_updates(graph: CompiledStateGraph, user_input: GraphState, config: dict):
    """
    流式处理图更新并返回最终结果。

    参数:
        graph (CompiledStateGraph): 编译好的状态图
        user_input (GraphState): 用户输入的状态
        config (dict): 配置字典

    返回:
        generator: 生成器对象，逐步返回图更新的内容
    """

    for chunk, _ in graph.stream(user_input, config, stream_mode="messages"):
        yield chunk.content
```

## 运行指南

### 在控制台中测试程序

在 `main.py` 文件中，定义了一个命令行程序，用户可以通过输入问题与智能体进行交互。

```python
import uuid
from dotenv import load_dotenv
from langchain.schema import Document
from langchain_core.messages import AIMessage, HumanMessage
from chains.models import load_vector_store
from graph.graph import create_graph, stream_graph_updates, GraphState

def main():
    # langchain.debug = True  # 启用langchain调试模式，可以获得如完整提示词等信息
    load_dotenv(verbose=True)  # 加载环境变量配置

    # 创建状态图以及对话相关的设置
    config = {"configurable": {"thread_id": uuid.uuid4().hex, "vectorstore": load_vector_store("nomic-embed-text")}}  
    state = GraphState(
        model_name="qwen2.5:7b",
        type="chat",
        documents=[Document(page_content="upload_files/test.pdf")],
    )
    graph = create_graph()

    # 对话
    while True:
        user_input = input("User: ")
        if user_input.lower() in ["exit", "quit"]:
            break
        state["messages"] = HumanMessage(user_input)
        # 流式获取AI的回复
        for answer in stream_graph_updates(graph, state, config):
            print(answer, end="")
        print()

    # 打印对话历史
    print("\nHistory: ")
    for message in graph.get_state(config).values["messages"]:
        if isinstance(message, AIMessage):
            prefix = "AI"
        else:
            prefix = "User"
        print(f"{prefix}: {message.content}")

if __name__ == "__main__":
    main()
```

### 使用 Streamlit 构建前端页面

在 `app.py` 文件中，使用 `Streamlit` 构建了一个简单的前端界面，用户可以通过输入框与智能体进行交互。完整的程序代码如下：

![](2.png)

```python
import uuid
import datetime
from dotenv import load_dotenv
from langchain.schema import Document
import streamlit as st
from streamlit_extras.bottom_container import bottom
from chains.models import load_vector_store
from graph.graph import create_graph, stream_graph_updates, GraphState

# 设置上传文件的存储路径
file_path = "upload_files/"
# 加载环境变量
load_dotenv(verbose=True)

def upload_pdf(file):
    """保存上传的文件并返回文件路径"""
    with open(file_path + file.name, "wb") as f:
        f.write(file.getbuffer())
        return file_path + file.name

# 设置页面配置信息
st.set_page_config(
    page_title="AI-Powerwd Assistant",
    page_icon="🌐",
    layout="wide"
)

# 初始化会话状态变量，创建图
if "graph" not in st.session_state:
    st.session_state.graph = create_graph()
# 初始化会话ID和向量存储
if "config" not in st.session_state:
    st.session_state.config = {"configurable": {"thread_id": uuid.uuid4().hex, "vectorstore": load_vector_store("nomic-embed-text")}}
# 初始化对话历史记录
if "history" not in st.session_state:
    st.session_state.history = []
# 初始化上传状态、模型名称和对话类型
if "settings" not in st.session_state:
    st.session_state.settings = {"uploaded": False, "model_name": "qwen2.5:7b", "type": "chat"}

# 显示应用标题
st.header("👽 AI-Powerwd Assistant")

# 定义可选的模型
model_options = {"通义千问 2.5 7B": "qwen2.5:7b", "DeepSeek R1 7B": "deepseek-r1:7b"}
with st.sidebar:
    # 侧边栏设置部分
    st.header("设置")
    # 模型选择下拉框
    st.session_state.settings["model_name"] = model_options[st.selectbox("选择模型", model_options, index=list(model_options.values()).index(st.session_state.settings["model_name"]))]

    st.divider()

    # 显示版本信息
    st.text(f"{datetime.datetime.now().strftime('%Y.%m.%d')} - ZHANG GAOXING")

# 定义对话类型选项
type_options = {"🤖 对话": "chat", "🔍 联网搜索": "websearch", "👾 代码模式": "code"}
question = None
with bottom():
    # 底部容器，包含工具选择、文件上传和输入框
    st.session_state.settings["type"] = type_options[st.radio("工具选择", type_options.keys(), horizontal=True, label_visibility="collapsed", index=list(type_options.values()).index(st.session_state.settings["type"]))]
    # 文件上传组件
    uploaded_file = st.file_uploader("上传文件", type=["pdf", "docx", "xlsx", "txt", "md"], accept_multiple_files=False, label_visibility="collapsed")
    # 聊天输入框
    question = st.chat_input('输入你要询问的内容')

# 显示历史对话内容
for message in st.session_state.history:
    with st.chat_message(message["role"]):
      st.markdown(message["content"])  

# 处理用户提问
if question: 
    # 显示用户问题
    with st.chat_message("user"):
        st.markdown(question)

    # 准备请求状态
    state = []
    if st.session_state.settings["type"] == "code":
        # 代码模式使用专门的代码模型
        state = {"model_name": "qwen2.5-coder:7b", "messages": [{"role": "user", "content": question}], "type": "chat", "documents": []}
    else:
        # 其他模式使用选择的模型
        state = {"model_name": st.session_state.settings["model_name"], "messages": [{"role": "user", "content": question}], "type": st.session_state.settings["type"], "documents": []}

    # 处理文件上传
    if uploaded_file:
        state["type"] = "file"
        if not st.session_state.settings["uploaded"]:
            # 保存上传的文件
            file_path = upload_pdf(uploaded_file)
            # 添加文档到请求
            state["documents"].append(Document(page_content=file_path))
            st.session_state.settings["uploaded"] = True

    # 获取AI回答并以流式方式显示
    answer = st.chat_message("assistant").write_stream(stream_graph_updates(st.session_state.graph, state, st.session_state.config))

    # 将对话保存到历史记录
    st.session_state.history.append({"role": "user", "content": question})
    st.session_state.history.append({"role": "assistant", "content": answer})
```
