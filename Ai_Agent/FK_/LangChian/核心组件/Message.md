# Basic usage（基本用法）

使用 Messages 最简单的方式是创建消息对象（Message Objects），并在调用模型时将它们传递给模型。

```python
from langchain.chat_models import init_chat_model
from langchain.messages import HumanMessage, AIMessage, SystemMessage

model = init_chat_model("gpt-5-nano")

system_msg = SystemMessage("You are a helpful assistant.")
human_msg = HumanMessage("Hello, how are you?")

# 与聊天模型一起使用
messages = [system_msg, human_msg]
response = model.invoke(messages)  # 返回 AIMessage
```

多轮 Agent 会不断累积较长的消息历史（Message History）。

LangSmith 会记录每一次对话、工具执行结果以及模型响应，以便你检查完整的对话过程。

请按照 **Tracing Quickstart（追踪快速入门）** 启用 Tracing（追踪）。

我们还建议配置 **LangSmith Engine**，它能够监控你的 Traces（追踪记录）、检测问题，并提出修复建议。

---

## Text prompts（文本提示）

文本提示（Text Prompts）就是普通字符串。

它非常适合那些**不需要保留对话历史**的简单文本生成任务。

```python
response = model.invoke("Write a haiku about spring")
```

以下场景适合使用 **Text Prompt**：

- 只有一次独立的请求（Single, Standalone Request）
- 不需要保留对话历史
- 希望代码尽可能简单

---

## Message prompts（消息提示）

另一种方式是向模型传入**消息对象列表（List of Message Objects）**。

```python
from langchain.messages import SystemMessage, HumanMessage, AIMessage

messages = [
    SystemMessage("You are a poetry expert"),
    HumanMessage("Write a haiku about spring"),
    AIMessage("Cherry blossoms bloom...")
]

response = model.invoke(messages)
```

以下场景适合使用 **Message Prompt**：

- 管理多轮对话（Multi-turn Conversations）
- 处理多模态内容（图片、音频、文件等）
- 包含系统指令（System Instructions）

---

## Dictionary format（字典格式）

你也可以直接使用 **OpenAI Chat Completions** 的消息格式来表示消息。

```python
messages = [
    {"role": "system", "content": "You are a poetry expert"},
    {"role": "user", "content": "Write a haiku about spring"},
    {"role": "assistant", "content": "Cherry blossoms bloom..."}
]

response = model.invoke(messages)
```

# Message types（消息类型）

LangChain 提供了四种标准消息类型：

- **System message（系统消息）** —— 告诉模型应如何行为，并为交互提供上下文。
- **Human message（用户消息）** —— 表示用户输入以及与模型之间的交互。
- **AI message（AI 消息）** —— 表示模型生成的响应，包括文本内容、工具调用以及元数据。
- **Tool message（工具消息）** —— 表示工具调用执行后的输出结果。 :contentReference[oaicite:0]{index=0}

---

## System message（系统消息）

`SystemMessage` 表示一组初始指令，用于引导（Prime）模型的行为。

你可以使用系统消息来：

- 设置回答风格（Tone）
- 定义模型角色（Role）
- 制定回答规范（Guidelines）

---

### 基础指令（Basic instructions）

```python
system_msg = SystemMessage("You are a helpful coding assistant.")

messages = [
    system_msg,
    HumanMessage("How do I create a REST API?")
]

response = model.invoke(messages)
```

---

### 详细角色设定（Detailed persona）

```python
from langchain.messages import SystemMessage, HumanMessage

system_msg = SystemMessage("""
You are a senior Python developer with expertise in web frameworks.
Always provide code examples and explain your reasoning.
Be concise but thorough in your explanations.
""")

messages = [
    system_msg,
    HumanMessage("How do I create a REST API?")
]

response = model.invoke(messages)
```

---

## Human message（用户消息）

`HumanMessage` 表示用户输入以及与模型之间的交互。

它可以包含：

- 文本（Text）
- 图片（Images）
- 音频（Audio）
- 文件（Files）
- 以及任意数量的多模态内容（Multimodal Content）。 :contentReference[oaicite:1]{index=1}

---

### 文本内容（Text content）

#### Message 对象

