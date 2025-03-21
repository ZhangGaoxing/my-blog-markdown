[TOC]

检索增强生成（Retrieval-Augmented Generation，RAG）是一种优化大型语言模型输出的方法，允许模型在生成回答前，从外部知识库中检索相关信息，而非仅依赖模型内部训练的知识。通过引用外部知识库的信息来生成更准确、实时且可靠的内容，并解决知识过时和幻觉的问题。下面将介绍使用 LangChain 和 Ollama 实现一个本地知识库应用。

## 基础概念

### 什么是 LangChain

LangChain 是一个大语言模型（LLM）**编程框架**，其目的是简化基于大语言模型的应用开发，统一不同大模型的调用方式，开发者无需关心底层 API 差异。

### 什么是 Ollama

Ollama 是一个开源的本地大语言模型**运行框架**，其目的是简化在本地设备上部署和运行大型语言模型的流程。它通过容器化管理和标准化接口，让用户能够快速、便捷地使用如 Qwen、Deepseek 等主流模型，无需复杂配置。

## 环境搭建与配置

### 安装 Ollama

访问 https://ollama.com/download 下载对应系统的安装文件。Windows 系统执行 `.exe` 文件安装，Linux 系统执行以下命令安装：

```shell
curl -fsSL https://ollama.com/install.sh | sh
```

访问 https://ollama.com/search 可以查看 Ollama 支持的模型。使用命令行可以下载并运行模型，例如运行 `Qwen2.5 7B` 模型：

```shell
ollama run qwen2.5
```

### 安装 LangChain

创建 Python 虚拟环境后，执行以下命令：

```shell
pip install langchain langchain-ollama langchain-chroma langchain-community chromadb unstructured jq
```

## 文档加载

知识库中的文档可以使用 JSON、TXT、PDF、Markdown、网页等格式的数据，只需要选择 LangChain 中合适的解析器即可。例如常见的有：
* `DirectoryLoader`：批量加载文件夹中的文档。
* `JSONLoader`：加载 JSON 格式的文档。
* `PDFPlumberLoader`：处理 PDF 中的内容。
* `WebBaseLoader`：爬取网页的内容。

下面使用上一篇博客爬取的新闻数据作为示例，展示如何将文本数据加载到程序中。

```json
# 数据示例
{"date": "2024年2月27日", "author": "党政办", "title": "各部门固定电话一览表", "content": "...", "url": "http://www.xzcit.cn/2018/0710/c2031a53936/page.htm"}
{"date": "2025年3月6日", "author": "保卫处", "title": "我校召开2025年安全工作大会", "content": "...", "url": "https://www.xzcit.cn/2025/0305/c275a56681/page.htm"}
{"date": "2025年3月3日", "author": "党委宣传部", "title": "我校举行2024-2025第二学期开学升国旗仪式暨“开学第一课”", "content": "...", "url": "https://www.xzcit.cn/2025/0303/c275a56667/page.htm"}
```

### 加载 JSON 数据

加载 JSON 数据可以使用 `JSONLoader` 加载，`json_lines=True` 表示 JSON 文件的格式为 `.jsonl`。

```python
from langchain_ollama.embeddings import OllamaEmbeddings
from langchain.document_loaders import JSONLoader

documents = JSONLoader(file_path='./documents/articles.jsonl', jq_schema='.', text_content=False, json_lines=True).load()
```

打印变量 `documents` 查看结果，`page_content` 为加载文档的内容。可以观察到，JSON 文件中的所有数据、所有字段都被加载到了 `documents` 变量中。

```shell
>>> documents[0]
Document(metadata={'source': 'D:\\GitHub\\langchain-ollama\\documents\\articles.jsonl', 'seq_num': 1}, page_content='{"date": "2024\\u5e742\\u670827\\u65e5", "author": "\\u515a\\u653f\\u529e", "title": "\\u5404\\u90e8\\u95e8\\u56fa\\u5b9a\\u7535\\u8bdd\\u4e00\\u89c8\\u8868", "content": "...", "url": "http://www.xzcit.cn/2018/0710/c2031a53936/page.htm"}')
```

