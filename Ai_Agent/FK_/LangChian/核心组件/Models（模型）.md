大型语言模型（LLM）是一种强大的 AI 工具，能够像人类一样理解和生成文本。它们用途广泛，可以完成内容编写、语言翻译、文本摘要以及问答等任务，而无需针对每项任务进行专门训练。

除了文本生成之外，许多模型还支持以下能力：

- **Tool calling（工具调用）** —— 调用外部工具（例如数据库查询或 API 调用），并利用返回结果生成回复。
- **Structured output（结构化输出）** —— 将模型输出限制为指定的格式。
- **Multimodality（多模态）** —— 处理和返回除文本之外的数据，例如图片、音频和视频。
- **Reasoning（推理）** —— 模型通过多步推理得出最终结论。

模型是 Agent 的推理引擎。

它决定 Agent 的决策过程，包括：

- 调用哪些工具；
- 如何解释工具返回的结果；
- 何时给出最终答案。

你所选择模型的质量与能力，会直接影响 Agent 的基础可靠性和性能。

不同模型擅长不同任务：

- 有些模型更擅长遵循复杂指令；
- 有些模型更擅长结构化推理；
- 有些模型支持更大的上下文窗口，可以处理更多信息。

LangChain 提供统一的模型接口，可以接入众多模型提供商，因此你能够方便地在不同模型之间切换，并选择最适合自己业务场景的模型。

有关特定模型提供商的集成方式和能力，请参阅对应提供商的 Chat Model 文档。

---

# Basic usage（基本用法）

模型可以通过两种方式使用：

1. **与 Agent 配合使用**

   在创建 Agent 时，可以动态指定所使用的模型。

2. **独立使用**

   模型可以直接调用（不经过 Agent 循环），用于文本生成、文本分类或信息提取，而无需使用 Agent 框架。

同一套模型接口同时适用于以上两种场景，因此你可以先从简单的模型调用开始，再根据需要扩展到更复杂的 Agent 工作流。

---

# Initialize a model（初始化模型）

在 LangChain 中，初始化独立聊天模型最简单的方法是使用 `init_chat_model`。

目前支持多个模型提供商，例如：

- OpenAI
- Anthropic
- Azure
- Google Gemini
- AWS Bedrock
- HuggingFace
- OpenRouter

## OpenAI

安装：

```bash
pip install -U "langchain[openai]"
```

初始化：

```python
import os
from langchain.chat_models import init_chat_model

os.environ["OPENAI_API_KEY"] = "sk-..."

model = init_chat_model("gpt-5.2")
```

## Anthropic

安装：

```bash
pip install -U "langchain[anthropic]"
```

初始化：

```python
import os
from langchain.chat_models import init_chat_model

os.environ["ANTHROPIC_API_KEY"] = "sk-..."

model = init_chat_model("claude-sonnet-4-6")
```

Azure、Google Gemini、AWS Bedrock、HuggingFace、OpenRouter 的初始化方式与上述示例类似，只需安装对应集成包、配置相应的 API Key，然后调用 `init_chat_model()` 初始化对应模型即可。

---

## Invoke（调用）

模型接收消息（messages）作为输入，并在生成完整响应后返回消息。

---

## Stream（流式输出）

调用模型时实时返回生成内容，而不是等待全部生成完成。

---

## Batch（批量调用）

一次向模型发送多个请求，以获得更高的处理效率。

除了 Chat Models 之外，LangChain 还支持其他相关组件，例如：

- Embedding Models（嵌入模型）
- Vector Stores（向量数据库）

# Parameters（参数）

聊天模型（Chat Model）接受一组参数来配置其行为。支持的参数会因模型和提供商而有所不同，但以下是一些标准参数：

## `model`

**类型：** `string`

**必填**

要与某个提供商一起使用的特定模型名称或标识符。

你也可以通过 `provider:model` 的格式，在一个参数中同时指定模型提供商和模型名称，例如：

```text
openai:o1
```

---

## `api_key`

