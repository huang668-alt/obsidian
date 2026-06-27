工具（Tools）扩展了 Agent 的能力，使其不仅能够生成文本，还能够：  
  
- 获取实时数据（Fetch Real-time Data）  
- 执行代码（Execute Code）  
- 查询外部数据库（Query External Databases）  
- 在现实世界中执行操作（Take Actions in the World）  
  
从底层实现来看，**Tool 本质上是一个可调用函数（Callable Function）**，它具有定义明确的输入（Inputs）和输出（Outputs），并会传递给聊天模型（Chat Model）。  
  
模型会根据当前对话上下文（Conversation Context）自主决定：  
  
- 是否需要调用某个工具（Whether to Invoke a Tool）  
- 调用哪个工具（Which Tool to Call）  
- 传递哪些输入参数（What Input Arguments to Provide）  
  
有关模型如何处理工具调用（Tool Calls）的详细说明，请参阅 **Tool calling（工具调用）** 文档。  
  
你还可以借助 **LangSmith** 对工具调用过程进行追踪（Trace Tool Calls），并调试执行过程中出现的错误。  
  
请按照 **Tracing Quickstart（追踪快速入门）** 完成 Tracing（追踪）配置。  
  
此外，我们建议配置 **LangSmith Engine**。  
  
它能够：  
  
- 监控你的 Traces（追踪记录）  
- 自动检测问题（Detect Issues）  
- 提供修复建议（Propose Fixes）

# Create tools（创建工具）

## Basic tool definition（基本工具定义）

创建工具最简单的方式是使用 `@tool` 装饰器。

默认情况下，**函数的 Docstring（文档字符串）** 会作为工具的描述（Description），帮助模型理解**什么时候应该使用该工具**。

```python
from langchain.tools import tool

@tool
def search_database(query: str, limit: int = 10) -> str:
    """Search the customer database for records matching the query.

    Args:
        query: Search terms to look for
        limit: Maximum number of results to return
    """
    return f"Found {limit} results for '{query}'"
```

**Type Hints（类型注解）是必须的**，因为它们用于定义工具的输入 Schema（Input Schema）。

Docstring 应当做到：

- 信息明确（Informative）
- 简洁易懂（Concise）

这样有助于模型理解工具的用途。

---

### Server-side tool use（服务端工具）

部分聊天模型内置了工具，例如：

- Web Search（网页搜索）
- Code Interpreter（代码解释器）

这些工具会在**服务端（Server-side）**执行。

详细内容请参阅 **Server-side Tool Use（服务端工具调用）**。

---

### Tool 命名建议

建议工具名称使用 **snake_case（蛇形命名）**。

例如：

```text
web_search
```

而不是：

```text
Web Search
```

原因是：

部分模型提供商对于包含：

- 空格（Spaces）
- 特殊字符（Special Characters）

的工具名称可能无法正常处理，甚至直接返回错误。

为了获得更好的兼容性，建议工具名称只包含：

- 字母（Alphanumeric Characters）
- 下划线（_）
- 连字符（-）

---

## Customize tool properties（自定义工具属性）

### Custom tool name（自定义工具名称）

默认情况下，工具名称来自函数名（Function Name）。

如果需要更具描述性的名称，可以手动指定。

```python
@tool("web_search")  # 自定义工具名称
def search(query: str) -> str:
    """Search the web for information."""
    return f"Results for: {query}"

print(search.name)

# web_search
```

---

### Custom tool description（自定义工具描述）

你也可以覆盖自动生成的工具描述，以便为模型提供更加清晰的指导。

```python
@tool(
    "calculator",
    description="Performs arithmetic calculations. Use this for any math problems."
)
def calc(expression: str) -> str:
    """Evaluate mathematical expressions."""
    return str(eval(expression))
```

---

## Advanced schema definition（高级 Schema 定义）

对于复杂输入，可以使用 **Pydantic Model** 或 **JSON Schema** 定义工具参数。

下面展示 Pydantic 示例。

```python
from pydantic import BaseModel, Field
from typing import Literal

class WeatherInput(BaseModel):
    """Input for weather queries."""

    location: str = Field(
        description="City name or coordinates"
    )

    units: Literal["celsius", "fahrenheit"] = Field(
        default="celsius",
        description="Temperature unit preference"
    )

    include_forecast: bool = Field(
        default=False,
        description="Include 5-day forecast"
    )


@tool(args_schema=WeatherInput)
def get_weather(
    location: str,
    units: str = "celsius",
    include_forecast: bool = False
) -> str:
    """Get current weather and optional forecast."""

    temp = 22 if units == "celsius" else 72

    result = (
        f"Current weather in {location}: "
        f"{temp} degrees {units[0].upper()}"
    )

    if include_forecast:
        result += "\nNext 5 days: Sunny"

    return result
```

