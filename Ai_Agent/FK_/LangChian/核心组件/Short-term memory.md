# Overview（概述）

Memory（记忆）是一种能够记住之前交互信息的系统。

对于 AI Agent 来说，Memory 非常重要，因为它能够：

- 记住之前的交互。
- 从用户反馈中学习。
- 根据用户偏好进行调整。

随着 Agent 处理越来越复杂的任务，并与用户进行越来越多的交互，这种能力对于提高效率和用户满意度变得至关重要。

Short-term memory（短期记忆）允许应用在单个 Thread（线程）或 Conversation（会话）中记住之前的交互。

Thread（线程）用于组织同一会话中的多次交互，类似于电子邮件中将多封邮件组织成一个会话。

Conversation history（对话历史）是最常见的一种短期记忆形式。长时间的对话会给当前的大语言模型（LLM）带来挑战：完整的历史记录可能无法放入模型的 Context Window（上下文窗口）中，从而导致上下文丢失或产生错误。

即使模型支持完整的上下文长度，大多数 LLM 在处理超长上下文时仍然表现较差。它们容易受到过期内容或无关内容的干扰，同时响应速度更慢、成本更高。

Chat Model（聊天模型）通过 Messages（消息）接收上下文，其中包括：

- Instructions（指令），即 System Message（系统消息）。
- Inputs（输入），即 Human Message（用户消息）。

在聊天应用中，消息会在人类输入和模型回复之间不断交替，因此 Messages 列表会随着时间不断增长。

由于 Context Window（上下文窗口）存在限制，许多应用都会采用一些技术来移除或“遗忘”那些已经过时的信息。

# Usage（使用）

要为 Agent 添加 Short-term Memory（短期记忆，即线程级持久化），需要在创建 Agent 时指定一个 Checkpointer（检查点存储器）。

LangChain 的 Agent 将 Short-term Memory（短期记忆）作为 Agent State（状态）的一部分进行管理。

通过将这些状态存储到 Graph（图）的 State（状态）中，Agent 可以访问某个对话的完整上下文，同时保持不同 Thread（线程）之间的相互隔离。

State（状态）会通过 Checkpointer（检查点存储器）持久化到数据库（或内存）中，因此线程可以在任何时候恢复。

当 Agent 被调用，或某一步骤（例如 Tool Call（工具调用））完成时，Short-term Memory（短期记忆）都会更新；而在每一步开始时，都会读取对应的 State（状态）。

```python
from langchain.agents import create_agent
from langgraph.checkpoint.memory import InMemorySaver


def get_user_info() -> str:
    """Look up information about the current user."""
    return "No user profile on file."


agent = create_agent(
    model="openai:gpt-5.5",
    tools=[get_user_info],
    checkpointer=InMemorySaver(),
)

thread_config = {"configurable": {"thread_id": "1"}}
response = agent.invoke(
    {"messages": [{"role": "user", "content": "Hi! My name is Bob."}]},
    thread_config,
)["messages"][-1].content

print(response)  # "Hi Bob! Nice to see you here. How are you doing?"

response = agent.invoke(
    {"messages": [{"role": "user", "content": "What's my name?"}]},
    thread_config,
)["messages"][-1].content

print(response)  # "You are Bob!"
```

## In production（生产环境）

在生产环境中，应使用由数据库支持的 Checkpointer（检查点存储器）：

```bash
pip install langgraph-checkpoint-postgres
```

```python
from langchain.agents import create_agent
from langgraph.checkpoint.postgres import PostgresSaver

def get_user_info() -> str:
    """Look up information about the current user."""
    return "No user profile on file."

DB_URI = "postgresql://postgres:postgres@localhost:5432/postgres?sslmode=disable"

with PostgresSaver.from_conn_string(DB_URI) as checkpointer:
    checkpointer.setup()  # 自动在 PostgreSQL 中创建数据表

    agent = create_agent(
        "gpt-5.5",
        tools=[get_user_info],
        checkpointer=checkpointer,
    )
```

有关更多 Checkpointer（检查点存储器）选项（包括 SQLite、Postgres 和 Azure Cosmos DB），请参阅 Persistence（持久化）文档中的 Checkpointer Libraries（检查点库）列表。