**类型：** `string`

用于向模型提供商进行身份验证的 API Key。

通常在申请模型访问权限时获得，并且一般通过环境变量进行配置。

---

## `temperature`

**类型：** `number`

控制模型输出的随机性。

- 值越高，回复越具有创造性。
- 值越低，回复越稳定、确定性越强。

---

## `max_tokens`

**类型：** `number`

限制模型回复可生成的最大 Token 数量。

该参数实际上控制了回复内容的最大长度。

---

## `timeout`

**类型：** `number`

等待模型响应的最长时间（单位：秒）。

如果超过该时间仍未收到响应，则请求会被取消。

---

## `max_retries`

**类型：** `number`

**默认值：** `6`

请求失败时允许自动重试的最大次数。

当出现以下情况时，LangChain 会自动重试：

- 网络超时
- 请求频率限制（429）
- 服务器错误（5xx）

重试采用**指数退避（Exponential Backoff）**并加入**随机抖动（Jitter）**策略，以减少大量重试请求同时发生。

以下错误不会自动重试：

- 401（Unauthorized，未授权）
- 404（Not Found，资源不存在）
- 其他客户端错误

对于运行时间较长、网络环境不稳定的 Agent 任务，可以考虑将该值提高到 **10~15**。

---

使用 `init_chat_model` 时，可以通过 `**kwargs` 直接传递这些参数：

```python
model = init_chat_model(
    "claude-sonnet-4-6",

    # 传递给模型的参数
    temperature=0.7,
    timeout=30,
    max_tokens=1000,
    max_retries=6,  # 默认值；网络不稳定时可适当提高
)
```

不同聊天模型集成还可能提供额外参数，用于控制提供商特有的功能。

例如，`ChatOpenAI` 提供了 `use_responses_api` 参数，用于决定使用 OpenAI 的 **Responses API** 还是 **Completions API**。

要查看某个聊天模型支持的全部参数，请参阅对应的 **Chat Model Integrations（聊天模型集成）** 页面。

---
# Invocation（调用）

聊天模型必须经过调用（Invoke）才能生成输出。

LangChain 提供了三种主要的调用方式，每种方式都适用于不同的使用场景。

## Invoke（普通调用）

调用模型最直接的方法是使用 `invoke()`。

它既可以接收**单条消息（Single message）**，也可以接收**消息列表（List of messages）**作为输入。

### 单条消息

```python
response = model.invoke("Why do parrots have colorful feathers?")
print(response)
```

---

聊天模型还可以接收**消息列表**来表示整个对话历史。

每条消息都包含一个 **role（角色）**，用于告诉模型该消息是谁发送的。

有关角色（roles）、消息类型（types）以及消息内容（content）的更多信息，请参阅 **Messages** 指南。

### 字典格式（Dictionary format）

```python
conversation = [
    {"role": "system", "content": "You are a helpful assistant that translates English to French."},
    {"role": "user", "content": "Translate: I love programming."},
    {"role": "assistant", "content": "J'adore la programmation."},
    {"role": "user", "content": "Translate: I love building applications."}
]

response = model.invoke(conversation)
print(response)  # AIMessage("J'adore créer des applications.")
```

### Message 对象格式（Message objects）

```python
from langchain.messages import HumanMessage, AIMessage, SystemMessage

conversation = [
    SystemMessage("You are a helpful assistant that translates English to French."),
    HumanMessage("Translate: I love programming."),
    AIMessage("J'adore la programmation."),
    HumanMessage("Translate: I love building applications.")
]

response = model.invoke(conversation)
print(response)  # AIMessage("J'adore créer des applications.")
```

---

如果调用返回的是 **字符串（string）**，请确认你使用的是 **Chat Model（聊天模型）**，而不是传统的 **LLM**。

传统的文本补全模型（Text-completion LLM）会直接返回字符串。

LangChain 中的聊天模型通常以 **`Chat`** 开头，例如：

- `ChatOpenAI`

---

## Stream（流式输出）

