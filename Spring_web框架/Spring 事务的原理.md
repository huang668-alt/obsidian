
在 Spring 中，声明式事务 `@Transactional` 的底层原理，可以用一句话来概括：**AOP（动态代理）负责织入生命周期，ThreadLocal 负责绑定连接，而 TransactionManager 负责执行具体的提交与回滚。**

它绝非什么魔法，而是由一系列严密的组件协同完成的。我们可以将其拆解为：**核心三驾马车**、**连接绑定机制** 以及 **完整的拦截工作流**。

## 一、 核心的“三驾马车”（三大核心接口）

Spring 并没有自己实现事务，它只是做了一层优雅的封装。整个事务抽象体系由三个核心接口支撑：

1. **`PlatformTransactionManager`（事务管理器）：**
    
    - **角色：** 绝对的执行者。
        
    - **职责：** 负责定义 `getTransaction()`（获取事务）、`commit()`（提交）和 `rollback()`（回滚）的行为。它有多个实现类，如果你用 MyBatis/JDBC，Spring 会为你注入 `DataSourceTransactionManager`；如果你用 Hibernate，则是 `JpaTransactionManager`。
        
2. **`TransactionDefinition`（事务定义信息）：**
    
    - **角色：** 事务的属性蓝图。
        
    - **职责：** 里面保存了你在 `@Transactional` 注解里配置的各种参数，包括：**隔离级别（Isolation）**、**传播行为（Propagation）**、**超时时间（Timeout）** 以及 **只读属性（ReadOnly）**。
        
3. **`TransactionStatus`（事务运行状态）：**
    
    - **角色：** 事务的实时记分板。
        
    - **职责：** 记录了当前事务在运行过程中的状态，比如：是不是一个新事务、事务是否已经结束、是否被标记为了 `setRollbackOnly()`（只回滚）。
        

## 二、 核心暗器：`ThreadLocal` 与连接同步

在一段事务代码里，你可能会调用 `orderDao.insert()` 和 `stockDao.update()`。这两个 DAO 操作必须使用**同一个数据库连接（Connection）**，否则它们各自提交，事务就失去了原子性。

Spring 是如何保证它们拿到同一个连接的？答案是 **`TransactionSynchronizationManager`**。

- 当事务开启时，事务管理器会从数据库连接池（如 Druid、HikariCP）里借一个 `Connection` 对象。
    
- 接着，Spring 会把这个 `Connection` 丢进 `TransactionSynchronizationManager` 内部的 **`ThreadLocal`** 中。
    
- **效果：** 只要接下来的 DAO 操作处于**同一个线程**内，MyBatis 或 JdbcTemplate 在执行 SQL 时，都会先去这个 `ThreadLocal` 里看一眼：“有没有老大约束好的连接？”如果有，直接拿来用。这就巧妙地保证了整个事务链条中连接的唯一性。
    

## 三、 完整的工作流程：代理对象的“两阶段拦截”

当你调用一个贴了 `@Transactional` 的 Service 方法时，Spring AOP 派出的核心拦截器 **`TransactionInterceptor`** 就会全面接管这个调用。整个执行过程如下：

```
 用户请求 ──> [ 代理对象 Proxy ]
                   │
                   ├──> 1. 开启事务 (获取连接，关闭 autoCommit，绑定 ThreadLocal)
                   ├──> 2. 执行业务 (调用你的原生 Service 方法)
                   │         ├──> 成功 ──> 3. 提交事务 (commit，清理 ThreadLocal)
                   │         └──> 失败 ──> 3. 回滚事务 (rollback，清理 ThreadLocal)
                   V
 返回结果
```

### 1. 事务前置准备

- **生成代理：** Spring 启动时，发现你的类有 `@Transactional`，就会用 JDK 动态代理或 CGLIB 为你生成一个代理对象。
    
- **开启拦截：** 你调用方法时，实际调用的是代理对象。`TransactionInterceptor` 优先拦截。
    
- **关闭自动提交：** 拦截器通过调用底层连接的 `connection.setAutoCommit(false)`，把数据库默认的“单条 SQL 自动提交”转为“手动批处理模式”。
    
- **绑定线程：** 将该连接绑定到当前线程的 `ThreadLocal`。
    

### 2. 执行目标业务

- 代理对象通过 `MethodInvocation.proceed()` 调用你真正写在 Service 里的业务代码。此时各路 DAO 开始愉快地共享同一个连接，疯狂执行 SQL。
    

### 3. 后置决议与收尾

- **场景 A（顺利通过）：** 业务代码执行完毕，没有抛出异常。代理对象调用 `transactionManager.commit()`，通知数据库真正落盘。
    
- **场景 B（悲剧发生）：** 业务代码抛出了异常（默认是 `RuntimeException` 或 `Error`）。代理对象捕捉到后，立刻调用 `transactionManager.rollback()`，让数据库将刚才的一系列操作全部撤销。
    
- **擦屁股（Finally）：** 无论成功还是失败，Spring 都会执行清扫工作：
    
    1. 将该连接从 `ThreadLocal` 中彻底移除（防止内存泄漏或连接污染）。
        
    2. 恢复连接的自动提交属性 `connection.setAutoCommit(true)`。
        
    3. 把连接归还给连接池，供其他线程使用。
        

## 四、 核心总结

Spring 事务的本质，就是**通过 AOP 在你的核心业务前后，自动帮你加上了 `try-catch`、`connection.commit()` 和 `rollback()` 的模板代码**。

而它最精妙的设计，就在于**用 `ThreadLocal` 解决了多层 Service 和 DAO 之间“跨方法传递数据库连接”的痛点**，让我们可以完全把精力专注于写具体的 SQL，而不用手动传来传去 Connection 对象。