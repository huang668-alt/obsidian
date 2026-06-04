
好的，**Redis** 和 **分布式锁** 是面试中的绝对核心，几乎是必问内容。我帮你整理成 **高频面试题 + 分布式锁深度解析** 两个部分。

---

# 第一部分：Redis 高频面试题

## 1. Redis 支持哪些数据类型？各自适用什么场景？（必问）

| 类型 | 底层结构 | 场景 |
|------|----------|------|
| **String** | SDS（简单动态字符串） | 缓存、计数器、分布式锁、Session 共享 |
| **Hash** | dict + ziplist | 存储对象（如用户信息、购物车） |
| **List** | quicklist | 消息队列、最新消息列表（如评论时间线） |
| **Set** | dict + intset | 去重、共同好友、标签、抽奖 |
| **Sorted Set (ZSet)** | ziplist + skiplist | 排行榜、延时队列、带权重的任务调度 |
| **BitMap** | String 的位操作 | 签到统计、在线状态 |
| **HyperLogLog** | 概率数据结构 | UV 统计（允许误差） |
| **Geo** | ZSet 的封装 | 附近的人、距离计算 |
| **Stream** | 消息队列结构 | 持久化消息队列（类似 Kafka） |

---

## 2. Redis 为什么这么快？（必问）

**要点**（多维度回答）：

| 维度 | 说明 |
|------|------|
| **纯内存** | 数据存储在内存，读写速度纳秒级 |
| **单线程** | 避免上下文切换和锁竞争（6.0 后 I/O 多线程，但命令执行仍是单线程） |
| **I/O 多路复用** | 一个线程监听多个连接，基于 epoll（Linux） |
| **高效数据结构** | SDS、跳表、压缩列表等针对性优化 |
| **C 语言实现** | 更接近底层，内存操作高效 |

**面试回答模板**：
> "Redis 快的原因：纯内存操作（核心）；单线程避免了锁开销和上下文切换；I/O 多路复用模型（epoll）可处理大量并发连接；底层数据结构针对不同场景做了极致优化。"

---

## 3. Redis 的持久化机制有哪些？区别是什么？（必问）

| 机制 | 原理 | 优点 | 缺点 |
|------|------|------|------|
| **RDB** | 快照，fork 子进程生成内存快照 | 文件小、恢复快 | 可能丢失最后一次快照后的数据 |
| **AOF** | 追加写命令日志 | 数据更完整（可配每秒同步） | 文件大、恢复慢 |
| **混合持久化（4.0+）** | RDB 文件 + AOF 增量 | 兼顾恢复速度和数据完整性 | 版本兼容性问题 |

**生产推荐**：AOF + RDB 混合模式，或至少开启 AOF（`appendfsync everysec`）

---

## 4. Redis 的过期删除策略有哪些？

**三种策略结合**：

| 策略 | 说明 |
|------|------|
| **惰性删除** | 访问时检查是否过期，过期则删除 |
| **定期删除** | 每秒执行 10 次，随机抽取一批 key 检查并删除过期 key |
| **内存淘汰（maxmemory）** | 内存满时，根据淘汰策略删除 key |

**内存淘汰策略**：
- `noeviction`：不淘汰，写操作报错
- `allkeys-lru`：所有 key 中淘汰最近最少使用的
- `volatile-lru`：仅设置了过期时间的 key 中淘汰 LRU
- `allkeys-lfu` / `volatile-lfu`（4.0+）：淘汰最不经常使用的
- `allkeys-random` / `volatile-random`：随机淘汰
- `volatile-ttl`：淘汰剩余 TTL 最短的

> **推荐**：`allkeys-lru`

---

## 5. 什么是缓存穿透、缓存击穿、缓存雪崩？如何解决？（超高频）

| 问题 | 描述 | 解决方案 |
|------|------|----------|
| **穿透** | 查询不存在的数据，请求打到 DB | 布隆过滤器、缓存空对象（短期） |
| **击穿** | 热点 key 过期，大量请求瞬间打到 DB | 互斥锁（setnx）、逻辑过期、永不过期 |
| **雪崩** | 大量 key 同时过期，或 Redis 宕机 | 过期时间加随机值、高可用主从/集群 |