使用 Pydantic 可以：

- 定义复杂输入对象
- 添加字段描述（Field Description）
- 设置默认值（Default Value）
- 限制字段取值（如 `Literal`）
- 自动完成输入校验（Validation）

---

## Reserved argument names（保留参数名称）

以下参数名称属于 **LangChain 保留名称（Reserved Names）**，**不能作为工具参数使用**。

否则将在运行时（Runtime）抛出错误。

| 参数名 | 用途 |
|--------|------|
| `config` | 保留用于向工具内部传递 `RunnableConfig`。 |
| `runtime` | 保留用于 `ToolRuntime` 参数（访问状态、上下文以及 Store）。 |

如果需要访问运行时信息，应使用 **ToolRuntime** 参数，而不要自己定义名为：

- `config`
- `runtime`

的函数参数。

如果你正在使用：

- `InjectedState`
- `InjectedStore`
- `get_runtime()`
- `InjectedToolCallId`

请参阅 **Migrate from older injection patterns（从旧版注入方式迁移）** 文档。


# Access context（访问上下文）

当工具能够访问运行时信息（Runtime Information）时，它们的能力将得到极大增强，例如：

- 对话历史（Conversation History）
- 用户数据（User Data）
- 持久化内存（Persistent Memory）

本节介绍如何在工具内部访问和更新这些信息。

工具可以通过 **`ToolRuntime`** 参数访问运行时上下文。

`ToolRuntime` 提供了以下能力：

| 组件 | 描述 | 使用场景 |
|------|------|---------|
| **State** | 短期记忆（Short-term Memory），仅存在于当前对话，包括消息历史、计数器、自定义字段等可变数据。 | 获取聊天历史、统计 Tool 调用次数等。 |
| **Context** | 调用时传入的不可变配置（Immutable Configuration），例如用户 ID、Session 信息等。 | 根据用户身份实现个性化回复。 |
| **Store** | 长期记忆（Long-term Memory），持久化存储，可跨多个会话保存数据。 | 保存用户偏好、知识库等。 |
| **Stream Writer** | 在工具执行期间实时输出更新。 | 向用户展示长时间任务的执行进度。 |
| **Execution Info** | 当前执行的信息，包括 Thread ID、Run ID、重试次数等。 | 获取执行标识，根据重试状态调整行为。 |
| **Server Info** | 在 LangGraph Server 上运行时提供的服务器元数据，例如 Assistant ID、Graph ID、认证用户等。 | 获取 Assistant ID、Graph ID 或当前登录用户。 |
| **Config** | 当前执行对应的 `RunnableConfig`。 | 获取 Callback、Tag、Metadata 等。 |
| **Tool Call ID** | 当前 Tool 调用的唯一标识。 | 日志记录、关联 Tool 调用和模型响应。 |

---

## Short-term memory（State，短期记忆）

State 表示**当前会话生命周期内**存在的短期记忆。

其中包括：

- 消息历史（Messages）
- 你在 Graph State 中定义的任何自定义字段。

只需在 Tool 参数中添加：

```python
runtime: ToolRuntime
```

即可访问 State。

该参数会：

- 自动注入（Automatically Injected）
- 对 LLM 完全隐藏（Hidden from the LLM）

因此，它不会出现在 Tool Schema 中。

---

### Access state（读取 State）

工具可以通过：

```python
runtime.state
```

访问当前会话状态。

```python
from langchain.tools import tool, ToolRuntime
from langchain.messages import HumanMessage

@tool
def get_last_user_message(runtime: ToolRuntime) -> str:
    """Get the most recent message from the user."""

    messages = runtime.state["messages"]

    # 找到最后一条 HumanMessage
    for message in reversed(messages):
        if isinstance(message, HumanMessage):
            return message.content

    return "No user messages found"


# 读取自定义 State 字段
@tool
def get_user_preference(
    pref_name: str,
    runtime: ToolRuntime
) -> str:
    """Get a user preference value."""

    preferences = runtime.state.get(
        "user_preferences",
        {}
    )

    return preferences.get(pref_name, "Not set")
```

对于上述示例，模型只能看到：

```python
pref_name
```

而不会看到：

```python
runtime
```