有些时候并不是所有字段都是我们需要的，可以通过 `jq_schema` 参数指定需要的字段，下面给出了几种常见的示例：

```
JSON        -> [{"text": ...}, {"text": ...}, {"text": ...}]
jq_schema   -> ".[].text"

JSON        -> {"key": [{"text": ...}, {"text": ...}, {"text": ...}]}
jq_schema   -> ".key[].text"

JSON        -> ["...", "...", "..."]
jq_schema   -> ".[]"

JSON        -> {"...", "...", "..."}
jq_schema   -> "."
```

刚刚打印的变量 `documents` 中，字段 `metadata` 用于存储文档的元数据。在一些高级检索场景中，除了使用文本内容的向量表示进行相似性搜索，还可以将元数据作为辅助信息进行加权或筛选。下面的例子将 JSON 中的 `date`、`author`、`url` 字段添加至 `metadata` 中。

```python
def metadata_func(record: dict, metadata: dict) -> dict:
    """添加元数据"""
    metadata["title"] = record.get("title")
    metadata["author"] = record.get("author")
    metadata["date"] = record.get("date")
    metadata["source"] = record.get("url")
    return metadata

documents = JSONLoader(file_path='documents/articles.jsonl', jq_schema='.', content_key='content', metadata_func=metadata_func, json_lines=True).load()
```

再次打印变量 `documents` 查看结果，可以看到 `metadata` 中新增了 `title`、`author`、`date`、`source` 字段，`page_content` 中的内容变为了 JSON 中的 `content` 字段。

```shell
>>> documents[0]
Document(metadata={'source': 'http://www.xzcit.cn/2018/0710/c2031a53936/page.htm', 'seq_num': 1, 'title': '各部门固定电话一览表', 'author': '党政办', 'date': '2024年2月27日'}, page_content='...')
```

### 加载文件夹中的文档

更常见的情况，数据不会只在一个文档中，如果文档是分散在文件夹中的，可以使用 `DirectoryLoader` 加载，它会递归加载文件夹中的所有文件。

```python
from langchain.document_loaders import DirectoryLoader, TextLoader

text_loader_kwargs={'encoding': 'utf-8'}    # 设置文本加载器的参数
documents = DirectoryLoader(path='./documents', glob='**/*.txt', show_progress=True, use_multithreading=True, loader_cls=TextLoader, loader_kwargs=text_loader_kwargs).load()
```

如果加载的单个文档过长，还需要将长文本分割为合理长度的块（Chunk），避免超出 LLM 上下文窗口。`RecursiveCharacterTextSplitter` 按优先级顺序使用一系列分隔符来分割文本，分割有助于保持文本的逻辑结构，比如保留完整的句子或段落。这里设置了一系列分割符 `separators`，例如换行、空格、英文和中文标点符号。使用 `chunk_size` 指定了分割后的文本块的长度为 1000。并设置 `chunk_overlap` 相邻文本块之间的重叠长度为 20，设置重叠可以使信息不会因为在边界处被切断而丢失。

```python
from langchain.text_splitter import RecursiveCharacterTextSplitter

text_splitter = RecursiveCharacterTextSplitter(
    separators=[
        "\n\n",
        "\n",
        " ",
        ".",
        ",",
        "\u200B",  # Zero-width space
        "\uff0c",  # Fullwidth comma
        "\u3001",  # Ideographic comma
        "\uff0e",  # Fullwidth full stop
        "\u3002",  # Ideographic full stop
        "",
    ], chunk_size=1000, chunk_overlap=20, add_start_index=True)
split_docs = text_splitter.split_documents(documents)
```

### 文本向量化

大模型无法直接理解知识库当中的文本，我们需要将人类可读的文本数据转换成机器能够理解和处理的数值形式，这个过程叫做文本向量化。如果将大模型通过词条查找内容的过程比作查字典，那么文本向量化就是在编写字典。这里使用向量化的一种方式—— Embedding（嵌入），能够使向量捕捉词语之间的语义关系、语法相似性等信息。`OllamaEmbeddings` 是一个大语言模型的 Embedding 工具，将文本数据转换为向量表示，以便于后续的检索和生成。在使用之前还需要下载 Embedding 模型，例如在命令行中执行 `ollama pull nomic-embed-text`。向量化后还需要使用向量数据库进行存储，例如 `FAISS`、`Chroma` 等。