# Customizing agent memory（自定义 Agent Memory）

默认情况下，Agent 使用 AgentState（Agent 状态）来管理 Short-term Memory（短期记忆），其中主要通过 `messages` 字段管理 Conversation History（对话历史）。

你可以继承 AgentState 来添加额外的字段。

自定义的 State Schema（状态结构）可以通过 `state_schema` 参数传递给 `create_agent`。

```python
from langchain.agents import create_agent, AgentState
from langgraph.checkpoint.memory import InMemorySaver


class CustomAgentState(AgentState):
    user_id: str
    preferences: dict

agent = create_agent(
    "gpt-5.5",
    tools=[get_user_info],
    state_schema=CustomAgentState,
    checkpointer=InMemorySaver(),
)

# 调用 invoke 时可以传入自定义状态
result = agent.invoke(
    {
        "messages": [{"role": "user", "content": "Hello"}],
        "user_id": "user_123",
        "preferences": {"theme": "dark"}
    },
    {"configurable": {"thread_id": "1"}}
)
```

# Common patterns（常见模式）

启用 Short-term Memory（短期记忆）后，长时间的对话可能会超出 LLM 的 Context Window（上下文窗口）。

常见的解决方案包括：

- **Trim messages（裁剪消息）**
  - 在调用 LLM 之前移除最前面或最后面的 N 条消息。

- **Delete messages（删除消息）**
  - 从 LangGraph State（状态）中永久删除消息。

- **Summarize messages（总结消息）**
  - 对历史消息进行总结，并用摘要替换这些历史消息。

- **Custom strategies（自定义策略）**
  - 自定义策略（例如消息过滤等）。

这样可以让 Agent 在不超出 LLM Context Window（上下文窗口）的情况下，继续跟踪整个对话。

---

## Trim messages（裁剪消息）

大多数 LLM 都有最大支持的 Context Window（上下文窗口），通常以 Token 数量表示。

决定何时裁剪消息的一种方式，是统计 Message History（消息历史）的 Token 数量，并在其接近 Context Window（上下文窗口）限制时进行裁剪。

如果你使用的是 LangChain，可以使用 `trim_messages` 工具，并指定：

- 保留多少 Token。
- 处理边界时所采用的策略（例如保留最后 `max_tokens` 个 Token）。

要在 Agent 中裁剪 Message History（消息历史），可以使用 `@before_model` 中间件装饰器：

```python
from langchain.messages import RemoveMessage
from langgraph.graph.message import REMOVE_ALL_MESSAGES
from langgraph.checkpoint.memory import InMemorySaver
from langchain.agents import create_agent, AgentState
from langchain.agents.middleware import before_model
from langgraph.runtime import Runtime
from langchain_core.runnables import RunnableConfig
from typing import Any


@before_model
def trim_messages(state: AgentState, runtime: Runtime) -> dict[str, Any] | None:
    """Keep only the last few messages to fit context window."""
    messages = state["messages"]

    if len(messages) <= 3:
        return None  # No changes needed

    first_msg = messages[0]
    recent_messages = messages[-3:] if len(messages) % 2 == 0 else messages[-4:]
    new_messages = [first_msg] + recent_messages

    return {
        "messages": [
            RemoveMessage(id=REMOVE_ALL_MESSAGES),
            *new_messages
        ]
    }

agent = create_agent(
    "gpt-5.5",
    tools=[...],
    middleware=[trim_messages],
    checkpointer=InMemorySaver(),
)

config: RunnableConfig = {"configurable": {"thread_id": "1"}}

agent.invoke({"messages": "hi, my name is bob"}, config)
agent.invoke({"messages": "write a short poem about cats"}, config)
agent.invoke({"messages": "now do the same but for dogs"}, config)
final_response = agent.invoke({"messages": "what's my name?"}, config)

final_response["messages"][-1].pretty_print()
"""
================================== Ai Message ==================================

Your name is Bob. You told me that earlier.
If you'd like me to call you a nickname or use a different name, just say the word.
"""
```

---

## Delete messages（删除消息）

你可以从 Graph State（图状态）中删除消息，以管理 Message History（消息历史）。

当你希望删除指定消息，或者清空整个消息历史时，这种方式非常有用。