因为 `runtime` 对模型是隐藏的。

---

### Update state（更新 State）

使用 **`Command`** 可以更新 Agent 的 State。

适用于需要修改自定义状态字段的工具。

更新 State 时，应同时返回一个 `ToolMessage`，这样模型才能看到 Tool 的执行结果。

```python
from langchain.agents import AgentState
from langchain.messages import ToolMessage
from langchain.tools import ToolRuntime, tool
from langgraph.types import Command


class CustomState(AgentState):
    user_name: str


@tool
def set_user_name(
    new_name: str,
    runtime: ToolRuntime[None, CustomState]
) -> Command:
    """Set the user's name in the conversation state."""

    return Command(
        update={
            "user_name": new_name,
            "messages": [
                ToolMessage(
                    content=f"User name set to {new_name}.",
                    tool_call_id=runtime.tool_call_id,
                )
            ],
        }
    )
```

当 Tool 更新 State 时，建议为这些字段定义 **Reducer（归约器）**。

原因是：

LLM 可能会**并行调用多个 Tool**。

如果多个 Tool 同时修改同一个 State 字段，Reducer 将决定如何解决这些冲突。

---

## Context（上下文）

Context 用于保存**调用时传入的不可变配置（Immutable Configuration）**。

适用于：

- 用户 ID
- Session 信息
- 应用配置

这些数据在一次会话期间不会发生变化。

需要注意：

```python
thread_id
```

用于标识一个会话（Conversation）。

它负责：

- 消息历史
- Checkpoint

而：

```python
context
```

则负责：

- 当前运行所需的数据（Per-run Data）

例如：

- Tool
- Middleware

都会读取它。

生产环境通常同时传入：

- 一个固定的 `thread_id`
- 每次调用都传入 `context`

通过：

```python
runtime.context
```

访问 Context。

下面示例展示了如何与 `thread_id` 一起使用。

```python
from dataclasses import dataclass

from langchain.agents import create_agent
from langchain.tools import tool, ToolRuntime
from langchain_core.utils.uuid import uuid7
from langchain_openai import ChatOpenAI


USER_DATABASE = {
    "user123": {
        "name": "Alice Johnson",
        "account_type": "Premium",
        "balance": 5000,
        "email": "alice@example.com",
    },
    "user456": {
        "name": "Bob Smith",
        "account_type": "Standard",
        "balance": 1200,
        "email": "bob@example.com",
    },
}


@dataclass
class UserContext:
    user_id: str


@tool
def get_account_info(
    runtime: ToolRuntime[UserContext]
) -> str:
    """Get the current user's account information."""

    user_id = runtime.context.user_id

    if user_id in USER_DATABASE:
        user = USER_DATABASE[user_id]

        return (
            f"Account holder: {user['name']}\n"
            f"Type: {user['account_type']}\n"
            f"Balance: ${user['balance']}"
        )

    return "User not found"


model = ChatOpenAI(model="openai:gpt-5.5")

agent = create_agent(
    model,
    tools=[get_account_info],
    context_schema=UserContext,
    system_prompt="You are a financial assistant.",
)

result = agent.invoke(
    {
        "messages": [
            {
                "role": "user",
                "content": "What's my current balance?"
            }
        ]
    },
    config={
        "configurable": {
            "thread_id": str(uuid7())
        }
    },
    context=UserContext(user_id="user123"),
)
```

---

## Long-term memory（Store，长期记忆）

`BaseStore` 提供持久化存储能力。

与 **State（短期记忆）** 不同：

Store 中保存的数据可以跨多个会话长期存在。

通过：

```python
runtime.store
```

访问 Store。

Store 使用：

```
(namespace, key)
```

组织数据。

生产环境建议使用：

- `PostgresStore`

而不是：

- `InMemoryStore`

具体配置请参考 Memory 文档。

```python
from typing import Any
from langgraph.store.memory import InMemoryStore
from langchain.agents import create_agent
from langchain.tools import tool, ToolRuntime
from langchain_openai import ChatOpenAI

# 读取长期记忆
@tool
def get_user_info(user_id: str, runtime: ToolRuntime) -> str:
    """Look up user info."""

    store = runtime.store

    user_info = store.get(("users",), user_id)

    return (
        str(user_info.value)
        if user_info
        else "Unknown user"
    )


# 保存长期记忆
@tool
def save_user_info(
    user_id: str,
    user_info: dict[str, Any],
    runtime: ToolRuntime
) -> str:
    """Save user info."""

    store = runtime.store

    store.put(("users",), user_id, user_info)

    return "Successfully saved user info."
```

