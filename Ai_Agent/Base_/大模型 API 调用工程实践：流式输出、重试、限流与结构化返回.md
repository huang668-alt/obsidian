很多 AI 应用的第一个版本都很“顺”：本地调通一个大模型 API，页面上能看到回答，Demo 就算跑起来了。

但一上生产，麻烦马上变得具体：

+ 用户等了 8 秒还看不到第一个字，以为系统卡死，直接刷新页面。
+ 模型返回了一半 JSON，前端解析失败，后端日志里只有一串残缺的 `{"answer": "根因是`。
+ 供应商偶发 429，你的服务开始疯狂重试，越重试越被限流。
+ 用户点了取消，浏览器断开了，但后端还在消耗 Token。
+ 同一个业务请求因为重试执行了两次，落库、扣费、发通知全重复了。

Guide 见过太多这样的事故。真正难的并非”怎么发一个 HTTP 请求给模型”，难点在于**如何把大模型 API 当成一个不稳定、昂贵、受配额约束的外部依赖来治理**。

本文覆盖：

1. **完整链路**：一次 AI 请求从业务入口、Prompt 组装、模型网关、供应商 API 到流式响应、解析、落库、观测是怎么跑起来的。
2. **流式输出**：Streaming 为什么能降低 TTFT，SSE、WebSocket、HTTP chunked 分别适合什么场景，后端如何处理取消、超时、断流和重连。
3. **重试与幂等**：哪些错误可以重试，哪些不能，指数退避、抖动、幂等 Key、请求去重和重复响应怎么设计。
4. **限流与配额**：用户级、租户级、模型级、供应商级限流怎么分层，Token 预算、429 处理、排队、降级和熔断怎么落地。
5. **结构化返回**：JSON Mode、JSON Schema、Structured Outputs 和 Function Calling 的工程价值，以及失败兜底策略。

## 一次生产级 LLM 调用包含哪些阶段？
很多人排查大模型调用问题时，只盯着供应商返回了什么。这个视角太窄。

一次生产级 LLM 调用，本质上是一条跨业务系统、上下文系统、模型网关、外部供应商和前端展示层的链路。任何一段没有治理好，最后都会表现成“模型不稳定”。

拆开看，一次请求通常包含 8 个阶段：

1. **业务请求进入**：校验用户身份、租户、套餐、功能权限、请求大小。
2. **上下文组装**：拼 System Prompt、用户输入、历史消息、RAG 证据、工具 Schema、输出格式约束。
3. **Token 预算预估**：估算输入 Token，预留输出 Token，决定是否裁剪历史、压缩上下文或换小模型。
4. **模型网关路由**：选择模型、供应商、区域、超时参数、重试策略、限流桶。
5. **供应商 API 调用**：同步返回或流式返回，可能经过 SSE、WebSocket 或普通 HTTP 响应体。
6. **响应解析**：处理 delta、finish reason、tool call、usage、拒答、结构化 JSON、异常中断。
7. **状态回写**：保存完整回答、增量片段、Token 用量、调用成本、失败原因和业务状态。
8. **观测与告警**：记录 traceId、providerRequestId、TTFT、总耗时、重试次数、429 次数、解析失败率。

很多团队栽的最多的一件事：**把模型网关当成透明代理**。它不是代理，它是 AI 应用的稳定性控制面。

如果没有网关，每个业务系统都会自己处理 API Key、超时、重试、限流、日志、供应商切换。短期看省事，长期一定变成事故放大器。Guide 的建议是：哪怕第一版很轻，也要把模型调用收口到一个统一的 `LLMGateway`。

## 同步返回和流式返回有什么区别？
默认的同步调用很好理解：后端发起请求，模型生成完全部内容后，一次性返回完整结果。

流式输出则是边生成边返回。模型每产生一段文本或一个事件，供应商就通过长连接把增量推给调用方。OpenAI 官方文档把 HTTP streaming 放在 SSE 场景下描述；Anthropic Messages API 也支持通过 SSE 增量返回事件；Gemini API 同样提供标准、流式和实时相关接口。具体字段和模型能力会变，**以官方文档最新展示为准**。

**为什么 Streaming 能降低 TTFT？**

TTFT（Time To First Token）指从请求发出到收到第一个可展示 Token 的时间。

同步返回时，用户要等模型生成完整答案。例如模型要生成 800 个 Token，后端必须等这 800 个 Token 都完成才把结果返回。

流式返回时，用户只要等模型开始生成第一个片段，就能看到内容逐步出现。

流式输出不是性能魔法。它没有让模型少算 Token，也不会天然省钱。它只是把等待过程拆成了可感知的进度，让用户觉得系统“活着”。

| 对比项 | 同步返回 | 流式返回 |
| --- | --- | --- |
| 首字延迟 | 高，需要等完整结果 | 低，收到第一个片段即可展示 |
| 端到端总耗时 | 取决于完整生成时间 | 通常仍取决于完整生成时间 |
| 前端体验 | 像提交表单后等待结果 | 像聊天软件逐字出现 |
| 后端实现 | 简单，拿到完整字符串再处理 | 复杂，需要处理增量事件、取消、断流 |
| 结构化解析 | 简单，完整 JSON 一次解析 | 需要缓存完整内容，或使用增量解析器 |
| 适合场景 | 短文本、后台任务、严格事务 | 聊天、写作、报告生成、长回答 |
| 不适合场景 | 用户强交互的长回答 | 强事务、必须一次性校验完整结果的链路 |


Guide 的经验：面向用户展示的长文本默认用流式，后台批处理和强结构化任务默认用同步。

## ⭐️ SSE、WebSocket 和 HTTP chunked 这三种流式协议怎么选
流式输出有几种常见承载方式，别把它们混成一个东西。