要从 Graph State（图状态）中删除消息，可以使用 `RemoveMessage`。

要使 `RemoveMessage` 生效，需要使用带有 `add_messages` reducer 的 State Key（状态字段）。

默认的 `AgentState` 已经提供了这一能力。

删除指定消息：

```python
from langchain.messages import RemoveMessage

def delete_messages(state):
    messages = state["messages"]
    if len(messages) > 2:
        # remove the earliest two messages
        return {"messages": [RemoveMessage(id=m.id) for m in messages[:2]]}
```

删除全部消息：

```python
from langgraph.graph.message import REMOVE_ALL_MESSAGES

def delete_messages(state):
    return {"messages": [RemoveMessage(id=REMOVE_ALL_MESSAGES)]}
```

删除消息时，请确保最终保留下来的 Message History（消息历史）仍然是合法的。

请检查你所使用的 LLM Provider（模型提供商）的限制。例如：

- 有些 Provider 要求消息历史必须以 User Message（用户消息）开始。
- 大多数 Provider 要求包含 Tool Call（工具调用）的 Assistant Message（助手消息）后面必须紧跟对应的 Tool Result Message（工具结果消息）。

```python
from langchain.messages import RemoveMessage
from langchain.agents import create_agent, AgentState
from langchain.agents.middleware import after_model
from langgraph.checkpoint.memory import InMemorySaver
from langgraph.runtime import Runtime
from langchain_core.runnables import RunnableConfig


@after_model
def delete_old_messages(state: AgentState, runtime: Runtime) -> dict | None:
    """Remove old messages to keep conversation manageable."""
    messages = state["messages"]
    if len(messages) > 2:
        # remove the earliest two messages
        return {"messages": [RemoveMessage(id=m.id) for m in messages[:2]]}
    return None


agent = create_agent(
    "gpt-5-nano",
    tools=[...],
    system_prompt="Please be concise and to the point.",
    middleware=[delete_old_messages],
    checkpointer=InMemorySaver(),
)

config: RunnableConfig = {"configurable": {"thread_id": "1"}}

stream = agent.stream_events(
    {"messages": [{"role": "user", "content": "hi! I'm bob"}]},
    config,
    version="v3",
)
for snapshot in stream.values:
    print([(message.type, message.content) for message in snapshot["messages"]])

stream = agent.stream_events(
    {"messages": [{"role": "user", "content": "write a short poem about cats"}]},
    config,
    version="v3",
)
for snapshot in stream.values:
    print([(message.type, message.content) for message in snapshot["messages"]])

stream = agent.stream_events(
    {"messages": [{"role": "user", "content": "what's my name?"}]},
    config,
    version="v3",
)
for snapshot in stream.values:
    print([(message.type, message.content) for message in snapshot["messages"]])

[('human', "hi! I'm bob")]
[('human', "hi! I'm bob"), ('ai', 'Hi Bob! Nice to meet you. How can I help you today? I can answer questions, brainstorm ideas, draft text, explain things, or help with code.')]
[('human', "hi! I'm bob"), ('ai', 'Hi Bob! Nice to meet you. How can I help you today? I can answer questions, brainstorm ideas, draft text, explain things, or help with code.'), ('human', "write a short poem about cats")]
[('human', "hi! I'm bob"), ('ai', 'Hi Bob! Nice to meet you. How can I help you today? I can answer questions, brainstorm ideas, draft text, explain things, or help with code.'), ('human', "write a short poem about cats"), ('ai', 'There once was a cat on a wall, Who barely moved at all...')]
[('human', 'write a short poem about cats'), ('ai', 'There once was a cat on a wall, Who barely moved at all...')]
[('human', 'write a short poem about cats'), ('ai', 'There once was a cat on a wall, Who barely moved at all...'), ('human', "what's my name?")]
[('human', 'write a short poem about cats'), ('ai', 'There once was a cat on a wall, Who barely moved at all...'), ('human', "what's my name?"), ('ai', "I don't know your name - you haven't told me!")]
[('human', "what's my name?"), ('ai', "I don't know your name - you haven't told me!")]
```

---

## Summarize messages（总结消息）

像上面介绍的 Trim Messages（裁剪消息）或 Delete Messages（删除消息）存在一个问题：裁剪或删除 Message Queue（消息队列）时，可能会丢失重要信息。

