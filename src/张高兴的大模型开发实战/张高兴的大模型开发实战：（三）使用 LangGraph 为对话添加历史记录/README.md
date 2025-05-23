[TOC]

在构建聊天机器人时，对话历史记录是提升用户体验的核心功能之一，用户希望机器人能够记住之前的对话内容，从而避免重复提问。LangGraph 是 LangChain 生态中一个工具，通过将应用逻辑组织成有向图（Graph）的形式，可以轻松实现对话历史的管理和复杂的对话流程。本文将通过一个示例，展示如何使用 LangGraph 实现这一功能。

在上一篇博客中提到，链（Chain）在 LangChain 中是一种基本的构建块，用于将多个 LLM 调用和工具调用链接在一起。然而，链在处理复杂、动态的对话流程时存在一些局限性，例如，链通常是线性的，这种线性结构只能按照预定义的顺序执行，限制了在对话中进行动态路由和条件分支的能力。LangGraph 的设计目标是提供一个更灵活、更强大的框架来构建复杂的智能体应用。

|  | LangGraph | LangChain |
| :-: | :-: | :-: |
| 核心设计 | 循环图结构：支持条件分支、循环和反馈机制，适合复杂多步骤任务。 | 线性流程（DAG）：以链式结构为主，适合线性任务（如文档检索、文本生成）。 |
| 控制能力 | 高度可控：通过节点（Node）和边（Edge）精细控制流程，支持条件逻辑和动态修改。 | 中等可控：依赖链式编排，灵活性较低，难以处理复杂循环或动态分支。 |
| 持久化与状态管理 | 内置持久化：支持状态检查点（Checkpoints），可中断/恢复任务，适合长期任务。 | 基础记忆功能：依赖对话历史记录，但无法持久化复杂状态或跨会话共享。 |
| 人在环（Human-in-the-Loop） | 深度支持：可在任意节点插入人工审核、干预，适合医疗、金融等需人工决策的场景。 | 弱支持：需手动集成人工干预逻辑，流程中断后难以恢复。 |
| 多代理（Multi-Agent） | 原生支持：通过共享状态实现多Agent协作，适合复杂任务拆分与协同。 | 较弱：需手动协调多个链，难以实现动态任务分配。 |
| 错误处理 | 容错性强：支持失败节点跳转或重试，流程可恢复。 | 基础重试：依赖单链重试，无法处理复杂流程中的错误传播。 |
| 适用场景 | 复杂多步骤任务、需人工干预的场景（如医疗诊断）、多Agent协作系统、长期任务（如持续对话） | 线性任务（文档检索、文本生成）、快速原型开发、简单对话系统 |
| 开发复杂度 | 中等：需定义节点、边和状态，但提供了灵活的编排能力。 | 低：开箱即用的链式结构，适合快速开发。 |

## 基础概念

LangGraph 的核心是 State Graph，它通过状态（State）、节点（Node）和边（Edge）的组合，定义对话的流程和逻辑。每个状态可以保存对话的上下文（如历史消息、总结等），节点定义了在不同状态下如何处理输入和生成输出，边定义了处理流程。

1. State（状态）
    用于存储对话中的临时数据，例如用户消息、模型响应、总结内容等。例如 `class State(MessagesState): messages: str` 表示一个状态，其中 `messages` 字段用于存储对话的具体信息。
2. Node（节点）
    定义了对话流程中的具体操作，通常是具体的函数，例如调用模型、判断是否需要总结、生成总结等。
3. Edge（边）
    用于连接不同的节点，定义了节点之间的关系和流程。边可以包含条件逻辑、循环、分支等，用于控制对话流程的走向。

我们来看一个最简单的示例，下图是一个 LangGraph 实现的聊天机器人。

![](1.png)

起始节点为 `__start__`，结束节点为 `__end__`，`chatbot` 表示调用大模型处理对话。`__start__` 节点存储了应用的 `State` 数据。节点之间带箭头的线段表示边，实线代表`普通边 →`，虚线代表`条件边 ⇢`，条件边根据当前的具体条件而选择哪一条边执行，选择不同的边，则到达的节点不同。

## 环境搭建与配置

在上一篇博客创建的 Python 虚拟环境中执行以下命令，安装需要的包：

```shell
pip install langgraph langgraph-checkpoint-postgres psycopg[binary,pool]
```

## 将对话历史存储至内存

在开始之前，先构建一个图，实现一个最简单的聊天机器人。

```python
from typing import Annotated
from typing_extensions import TypedDict
from langgraph.graph import StateGraph, START, END
from langgraph.graph.message import add_messages
from langchain_ollama import ChatOllama

class State(TypedDict):
    """存储对话状态信息"""
    messages: Annotated[list, add_messages]

def chatbot(state: State):
    """调用模型处理对话"""
    return {"messages": [llm.invoke(state["messages"])]}

llm = ChatOllama(model="qwen2.5:1.5b")

# 创建图
graph_builder = StateGraph(State)
graph_builder.add_node("chatbot", chatbot)  # 添加节点
graph_builder.add_edge(START, "chatbot")    # 添加边
graph_builder.add_edge("chatbot", END)
graph = graph_builder.compile()
```

使用下面的代码输出图的结构：