这样，即使新的对话开始，也能够继续读取之前保存的数据。

---

## Stream writer（流式输出）

可以通过：

```python
runtime.stream_writer
```

在 Tool 执行期间实时输出更新。

适用于：

- 长时间运行的任务
- 向用户展示执行进度

```python
from langchain.tools import tool, ToolRuntime

@tool
def get_weather(
    city: str,
    runtime: ToolRuntime
) -> str:
    """Get weather for a given city."""

    writer = runtime.stream_writer

    writer(f"Looking up data for city: {city}")
    writer(f"Acquired data for city: {city}")

    return f"It's always sunny in {city}!"
```

如果使用：

```python
runtime.stream_writer
```

则 Tool 必须运行在 **LangGraph Execution Context** 中。

详情请参考 **Streaming** 文档。

---

## Execution info（执行信息）

通过：

```python
runtime.execution_info
```

可以访问当前执行的信息。

包括：

- Thread ID
- Run ID
- Retry 次数

```python
from langchain.tools import tool, ToolRuntime

@tool
def log_execution_context(runtime: ToolRuntime) -> str:
    """Log execution identity information."""

    info = runtime.execution_info

    print(f"Thread: {info.thread_id}, Run: {info.run_id}")
    print(f"Attempt: {info.node_attempt}")

    return "done"
```

需要：

```
deepagents>=0.5.0
```

或：

```
langgraph>=1.1.5
```

---

## Server info（服务器信息）

如果 Tool 运行在 **LangGraph Server** 上，可以通过：

```python
runtime.server_info
```

获取服务器相关信息。

包括：

- Assistant ID
- Graph ID
- 当前认证用户

```python
from langchain.tools import tool, ToolRuntime

@tool
def get_assistant_scoped_data(
    runtime: ToolRuntime
) -> str:
    """Fetch data scoped to the current assistant."""

    server = runtime.server_info

    if server is not None:
        print(
            f"Assistant: {server.assistant_id}, "
            f"Graph: {server.graph_id}"
        )

        if server.user is not None:
            print(f"User: {server.user.identity}")

    return "done"
```

如果 Tool 并未运行在 LangGraph Server（例如本地开发或测试），则：

```python
runtime.server_info
```

的值为：

```python
None
```

同样需要：

```
deepagents>=0.5.0
```

或：

```
langgraph>=1.1.5
```

最后，官方建议阅读：

**Migrate from older injection patterns（从旧版注入方式迁移）**，了解如何从旧版运行时注入方式迁移到 `ToolRuntime`。


# Tool execution（工具执行）

在 LangChain 中，Tool（工具）通常由 **Agent** 使用（例如通过 `create_agent` 创建）。

Tool 的错误处理（Error Handling）通常通过 **Middleware（中间件）** 来配置。

如果你使用 **LangGraph Workflow**，则 Tool 的执行由 **ToolNode** 负责。

关于 Graph API 的使用方式，以及 Tool 如何访问：

- 当前 Graph State
- 当前运行（Run）上下文（Context）

请参考 **ToolNode** 文档。

---

## Tool return values（Tool 返回值）

Tool 可以返回不同类型的数据，根据你的需求选择合适的返回方式。

可以返回：

- 字符串（String）
- 对象（Object）
- Command（修改 State）
- 多模态内容（Multimodal Content）

---

## Return a string（返回字符串）

如果 Tool 只是返回一段普通文本，让模型继续阅读并推理，可以直接返回字符串。

```python
from langchain.tools import tool

@tool
def get_weather(city: str) -> str:
    """Get weather for a city."""
    return f"It is currently sunny in {city}."
```

### 行为（Behavior）

返回字符串后：

- 自动转换为 `ToolMessage`
- 模型读取这段文本
- 再决定下一步该如何回复

不会修改 Agent 的 State。

除非：

- 后续模型修改
- 或其他 Tool 修改

适用于：

> 返回的是自然语言文本（Human-readable Text）。

---

## Return an object（返回对象）

如果 Tool 返回的是结构化数据，可以直接返回对象（例如 `dict`）。

```python
from langchain.tools import tool

@tool
def get_weather_data(city: str) -> dict:
    """Get structured weather data for a city."""
    return {
        "city": city,
        "temperature_c": 22,
        "conditions": "sunny",
    }
```

### 行为（Behavior）

返回对象后：

- 自动序列化（Serialize）
- 作为 Tool Output 返回给模型

