代理是一种模型，它会持续调用各种工具，直到完成指定的任务为止。![Core agent loop diagram](https://mintcdn.com/langchain-5e9cc07a/jtty0O--UJOKG0nK/oss/images/core_agent_loop.svg?fit=max&auto=format&n=jtty0O--UJOKG0nK&q=85&s=4b4cbb497b6273758a565de1bc90ece0)
> [!NOTE]  
>**Agent = Model + Harness**
>harness的任务是: 在正确的时间为给定的任务提供正确的上下文。

“**Harness**”指的是围绕那个循环的所有元素：模型、提示语、工具，以及任何能够影响其行为的附加组件。
``` Python
	from langchain.agents import create_agent
	agent = create_agent(model="openai:gpt-5.5", tools=tools)
```

## 核心组件

![[Pasted image 20260627115909.png]]
### models

``` Python
	from langchain.agents import create_agent
	agent = create_agent(model="openai:gpt-5.5", tools=tools)
```

### Tools
``` python
from langchain.agents import create_agent

agent = create_agent(model="openai:gpt-5.5", tools=tools)
```

### System_prompt

``` python 
agent = create_agent(
    model="openai:gpt-5.5",
    tools=tools,
    system_prompt="You are a helpful assistant. Be concise and accurate.",
)
```

### Structured output

``` Python
from pydantic import BaseModel
from langchain.agents import create_agent


class Answer(BaseModel):
    summary: str
    confidence: float


agent = create_agent(model="openai:gpt-5.5", tools=tools, response_format=Answer)
result = agent.invoke({"messages": [{"role": "user", "content": "Summarize AI trends"}]})
result["structured_response"]  # Answer(summary=..., confidence=...)

```

## 调用

你可以通过发送消息来调用某个代理。实际上，这是通过向代理的 `State` 参数传递更新信息来实现的。所有代理都在其状态中保存了一系列的消息；要调用代理，只需传递一个新的消息，同时附带 `thread_id` 参数，这样代理就能保存并继续处理之前的对话历史。

```python
# 导入创建 Agent 的函数
from langchain.agents import create_agent

# 导入 uuid7，用于生成唯一的会话 ID（thread_id）
# 每一个 thread_id 都代表一段独立的对话
from langchain_core.utils.uuid import uuid7

# 导入内存检查点（Checkpoint）实现
# InMemorySaver 会将对话历史保存在内存中，程序退出后数据会丢失
from langgraph.checkpoint.memory import InMemorySaver


# 创建一个 Agent
agent = create_agent(
    # 指定使用的大语言模型
    model="openai:gpt-5.5",

    # Agent 可以调用的工具列表
    # 这里为空列表，因此 Agent 只能依赖模型自身回答问题
    tools=[],

    # 指定 Checkpointer，用于保存每轮对话历史
    # LangChain v1 的短期记忆就是依靠 Checkpointer 实现的
    checkpointer=InMemorySaver(),
)


# 创建 Agent 的运行配置（Runtime Config）
config = {
    "configurable": {
        # 为当前会话生成唯一的线程 ID
        # 同一个 thread_id 表示同一段对话
        # 后续只要继续使用这个 thread_id，
        # Agent 就能够读取之前保存的聊天记录
        "thread_id": str(uuid7())
    }
}


# ==========================
# 第一轮对话
# ==========================

result = agent.invoke(
    {
        "messages": [
            {
                # 消息角色
                "role": "user",

                # 用户输入
                "content": "What's the weather in San Francisco?"
            }
        ]
    },

    # 将 thread_id 一起传入
    # Agent 会自动保存这一轮聊天记录
    config=config,
)


# ==========================
# 第二轮对话
# ==========================

result = agent.invoke(
    {
        "messages": [
            {
                "role": "user",

                # 用户没有再次说明城市
                # 这里只问："明天呢？"
                "content": "What about tomorrow?"
            }
        ]
    },

    # 注意：
    # 这里继续使用相同的 thread_id
    # Agent 会自动读取上一轮历史消息
    config=config,
)

# 由于 thread_id 没有改变，
# Agent 能够理解：
#
# 第一轮：
# "What's the weather in San Francisco?"
#
# 第二轮：
# "What about tomorrow?"
#
# 实际理解为：
#
# "What about the weather in San Francisco tomorrow?"
#
# 这就是 LangChain v1 短期记忆（Short-term Memory）的工作方式。
#
# Agent 并不是自己记住了历史，
# 而是在调用模型之前，
# LangGraph 的 Checkpointer 自动读取历史消息，
# 将历史 Messages 与当前用户输入一起发送给模型。
```

> [!IMPORTANT]  
> **要使用 `thread_id` 持久化对话历史，Agent 必须配置一个 `checkpointer`（检查点存储器）。**
> 
> 如果 Agent 部署在 **LangSmith** 上，系统会自动为 Agent 配置一个 `checkpointer`，无需手动设置。
> 
> 如果是在本地运行，则需要显式传入一个 `checkpointer`，例如：
> 
> ```python
> from langgraph.checkpoint.memory import InMemorySaver
> 
> agent = create_agent(
>     model="openai:gpt-5.5",
>     tools=[],
>     checkpointer=InMemorySaver(),
> )
> ```
> 
> `checkpointer` 的作用是保存和恢复每个 `thread_id` 对应的会话历史。当后续请求使用相同的 `thread_id` 时，Agent 会自动读取之前保存的消息，将历史对话与当前输入一起发送给模型，从而实现短期记忆（Short-term Memory）。


**如果除了会话配置外，还需要在每次运行（Run）时向工具（Tools）和中间件（Middleware）传递一些运行时配置（例如用户 ID、API Key 或功能开关等），可以通过 `context` 参数进行传递，并与 `config` 一起使用。**

在创建 Agent 时，可以使用 `context_schema` 定义运行时上下文（Context）的数据结构，随后在工具（Tool）或中间件（Middleware）中通过 `runtime.context` 访问这些数据。

常见的运行时上下文包括：

- `user_id`：当前用户的唯一标识。
    
- `api_key`：调用第三方服务时使用的 API Key。
    
- `feature_flags`：功能开关，用于控制某些功能是否启用。
    
- `tenant_id`：多租户系统中的租户标识。
    
- `locale`：用户的语言或地区设置。
    

需要注意的是，**`context` 与 `config` 的作用并不相同：**

- **`config`**：用于控制 Agent 的运行行为，例如 `thread_id`、回调（Callbacks）、超时等运行配置。
    
- **`context`**：用于传递业务相关的运行时数据，供工具（Tools）和中间件（Middleware）在执行过程中读取和使用，而不会直接影响 Agent 的运行机制。

```python
# 导入 dataclass，用于定义运行时上下文的数据结构
from dataclasses import dataclass

# 导入创建 Agent 的函数
from langchain.agents import create_agent

# 导入 uuid7，用于生成唯一的会话 ID（thread_id）
from langchain_core.utils.uuid import uuid7

# 导入内存检查点，用于保存会话历史
from langgraph.checkpoint.memory import InMemorySaver


# 定义运行时上下文（Context）的数据结构
@dataclass
class Context:
    # 当前用户的唯一标识
    user_id: str


# 创建 Agent
agent = create_agent(
    # 指定使用的大模型
    model="openai:gpt-5.5",

    # Agent 可调用的工具列表
    tools=[],

    # 指定运行时上下文的数据结构
    context_schema=Context,

    # 配置 Checkpointer，用于保存和恢复会话历史
    checkpointer=InMemorySaver(),
)

# 调用 Agent
result = agent.invoke(
    {
        # 用户发送的消息
        "messages": [
            {
                "role": "user",
                "content": "What's the weather in San Francisco?"
            }
        ]
    },

    # Agent 的运行配置
    config={
        "configurable": {
            # 当前会话的唯一标识
            "thread_id": str(uuid7())
        }
    },

    # 本次运行的上下文数据
    context=Context(
        # 当前用户 ID，可在 Tool 和 Middleware 中通过 runtime.context.user_id 获取
        user_id="user-123"
    ),
)
```

`thread_id` 用于标识和隔离一段对话，它负责管理当前会话的**消息历史（Message History）** 和**检查点（Checkpoints）**。只要多次请求使用相同的 `thread_id`，Agent 就能够自动恢复之前的对话内容，实现会话连续性。

而 `context` 用于携带**每次运行（Per-run）** 所需的业务数据，例如用户 ID、API Key、z

二者职责不同，但相互配合，共同完成一次完整的 Agent 调用过程。

## Streaming

`invoke()` 会在 **Agent 完成整个运行（Run）后** 一次性返回最终结果。

如果 Agent 在执行过程中需要调用多个工具（Tool Calls），用户通常希望在最终结果返回之前，就能够看到执行进度，例如当前生成的内容、正在调用的工具以及工具返回的结果。

此时，可以使用 **Streaming（流式输出）**。流式输出会在 Agent 运行过程中持续返回中间状态（Intermediate State），让用户实时看到消息生成和工具调用过程，而无需等待整个任务结束。

```python
from langchain.messages import AIMessage, HumanMessage

# 创建事件流
stream = agent.stream_events(
    {
        "messages": [
            {
                "role": "user",
                "content": "Search for AI news and summarize the findings"
            }
        ]
    },
    # 使用 v3 版本事件协议
    version="v3",
)

# 持续读取事件流中的状态快照（Snapshot）
for snapshot in stream.values:

    # 每个 snapshot 都包含当前 Agent 的完整状态
    latest_message = snapshot["messages"][-1]

    # 如果当前消息包含文本内容
    if latest_message.content:

        # 用户消息
        if isinstance(latest_message, HumanMessage):
            print(f"User: {latest_message.content}")

        # Agent 回复
        elif isinstance(latest_message, AIMessage):
            print(f"Agent: {latest_message.content}")

    # 如果当前消息没有文本，而是工具调用
    elif latest_message.tool_calls:

        # 输出当前正在调用的工具名称
        print(
            f"Calling tools: "
            f"{[tc['name'] for tc in latest_message.tool_calls]}"
        )
```

如果想进一步了解 Streaming 支持的各种模式（Streaming Modes）、事件类型（Event Types）以及常见的前端展示方式（UI Patterns），可以参考 **Streaming** 章节。

# Configure the Harness（配置 Agent 运行框架）

`create_agent` 具有很强的扩展能力，而 **Middleware（中间件）** 是 LangChain v1 中实现扩展的核心机制。

每一个 Middleware 都只负责一个独立的功能（Concern），并会在 Agent 执行流程（Agent Loop）的合适阶段自动介入执行。多个 Middleware 之间彼此独立，可以自由组合，因此你只需要添加当前业务真正需要的功能，而无需引入其他无关组件。

对于一些常见场景，LangChain 官方已经提供了开箱即用的预置 Middleware（Prebuilt Middleware）；如果默认功能无法满足需求，也可以编写自定义 Middleware 来扩展 Agent 的能力。

## Agent Harness（Agent 运行框架）提供的能力

### 1. Execution Environment（执行环境）

Agent 最大的优势就是不仅能够思考，还能够真正执行任务。

执行环境主要负责提供：

- Tool
    
- 文件读写
    
- Shell
    
- Python
    
- Sandbox
    

示例：

```python
# 导入创建 Agent 的函数
from langchain.agents import create_agent

# 导入状态存储后端（State Backend）
# 用于保存 Agent 在运行过程中的状态数据
from deepagents.backends import StateBackend

# 导入文件系统中间件（Filesystem Middleware）
# 为 Agent 提供文件读写能力
from deepagents.middleware import FilesystemMiddleware


# 创建 Agent
agent = create_agent(

    # 指定使用的大语言模型
    model="openai:gpt-5.5",

    # Agent 可调用的工具列表
    tools=[search],

    # 配置 Agent 使用的中间件
    middleware=[

        # 为 Agent 添加文件系统能力
        FilesystemMiddleware(

            # 指定文件系统使用的状态存储后端
            # StateBackend 用于保存 Agent 创建、修改的文件及相关状态
            backend=StateBackend()
        )
    ],
)
```

这里为 Agent 添加了文件系统能力，使 Agent 可以跨多轮对话读取和保存文件。

### 2. Context Management（上下文管理）

模型能够处理的 Token 数量是有限的。

随着 Agent 持续运行：

```
历史消息
      ↓
工具返回结果
      ↓
推理过程
      ↓
越来越长
      ↓
Context Window 被占满
```

因此官方提供了一系列 Middleware 来管理上下文。

例如：

```python
# 配置 Agent 使用的中间件
middleware=[

    # 为 Agent 提供文件系统能力
    # 支持读取、创建、修改和保存文件
    FilesystemMiddleware(...),

    # 自动压缩历史对话和中间推理过程
    # 防止 Context Window 被过长的历史消息占满
    SummarizationMiddleware(...),

    # 为 Agent 加载长期记忆
    # 例如从 AGENTS.md 或数据库中读取长期保存的信息
    MemoryMiddleware(...),

    # 为 Agent 加载技能（Skills）
    # 根据当前任务按需加载技能，而不是一次性全部加载
    SkillsMiddleware(...),
]
```

各组件作用如下：

- FilesystemMiddleware：保存文件。
    
- SummarizationMiddleware：自动压缩历史对话。
    
- MemoryMiddleware：加载长期记忆。
    
- SkillsMiddleware：按需加载技能，而不是一次加载全部内容。

### 3. Planning and Delegation（规划与任务委派）

复杂任务通常需要拆分成多个子任务。

例如：

```
用户需求
      │
      ▼
Main Agent
      │
 ┌────┴────┐
 ▼         ▼
Research  Coding
Agent      Agent
 │          │
 └────┬─────┘
      ▼
汇总结果
```

示例：

```python
# 配置 Agent 使用的中间件
middleware=[

    # 为 Agent 提供文件系统能力
    # 支持跨多轮对话读取、创建、修改和保存文件
    FilesystemMiddleware(...),

    # 为 Agent 添加任务规划能力
    # 将复杂任务拆分为多个待办事项（Todo），并跟踪执行进度
    TodoListMiddleware(),

    # 为 Agent 添加子 Agent（Sub Agent）能力
    # 将复杂任务委派给多个子 Agent 独立执行，再汇总执行结果
    SubAgentMiddleware(...),
]
```

这样主 Agent 可以专注于协调，而不是亲自完成所有工作。
### 4. Name your agent（为 Agent 命名）

可以为 Agent 指定一个名称：

```python
agent = create_agent(
    model="openai:gpt-5.5",
    tools=tools,
    name="research_assistant"
)
```

这个名称主要用于：

- 多 Agent 系统
    
- LangGraph
    
- Sub Agent
    
- 调试日志
    

方便区分不同 Agent。

### 5. Fault Tolerance（容错）

生产环境中常见问题：

- API 超时
    
- 网络异常
    
- Rate Limit
    
- 模型不可用
    

官方提供：

```python
# 配置 Agent 使用的中间件
middleware=[

    # 模型调用失败时自动重试
    # 最多重试 3 次，例如处理网络异常、API 超时、Rate Limit 等问题
    ModelRetryMiddleware(
        max_retries=3
    ),

    # 工具调用失败时自动重试
    # 最多重试 2 次，例如处理工具执行异常或临时错误
    ToolRetryMiddleware(
        max_retries=2
    ),
]
```

这样可以显著提高 Agent 的稳定性。

### 6. Guardrails（安全护栏）

安全策略最好不要依赖 Prompt，而应该由程序保证。

例如：

```python
middleware=[
    PIIMiddleware("email")
]
```

表示：

所有数据在进入模型之前都会自动检测 Email。

如果发现隐私信息，可以：

- 删除
    
- 脱敏
    
- 拒绝继续执行
### 7. Steering（人工干预）

Human-in-the-loop 是生产级 Agent 非常重要的能力。

例如：

```python
middleware=[
    HumanInTheLoopMiddleware(
        interrupt_on={
            "write_file": True
        }
    )
]
```

表示：

当 Agent 调用 `write_file` 工具时：

```
Agent
    │
    ▼
准备写文件
    │
    ▼
暂停执行
    │
    ▼
等待人工确认
    │
 ┌──┴───────┐
 │          │
批准      拒绝
 │          │
 ▼          ▼
继续执行   结束
```

只有用户确认之后，Agent 才会继续执行写文件操作。
### 小结

LangChain v1 将 Agent 的扩展能力统一抽象为 **Middleware**。

不同 Middleware 分别负责：

|Middleware 类型|作用|
|---|---|
|Filesystem|文件系统|
|Memory|长期记忆|
|Summarization|历史摘要|
|Skills|技能加载|
|TodoList|任务规划|
|SubAgent|多 Agent 协作|
|Retry|自动重试|
|PII|隐私保护|
|Human-in-the-loop|人工审批|

这种设计使得 Agent 具有高度模块化的能力，开发者可以根据实际业务需求自由组合 Middleware，快速构建功能丰富且可扩展的智能体系统。