```python
png = graph.get_graph().draw_mermaid_png()
with open("chatbot.png", "wb") as f:
    f.write(png)
```

![](1.png)

接下来，使用 `graph.stream()` 方法执行图，即可开始对话。

```python
events = graph.stream({"messages": [{"role": "user", "content": "你可以做些什么？"}]})
for event in events:
    last_event = event
print("AI: ", last_event["messages"][-1].content)
```

下面使用 `MemorySaver` 将对话历史存储在内存中。

```python
from langgraph.checkpoint.memory import MemorySaver

checkpointer = MemorySaver()
# 创建图
# ...
graph = graph_builder.compile(checkpointer=checkpointer)
```

在对话时要记录对话历史，还需要在 `graph.stream()` 方法中传入 `config` 参数，`thread_id` 用于标识对话的唯一性，不同的对话 `thread_id` 不同。

```python
import uuid

config = {"configurable": {"thread_id": uuid.uuid4().hex}}
events = graph.stream({"messages": [{"role": "user", "content": "你好，我的名字是张三"}]}, config)
```

最后，我们将对话的代码封装成 `stream_graph_updates()` 方法，通过对话检测一下历史信息是否被正确保存。

```python
def stream_graph_updates(user_input: str, config: dict):
    """对话"""
    events = graph.stream({"messages": [{"role": "user", "content": user_input}]}, config, stream_mode="values")
    for event in events:
        last_event = event
    print("AI: ", last_event["messages"][-1].content)

if __name__ == "__main__":
    config = {"configurable": {"thread_id": uuid.uuid4().hex}}

    while True:
        user_input = input("User: ")    # 用户输入问题进行对话
        if user_input.lower() in ["exit", "quit"]:
            break
        stream_graph_updates(user_input, config)

    print("\nHistory: ")    # 输出对话历史
    for message in graph.get_state(config).values["messages"]:
        if isinstance(message, AIMessage):
            prefix = "AI"
        else:
            prefix = "User"
        print(f"{prefix}: {message.content}")
```

```shell
User: 你好，我的名字是张三
AI:  你好！很高兴认识你。有什么可以帮忙的吗？
User: 我叫什么名字
AI:  你的名字确实是“张三”。很高兴认识你！有什么问题或需要帮助的地方吗？
```

## 将对话历史存储至 PostgreSQL

对话历史存储至内存中，当应用关闭时，对话历史也会消失，有时无法满足持久化的需求。LangGraph 提供了一些数据库持久化方式，支持的数据库有 PostgreSQL、MongoDB、Redis。下面使用 PostgreSQL 数据库为例。在开始之前，执行以下命令创建一个 PostgreSQL 数据库：

```shell
psql -U postgres -c "CREATE DATABASE llm"
```

接着，在代码中替换 `MemorySaver` 为 `PostgresSaver`，连接并初始化数据库：

```python
from psycopg import Connection
from langgraph.checkpoint.postgres import PostgresSaver

DB_URI = "postgresql://postgres:YOUR_PASSW0RD@localhost:5432/llm"   # 记得替换数据库密码

conn = Connection.connect(DB_URI)   # 连接数据库
checkpointer = PostgresSaver(conn)
checkpointer.setup()    # 初始化数据库
```

使用数据库管理工具查看数据库，可以看到 LangGraph 在数据库初始化时帮我们创建了四张表：`checkpoint`、`checkpoint_blobs`、`checkpoint_writes`、`checkpoint_migrations`。

![](2.png)

完整的程序代码如下：

```python
import uuid
from typing import Annotated
from typing_extensions import TypedDict
from langgraph.graph import StateGraph, START, END
from langgraph.graph.message import add_messages
from langchain_ollama import ChatOllama
from langchain_core.messages import AIMessage, HumanMessage
from psycopg import Connection
from langgraph.checkpoint.postgres import PostgresSaver

class State(TypedDict):
    messages: Annotated[list, add_messages]

def chatbot(state: State):
    return {"messages": [llm.invoke(state["messages"])]}

DB_URI = "postgresql://postgres:%40Passw0rd@localhost:5432/llm"

llm = ChatOllama(model="qwen2.5:1.5b")

conn = Connection.connect(DB_URI)
checkpointer = PostgresSaver(conn)
checkpointer.setup()

graph_builder = StateGraph(State)
graph_builder.add_node("chatbot", chatbot)
graph_builder.add_edge(START, "chatbot")
graph_builder.add_edge("chatbot", END)

graph = graph_builder.compile(checkpointer=checkpointer)

def stream_graph_updates(user_input: str, config: dict):
    events = graph.stream({"messages": [{"role": "user", "content": user_input}]}, config, stream_mode="values")
    for event in events:
        last_event = event
    print("AI: ", last_event["messages"][-1].content)

if __name__ == "__main__":
    config = {"configurable": {"thread_id": uuid.uuid4().hex}}

    while True:
        user_input = input("User: ")
        if user_input.lower() in ["exit", "quit"]:
            break
        stream_graph_updates(user_input, config)

    print("\nHistory: ")
    for message in checkpointer.get(config)["channel_values"]["messages"]:
        if isinstance(message, AIMessage):
            prefix = "AI"
        else:
            prefix = "User"
        print(f"{prefix}: {message.content}")

    conn.close()
```