```python
response = model.invoke([
    HumanMessage("What is machine learning?")
])
```

---

### Message 元数据（Message metadata）

#### 添加元数据（Add metadata）

```python
human_msg = HumanMessage(
    content="Hello!",
    name="alice",  # 可选：标识不同用户
    id="msg_123",  # 可选：用于 Tracing 的唯一标识
)
```

`name` 字段的行为因模型提供商而异。

有些 Provider 会使用它来标识用户，而有些则会忽略它。

具体行为请参考对应模型提供商的文档。 :contentReference[oaicite:2]{index=2}

---

## AI message（AI 消息）

`AIMessage` 表示模型调用后的输出结果。

它可以包含：

- 多模态数据（Multimodal Data）
- 工具调用（Tool Calls）
- Provider 特有的元数据（Provider-specific Metadata）

这些信息都可以在后续程序中访问。

```python
response = model.invoke("Explain AI")

print(type(response))
# <class 'langchain.messages.AIMessage'>
```

`AIMessage` 对象由模型调用返回，其中包含本次响应的全部元数据。

不同模型提供商对于各种 Message 类型的重要程度（Weight）和上下文处理方式（Contextualization）可能有所不同。

因此，有时手动创建一个新的 `AIMessage` 并将其插入消息历史，就像它是模型之前生成的一样，会非常有帮助。

```python
from langchain.messages import AIMessage, SystemMessage, HumanMessage

# 手动创建 AIMessage（例如用于补充对话历史）
ai_msg = AIMessage("I'd be happy to help you with that question!")

# 添加到消息历史
messages = [
    SystemMessage("You are a helpful assistant"),
    HumanMessage("Can you help me?"),
    ai_msg,  # 像模型生成的一样插入历史记录
    HumanMessage("Great! What's 2+2?")
]

response = model.invoke(messages)
```

---

### Tool calls（工具调用）

当模型发起工具调用时，这些调用信息会包含在 `AIMessage` 中。

```python
from langchain.chat_models import init_chat_model

model = init_chat_model("gpt-5-nano")

def get_weather(location: str) -> str:
    """Get the weather at a location."""
    ...

model_with_tools = model.bind_tools([get_weather])

response = model_with_tools.invoke(
    "What's the weather in Paris?"
)

for tool_call in response.tool_calls:
    print(f"Tool: {tool_call['name']}")
    print(f"Args: {tool_call['args']}")
    print(f"ID: {tool_call['id']}")
```

除了 Tool Calls 外，消息内容中还可能包含其他结构化数据，例如：

- Reasoning（推理）
- Citations（引用） :contentReference[oaicite:3]{index=3}

---

### Token usage（Token 使用情况）

`AIMessage` 可以通过 `usage_metadata` 字段保存 Token 数量及其他使用统计信息。

```python
from langchain.chat_models import init_chat_model

model = init_chat_model("gpt-5-nano")

response = model.invoke("Hello!")

response.usage_metadata

{
    'input_tokens': 8,
    'output_tokens': 304,
    'total_tokens': 312,
    'input_token_details': {
        'audio': 0,
        'cache_read': 0
    },
    'output_token_details': {
        'audio': 0,
        'reasoning': 256
    }
}
```

更多信息请参阅 **UsageMetadata** 文档。 :contentReference[oaicite:4]{index=4}

---

### Streaming 与 Chunks（流式输出与消息分块）

在流式输出过程中，你会收到多个 `AIMessageChunk` 对象。

这些对象可以组合成一个完整的 `AIMessage`。

```python
chunks = []
full_message = None

for chunk in model.stream("Hi"):
    chunks.append(chunk)
    print(chunk.text)

    full_message = (
        chunk if full_message is None
        else full_message + chunk
    )
```

更多内容：

- Streaming tokens from chat models（聊天模型 Token 流式输出）
- Streaming tokens and/or steps from agents（Agent Token 或步骤流式输出）

---

## Tool message（工具消息）

对于支持 Tool Calling 的模型，`AIMessage` 可以包含工具调用。

`ToolMessage` 用于将**单次工具执行结果**返回给模型。

工具也可以直接生成 `ToolMessage` 对象。

下面是一个简单示例。

