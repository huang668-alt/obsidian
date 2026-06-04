
好的，**WebSocket** 也是面试中常见的高频题，尤其是涉及**即时通讯、实时推送**的场景。

我帮你把 WebSocket 的核心知识点整理成 **6 个高频面试题**，同样附带要点和回答模板。

---

## 1. 什么是 WebSocket？为什么要用它？（必问）

**要点**：

WebSocket 是一种**全双工通信协议**，在单个 TCP 连接上实现客户端和服务端之间的双向实时通信。

**对比 HTTP**：

| 特性 | HTTP | WebSocket |
|------|------|------------|
| 通信方向 | 半双工（客户端请求，服务端响应） | 全双工（双方可随时主动发送） |
| 实时性 | 差（需轮询、长轮询） | 好（推送即达） |
| 连接方式 | 短连接（每次请求重新建立） | 长连接（建立后保持） |
| 头部开销 | 大（每次请求都带 HTTP 头） | 小（建立后只有少量控制帧） |
| 服务端推送 | 不支持（需要轮询模拟） | 原生支持 |
| 适用场景 | 普通 Web 请求 | 聊天、游戏、股票行情、协同编辑 |

**一句话总结**：
> HTTP 适合请求-响应模式，WebSocket 适合需要服务端主动推送数据的实时场景，避免了 HTTP 轮询的性能浪费。

---

## 2. WebSocket 的连接流程是怎样的？（必问）

**要点**：WebSocket 连接分为两个阶段：**握手（HTTP 升级）** → **数据传输（WebSocket 帧）**

### 第 1 步：客户端发起握手请求（HTTP）

```http
GET /chat HTTP/1.1
Host: example.com
Upgrade: websocket          // 告诉服务器：我要升级协议
Connection: Upgrade
Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==  // 随机密钥
Sec-WebSocket-Version: 13   // WebSocket 版本
```

### 第 2 步：服务端响应握手

```http
HTTP/1.1 101 Switching Protocols  // 101 表示切换协议
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: s3pPLMBiTxaQ9kYGzzhZRbK+xOo=  // 基于客户端密钥计算
```

### 第 3 步：建立连接，开始数据传输

- 握手完成后，TCP 连接保持，双方可以随时发送数据
- 数据以**帧（Frame）**的形式传输，包括文本、二进制、ping/pong 心跳等

**流程图**：
```
Client                              Server
  |--------HTTP Upgrade Request------>|
  |<-----101 Switching Protocols------|
  |--------WebSocket Frame----------->|
  |<-------WebSocket Frame-----------|
  |<-------WebSocket Frame (推送)-----|
  |--------Close Frame--------------->|
```

**面试回答模板**：
> "WebSocket 通过 HTTP 协议进行握手，客户端在请求头中带上 `Upgrade: websocket` 和 `Sec-WebSocket-Key`，服务端验证后返回 `101 Switching Protocols`，协议升级成功。之后双方就可以通过 TCP 连接自由发送 WebSocket 帧进行双向通信。"

---

## 3. Spring Boot 中如何集成 WebSocket？（高频实操题）

**要点**：Spring Boot 提供了两种方式：STOMP（高级）和原生 WebSocket。

### 方式一：原生 WebSocket（简单，适合面试讲）

**第 1 步**：引入依赖
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-websocket</artifactId>
</dependency>
```

**第 2 步**：配置 WebSocket
```java
@Configuration
@EnableWebSocket
public class WebSocketConfig implements WebSocketConfigurer {
    
    @Override
    public void registerWebSocketHandlers(WebSocketHandlerRegistry registry) {
        registry.addHandler(new MyWebSocketHandler(), "/ws")
                .setAllowedOrigins("*");  // 允许跨域
    }
}
```

**第 3 步**：实现 WebSocketHandler
```java
@Component
public class MyWebSocketHandler extends TextWebSocketHandler {
    
    // 存储所有在线会话
    private static final CopyOnWriteArraySet<WebSocketSession> sessions = new CopyOnWriteArraySet<>();
    
    // 连接建立后
    @Override
    public void afterConnectionEstablished(WebSocketSession session) {
        sessions.add(session);
        System.out.println("新连接：" + session.getId());
    }
    
    // 收到消息
    @Override
    protected void handleTextMessage(WebSocketSession session, TextMessage message) {
        // 广播给所有客户端
        for (WebSocketSession s : sessions) {
            s.sendMessage(new TextMessage("广播：" + message.getPayload()));
        }
    }
    