模型可以直接读取：

```text
city
temperature_c
conditions
```

等字段进行推理。

同样：

不会修改 Graph State。

适用于：

> 后续推理需要明确字段，而不是自由文本。

---

## Return multimodal content（返回多模态内容）

Tool 不仅可以返回文本，还可以返回：

- 图片
- 音频
- 视频
- 文本 + 图片

前提是模型支持对应模态。

例如：

```python
from langchain.tools import tool

@tool
def capture_screenshot() -> list[dict]:
    """Capture a screenshot of the current page."""

    return [
        {
            "type": "text",
            "text": "Screenshot of the current page:"
        },
        {
            "type": "image",
            "url": "https://example.com/page.png"
        },
    ]
```

### 行为（Behavior）

返回值会转换成：

```text
ToolMessage
```

其中包含：

- Text Block
- Image Block
- Audio Block
- Video Block

执行完成后可以通过：

```python
message.content_blocks
```

读取标准化后的 Content Block。

需要注意：

模型必须支持你返回的数据类型。

例如：

- Image
- Audio
- Video

具体要求请参考：

- Multimodal Messages
- Multimodal Tool Content

---

## Return a Command（返回 Command）

如果 Tool 不只是返回数据，而是需要**修改 Agent State**，则应该返回：

```python
Command
```

例如：

修改用户偏好、

修改会话状态、

修改 Agent State。

建议同时返回：

```text
ToolMessage
```

这样模型才能知道 Tool 是否执行成功。

```python
from langchain.messages import ToolMessage
from langchain.tools import ToolRuntime, tool
from langgraph.types import Command

@tool
def set_language(
    language: str,
    runtime: ToolRuntime
) -> Command:
    """Set the preferred response language."""

    return Command(
        update={
            "preferred_language": language,
            "messages": [
                ToolMessage(
                    content=f"Language set to {language}.",
                    tool_call_id=runtime.tool_call_id,
                )
            ],
        }
    )
```

### 行为（Behavior）

Command 会：

- 使用 `update` 修改 State
- 后续所有节点都能读取新的 State
- 并发修改同一字段时，应使用 Reducer 解决冲突

适用于：

> Tool 除了返回数据，还需要修改 Agent 状态。

---

## Return directly from a tool（直接返回 Tool 结果）

默认情况下：

```
用户
   ↓
LLM
   ↓
Tool
   ↓
LLM
   ↓
最终回复
```

Tool 返回结果后，还会再次交给 LLM。

如果：

```python
return_direct=True
```

Agent 会直接结束流程。

```python
from langchain.agents import create_agent
from langchain.tools import tool
from langchain_openai import ChatOpenAI

@tool(return_direct=True)
def fetch_order_status(order_id: str) -> str:
    """Fetch the current status of a customer order."""

    return (
        f"Order {order_id} is shipped "
        "and will arrive in 2 days."
    )


agent = create_agent(
    ChatOpenAI(model="openai:gpt-5.5"),
    tools=[fetch_order_status],
)
```

例如：

用户：

> 查询订单状态

流程变成：

```
用户
    ↓
LLM
    ↓
Tool
    ↓
直接返回用户
```

不会再次调用 LLM。

---

### Behavior

Tool 正常执行。

返回值仍然包装成：

```
ToolMessage
```

但是：

Agent 不会继续循环。

直接把 Tool 返回给用户。

---

### 多 Tool 调用时

如果一次调用了多个 Tool：

只有**全部 Tool 都设置了：**

```python
return_direct=True
```

才会直接结束。

否则：

仍然继续走 Agent Loop。

---

### 什么时候适合使用？

适用于：

✅ Tool 已经返回最终答案

例如：

- 查询订单
- 查询天气
- 查询物流
- 查询数据库

无需模型继续总结。

还适用于：

- 节省一次 LLM 调用
- 保证输出完全一致
- 不希望模型重新润色结果

---

### 不适合

如果 Tool 返回的是：

- 大量数据
- 需要总结
- 需要推理
- 需要继续调用其他 Tool

不要使用：

```python
return_direct=True
```

---

## Error handling（错误处理）

Tool 的异常推荐通过 **Middleware（中间件）** 统一处理。

例如：