大多数模型都支持在生成内容的过程中**实时流式返回输出**。

通过逐步显示生成内容，流式输出可以显著提升用户体验，尤其是在回复内容较长时。

调用 `stream()` 会返回一个**迭代器（Iterator）**，随着模型不断生成内容，它会持续产生新的输出块（Chunk）。

你可以使用循环实时处理这些输出。

### 基本文本流式输出（Basic text streaming）

```python
for chunk in model.stream("Why do parrots have colorful feathers?"):
    print(chunk.text, end="|", flush=True)
```

---

与 `invoke()` 不同：

- `invoke()` 会在模型生成完整回复后返回一个 `AIMessage`。
- `stream()` 会持续返回多个 `AIMessageChunk` 对象。

每一个 `AIMessageChunk` 都包含回复内容的一部分。

这些 Chunk 可以通过相加（Summation）的方式组合成完整消息。

### 构建完整 AIMessage

```python
full = None  # None | AIMessageChunk

for chunk in model.stream("What color is the sky?"):
    full = chunk if full is None else full + chunk
    print(full.text)

# The
# The sky
# The sky is
# The sky is typically
# The sky is typically blue
# ...

print(full.content_blocks)

# [{"type": "text", "text": "The sky is typically blue..."}]
```

---

最终得到的消息与 `invoke()` 返回的消息完全一致。

例如，它同样可以：

- 添加到聊天历史（Message History）
- 再次作为上下文传递给模型

---

只有当程序中的所有步骤都支持流式处理时，Streaming 才能够正常工作。

例如，如果某个应用必须等到全部输出完成之后才能继续处理，那么它就不属于支持 Streaming 的应用。

---

### 高级 Streaming 主题（Advanced streaming topics）

#### Streaming Events（流式事件）

LangChain 聊天模型还支持通过 `astream_events()` 流式返回**语义事件（Semantic Events）**。

这种方式能够：

- 根据事件类型过滤数据
- 获取更多元数据（Metadata）
- 在后台自动聚合完整消息

示例：

```python
async for event in model.astream_events("Hello"):

    if event["event"] == "on_chat_model_start":
        print(f"Input: {event['data']['input']}")

    elif event["event"] == "on_chat_model_stream":
        print(f"Token: {event['data']['chunk'].text}")

    elif event["event"] == "on_chat_model_end":
        print(f"Full message: {event['data']['output'].text}")

    else:
        pass
```

输出示例：

```text
Input: Hello
Token: Hi
Token:  there
Token: !
Token:  How
Token:  can
Token:  I
...
Full message: Hi there! How can I help today?
```

有关事件类型及更多细节，请参阅 `astream_events()` 的参考文档。

---

#### 自动 Streaming（Auto-streaming）

LangChain 可以自动启用 Streaming。

即使你的代码调用的是：

```python
model.invoke(...)
```

只要 LangChain 检测到整个应用正在以流式模式运行，它就会自动切换到底层 Streaming 模式。

例如：

在 LangGraph Agent 中，你可以在节点（Node）中调用：

```python
model.invoke(...)
```

LangChain 会自动将其转换为 Streaming，而无需修改你的代码。

---

#### 工作原理（How it works）

当调用 `invoke()` 时，如果 LangChain 检测到整个应用正在进行 Streaming，它会自动切换到内部 Streaming 模式。

对于调用者来说：

```python
result = model.invoke(...)
```

返回结果不会发生变化。

但是在模型流式生成期间，LangChain 会自动触发 Callback 系统中的：

```text
on_llm_new_token
```

事件。

这些 Callback 会让：

- LangGraph `stream()`
- LangGraph `astream_events()`

能够实时显示模型输出。

## Batch（批量调用）

将一组彼此独立的请求批量发送给模型，可以显著提升性能并降低成本，因为这些请求可以并行处理。

### Batch

```python
responses = model.batch([
    "Why do parrots have colorful feathers?",
    "How do airplanes fly?",
    "What is quantum computing?"
])

for response in responses:
    print(response)
```