因此，一些应用更适合采用更高级的方法：使用 Chat Model（聊天模型）对 Message History（消息历史）进行总结。

### Summary（摘要）

要在 Agent 中总结 Message History（消息历史），可以使用内置的 `SummarizationMiddleware`。

```python
from langchain.agents import create_agent
from langchain.agents.middleware import SummarizationMiddleware
from langgraph.checkpoint.memory import InMemorySaver
from langchain_core.runnables import RunnableConfig


checkpointer = InMemorySaver()

agent = create_agent(
    model="gpt-5.5",
    tools=[...],
    middleware=[
        SummarizationMiddleware(
            model="gpt-5.4-mini",
            trigger=("tokens", 4000),
            keep=("messages", 20)
        )
    ],
    checkpointer=checkpointer,
)

config: RunnableConfig = {"configurable": {"thread_id": "1"}}
agent.invoke({"messages": "hi, my name is bob"}, config)
agent.invoke({"messages": "write a short poem about cats"}, config)
agent.invoke({"messages": "now do the same but for dogs"}, config)
final_response = agent.invoke({"messages": "what's my name?"}, config)

final_response["messages"][-1].pretty_print()
"""
================================== Ai Message ==================================

Your name is Bob!
"""
```

# Access memory访问内存
你可以以多种方式访问和修改某个代理的短期记忆（状态）：
## Tools工具

### 在工具中读取短期记忆
使用 `runtime` 参数访问短期记忆中的状态（信息）。该参数被定义为 `ToolRuntime` 。

参数 `runtime` 在工具签名中是隐藏的（因此模型无法看到它），但工具可以通过该参数访问状态。

``` python
from langchain.agents import create_agent, AgentState
from langchain.tools import tool, ToolRuntime


class CustomState(AgentState):
    user_id: str

@tool
def get_user_info(
    runtime: ToolRuntime
) -> str:
    """Look up user info."""
    user_id = runtime.state["user_id"]
    return "User is John Smith" if user_id == "user_123" else "Unknown user"

agent = create_agent(
    model="gpt-5-nano",
    tools=[get_user_info],
    state_schema=CustomState,
)

result = agent.invoke({
    "messages": "look up user information",
    "user_id": "user_123"
})
print(result["messages"][-1].content)
# > User is John Smith.
```

### 从工具写入短期记忆

在执行过程中，可以修改代理的短期状态。您可以直接通过工具返回状态更新信息。这非常适用于保存中间结果，或者让后续的工具或提示能够访问这些信息。

``` python
from langchain.tools import tool, ToolRuntime
from langchain_core.runnables import RunnableConfig
from langchain.messages import ToolMessage
from langchain.agents import create_agent, AgentState
from langgraph.types import Command
from pydantic import BaseModel


class CustomState(AgentState):
    user_name: str

class CustomContext(BaseModel):
    user_id: str

@tool
def update_user_info(
    runtime: ToolRuntime[CustomContext, CustomState],
) -> Command:
    """Look up and update user info."""
    user_id = runtime.context.user_id
    name = "John Smith" if user_id == "user_123" else "Unknown user"
    return Command(update={
        "user_name": name,
        # update the message history
        "messages": [
            ToolMessage(
                "Successfully looked up user information",
                tool_call_id=runtime.tool_call_id
            )
        ]
    })

@tool
def greet(
    runtime: ToolRuntime[CustomContext, CustomState]
) -> str | Command:
    """Use this to greet the user once you found their info."""
    user_name = runtime.state.get("user_name", None)
    if user_name is None:
       return Command(update={
            "messages": [
                ToolMessage(
                    "Please call the 'update_user_info' tool it will get and update the user's name.",
                    tool_call_id=runtime.tool_call_id
                )
            ]
        })
    return f"Hello {user_name}!"

agent = create_agent(
    model="gpt-5-nano",
    tools=[update_user_info, greet],
    state_schema=CustomState,
    context_schema=CustomContext,
)

agent.invoke(
    {"messages": [{"role": "user", "content": "greet the user"}]},
    context=CustomContext(user_id="user_123"),
)
```

### 提示

在中间件中访问短期记忆（状态），根据对话历史或自定义状态字段来生成动态提示。