```python
from langchain_ollama.embeddings import OllamaEmbeddings
from langchain_chroma.vectorstores import Chroma

embeddings = OllamaEmbeddings(model="nomic-embed-text")
db = Chroma.from_documents(documents, embeddings, persist_directory="./embeddings")
```

等待这一过程完成后，试着检索文档，返回与查询语句最相似的文档。

```python
results = db.similarity_search("招聘公告")

for result in results:
    print("Title:", result.metadata["title"])
    print("Author:", result.metadata["author"])
    print("Date:", result.metadata["date"])
    print("Content:", result.page_content)
    print("------\n")
```

也可以使用向量进行检索，`k` 指定了返回结果的个数，`filter` 设置了元数据字段的搜索条件。

```python
query_vector = embeddings.embed_query("招聘公告")
results = db.similarity_search_by_vector(query_vector, k=1, filter={"author": "人事处"})
```

## 实现问答应用

首先引入相关的包并加载文档与模型。

```python
from langchain_ollama.llms import OllamaLLM
from langchain_ollama.embeddings import OllamaEmbeddings
from langchain_chroma import Chroma
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser
from langchain_core.runnables import RunnablePassthrough

model = OllamaLLM(model="qwen2.5:7b")
embeddings = OllamaEmbeddings(model="nomic-embed-text")
db = Chroma(embedding_function=embeddings, persist_directory='./embeddings')
retriever=db.as_retriever(search_kwargs={"k": 3})
```

接着使用 `ChatPromptTemplate` 定义与用户交互时使用的提示模板。这个模板包含了两个占位符 `{context}` 和 `{question}`，分别代表从知识库中检索到的相关内容和用户的提问。

```python
template = """你是徐州工业职业技术学院的专业助手，能够根据知识库中的新闻稿回答问题：
{context}
问题：{question}
回答应简洁且准确，避免编造信息。
"""
prompt = ChatPromptTemplate.from_template(template)
```

然后使用 LangChain 中的一个关键概念——**链式调用**，构建了一个数据处理管道：首先通过检索器获取相关上下文，然后将其与问题一起格式化成提示，接着发送给语言模型以生成回答，最后通过 `StrOutputParser` 将输出解析为字符串。

```python
chain = (
    {"context": retriever, "question": RunnablePassthrough()}
    | prompt
    | model
    | StrOutputParser()
)
```

最后使用 `invoke()` 方法执行管道，传入问题作为参数，即可得到回答。

```python
response = chain.invoke("学校最近一次招聘是在什么时候")
print(f"AI: {response}")
```

完整的程序如下：

```python
from langchain_ollama.llms import OllamaLLM
from langchain_ollama.embeddings import OllamaEmbeddings
from langchain_chroma import Chroma
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser
from langchain_core.runnables import RunnablePassthrough

model = OllamaLLM(model="qwen2.5:0.5b")
embeddings = OllamaEmbeddings(model="nomic-embed-text")
db = Chroma(embedding_function=embeddings, persist_directory='./embeddings')
retriever=db.as_retriever(search_kwargs={"k": 3})

template = """你是徐州工业职业技术学院的专业助手，能够根据知识库中的新闻稿回答问题：
{context}
问题：{question}
回答应简洁且准确，避免编造信息。
"""
prompt = ChatPromptTemplate.from_template(template)

chain = (
    {"context": retriever, "question": RunnablePassthrough()}
    | prompt
    | model
    | StrOutputParser()
)

def chat_with_system(question):
    response = chain.invoke(question)
    print(f"AI: {response}")
    return response

# 示例对话
if __name__ == "__main__":
    while True:
        user_input = input("You: ")
        if user_input.lower() in ["exit", "quit"]:
            break
        response = chat_with_system(user_input)
```