本节介绍的是聊天模型提供的 `batch()` 方法，它会在**客户端（Client-side）**并行执行多个模型调用。

它与模型提供商（如 OpenAI 或 Anthropic）提供的 **Batch API** 不同，两者并不是同一个功能。

---

默认情况下，`batch()` 会在**整个批次全部完成后**一次性返回所有结果。

如果希望**每个输入一完成就立即获得对应结果**，可以使用 `batch_as_completed()` 进行流式获取。

### 按完成顺序返回批量结果（Yield batch responses upon completion）

```python
for response in model.batch_as_completed([
    "Why do parrots have colorful feathers?",
    "How do airplanes fly?",
    "What is quantum computing?"
]):
    print(response)
```

使用 `batch_as_completed()` 时，返回结果**可能不会按照输入顺序排列**。

每个返回结果都会包含对应的**输入索引（Input Index）**，你可以利用该索引将结果重新排列为原始输入顺序。

---

当使用 `batch()` 或 `batch_as_completed()` 处理大量输入时，你可能希望限制**最大并行调用数量**。

可以通过在 `RunnableConfig` 字典中设置 `max_concurrency` 属性来实现。

### 设置最大并发数（Batch with max concurrency）

```python
model.batch(
    list_of_inputs,
    config={
        "max_concurrency": 5,  # 最多同时进行 5 个并行调用
    }
)
```

有关 `RunnableConfig` 支持的完整配置项，请参阅 **RunnableConfig** 参考文档。

# Tool calling（工具调用）

模型可以请求调用工具（Tool），以执行诸如从数据库获取数据、搜索网页或运行代码等任务。

一个工具（Tool）由以下两部分组成：

1. **Schema（模式）**
   - 工具名称（Name）
   - 工具描述（Description）
   - 参数定义（Argument Definitions，通常为 JSON Schema）

2. **Function（函数）或 Coroutine（协程）**
   - 实际执行任务的代码。

你可能会听到 **Function Calling（函数调用）** 这一术语，在 LangChain 中，它与 **Tool Calling（工具调用）** 是可以互换使用的。

下面展示了用户与模型之间工具调用的基本流程：

```text
                 User（用户）
                       │
                       │
 "What's the weather in SF and NYC?"
                       │
                       ▼
                 Model（模型）
      分析用户请求，决定需要调用哪些工具
                       │
          ┌────────────┴────────────┐
          ▼                         ▼
get_weather("San Francisco")   get_weather("New York")
          │                         │
          ▼                         ▼
      SF 天气数据               NYC 天气数据
          └────────────┬────────────┘
                       ▼
                 Model（模型）
      处理工具返回结果并生成最终回复
                       │
                       ▼
"SF: 72°F sunny, NYC: 68°F cloudy"
```

为了让模型能够使用你定义的工具，必须先通过 `bind_tools()` 将工具绑定到模型。

完成绑定后，在后续调用中，模型可以根据需要自主选择调用任意一个已绑定的工具。

一些模型提供商还提供了**内置工具（Built-in Tools）**，可以通过模型参数或调用参数启用（例如 `ChatOpenAI`、`ChatAnthropic`）。

具体支持情况请查阅对应模型提供商的参考文档。

关于工具的创建方式以及更多配置选项，请参阅 **Tools（工具）** 指南。

---

## Binding user tools（绑定用户自定义工具）

```python
from langchain.tools import tool

@tool
def get_weather(location: str) -> str:
    """Get the weather at a location."""
    return f"It's sunny in {location}."


model_with_tools = model.bind_tools([get_weather])

response = model_with_tools.invoke("What's the weather like in Boston?")

for tool_call in response.tool_calls:
    # 查看模型生成的工具调用
    print(f"Tool: {tool_call['name']}")
    print(f"Args: {tool_call['args']}")
```

当绑定的是**用户自定义工具**时，模型返回的响应中会包含一个**工具执行请求（Tool Call）**。