    // 连接关闭
    @Override
    public void afterConnectionClosed(WebSocketSession session, CloseStatus status) {
        sessions.remove(session);
    }
}
```

**第 4 步**：客户端 JS 连接
```javascript
const ws = new WebSocket("ws://localhost:8080/ws");

ws.onopen = function() {
    console.log("连接成功");
    ws.send("Hello Server");
};

ws.onmessage = function(event) {
    console.log("收到消息：" + event.data);
};
```

### 方式二：STOMP（功能更丰富，支持订阅、消息路由）

适合复杂场景，Spring 官方推荐，这里面试问到可以说"STOMP 是在 WebSocket 之上的子协议，提供了类似消息队列的语义"。

---

## 4. WebSocket 如何保持连接（心跳机制）？（高频）

**要点**：

WebSocket 有两层心跳：

### 底层：WebSocket 协议自身的 Ping/Pong 帧
- 帧类型：`0x9`（Ping）、`0xA`（Pong）
- 一方发 Ping，另一方必须回 Pong
- 如果一段时间没收到 Pong，可以认为连接已断开

### 应用层：业务心跳
如果网络中间件（如负载均衡器）会关闭空闲连接，需要在应用层发心跳消息：

```javascript
// 客户端定时发送心跳
setInterval(() => {
    ws.send(JSON.stringify({type: "ping"}));
}, 30000);  // 30秒
```

```java
// 服务端
@Override
protected void handleTextMessage(WebSocketSession session, TextMessage message) {
    String payload = message.getPayload();
    if ("ping".equals(payload)) {
        session.sendMessage(new TextMessage("pong"));
        return;
    }
    // 处理业务消息...
}
```

**面试回答模板**：
> "WebSocket 心跳有两种：协议层有 Ping/Pong 帧；应用层可以在业务消息中定义 ping/pong。一般设置 30 秒的心跳间隔，超过 60 秒没收到响应就主动重连，避免被网络中间件断开。"

---

## 5. WebSocket 如何处理断线重连？（重要）

**要点**：断线重连必须在客户端实现，服务端配合处理。

### 客户端重连策略（指数退避）

```javascript
let reconnectInterval = 1000;   // 初始 1 秒
let maxInterval = 30000;        // 最大 30 秒