```
from langchain.agents import create_agent
from typing import TypedDict
from langchain.agents.middleware import dynamic_prompt, ModelRequest


class CustomContext(TypedDict):
    user_name: str


def get_weather(city: str) -> str:
    """Get the weather in a city."""
    return f"The weather in {city} is always sunny!"


@dynamic_prompt
def dynamic_system_prompt(request: ModelRequest) -> str:
    user_name = request.runtime.context["user_name"]
    system_prompt = f"You are a helpful assistant. Address the user as {user_name}."
    return system_prompt


agent = create_agent(
    model="gpt-5-nano",
    tools=[get_weather],
    middleware=[dynamic_system_prompt],
    context_schema=CustomContext,
)

result = agent.invoke(
    {"messages": [{"role": "user", "content": "What is the weather in SF?"}]},
    context=CustomContext(user_name="John Smith"),
)
for msg in result["messages"]:
    msg.pretty_print()

```

输出

```
================================ Human Message =================================

What is the weather in SF?
================================== Ai Message ==================================
Tool Calls:
  get_weather (call_WFQlOGn4b2yoJrv7cih342FG)
 Call ID: call_WFQlOGn4b2yoJrv7cih342FG
  Args:
    city: San Francisco
================================= Tool Message =================================
Name: get_weather

The weather in San Francisco is always sunny!
================================== Ai Message ==================================

Hi John Smith, the weather in San Francisco is always sunny!
```

### 模型之前

在 `@before_model` 中间件中访问短期记忆（状态），以便在调用模型之前处理消息。


![[Pasted image 20260627134131.png]]

```
from langchain.messages import RemoveMessage
from langgraph.graph.message import REMOVE_ALL_MESSAGES
from langgraph.checkpoint.memory import InMemorySaver
from langchain.agents import create_agent, AgentState
from langchain.agents.middleware import before_model
from langchain_core.runnables import RunnableConfig
from langgraph.runtime import Runtime
from typing import Any


@before_model
def trim_messages(state: AgentState, runtime: Runtime) -> dict[str, Any] | None:
    """Keep only the last few messages to fit context window."""
    messages = state["messages"]

    if len(messages) <= 3:
        return None  # No changes needed

    first_msg = messages[0]
    recent_messages = messages[-3:] if len(messages) % 2 == 0 else messages[-4:]
    new_messages = [first_msg] + recent_messages

    return {
        "messages": [
            RemoveMessage(id=REMOVE_ALL_MESSAGES),
            *new_messages
        ]
    }


agent = create_agent(
    "gpt-5-nano",
    tools=[],
    middleware=[trim_messages],
    checkpointer=InMemorySaver()
)

config: RunnableConfig = {"configurable": {"thread_id": "1"}}

agent.invoke({"messages": "hi, my name is bob"}, config)
agent.invoke({"messages": "write a short poem about cats"}, config)
agent.invoke({"messages": "now do the same but for dogs"}, config)
final_response = agent.invoke({"messages": "what's my name?"}, config)

final_response["messages"][-1].pretty_print()
"""
================================== Ai Message ==================================

Your name is Bob. You told me that earlier.
If you'd like me to call you a nickname or use a different name, just say the word.
"""
```

### 模型之后

在 [`@after_model`](https://reference.langchain.com/python/langchain/agents/middleware/types/after_model) 中间件中访问短期记忆（状态），以便在模型调用后处理消息。

![[Pasted image 20260627134228.png]]

```
from langchain.messages import RemoveMessage
from langgraph.checkpoint.memory import InMemorySaver
from langchain.agents import create_agent, AgentState
from langchain.agents.middleware import after_model
from langgraph.runtime import Runtime


@after_model
def validate_response(state: AgentState, runtime: Runtime) -> dict | None:
    """Remove messages containing sensitive words."""
    STOP_WORDS = ["password", "secret"]
    last_message = state["messages"][-1]
    if any(word in last_message.content for word in STOP_WORDS):
        return {"messages": [RemoveMessage(id=last_message.id)]}
    return None

agent = create_agent(
    model="gpt-5-nano",
    tools=[],
    middleware=[validate_response],
    checkpointer=InMemorySaver(),
)
```