**代码示例（互斥锁解决击穿）**：
```java
public Object getData(String key) {
    Object value = redis.get(key);
    if (value != null) return value;
    
    // 尝试获取锁
    String lockKey = "lock:" + key;
    if (redis.setnx(lockKey, "1", 10, TimeUnit.SECONDS)) {
        try {
            value = db.query(key);
            redis.set(key, value, 3600, TimeUnit.SECONDS);
        } finally {
            redis.del(lockKey);
        }
    } else {
        // 没拿到锁，等待重试
        Thread.sleep(50);
        return getData(key);
    }
    return value;
}
```

---

# 第二部分：分布式锁（**面试重灾区**）

## 6. 分布式锁需要满足什么条件？

| 条件 | 说明 |
|------|------|
| **互斥** | 同一时刻只有一个客户端持有锁 |
| **防死锁** | 锁最终能被释放（设置过期时间） |
| **可重入** | 同一线程可重复获取锁（可选） |
| **高可用** | 锁服务本身要稳定（Redis 主从/集群） |
| **高性能** | 加锁解锁延迟低 |

---

## 7. Redis 分布式锁怎么实现？（手写核心代码）

### 基础版（有 Bug，面试先说这个再优化）

```java
// ❌ 问题：setnx 和 expire 不是原子操作，可能 setnx 成功后宕机，锁永不释放
Long result = jedis.setnx("lock:order", "thread-1");
if (result == 1) {
    jedis.expire("lock:order", 30);
    try {
        // 业务逻辑
    } finally {
        jedis.del("lock:order");
    }
}
```

### 优化版（原子加锁 + 唯一标识）

```java
// ✅ 正确的 Redis 分布式锁
public boolean tryLock(String key, String value, int expireSeconds) {
    // SET key value NX EX seconds（原子操作）
    String result = jedis.set(key, value, "NX", "EX", expireSeconds);
    return "OK".equals(result);
}

public void unlock(String key, String value) {
    // 使用 Lua 脚本保证原子性：只有值匹配才删除（防止误删别人的锁）
    String luaScript = 
        "if redis.call('get', KEYS[1]) == ARGV[1] then " +
        "    return redis.call('del', KEYS[1]) " +
        "else " +
        "    return 0 " +
        "end";
    jedis.eval(luaScript, Collections.singletonList(key), Collections.singletonList(value));
}
```

### 完整使用示例

```java
public void handleOrder(String orderId) {
    String lockKey = "lock:order:" + orderId;
    String lockValue = UUID.randomUUID().toString();  // 唯一标识
    
    try {
        // 尝试加锁，10秒过期
        if (tryLock(lockKey, lockValue, 10)) {
            // 执行业务
            processOrder(orderId);
        } else {
            // 获取锁失败，重试或返回失败
            throw new BusinessException("系统繁忙，请稍后重试");
        }
    } finally {
        unlock(lockKey, lockValue);
    }
}
```

---

## 8. Redis 分布式锁有什么问题？RedLock 是什么？

### 问题点

| 问题 | 说明 |
|------|------|
| **锁过期误删** | 业务执行超过锁过期时间，锁被自动释放，其他线程拿到锁，当前线程完成后会误删别人的锁（已用唯一标识 + Lua 解决） |
| **锁续期** | 业务没执行完锁就过期了，需要看门狗机制 |
| **主从切换锁丢失** | 主节点加锁成功但未同步到从节点，主宕机后从晋升为主，锁丢失（Redis 分布式锁最大痛点） |
| **时钟漂移** | 不同节点时间不一致导致锁判断问题 |

### RedLock 算法（Redis 作者提出）

**核心思想**：不依赖单点，向多个 Redis 主节点（N 个，通常 N 为奇数）请求锁，超过半数成功才算加锁成功。

```java
// Redisson 已经实现了 RedLock
RLock lock1 = redisson1.getLock("myLock");
RLock lock2 = redisson2.getLock("myLock");
RLock lock3 = redisson3.getLock("myLock");

RedissonRedLock redLock = new RedissonRedLock(lock1, lock2, lock3);
redLock.lock();
try {
    // 业务
} finally {
    redLock.unlock();
}
```