```python
from langchain.messages import AIMessage
from langchain.messages import ToolMessage

# 模型发起 Tool Call 后
# （这里为了简洁，手动创建消息）
ai_message = AIMessage(
    content=[],
    tool_calls=[{
        "name": "get_weather",
        "args": {"location": "San Francisco"},
        "id": "call_123"
    }]
)

# 执行工具并创建 ToolMessage
weather_result = "Sunny, 72°F"

tool_message = ToolMessage(
    content=weather_result,
    tool_call_id="call_123"  # 必须与 Tool Call 的 ID 保持一致
)

# 继续对话
messages = [
    HumanMessage("What's the weather in San Francisco?"),
    ai_message,      # 模型发出的 Tool Call
    tool_message,    # 工具执行结果
]

response = model.invoke(messages)
```

---

### Attributes（属性）

`artifact` 字段用于保存**不会发送给模型**，但可以通过程序访问的附加数据。

它适用于保存：

- 原始结果（Raw Results）
- 调试信息（Debug Information）
- 下游处理所需的数据（Downstream Processing）

而不会污染模型的上下文（Context）。 :contentReference[oaicite:5]{index=5}


# Message content（消息内容）

你可以将消息（Message）的 **Content（内容）** 理解为发送给模型的数据载荷（Payload）。

每条消息都具有一个 `content` 属性，它采用**宽松类型（Loosely-typed）**设计，支持以下数据类型：

- 字符串（String）
- 未指定类型对象（如 Dictionary）的列表

这种设计使 LangChain Chat Model 能够直接支持模型提供商（Provider）的原生数据结构，例如：

- 多模态内容（Multimodal Content）
- 其他 Provider 特有的数据结构

除此之外，LangChain 还为以下内容提供了专门的内容类型（Content Types）：

- 文本（Text）
- 推理（Reasoning）
- 引用（Citations）
- 多模态数据（Multimodal Data）
- 服务端工具调用（Server-side Tool Calls）
- 其他消息内容

详见下方 **Content Blocks（内容块）**。

---

LangChain Chat Model 会从消息对象的 `content` 属性读取消息内容。

`content` 可以是以下三种形式之一：

1. 一个字符串（String）
2. Provider 原生格式（Provider-native Format）的 Content Block 列表
3. LangChain 标准 Content Block 列表

下面展示一个使用多模态输入（Multimodal Input）的示例。

```python
from langchain.messages import HumanMessage

# 字符串内容
human_message = HumanMessage("Hello, how are you?")

# Provider 原生格式（例如 OpenAI）
human_message = HumanMessage(content=[
    {"type": "text", "text": "Hello, how are you?"},
    {"type": "image_url", "image_url": {
        "url": "https://example.com/image.jpg"
    }}
])

# LangChain 标准 Content Block
human_message = HumanMessage(content_blocks=[
    {"type": "text", "text": "Hello, how are you?"},
    {"type": "image", "url": "https://example.com/image.jpg"},
])
```

初始化消息时指定 `content_blocks` 后，仍然会填充消息对象的 `content` 属性，但同时还能提供一个**类型安全（Type-safe）**的接口。 :contentReference[oaicite:0]{index=0}

---

## Standard content blocks（标准内容块）

LangChain 提供了一套**跨模型提供商统一的消息内容表示方式**。

消息对象实现了 `content_blocks` 属性。

访问该属性时，LangChain 会**延迟解析（Lazily Parse）** `content` 属性，并转换为统一、类型安全（Type-safe）的标准表示。

例如：

由 `ChatAnthropic` 或 `ChatOpenAI` 生成的消息可能分别包含 Provider 自己定义的：

- `thinking`
- `reasoning`

内容块。

LangChain 会自动将它们解析为统一的 `ReasoningContentBlock`。

```python
from langchain.messages import AIMessage

message = AIMessage(
    content=[
        {
            "type": "thinking",
            "thinking": "...",
            "signature": "WaUjzkyp..."
        },
        {
            "type": "text",
            "text": "..."
        },
    ],
    response_metadata={
        "model_provider": "anthropic"
    }
)

message.content_blocks

[
    {
        "type": "reasoning",
        "reasoning": "...",
        "extras": {
            "signature": "WaUjzkyp..."
        }
    },
    {
        "type": "text",
        "text": "..."
    }
]
```