如果你是**单独使用模型**（而不是使用 Agent），那么需要由你自己：

1. 执行模型请求调用的工具；
2. 获取工具执行结果；
3. 将工具执行结果返回给模型，以便模型在后续推理过程中使用。

如果使用的是 **Agent**，则 **Agent Loop（Agent 循环）** 会自动完成整个工具执行流程，你无需手动处理。

# Structured output（结构化输出）

可以要求模型按照指定的 **Schema（模式）** 返回响应。

这对于确保模型输出能够被轻松解析，并在后续处理中直接使用非常有用。

LangChain 支持多种 Schema 类型以及多种实现结构化输出的方法。

要了解更多内容，请参阅 **Structured output** 文档。

支持的 Schema 类型包括：

- Pydantic
- TypedDict
- JSON Schema

其中，**Pydantic 模型**功能最丰富，支持：

- 字段校验（Field Validation）
- 字段描述（Field Description）
- 嵌套结构（Nested Structures）

```python
from pydantic import BaseModel, Field

class Movie(BaseModel):
    """A movie with details."""
    title: str = Field(description="The title of the movie")
    year: int = Field(description="The year the movie was released")
    director: str = Field(description="The director of the movie")
    rating: float = Field(description="The movie's rating out of 10")

model_with_structure = model.with_structured_output(Movie)

response = model_with_structure.invoke(
    "Provide details about the movie Inception"
)

print(response)
# Movie(
#     title="Inception",
#     year=2010,
#     director="Christopher Nolan",
#     rating=8.8
# )
```

---

## 结构化输出的关键注意事项（Key considerations for structured output）

### Method 参数

部分模型提供商支持多种生成结构化输出的方法：

- **`json_schema`**
  
  使用模型提供商原生支持的 **JSON Schema** 结构化输出能力。

- **`function_calling`**
  
  通过强制模型进行一次符合指定 Schema 的 **Tool Calling（函数调用）** 来生成结构化输出。

- **`json_mode`**
  
  一些模型提供商提供的 `json_schema` 的早期实现方式。

  它能够保证输出是合法的 JSON，但需要你在 Prompt 中自行描述 JSON 的结构。

---

### Include raw（返回原始响应）

将 `include_raw=True` 传递给 `with_structured_output()` 后，可以同时获得：

- 解析后的结构化对象（Parsed Output）
- 原始的 `AIMessage`

这样可以方便获取 Token 使用量、响应元数据等信息。

---

### Validation（数据校验）

不同 Schema 类型的数据校验方式不同：

- **Pydantic**

  自动进行字段校验（Automatic Validation）。

- **TypedDict**

  需要开发者自行完成数据校验。

- **JSON Schema**

  同样需要开发者手动进行数据校验。

---
# Advanced topics（高级主题）

## Model profiles（模型配置文件）

> **Model profiles** 需要 `langchain>=1.1`。

LangChain 聊天模型可以通过 `profile` 属性公开一个字典，用于描述模型支持的功能和能力：

```python
model.profile

# {
#   "max_input_tokens": 400000,
#   "image_inputs": True,
#   "reasoning_output": True,
#   "tool_calling": True,
#   ...
# }
```

完整字段请参考 API Reference（API 参考文档）。

大部分模型配置数据由 **models.dev** 项目提供，这是一个用于提供模型能力数据的开源项目。

为了更好地配合 LangChain 使用，这些数据还增加了一些额外字段，并会随着上游项目的发展持续保持同步。

模型配置数据可以帮助应用根据模型能力动态调整行为。例如：

- Summarization Middleware（摘要中间件）可以根据模型上下文窗口大小自动触发摘要。
- `create_agent` 中的 Structured Output（结构化输出）策略可以自动推断（例如检查模型是否支持原生 Structured Output）。
- 可以根据模型支持的模态（Modality）以及最大输入 Token 数限制模型输入。
- Deep Agents Code 会根据模型配置过滤模型选择器，仅显示支持 `tool_calling` 和文本输入输出（Text I/O）的模型，同时展示上下文窗口大小和能力标志。