**面试评价**：
> "RedLock 理论上解决了主从切换丢锁问题，但实现复杂，且存在争议（Martin Kleppmann 曾发文指出其不完美）。实际生产中，如果对锁的强一致性要求极高（如金融场景），建议用 ZooKeeper 或 etcd；允许极小概率丢锁的场景，单节点 Redis + 唯一标识 + 续期就够用了。"

---

## 9. Redisson 实现分布式锁（生产推荐）

**Redisson 自动解决了**：
- 原子加锁 + 自动续期（看门狗）
- 可重入
- 自动解锁

```java
@Autowired
private RedissonClient redissonClient;

public void handleOrder(String orderId) {
    RLock lock = redissonClient.getLock("lock:order:" + orderId);
    
    try {
        // 尝试加锁，最多等待 100 秒，锁 30 秒后自动释放（有看门狗会自动续期）
        if (lock.tryLock(100, 30, TimeUnit.SECONDS)) {
            processOrder(orderId);
        }
    } catch (InterruptedException e) {
        Thread.currentThread().interrupt();
    } finally {
        if (lock.isHeldByCurrentThread()) {
            lock.unlock();
        }
    }
}
```

---

## 10. Redis 分布式锁 vs ZooKeeper 分布式锁

| 维度 | Redis | ZooKeeper |
|------|-------|-----------|
| 一致性 | 弱（主从异步复制可能丢锁） | 强（ZAB 协议，CP 模型） |
| 高可用 | AP 模型 | CP 模型 |
| 性能 | 极高 | 一般（需半数确认） |
| 实现复杂度 | 简单 | 中等 |
| 死锁防止 | 过期时间 | Session 临时节点 |
| 适用场景 | 允许锁丢失的高并发场景（如秒杀防重） | 对一致性要求高的场景（如 leader 选举、配置管理） |

**面试回答**：
> "大部分业务场景下，Redis 分布式锁够用了。如果对强一致性要求极高（比如分布式调度、选主），我会选择 ZooKeeper；如果追求极致性能且允许极低概率的锁失效，选 Redis。"

---

## 面试高频追问汇总

### Q：锁过期了业务没执行完怎么办？

> 用 Redisson 的看门狗机制，每 10 秒自动续期一次。手写实现的话，可以起一个定时任务在锁过期前（比如过期时间 30 秒，在 20 秒时）检查并续期。

### Q：主从切换丢锁怎么解决？

> 用 RedLock 或改用 ZooKeeper。实际生产如果锁涉及的金额敏感，我会选择后者；如果是防重复提交等场景，丢锁概率极低，可接受。

### Q：可重入锁怎么实现？

> 在 value 里存当前线程标识 + 重入次数（如 JSON 格式），每次加锁时判断标识是否相同，相同则次数 +1；解锁时次数 -1，减到 0 才真正删除。

```java
// value 示例
{
    "threadId": "main-thread-1",
    "count": 3
}
```

---

## 一张图总结分布式锁加锁流程

```
客户端 A                                 Redis
    |                                       |
    |------- SET lock key value NX EX 30 --->|
    |<-------------- OK --------------------|
    |                                       |
    | (业务执行中，看门狗每10秒续期)           |
    |------- EXPIRE lock 30 --------------->|
    |<-------------- 1 --------------------|
    |                                       |
    | (业务完成)                             |
    |------- Lua脚本（值匹配才删锁）--------->|
    |<-------------- 1 --------------------|
```

---

## 面试准备优先级

| 优先级 | 内容 |
|--------|------|
| ⭐⭐⭐ | 分布式锁的正确实现（SET NX EX + Lua 解锁） |
| ⭐⭐⭐ | 缓存穿透/击穿/雪崩及解决方案 |
| ⭐⭐⭐ | Redis 数据类型及应用场景 |
| ⭐⭐ | 持久化机制、过期策略 |
| ⭐⭐ | Redis 为什么快、Redisson 看门狗 |
| ⭐ | RedLock、与 ZooKeeper 对比 |

明天面试如果问到分布式锁，先把 **SET NX EX + 唯一标识 + Lua 解锁** 这套讲清楚，再提一下主从切换可能丢锁的问题，面试官会觉得你思考得很全面。

需要我把某个点再展开详细讲吗？