```python
from collections.abc import Callable

from langchain.agents import create_agent
from langchain.agents.middleware import wrap_tool_call
from langchain.messages import ToolMessage
from langchain.tools.tool_node import ToolCallRequest


@wrap_tool_call
def handle_tool_errors(
    request: ToolCallRequest,
    handler: Callable[[ToolCallRequest], ToolMessage],
) -> ToolMessage:
    """Convert tool exceptions into ToolMessages the model can handle."""

    try:
        return handler(request)

    except Exception as e:
        return ToolMessage(
            content=(
                "Tool error: "
                f"Please check your input and try again. ({e})"
            ),
            tool_call_id=request.tool_call["id"],
        )


agent = create_agent(
    model="openai:gpt-5.5",
    tools=[],
    middleware=[handle_tool_errors],
)
```

这样：

即使 Tool 抛出异常，

模型收到的仍然是：

```text
ToolMessage
```

而不是程序崩溃。

---

## State injection（State 注入）

Tool 可以通过：

```python
ToolRuntime
```

访问 Graph State。

例如：

```python
from langchain.tools import tool, ToolRuntime

@tool
def get_message_count(
    runtime: ToolRuntime
) -> str:
    """Get the number of messages in the conversation."""

    messages = runtime.state["messages"]

    return f"There are {len(messages)} messages."
```

更多关于：

- State
- Context
- Store
- Streaming

请参考上一章节 **Access Context**。

---

# Dynamic tool selection（动态 Tool 选择）

动态 Tool 指：

**运行期间（Runtime）**

决定哪些 Tool 可以提供给模型，

而不是 Agent 创建时全部固定。

原因：

Tool 太多：

- 增加 Prompt 长度
- 增加错误率
- 模型更容易选错 Tool

Tool 太少：

- 功能受限

因此：

可以根据：

- 登录状态
- 用户权限
- Feature Flag
- 当前对话阶段

动态调整 Tool。

---

## 两种方式

动态 Tool 有两种实现方式：

1.

过滤已注册 Tool（Filtering Pre-registered Tools）

2.

运行时注册 Tool（Runtime Tool Registration）

---

## Filtering pre-registered tools（过滤已注册 Tool）

如果所有 Tool 都已知，

可以全部注册，

然后运行时筛选。

例如：

```python
from langchain.agents import create_agent
from langchain.agents.middleware import (
    wrap_model_call,
    ModelRequest,
    ModelResponse,
)
from typing import Callable


@wrap_model_call
def state_based_tools(
    request: ModelRequest,
    handler: Callable[[ModelRequest], ModelResponse]
) -> ModelResponse:
    """根据 State 动态过滤 Tool。"""

    state = request.state

    is_authenticated = state.get(
        "authenticated",
        False
    )

    message_count = len(state["messages"])

    if not is_authenticated:
        tools = [
            t
            for t in request.tools
            if t.name.startswith("public_")
        ]
        request = request.override(tools=tools)

    elif message_count < 5:
        tools = [
            t
            for t in request.tools
            if t.name != "advanced_search"
        ]
        request = request.override(tools=tools)

    return handler(request)


agent = create_agent(
    model="gpt-5.5",
    tools=[
        public_search,
        private_search,
        advanced_search,
    ],
    middleware=[state_based_tools],
)
```

适用于：

- Tool 集合固定
- 根据权限动态开放

例如：

登录前：

```
public_search
```

登录后：

```
private_search
```

---

## Headless tools（无头工具）

Headless Tool 是一种**只有定义（Schema），没有服务器实现（Implementation）**的 Tool。

它真正运行的位置不是服务器，而是：

- 浏览器
- 客户端 App
- 第三方服务
- 人工审批流程

例如：

浏览器才能调用：

- Clipboard API
- Geolocation
- IndexedDB
- Canvas
- File Picker

因此：

Tool Schema 注册到 Agent。

真正执行：

交给客户端。

执行流程：

```
LLM
   ↓
Headless Tool
   ↓
Graph Interrupt
   ↓
Browser 执行
   ↓
Resume Graph
```

这样既能保护本地数据，

又无需服务器参与执行。

---

## Prebuilt tools（预构建工具）

LangChain 官方提供了大量现成 Tool。

例如：

- Web Search
- Code Interpreter
- SQL
- Database
- Vector Database
- API

无需自己开发，

直接集成即可。

完整列表请参考：

**Tools & Toolkits Integration**。

---

# Server-side tool use（服务端工具）

部分模型（例如 OpenAI）自带 Tool。

例如：

- Web Search
- Code Interpreter

这些 Tool：

无需自己编写。

也无需自己部署。

直接由：

**模型提供商（Model Provider）**

在服务器端执行。

具体开启方式请查看：

- 对应模型集成文档（Chat Model Integration）
- Tool Calling 文档