### Updating or overwriting profile data（更新或覆盖模型配置）

> **Model profiles** 目前属于 Beta 功能，其数据格式未来可能发生变化。

---

## Multimodal（多模态）

部分模型能够处理和返回除文本之外的数据，例如：

- 图片（Images）
- 音频（Audio）
- 视频（Video）

可以通过提供 **Content Blocks（内容块）** 向模型传递这些非文本数据。

所有具备多模态能力的 LangChain Chat Model 都支持：

- 跨 Provider 的标准数据格式（参见 Messages 指南）
- OpenAI Chat Completions 格式
- Provider 原生格式（例如 Anthropic 原生格式）

更多内容请参阅 **Messages Guide** 中的 Multimodal 章节。

部分模型还支持返回多模态内容。

如果模型生成了图片等内容，返回的 `AIMessage` 中将包含对应类型的 Content Blocks。

### Multimodal output（多模态输出）

```python
response = model.invoke("Create a picture of a cat")

print(response.content_blocks)

# [
#     {
#         "type": "text",
#         "text": "Here's a picture of a cat"
#     },
#     {
#         "type": "image",
#         "base64": "...",
#         "mime_type": "image/jpeg"
#     },
# ]
```

关于不同模型提供商支持情况，请参阅对应 Integration（集成）文档。

---

## Reasoning（推理）

许多模型具备执行**多步推理（Multi-step Reasoning）**的能力，以得出最终结论。

通常，它们会将复杂问题拆分成多个更容易处理的小步骤。

如果底层模型支持，你还可以获取模型的推理过程，以帮助理解模型是如何得到最终答案的。

### Stream reasoning output（流式推理输出）

```python
for chunk in model.stream("Why do parrots have colorful feathers?"):
    reasoning_steps = [
        r
        for r in chunk.content_blocks
        if r["type"] == "reasoning"
    ]

    print(reasoning_steps if reasoning_steps else chunk.text)
```

根据不同模型，你还可以指定模型进行推理时投入的推理强度（Reasoning Effort）。

例如：

- `"low"`
- `"high"`

或者使用整数 Token Budget（推理 Token 配额）。

有些模型还支持完全关闭推理功能。

具体支持方式请查阅对应聊天模型的 Integration（集成）页面或 API Reference。

---

## Local models（本地模型）

LangChain 支持在本地硬件上运行模型。

适用于以下场景：

- 数据隐私要求较高；
- 希望运行自定义模型；
- 希望避免云端模型带来的调用成本。

**Ollama** 是目前运行本地聊天模型和 Embedding 模型最简单的方法之一。

---

## Prompt caching（Prompt 缓存）

许多模型提供商提供 Prompt Caching（Prompt 缓存）功能，以降低重复处理相同 Token 时的延迟和成本。

缓存可以在三个层面启用：

### 1. Implicit provider caching（提供商自动缓存）

模型提供商自动命中缓存，无需任何配置。

例如：

- OpenAI
- Gemini

---

### 2. Provider-level explicit controls（提供商显式缓存控制）

部分 Provider 允许开发者手动指定缓存位置，以获得更好的控制能力或确保缓存生效。

例如：

- `ChatOpenAI`（通过 `prompt_cache_key`）
- Anthropic 的 `cache_control`
- Gemini
- AWS Bedrock 的 `cachePoint`

---

### 3. LangChain Middleware（LangChain 中间件）

对于 Agent，LangChain 可以通过 Middleware 自动优化稳定内容（如 System Prompt、Tool Definition）的缓存。

例如：

- `AnthropicPromptCachingMiddleware`
- `BedrockPromptCachingMiddleware`

---

Prompt 缓存通常只有在输入 Token 数达到最低阈值后才会生效。

具体要求请参考对应 Provider 的文档。

缓存命中情况会体现在模型响应的 Usage Metadata（使用统计信息）中。