| 方式 | 核心特点 | 适合场景 | 边界 |
| --- | --- | --- | --- |
| SSE | 浏览器原生 `EventSource`<br/>，服务端到客户端单向推送，格式是 `text/event-stream` | 文本聊天、模型增量输出、状态通知 | 单向通信；复杂双向控制需要额外 HTTP 请求 |
| WebSocket | 双向长连接，客户端和服务端都能随时发消息 | 实时语音、多人协作、需要频繁取消或插话 | 连接管理更复杂，网关、鉴权、心跳都要自己管好 |
| HTTP chunked | HTTP/1.1 的分块传输机制，响应体分块发送 | 后端到后端流式代理、低层传输 | 它是传输机制，不是应用事件协议；HTTP/2 之后有自己的流式机制 |


SSE 的优势是简单。浏览器端几行代码就能接收事件，服务端按 `data:` 一段段写出去即可。MDN 对 EventSource 的描述也强调了它和 WebSocket 的区别：SSE 是服务端到客户端的单向数据流。

WebSocket 适合更实时、更复杂的交互。比如语音 Agent 里，客户端要不断上传音频，服务端要不断返回 ASR、LLM、TTS 状态，还要支持用户中途打断。这种场景用 WebSocket 更自然。

HTTP chunked 更底层。很多服务端框架在没有 `Content-Length` 的情况下会用分块响应，它能实现“边写边发”，但不会帮你定义事件类型、重连语义、消息边界。业务层仍然要自己设计协议。

### SSE 协议的事件边界
SSE 在传输层仍是 HTTP，但**应用层是一份 UTF-8 纯文本协议**。每个事件由若干行字段组成，事件之间必须用**空行**结束，也就是连续两个换行符 `\n\n`。

常用字段如下：

| 字段 | 作用 |
| --- | --- |
| `data` | 业务载荷；允许多行 `data:`<br/>，客户端会按规范拼接 |
| `event` | 自定义事件名；浏览器默认事件类型是 `message` |
| `id` | 事件序号；配合浏览器重连语义可做断点提示 |
| `retry` | 建议的重连间隔（毫秒） |


`**\n\n**`** 是事件分隔符**。只要在“本应属于同一段模型增量”的字符串里出现了“裸的换行”，就有可能被客户端解析成“上一个事件已结束、下一个事件开始”。这是很多团队在 Demo 里没问题、一上对话界面加 Markdown 或列表就炸裂的根因。