请参阅对应模型提供商的 Integration（集成）指南，了解如何开始使用不同的推理模型。 :contentReference[oaicite:1]{index=1}

---

### Serializing standard content（序列化标准内容）

如果 LangChain 外部的应用需要访问标准 Content Block 表示，可以选择（Opt-in）将 Content Block 存储到消息的 `content` 中。

有两种方式：

设置环境变量：

```text
LC_OUTPUT_VERSION=v1
```

或者初始化 Chat Model 时指定：

```python
from langchain.chat_models import init_chat_model

model = init_chat_model(
    "gpt-5-nano",
    output_version="v1"
)
```

---

## Multimodal（多模态）

多模态（Multimodality）指的是处理不同类型数据的能力，例如：

- 文本（Text）
- 音频（Audio）
- 图片（Images）
- 视频（Video）

LangChain 为这些数据类型提供了统一的标准表示，可跨不同模型提供商使用。

聊天模型既可以：

- 接收多模态输入（Input）
- 生成多模态输出（Output）

下面展示几个多模态输入示例。

Content Block 中允许：

- 在顶层添加额外字段（Top-level Keys）
- 或放入：

```python
extras = {
    "key": value
}
```

例如：

OpenAI 在上传 PDF 时要求提供文件名（Filename）。

具体要求请查阅对应 Provider 文档。

---

### Image input（图片输入）

#### 从 URL 加载

```python
message = {
    "role": "user",
    "content": [
        {
            "type": "text",
            "text": "Describe the content of this image."
        },
        {
            "type": "image",
            "url": "https://example.com/path/to/image.jpg"
        },
    ]
}
```

---

#### 使用 Base64 图片

```python
message = {
    "role": "user",
    "content": [
        {
            "type": "text",
            "text": "Describe the content of this image."
        },
        {
            "type": "image",
            "base64": "AAAAIGZ0eXBtcDQyAAAAAGlzb21tcDQyAAACAGlzb2...",
            "mime_type": "image/jpeg",
        },
    ]
}
```

---

#### 使用 Provider 管理的 File ID

```python
message = {
    "role": "user",
    "content": [
        {
            "type": "text",
            "text": "Describe the content of this image."
        },
        {
            "type": "image",
            "file_id": "file-abc123"
        },
    ]
}
```

并非所有模型都支持所有文件类型。

支持的文件格式以及大小限制，请参考对应模型提供商的文档。 :contentReference[oaicite:2]{index=2}

---

## Content block reference（Content Block 参考）

Content Block 无论是在：

- 创建消息时
- 还是访问 `content_blocks` 属性时

都表示为一个**带类型（Typed）的字典列表**。

列表中的每一项都必须符合以下其中一种 Block 类型：

- Core（核心内容块）
- Multimodal（多模态内容块）
- Tool Calling（工具调用内容块）
- Server-Side Tool Execution（服务端工具执行内容块）
- Provider-Specific Blocks（Provider 专用内容块）

完整类型定义请参阅 API Reference（API 参考文档）。

Content Blocks 是 LangChain v1 引入的新特性。

它的目标是：

- 在不同模型提供商之间统一消息内容格式；
- 同时保持与现有代码的向后兼容（Backward Compatibility）。

需要注意的是：

Content Blocks **并不是** `content` 属性的替代品。

它只是一个新的属性，用于以统一的标准格式访问消息内容。 :contentReference[oaicite:3]{index=3}

---

# Use with chat models（与聊天模型配合使用）

Chat Model 接收一组消息对象（Sequence of Message Objects）作为输入，并返回一个 `AIMessage` 作为输出。

模型交互通常是**无状态（Stateless）**的。

因此，一个简单的聊天循环通常是：

不断向消息历史中追加新的消息，然后将完整消息列表再次传递给模型。

更多内容请参阅以下指南：

- 内置的对话历史持久化与管理功能（Built-in features for persisting and managing conversation histories）
- 上下文窗口管理策略，包括消息裁剪（Trimming）与消息摘要（Summarizing Messages） :contentReference[oaicite:4]{index=4}