function connect() {
    const ws = new WebSocket("ws://localhost:8080/ws");
    
    ws.onclose = function() {
        // 连接断开，尝试重连
        setTimeout(connect, reconnectInterval);
        // 指数退避，最大 30 秒
        reconnectInterval = Math.min(reconnectInterval * 1.5, maxInterval);
    };
    
    ws.onopen = function() {
        // 连接成功，重置重连间隔
        reconnectInterval = 1000;
        // 可能需要重新认证或订阅
        ws.send(JSON.stringify({type: "reconnect", userId: "xxx"}));
    };
}
```

### 服务端需要做的事情
- 维护用户 ID 与 Session 的映射
- 收到 `reconnect` 消息后，将新 Session 关联到原用户
- 可选：重发用户离线期间的消息（需要配合消息持久化）

---

## 6. WebSocket 和 SSE（Server-Sent Events）的区别？

**要点**：

| 特性 | WebSocket | SSE |
|------|-----------|-----|
| 方向 | 全双工（双向） | 半双工（服务端→客户端单向） |
| 协议 | ws:// / wss:// | HTTP（复用 HTTP） |
| 二进制数据 | 原生支持 | 只支持 UTF-8 文本 |
| 自动重连 | 需自己实现 | 浏览器内置 |
| 兼容性 | 优秀 | IE 不支持 |
| 适用场景 | 聊天、游戏、实时协作 | 新闻推送、股票行情、通知 |

**面试回答**：
> "WebSocket 是全双工的，客户端和服务端可以互发消息；SSE 是单通道的，只能服务端推给客户端。SSE 优势是依赖 HTTP，实现简单、自动重连；WebSocket 优势是双向通信和二进制传输。如果是实时监控、推送等单向场景，SSE 更轻量；如果是聊天或游戏，必须用 WebSocket。"

---

## 附加题：WebSocket 和 HTTP/2 的 Server Push 有什么区别？

| 维度 | WebSocket | HTTP/2 Server Push |
|------|-----------|---------------------|
| 通信模型 | 全双工，双向随时发 | 单向，服务端主动推资源（如 CSS、JS） |
| 消息格式 | 任意文本/二进制 | 只能是 HTTP 响应 |
| 推送时机 | 任意时刻 | 必须在客户端第一次请求时预判 |
| 应用场景 | 实时消息 | 预加载资源优化页面性能 |

> 两者不冲突，可以同时使用。

---

## 面试回答模板（Netty 实现 WebSocket）

> **面试官**：你们项目中 WebSocket 是怎么实现的？

> **你**：  
> "我们用的是 **Netty + WebSocket**，没有用 Spring Boot 内置的 WebSocket 方案。选择 Netty 是因为我们的推送服务需要支撑高并发（比如同时在线 10 万+），Netty 的 Reactor 线程模型和零拷贝机制更适合这种场景。
> 
> 下面我讲一下具体的实现细节。"

---

## 完整技术回答（按这个讲）

### 第 1 步：Netty 服务端启动配置

java

@Component
public class WebSocketServer {
    
    @PostConstruct
    public void start() throws InterruptedException {
        // Boss Group：负责接受新连接（1个线程）
        EventLoopGroup bossGroup = new NioEventLoopGroup(1);
        // Worker Group：负责处理 I/O 读写（CPU核心数 × 2）
        EventLoopGroup workerGroup = new NioEventLoopGroup();
        
        try {
            ServerBootstrap b = new ServerBootstrap();
            b.group(bossGroup, workerGroup)
             .channel(NioServerSocketChannel.class)
             .childHandler(new ChannelInitializer<SocketChannel>() {
                 @Override
                 protected void initChannel(SocketChannel ch) {
                     ChannelPipeline pipeline = ch.pipeline();
                     
                     // HTTP 编解码器（用于 WebSocket 握手）
                     pipeline.addLast(new HttpServerCodec());
                     // HTTP 聚合器（处理大消息）
                     pipeline.addLast(new HttpObjectAggregator(65536));
                     // WebSocket 帧聚合器（支持 WebSocket 协议）
                     pipeline.addLast(new WebSocketServerCompressionHandler());
                     // WebSocket 协议处理器：指定 WebSocket 路径，比如 /ws
                     pipeline.addLast(new WebSocketServerProtocolHandler("/ws", null, true));
                     // 业务处理器：处理 TextWebSocketFrame
                     pipeline.addLast(new WebSocketFrameHandler());
                 }
             });
            
            // 绑定端口
            ChannelFuture future = b.bind(8080).sync();
            future.channel().closeFuture().sync();
        } finally {
            bossGroup.shutdownGracefully();
            workerGroup.shutdownGracefully();
        }
    }
}

### 第 2 步：业务 Handler（处理 WebSocket 帧）

java

public class WebSocketFrameHandler extends SimpleChannelInboundHandler<TextWebSocketFrame> {
    
    // 存储用户ID与Channel的映射（用于点对点推送）
    // 分布式环境下需要改用 Redis
    private static final ConcurrentHashMap<String, Channel> userChannelMap = new ConcurrentHashMap<>();
    
    // 客户端连接建立后
    @Override
    public void handlerAdded(ChannelHandlerContext ctx) {
        System.out.println("新连接：" + ctx.channel().id().asShortText());
    }
    
    // 收到 WebSocket 消息（TextWebSocketFrame 是文本帧）
    @Override
    protected void channelRead0(ChannelHandlerContext ctx, TextWebSocketFrame msg) {
        String text = msg.text();
        System.out.println("收到消息：" + text);
        
        // 解析消息（假设是 JSON）
        JSONObject json = JSONObject.parseObject(text);
        String type = json.getString("type");
        
        if ("login".equals(type)) {
            // 用户登录，绑定 userId 和 Channel
            String userId = json.getString("userId");
            userChannelMap.put(userId, ctx.channel());
            // 回复登录成功
            ctx.channel().writeAndFlush(new TextWebSocketFrame("{\"type\":\"loginSuccess\"}"));
            
        } else if ("chat".equals(type)) {
            // 聊天消息：可能点对点或群发
            String toUserId = json.getString("toUserId");
            String content = json.getString("content");
            
            Channel targetChannel = userChannelMap.get(toUserId);
            if (targetChannel != null && targetChannel.isActive()) {
                // 点对点发送
                targetChannel.writeAndFlush(new TextWebSocketFrame(content));
            } else {
                // 用户离线，可以存消息到数据库
                saveOfflineMessage(toUserId, content);
            }
        } else if ("ping".equals(type)) {
            // 心跳响应 pong
            ctx.channel().writeAndFlush(new TextWebSocketFrame("{\"type\":\"pong\"}"));
        }
    }
    
    // 连接断开时
    @Override
    public void handlerRemoved(ChannelHandlerContext ctx) {
        // 从映射中移除
        userChannelMap.values().remove(ctx.channel());
        System.out.println("连接断开：" + ctx.channel().id().asShortText());
    }
    
    // 异常处理
    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) {
        cause.printStackTrace();
        ctx.close();
    }
}

### 第 3 步：心跳机制（Netty 自带 IdleStateHandler）

java

// 在 Pipeline 中添加心跳检测
pipeline.addLast(new IdleStateHandler(60, 0, 0, TimeUnit.SECONDS));  // 60秒读空闲
pipeline.addLast(new HeartbeatHandler());  // 自定义心跳处理器
// 心跳处理器
public class HeartbeatHandler extends ChannelInboundHandlerAdapter {
    @Override
    public void userEventTriggered(ChannelHandlerContext ctx, Object evt) {
        if (evt instanceof IdleStateEvent) {
            IdleStateEvent event = (IdleStateEvent) evt;
            if (event.state() == IdleState.READER_IDLE) {
                // 60秒没收到心跳，关闭连接
                System.out.println("客户端心跳超时，关闭连接");
                ctx.close();
            }
        }
    }
}

### 第 4 步：点对点推送（主动发消息）

java

@Service
public class PushService {
    
    // 主动给某用户推送消息
    public void pushToUser(String userId, String message) {
        Channel channel = WebSocketFrameHandler.userChannelMap.get(userId);
        if (channel != null && channel.isActive()) {
            channel.writeAndFlush(new TextWebSocketFrame(message));
        } else {
            // 用户离线，存 DB 或 MQ
            saveToOfflineQueue(userId, message);
        }
    }
    
    // 广播给所有在线用户
    public void broadcast(String message) {
        for (Channel channel : WebSocketFrameHandler.userChannelMap.values()) {
            if (channel.isActive()) {
                channel.writeAndFlush(new TextWebSocketFrame(message));
            }
        }
    }
}

### 第 5 步：分布式部署（面试加分）

如果面试官追问**多实例部署怎么推送**？

> "我们用 Redis 的 Pub/Sub 做跨实例消息广播。每个 Netty 服务实例订阅同一个 Redis 频道，收到推送消息时判断目标用户是否在自己维护的 userChannelMap 里，是的话直接推送，否则忽略。这样用户无论连到哪台机器都能收到消息。"

java

// Redis 订阅者
@Component
public class RedisPushSubscriber implements MessageListener {
    
    @Autowired
    private PushService pushService;
    
    @Override
    public void onMessage(Message message, byte[] pattern) {
        String msg = new String(message.getBody());
        JSONObject json = JSONObject.parseObject(msg);
        String userId = json.getString("userId");
        String content = json.getString("content");
        
        // 尝试在本实例推送
        pushService.pushToUser(userId, content);
    }
}

---

## 为什么用 Netty 而不是 Spring Boot 原生 WebSocket？（面试官会追问）

|对比项|Spring Boot WebSocket|Netty WebSocket|
|---|---|---|
|底层实现|依赖 Tomcat/Jetty 的 WebSocket 实现|自己用 NIO 实现，完全掌控|
|线程模型|容器线程模型（Tomcat 线程池）|Reactor 多线程模型|
|高并发能力|一般（受限于容器）|极高（Netty 本身就是为高并发设计）|
|内存控制|依赖容器|可精细化控制（池化 ByteBuf、堆外内存）|
|自定义程度|较低|完全可控（协议、编解码、线程模型）|
|学习曲线|简单|较陡|
|适用场景|普通推送、小规模应用|大型 IM、游戏服务器、10万+长连接|

**面试回答**：

> "之所以用 Netty 而不是 Spring Boot 原生 WebSocket，是因为我们预估在线用户会突破 10 万。Netty 的 Reactor 线程模型比 Tomcat 的线程模型更适合长连接场景，一个 EventLoop 可以管理成千上万个 Channel，线程数不随连接数增长。另外 Netty 的内存池化和零拷贝也能显著降低 GC 压力。"

---

## 一图总结 Netty WebSocket 架构

text

客户端 A (WebSocket)
        |
        | ws://
        ↓
[Netty Boss Group]  → 接受连接，注册到 Worker
        ↓
[Netty Worker Group] → 读数据，经过 Pipeline
        ↓
[Pipeline] → HttpServerCodec → HttpObjectAggregator → WebSocketServerProtocolHandler → WebSocketFrameHandler
        ↓
业务处理：解析 JSON → 根据消息类型处理 → 写入 TextWebSocketFrame
        ↓
响应客户端