Guide 在[《SpringAI 智能面试平台+RAG 知识库》](https://javaguide.cn/zhuanlan/interview-guide.html)的知识库问答里用的就是 SSE：模型一边生成，浏览器一边打字机展示；链路不长，但协议细节一个不落下。

### Spring Boot + Spring AI 的 SSE 写法
Java 侧常见做法是 `**Content-Type: text/event-stream**`，再用响应式流往外推。Spring 提供了 `ServerSentEvent<T>`，避免手写 `data:` 和 `\n\n` 拼串出错：

```java
// 定义一个 GET 接口，路径为 /chat/stream
// produces 指定返回的数据类型为 Server-Sent Events (SSE)，也就是 text/event-stream
@GetMapping(value = "/chat/stream", produces = MediaType.TEXT_EVENT_STREAM_VALUE)
public Flux<ServerSentEvent<String>> stream() {
    
    // 创建一个 Flux，按固定时间间隔发射递增的 Long 值（seq），这里每 500 毫秒一次
    return Flux.interval(Duration.ofMillis(500))
    
        // 将每个递增的 Long 值映射为一个 Server-Sent Event
        .map(seq -> ServerSentEvent.<String>builder()
            
            // SSE 的唯一 ID，通常用作事件标识
            .id(Long.toString(seq))
            
            // SSE 的事件类型，这里命名为 "token"，客户端可以根据类型处理不同事件
            .event("token")
            
            // SSE 的数据内容，这里简单返回字符串 "片段-" + 序号
            .data("片段-" + seq)
            
            // 如果连接中断，客户端或服务器可以在指定时间后自动重试，这里设置为 3 秒
            .retry(Duration.ofSeconds(3))
            
            // 构建 ServerSentEvent 对象
            .build()
        );
}
```

和大模型对接时，增量源头通常是 SDK 或框架暴露的流式接口。以 Spring AI 为例，`ChatClient` 侧启用流式后拿到 `Flux<String>`，再映射成 SSE 推给前端：

```java
Flux<String> tokens = chatClient.prompt()
    .system(systemPrompt)
    .user(userPrompt)
    .stream()
    .content();
```

```python
# 假设 chat_client 是你的 Python 聊天客户端对象
tokens = chat_client.prompt(
    system=system_prompt,
    user=user_prompt,
).stream()  # 返回异步生成器或迭代器

# 逐个 token 遍历
async for token in tokens:
    print(token, end="")  # 处理每个 token
```

工程上要心里有数：WebMVC + `Flux` 只是在 Controller 出口用了响应式类型做 SSE，底层仍是 Servlet 容器。线程池、连接数和超时仍要按「长请求」来治理；Java 21 虚拟线程可以把「占着一个平台线程傻等」的成本降下来，这对动辄数十秒的生成链路很实用。

### 模型正文换行导致的 SSE 截断
假设你把某个 token 或片段直接塞进 `data:`，而片段里含有真实的换行符 `\n`。协议眼里这就是「字段结束 / 新字段开始」，前端事件边界立刻错位。

血泪教训：别指望「模型不太会输出换行」——列表、代码块、道歉话术一来，线上必现。

一条务实的做法是在应用层约定转义，例如在出站前把 `\n`、`\r` 转成字面量 `\\n`、`\\r`，前端收到后再还原：

```java
.map(chunk -> ServerSentEvent.<String>builder()
     .data(chunk.replace("\n", "\\n").replace("\r", "\\r"))
     .build())
```

```java
const text = chunk.replace(/\\n/g, "\n").replace(/\\r/g, "\r");
```

更「协议原生」的做法也能做：把一行正文拆成多行 `data:`，由客户端按规范拼回一行内的 `\n`。选型核心是：团队要在服务端和前端固定同一种语义，并把单元测试覆盖到「含换行、含 CR、含空行」的片段。

### Nginx 与网关的流式配置
只要前面挂了 Nginx 或其它响应缓冲型网关，`text/event-stream` 可能被攒够一整块才下发，用户侧的 TTFT 体感瞬间回到同步接口。

最小改动通常是：

```nginx
# 匹配请求路径以 /api/ 开头的请求
location /api/ {

  # 将请求代理转发给名为 backend 的上游服务
  proxy_pass http://backend;

  # 关闭代理响应缓冲，保证流式输出能实时推送给客户端
  proxy_buffering off;

  # 禁用缓存，确保每次请求都直接获取最新数据
  proxy_cache off;

  # 设置代理读取超时时间为 300 秒，避免长时间响应被中断
  proxy_read_timeout 300s;

  # 清空 Connection 请求头，避免 SSE 或长连接被关闭
  proxy_set_header Connection "";

  # 向浏览器返回不缓存的响应头，确保客户端不使用缓存
  add_header Cache-Control no-cache;
}
```

再配合 `proxy_read_timeout`（或等价配置）把「长生成」守住，否则链路会在沉默超时处被中间件切断。

`proxy_read_timeout` 的作用是：**规定 Nginx 在代理后端服务时，最长允许多久收不到后端数据。超过这个时间，就认为后端失联并断开连接。**

如果少了 `proxy_buffering off`，前端经常会出现：后端明明在流式输出，但浏览器一直不显示，等全部生成完才一次性返回。

### 流式异常的四类场景
流式链路最容易出问题的地方，往往不是“怎么开始”，而是“怎么结束”。

**第一类：用户取消。**

用户关闭页面、点击停止生成、切换会话，都应该触发取消。后端要同时取消：

+ 到供应商 API 的请求。
+ 正在解析的响应流。
+ 后续 TTS、工具调用、落库任务。
+ 还没提交的增量缓存。

血泪教训：不要只在前端停止展示。前端停了，后端还在生成，账单照样跑。

**第二类：超时。**

超时至少分三层：

+ 连接超时：连不上供应商。
+ TTFT 超时：连接上了，但迟迟没有第一个事件。
+ 总时长超时：一直有输出，但超过业务可接受时间。

三者要分开记录。TTFT 超时通常指向模型排队、上下文过长或供应商抖动；总时长超时可能只是用户让模型写太长。

**第三类：断流。**

断流时不要轻易把半截内容当成成功。正确做法是记录 `finish_reason` 或最后事件状态，如果没有正常结束标记，就把本次调用标记为 `INTERRUPTED`，前端展示“已中断，可重新生成”，而不是悄悄落成完整答案。

**第四类：重连。**

SSE 的 `EventSource` 有自动重连能力，但大模型输出不是普通新闻推送。重连后是否能从断点续传，取决于你的服务端是否保存了事件序号、增量片段和供应商调用状态。多数情况下，供应商侧流已经断掉，无法真正从 Token 级别续上。

更稳的做法是：

+ 服务端为每个流式响应生成 `messageId` 和递增 `sequence`。
+ 已发送片段写入短期缓存。
+ 前端重连时先补发已缓存片段。
+ 如果供应商流已结束或失效，提示用户重新生成，而不是假装无缝续写。

## 哪些错误能重试，哪些不能重试？
重试是后端工程师最熟悉也最容易滥用的能力。

大模型 API 的重试有两个特殊点：

1. **请求贵**：失败请求也可能消耗配额，甚至已经消耗了部分 Token。
2. **输出非确定**：即使 Prompt 一样，第二次返回也可能和第一次不同。

### 错误类型对照表
| 类型 | 示例 | 是否建议重试 | 处理方式 |
| --- | --- | --- | --- |
| 网络瞬断 | 连接重置、DNS 抖动、读超时 | 可以 | 指数退避 + 抖动，限制最大次数 |
| 供应商 5xx | 500、502、503、504 | 可以 | 短暂重试，超过阈值切换模型或降级 |
| 供应商过载 | Anthropic 529、类似 overloaded 错误 | 可以 | 慢重试，必要时熔断该供应商 |
| 429 限流 | RPM、TPM、RPD、并发限制超出 | 谨慎 | 优先看 `Retry-After`<br/> 和限流头，排队或降级 |
| 流式中断 | 未收到正常结束事件 | 视场景 | 用户可见任务不自动重试，后台任务可幂等重试 |
| 400 参数错误 | Schema 不合法、字段缺失、上下文超限 | 不建议 | 修请求，不要重试同一 payload |
| 401/403 鉴权错误 | API Key 无效、权限不足 | 不建议 | 告警并停用对应 Key |
| 安全拒答 | 内容策略拒绝 | 不建议 | 进入业务拒答流程 |
| 解析失败 | JSON 不完整、字段类型错误 | 可有限重试 | 带失败原因二次修复，最多 1-2 次 |


OpenAI 官方限流文档建议对 rate limit error 使用随机指数退避，同时提醒失败请求也会计入每分钟限制；Anthropic 官方错误文档中明确列出了 429 rate limit、500 api error、504 timeout、529 overloaded 等错误类型。这里的结论不是某一家供应商专属，而是外部模型依赖的通用治理思路。

### 指数退避和抖动
**指数退避**的核心是：第 1 次失败等一小会儿，第 2 次失败等更久，第 3 次再更久，直到达到最大等待时间或最大重试次数。

**抖动（Jitter）**的核心是：不要让所有请求在同一时间点一起重试。否则系统刚从限流里恢复，马上又被同一批重试打爆。

一个实用公式：

```java
sleep = min(maxDelay, baseDelay * 2^retryCount) + random(0, jitter)
```

生产里别忘了加两条硬约束：

+ **最大重试次数**：通常 2-3 次足够，别无限重试。
+ **总体截止时间**：用户请求有整体 SLA，例如 15 秒，到点就失败，不要因为重试拖成 1 分钟。

### 幂等 Key 和去重机制
只要有重试，就必须讨论幂等。幂等 Key 可以由业务生成，例如：

```plain
tenantId:userId:conversationId:messageId:attemptGroup
```

服务端拿到请求后，先查这个 Key 是否已经存在：

+ 如果已经成功，直接返回历史结果。
+ 如果正在生成，返回同一个流式任务的订阅地址。
+ 如果失败且允许重试，创建新的 attempt，但仍然挂在同一个业务消息下。
+ 如果失败但不可重试，直接返回失败原因。

这能避免两个坑：

1. 用户狂点“重新发送”，后端创建多个模型调用。
2. 网关超时后自动重试，第一次其实已经成功落库，第二次又写了一条重复消息。

### 响应重复的处理
重试后的响应可能重复、冲突或部分重叠。

对聊天类应用，建议把一次用户消息下的多次模型调用区分为：

+ `message_id`：业务消息 ID，对用户可见。
+ `attempt_id`：模型调用尝试 ID，对系统可见。
+ `provider_request_id`：供应商请求 ID，用于排查。
+ `stream_sequence`：增量片段序号，用于去重和补发。

落库时，只允许一个 attempt 成为 `final`。其他 attempt 保留为诊断记录，不参与用户上下文。这样既能排查问题，又不会污染下一轮 Prompt。

## ⭐️ 为什么要限流？如何限流？
很多团队的限流意识，是从收到第一个 429（ 请求次数超过了服务器允许的限制（被限流了）。  ） 开始的。

这已经晚了。等供应商把你拦住，说明你的系统里根本没有容量管理。供应商的 429 是最后一道墙——如果你把它当容量规划工具用，迟早会在流量尖峰时被连续打脸。

### 限流的四层架构
| 层级 | 限制对象 | 核心目的 | 常见策略 |
| --- | --- | --- | --- |
| 用户级 | 单个用户或账号 | 防止滥用、误操作、脚本刷接口 | 每分钟请求数、每日 Token 上限 |
| 租户级 | 企业、团队、项目 | 控制套餐成本和公平性 | 月度配额、并发上限、优先级队列 |
| 模型级 | 某个模型或模型族 | 避免热门模型被打满 | 模型维度令牌桶、降级到备用模型 |
| 供应商级 | OpenAI、Anthropic、Gemini 等 | 保护外部依赖和 API Key | 全局 RPM、TPM、并发、熔断 |


Gemini 官方限流文档把限流维度拆成 RPM、输入 TPM、RPD，并说明限制按项目而不是单个 API Key 应用；OpenAI 官方文档也展示了请求数、Token 数、剩余额度等 rate limit header。具体数值和模型关系变化很快，生产系统不要把文档里的静态数字写死，要从控制台、响应头或配置中心动态管理。

### 为什么 Token 预算比请求数更重要
传统 API 限流通常按 QPS。大模型 API 只按 QPS 不够。

两个请求的成本可能差很多：

+ 请求 A：输入 500 Token，输出 100 Token。
+ 请求 B：输入 80K Token，输出 8K Token。

它们都是 1 次请求，但对模型推理、供应商配额和账单的压力完全不是一个量级。

所以限流至少要同时看：

+ **RPM（ Revolutions Per Minute  ）**：每分钟请求数。
+ **TPM（ Tokens Per Minute  ）**：每分钟 Token 数。
+ **并发数**：正在生成的请求数量。
+ **上下文大小**：单请求输入 Token。
+ **最大输出**：`max_tokens` 或类似参数。
+ **日/月预算**：租户或用户总成本。

Guide 的建议是：**先扣预算，再发请求**。

请求进入网关后，先估算 `input_tokens + reserved_output_tokens`，在用户、租户、模型、供应商几个桶里尝试扣减。扣不到就不要发给供应商，直接排队、降级或拒绝。

### 常见限流策略对比
| 策略 | 适合场景 | 优点 | 缺点 |
| --- | --- | --- | --- |
| 固定窗口 | 简单后台任务、管理接口 | 实现简单，容易统计 | 窗口边界容易突刺 |
| 滑动窗口 | 用户级请求限制 | 边界更平滑 | 实现和存储成本更高 |
| 令牌桶 | 模型调用、Token 预算 | 支持一定突发，工程上常用 | 参数需要调优 |
| 漏桶 | 严格平滑出流量 | 输出稳定，适合保护供应商 | 突发体验差 |
| 并发信号量 | 流式生成、长任务 | 能限制同时占用连接 | 不控制单个请求 Token 成本 |
| 优先级队列 | 多租户、多套餐 | 能保护高优先级请求 | 需要处理饥饿和超时 |


生产里通常不是选一个，而是组合：

+ 用户级：滑动窗口 + 日 Token 上限。
+ 租户级：令牌桶 + 月度预算
+ 模型级：令牌桶 + 并发信号量
+ 供应商级：全局令牌桶 + 熔断器
+ 流式请求：并发信号量 + 总时长限制

### 收到 429 应该怎么处理
HTTP 429 表示请求过多。后端处理 429 时，建议按这个顺序：

1. **读取 **`**Retry-After**`** 或供应商 rate limit header**：有明确恢复时间就尊重它。
2. **标记限流维度**：是请求数打满，还是 Token 打满，还是日配额耗尽。
3. **短请求可排队**：例如后台摘要任务可以进延迟队列。
4. **用户交互请求少重试**：用户等不起时，直接提示稍后再试或切换轻量模型。
5. **供应商连续 429 时熔断**：不要让所有请求继续撞墙。

一个典型降级链路：

```plain
优先模型可用 -> 正常调用
优先模型 429 -> 切备用同级模型
备用模型也限流 -> 切轻量模型并缩短输出
仍不可用 -> 排队或返回"当前请求繁忙"
```

这里要避免一个误区：降级不是偷偷变差。如果轻量模型会影响答案质量，要在业务层明确标记，例如“当前为快速模式，复杂问题建议稍后重试”。

## 为什么要结构化返回？
很多业务一开始这样写 Prompt：

```plain
请分析用户问题，输出 JSON，字段包括 intent、confidence、answer。
```

然后后端直接 `JSON.parse()`。

这在 Demo 阶段很常见，但生产环境会遇到各种边缘情况：

+ 模型在 JSON 前加了一句“好的，以下是结果”。
+ 字段缺失。
+ 枚举值乱写。
+ 数字返回成字符串。
+ 流式返回时只拿到半个对象。
+ 安全拒答时压根不是业务 Schema。

所以结构化返回的核心不只是“看起来像 JSON”，更关键的是**让模型输出能被程序稳定消费**。

### JSON Mode、JSON Schema 和 Structured Output 的区别
| 方式 | 约束强度 | 工程价值 | 风险 |
| --- | --- | --- | --- |
| 普通自然语言 | 几乎没有 | 适合展示型回答 | 不适合程序解析 |
| Prompt 要求 JSON | 弱 | 简单、跨模型 | 容易混入解释文本或缺字段 |
| JSON Mode | 中 | 通常能保证语法是 JSON | 不一定符合业务字段 Schema |
| JSON Schema | 强 | 明确字段、类型、必填、枚举 | 不同供应商支持子集不同 |
| Structured Outputs | 更强 | 供应商在解码或 SDK 层增强约束 | 受模型、SDK、Schema 子集限制 |
| Function Calling / Tool Use | 面向动作 | 适合让模型选择工具和参数 | 不是最终自然语言答案的万能替代 |


OpenAI 官方 Structured Outputs 文档强调可以让输出遵循开发者提供的 JSON Schema，并提供 `strict` 相关配置；Gemini 官方文档说明 structured output 使用 `response_format` 和 JSON Schema，且支持的是 JSON Schema 的子集；Anthropic 官方文档也提供 Structured Outputs 和 Strict tool use，二者解决的问题并不完全一样。具体模型、字段、Schema 子集变化较快，仍然以官方文档最新展示为准。

### 普通 JSON 和结构化输出的工程差异
普通自然语言返回像“人写给人看的说明”，结构化返回像“服务写给服务的接口”。

举个意图识别场景：

```json
{
  "intent": "refund_request",
  "confidence": 0.86,
  "entities": {
    "order_id": "202605080001",
    "reason": "商品破损"
  },
  "need_human_review": false
}
```

有了 Schema，后端可以做这些事：

+ `intent` 只能是有限枚举。
+ `confidence` 必须是数字。
+ `order_id` 可以为空，但类型必须稳定。
+ `need_human_review` 必须存在。
+ 解析失败时可以进入修复或人工兜底流程。

这就是结构化返回的价值：**把“模型生成”变成“可校验的数据契约”**。

### 结构化输出失败后如何兜底
结构化输出仍然可能失败。失败不一定是供应商能力问题，也可能是 Schema 太复杂、上下文冲突、输出被截断、安全策略拒答。

建议兜底分四级：

1. **本地校验**：用 JSON Schema、Jackson、Bean Validation 校验字段和类型。
2. **轻量修复**：只让模型修复格式，不重新生成业务内容。
3. **降级 Schema**：复杂对象拆成多个小对象，或先分类再抽取字段。
4. **人工或规则兜底**：高价值订单、金融、医疗、法务场景不要完全依赖自动修复。

一个实用原则：结构化返回失败时，不要把原始自然语言硬塞给下游系统。能展示给用户，不代表能被程序执行。

## Java 后端怎么落地 LLM 调用？
下面给一个简化版 Java 伪代码，重点不是绑定某个 SDK，而是展示工程结构：网关统一处理 Token 预算、限流、重试、流式解析、幂等和观测。

```java
/**
 * 大模型客户端抽象
 * 屏蔽底层差异——无论是 OpenAI、DeepSeek、Claude 还是本地模型，
 * 上层调用者只依赖这个接口，不感知具体实现。
 */
public interface LLMClient {

    /**
     * 同步调用：阻塞直到模型返回完整结果。
     * 适合短文本、延迟可控的场景。
     */
    LLMResponse chat(LLMRequest request);

    /**
     * 流式调用：Token 增量推送，边生成边消费。
     * 适合长文本、需要实时展示的场景。
     */
    void stream(LLMRequest request, StreamHandler handler);
}

/**
 * 流式事件处理器
 * 采用回调风格，与底层 SSE / WebSocket 解耦。
 * 实现类负责将事件转发给上游（HTTP 响应、消息队列等）。
 */
public interface StreamHandler {

    /** 流开始：收到首个 Token 前触发，携带本次调用的唯一 messageId。 */
    void onStart(String messageId);

    /**
     * 收到增量内容。
     *
     * @param messageId 本次调用 ID
     * @param sequence  严格递增序号，用于幂等去重和乱序检测
     * @param delta     本次新增的 Token 文本片段
     */
    void onDelta(String messageId, long sequence, String delta);

    /**
     * 流正常结束。
     * usage 包含本次实际消耗的 prompt/completion Token 数，用于计费和监控。
     */
    void onComplete(String messageId, LLMUsage usage);

    /** 流异常中断：error 携带原始异常，便于上游决策（重试/降级/报错）。 */
    void onError(String messageId, Throwable error);
}

/**
 * LLM 统一网关（Gateway Pattern）
 *
 * 将所有与模型调用相关的横切关注点集中在此处，业务层只管传 BusinessCommand、拿结果。
 *
 * 职责一览：
 *   1. 幂等控制   —— 防止网络重传导致同一请求被执行多次
 *   2. Token 预算 —— 估算本次调用的 Token 消耗，为限流提供依据
 *   3. RPM/TPM 限流 —— 按租户 + 模型维度控制请求速率和 Token 速率
 *   4. 重试机制   —— 对可恢复错误（限流、超时）自动退避重试
 *   5. 结构化输出校验 —— 确保模型返回的 JSON 符合业务 Schema
 *   6. 流式输出处理 —— 去重、缓冲、最终拼接校验
 *   7. 监控埋点   —— 成功/失败指标、延迟、Token 消耗全量上报
 */
public final class LLMGateway {

    /** 实际发起 HTTP 请求的模型客户端（如 OpenAI SDK 封装）。 */
    private final LLMClient client;

    /** 按租户 + 模型维度的双桶限流器（RPM + TPM）。 */
    private final RateLimiter rateLimiter;

    /**
     * 幂等记录存储（建议持久化，如 Redis / DB）。
     * 记录每个 idempotencyKey 的状态：RUNNING / SUCCESS / FAILED / INTERRUPTED。
     */
    private final IdempotencyStore idempotencyStore;

    /** Token 预算估算器：在真正调用前预估本次会消耗多少 Token。 */
    private final TokenEstimator tokenEstimator;

    /** 监控埋点客户端（Micrometer / OpenTelemetry 等）。 */
    private final Observation observation;

    public LLMGateway(LLMClient client, RateLimiter rateLimiter,
                      IdempotencyStore idempotencyStore,
                      TokenEstimator tokenEstimator, Observation observation) {
        this.client = client;
        this.rateLimiter = rateLimiter;
        this.idempotencyStore = idempotencyStore;
        this.tokenEstimator = tokenEstimator;
        this.observation = observation;
    }

    /**
     * 同步调用入口（含自动重试）。
     *
     * BusinessCommand 是一次完整业务请求的描述对象，携带：
     *   - 幂等键（idempotencyKey）
     *   - 租户 ID（tenantId）
     *   - 模型标识（model）
     *   - Prompt（systemPrompt / userPrompt）
     *   - 结构化输出 Schema
     *   - 超时时间等
     */
    public LLMResponse chatWithRetry(BusinessCommand command) {

        // ===== Step 1：幂等检查 =====
        // 同一 idempotencyKey 若已成功执行，直接返回历史结果，跳过后续所有流程。
        // 这是防止"支付重复扣款"类问题的第一道防线。
        String idemKey = command.idempotencyKey();
        IdempotencyRecord existed = idempotencyStore.find(idemKey);
        if (existed != null && existed.isSuccess()) {
            return existed.toResponse(); // 直接返回幂等缓存，不再调用模型
        }

        // ===== Step 2：构建请求 =====
        // 将业务语义（BusinessCommand）翻译成模型能理解的技术请求（LLMRequest）。
        LLMRequest request = buildRequest(command);

        // ===== Step 3：Token 预算估算 =====
        // 调用前预估 Token 数，为后续限流桶的扣减提供依据。
        // 注意：这是估算值，实际消耗以 response.usage() 为准。
        TokenBudget budget = tokenEstimator.estimate(request);

        // ===== Step 4：RPM / TPM 限流 =====
        // 按（租户, 模型）维度消耗令牌桶配额。
        // 若桶已空，此处会阻塞或抛出限流异常，防止打爆下游 API 配额。
        rateLimiter.acquire(command.tenantId(), request.model(), budget);

        // ===== Step 5：准备重试策略 =====
        // defaultPolicy() 通常包含：最大重试次数、指数退避、可重试异常白名单。
        RetryPolicy retryPolicy = RetryPolicy.defaultPolicy();
        Throwable lastError = null;

        // ===== Step 6：重试循环 =====
        for (int attempt = 0; attempt <= retryPolicy.maxRetries(); attempt++) {

            // 每次尝试生成独立的 attemptId，便于在幂等记录中区分不同轮次。
            String attemptId = idemKey + ":attempt:" + attempt;
            long startNanos = System.nanoTime();

            try {
                // 标记"执行中"，防止并发重入（分布式锁的轻量替代）。
                idempotencyStore.markRunning(idemKey, attemptId);

                // 调用底层模型，withAttemptId 会将 attemptId 注入请求头，
                // 供模型服务侧的幂等机制（如 OpenAI 的 request_id）使用。
                LLMResponse response = client.chat(request.withAttemptId(attemptId));

                // ----- 结构化输出校验 -----
                // 模型可能返回格式错误的 JSON；此处强制校验，不合规直接抛 NonRetryableLLMException。
                // 注意：校验失败不重试，因为重试大概率返回相同结果，属于 Prompt 设计问题。
                ParsedAnswer parsed = parseAndValidate(response.content(), command.schema());

                // 标记成功，持久化结果供后续幂等命中使用。
                idempotencyStore.markSuccess(idemKey, attemptId, response, parsed);

                // 上报成功指标：延迟、Token 消耗、重试次数。
                observation.recordSuccess(request, response.usage(), startNanos, attempt);

                return response;

            } catch (LLMException ex) {
                lastError = ex;

                // 上报失败指标，便于告警和 SLO 统计。
                observation.recordFailure(request, ex, startNanos, attempt);

                if (!retryPolicy.canRetry(ex, attempt)) {
                    // 不可重试（如鉴权失败、Schema 校验失败）：直接标记失败并上抛。
                    idempotencyStore.markFailed(idemKey, attemptId, ex);
                    throw ex;
                }

                // 可重试：按退避策略等待后进入下一轮。
                // nextDelay() 通常实现指数退避 + Jitter，避免惊群效应。
                sleep(retryPolicy.nextDelay(ex, attempt));
            }
        }

        // 超过最大重试次数，包装后上抛，lastError 保留根因。
        throw new LLMException("LLM request failed after retries", lastError);
    }

    /**
     * 流式调用入口。
     *
     * 与同步调用的主要差异：
     *   - 不重试（流中断后重试会导致下游重复消费）
     *   - 增量持久化每个 delta，支持断点续传
     *   - 在 onComplete 时才做最终结构化校验
     */
    public void stream(BusinessCommand command, StreamHandler downstream) {

        String idemKey = command.idempotencyKey();

        // enableStream() 在请求中设置 stream=true，告知模型以 SSE 方式返回。
        LLMRequest request = buildRequest(command).enableStream();

        TokenBudget budget = tokenEstimator.estimate(request);
        rateLimiter.acquire(command.tenantId(), request.model(), budget);

        String messageId = command.messageId();

        // StreamBuffer 负责：
        //   1. 按 sequence 去重（防止网络重传导致重复 Token）
        //   2. 按序拼接成完整文本，供 onComplete 校验使用
        StreamBuffer buffer = new StreamBuffer(messageId);

        idempotencyStore.markRunning(idemKey, messageId);

        client.stream(request, new StreamHandler() {

            @Override
            public void onStart(String ignored) {
                // 忽略底层 messageId，统一用业务层的 messageId 向下游透出。
                downstream.onStart(messageId);
            }

            @Override
            public void onDelta(String ignored, long sequence, String delta) {
                // 幂等去重：同一 sequence 的 delta 只处理一次。
                if (buffer.seen(sequence)) {
                    return;
                }

                buffer.append(sequence, delta); // 缓存到内存，供最终拼接

                // 持久化增量，支持客户端断线重连后从断点续传。
                idempotencyStore.appendDelta(messageId, sequence, delta);

                downstream.onDelta(messageId, sequence, delta); // 推给下游（如 SSE 客户端）
            }

            @Override
            public void onComplete(String ignored, LLMUsage usage) {
                String fullText = buffer.fullText(); // 拼接完整回答

                // 流完成后才有完整内容，此时做结构化校验。
                ParsedAnswer parsed = parseAndValidate(fullText, command.schema());

                // 持久化完整结果 + usage，供计费和幂等使用。
                idempotencyStore.markSuccess(idemKey, messageId, fullText, parsed, usage);

                downstream.onComplete(messageId, usage);
            }

            @Override
            public void onError(String ignored, Throwable error) {
                // 标记"中断"而非"失败"：保留已接收的部分内容，
                // 便于人工排查或后续断点续传。
                idempotencyStore.markInterrupted(idemKey, messageId, buffer.fullText(), error);

                downstream.onError(messageId, error);
            }
        });
    }

    /**
     * 将业务命令转换为模型请求。
     * 统一注入租户、消息 ID 等元数据，便于日志追踪和计费分摊。
     */
    private LLMRequest buildRequest(BusinessCommand command) {
        return LLMRequest.builder()
                .model(command.model())
                .systemPrompt(command.systemPrompt())
                .userPrompt(command.userPrompt())
                .context(command.context())
                .responseSchema(command.schema())   // 告知模型期望的输出格式（JSON Schema）
                .timeout(command.timeout())
                .metadata("tenantId", command.tenantId())
                .metadata("messageId", command.messageId())
                .build();
    }

    /**
     * 结构化输出校验。
     *
     * 将模型返回的字符串按 JSON Schema 解析并校验。
     * 失败抛 NonRetryableLLMException，重试循环会识别该异常并直接终止。
     */
    private ParsedAnswer parseAndValidate(String content, JsonSchema schema) {
        try {
            return ParsedAnswer.fromJson(content, schema);
        } catch (Exception ex) {
            throw new NonRetryableLLMException("Structured output validation failed", ex);
        }
    }

    /**
     * 重试等待。
     * 捕获 InterruptedException 并恢复中断标志，遵循 Java 并发规范。
     */
    private void sleep(Duration duration) {
        try {
            Thread.sleep(duration.toMillis());
        } catch (InterruptedException ex) {
            Thread.currentThread().interrupt(); // 恢复中断状态，让上层感知
            throw new LLMException("Retry sleep interrupted", ex);
        }
    }
}
```

这段代码有几个关键点：

+ **业务入口不直接调用供应商 SDK**，统一走 `LLMGateway`。
+ **先估算 Token 并扣限流桶**，避免发出去才发现没额度。
+ **幂等记录包住整次业务消息**，attempt 只是系统内部重试。
+ **同步和流式分开处理**，流式要记录 `sequence`，避免重连补发时重复。
+ **结构化解析在落库前做**，失败就进入失败状态，而不是污染业务数据。

真实项目里还要补充：

+ API Key 池和供应商路由。
+ 模型优先级和降级策略。
+ Prompt 版本号。
+ 响应内容安全审查。
+ usage 成本计算。
+ traceId 和 providerRequestId 对齐。
+ 流式取消信号向供应商请求传播。
+ SSE 出站契约：换行与事件边界的处理方式要与前端一致，网关关闭缓冲并放宽读超时。

## 没有指标就没有稳定性
AI 应用的观测不能只记录“调用成功/失败”。

至少要记录这些指标：

| 指标 | 含义 | 用途 |
| --- | --- | --- |
| TTFT | 首个 Token 返回时间 | 判断排队、上下文过长、供应商抖动 |
| E2E Latency | 端到端完成时间 | 判断用户体验和 SLA |
| Input Tokens | 输入 Token | 成本分析、上下文膨胀排查 |
| Output Tokens | 输出 Token | 成本分析、异常长回答排查 |
| Retry Count | 重试次数 | 识别供应商不稳定或策略过激 |
| 429 Rate | 限流比例 | 判断配额和限流桶是否合理 |
| Parse Failure Rate | 结构化解析失败率 | 判断 Schema、Prompt、模型适配问题 |
| Cancel Rate | 用户取消比例 | 判断响应太慢或生成太长 |
| Provider Error Rate | 供应商错误率 | 路由、降级、熔断依据 |


日志里建议带上这些字段：

```plain
trace_id
tenant_id
user_id
conversation_id
message_id
attempt_id
model
provider
prompt_version
input_tokens
output_tokens
ttft_ms
latency_ms
retry_count
finish_reason
error_type
provider_request_id
```

没有这些字段，线上排查会非常痛苦。用户说“刚才 AI 没返回”，你连是哪家供应商、哪个模型、哪次 attempt、有没有收到第一个 delta 都查不到。

## 面试问题
### 1. 大模型 API 调用的完整链路是什么
1. **业务请求进入**：校验用户身份、租户、套餐、功能权限、请求大小。
2. **上下文组装**：拼 System Prompt、用户输入、历史消息、RAG 证据、工具 Schema、输出格式约束。
3. **Token 预算预估**：估算输入 Token，预留输出 Token，决定是否裁剪历史、压缩上下文或换小模型。
4. **模型网关路由**：选择模型、供应商、区域、超时参数、重试策略、限流桶。
5. **供应商 API 调用**：同步返回或流式返回，可能经过 SSE、WebSocket 或普通 HTTP 响应体。
6. **响应解析**：处理 delta、finish reason、tool call、usage、拒答、结构化 JSON、异常中断。
7. **状态回写**：保存完整回答、增量片段、Token 用量、调用成本、失败原因和业务状态。
8. **观测与告警**：记录 traceId、providerRequestId、TTFT、总耗时、重试次数、429 次数、解析失败率。

核心点是：**LLM 调用不能只看作一个 HTTP 请求，它是一条需要治理的生产链路**。

### 2. Streaming 为什么能改善体验
Streaming 让模型边生成边返回，用户可以更早看到第一个 Token，因此降低 TTFT。它不保证总生成时间变短，也不天然减少 Token 成本。后端需要额外处理取消、超时、断流、重连、半成品 JSON 和增量落库。

### 3. SSE 和 WebSocket 怎么选
如果只是服务端向浏览器推模型文本，SSE 更简单，天然适合单向增量输出；落地时别忘了 `**text/event-stream**`** 对换行与事件边界敏感**，以及反向代理缓冲会把「流式」攒成「批量」。如果客户端也要频繁向服务端发数据，例如语音流、实时控制、多人协作、插话打断，WebSocket 更适合。HTTP chunked 更偏底层传输机制，业务层仍要自己定义消息边界和事件类型。

### 4. 哪些大模型 API 错误可以重试
**网络瞬断、连接重置、部分 5xx、504、供应商过载通常可以有限重试；**429 要结合 `Retry-After`、限流头、排队和降级处理；400 参数错误、401/403 鉴权错误、内容安全拒答通常不能重试。结构化解析失败可以做 1-2 次格式修复，但不要无限重试。

### 5. 为什么大模型调用必须做幂等
因为重试、用户重复点击、网关超时都会让同一个业务请求被执行多次。没有幂等 Key，就可能重复落库、重复扣费、重复发通知。正确做法是用业务消息 ID 生成幂等 Key，把多次模型调用 attempt 挂在同一条业务消息下，只允许一个 attempt 成为最终结果。

### 6. 限流为什么不能只按 QPS
因为大模型 API 的成本和压力主要由 Token 决定。一个 500 Token 请求和一个 80K Token 请求都是 1 次请求，但资源消耗差异很大。生产限流要同时看 RPM、TPM、并发数、上下文大小、最大输出和租户预算。

### 7. JSON Mode 和 Structured Outputs 有什么区别
JSON Mode 更关注“输出是合法 JSON”，但不一定符合你的业务 Schema。Structured Outputs 或 JSON Schema 约束更强，可以要求字段、类型、必填项、枚举等结构。Function Calling 或 Tool Use 更适合让模型产出工具调用参数。不同供应商支持的 Schema 子集不同，落地前要查官方文档并写兼容层。

### 8. 流式结构化返回怎么处理
不要一边收到 delta 一边直接 `JSON.parse()` 完整对象。更稳的做法是：增量阶段只展示文本或记录片段，等收到正常结束事件后拼成完整内容，再做 Schema 校验。若供应商支持结构化流式事件或 SDK accumulator，可以使用官方累积器；否则自己维护 buffer、sequence 和结束状态。

## 总结
收束一下这篇文章的几个工程判断：

+ **模型网关是稳定性入口**。路由、限流、重试、幂等、观测全在这里收口。没有网关的团队，每个业务模块各自处理 API Key 和重试逻辑，短期省事，长期一定出事故。
+ **Streaming 降低的是 TTFT，不是总成本**。它改善用户体感，但取消、超时、断流、重连和半成品 JSON 解析全是新问题。SSE 还要额外盯住事件边界、换行转义与 Nginx 缓冲——Guide 在项目里因为 `proxy_buffering` 没关，流式愣是变成了批量。
+ **重试必须和幂等绑定**。能重试的错误有限，不能让重试制造重复业务结果。用户狂点"重新发送"，后端如果没有幂等 Key 拦着，Token 账单和落库记录都会翻倍。
+ **限流不能只按 QPS**。一个 500 Token 请求和一个 80K Token 请求对供应商的压力差两个量级，必须同时看请求数、Token 数、并发和预算。
+ **结构化返回是数据契约**。JSON Schema、Structured Outputs、Tool Use 解决的是"让下游系统能稳定消费模型输出"，而不是"让输出看起来像 JSON"。
+ **没有观测就没有稳定性**。TTFT、usage、attempt、providerRequestId、parse failure rate——线上排查时少任何一个字段，都会让你多花几倍时间定位问题。

大模型 API 调用，本质上是接入一个聪明但昂贵、偶尔排队、会被限流、输出还需要校验的外部系统。把这套工程治理做到位，AI 应用才算真正从 Demo 走向生产。



