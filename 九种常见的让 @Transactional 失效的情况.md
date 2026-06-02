在 Spring 框架的日常开发中，`@Transactional` 绝对是出镜率最高的注解之一。然而，很多人在开发中经常会遇到“代码明明抛了异常，数据库却没回滚”的翻车现场。

Spring 的声明式事务底层极度依赖 **Spring AOP（动态代理）** 和 **`ThreadLocal`**。一旦你的代码避开了这两者的控制范围，事务就会瞬间失效。

以下是开发和面试中最常见的 **9 种让 `@Transactional` 失效的场景**及对应的自救指南。

### 1. 非 public 方法修饰

- **原理分析：** Spring 在激活事务意图时，底层 AbstractFallbackTransactionAttributeSource 类的 `computeTransactionAttribute` 方法会默认检查方法的修饰符。如果发现不是 `public`（如 `private`、`protected` 或 `default`），则会**直接忽略**，不为其创建事务属性。
    
- **自救办法：** 确保贴了 `@Transactional` 的方法必须是 `public` 的。
    

### 2. 类内部自调用（Self-Invocation）

- **原理分析：** 这是最经典的翻车场景。在一个 Service 类中，`methodA()` 没有事务，`methodB()` 贴了 `@Transactional`。如果在同一个类里直接用 `this.methodB()` 或直接调用 `methodB()`，事务会直接失效。
    
    > **核心原因：** 事务的本质是通过 **Proxy（代理对象）** 去调用方法。在类内部直接调用，走的是目标对象（`this`）的原生方法，直接绕过了代理对象，Spring 根本无法介入。
    
- **自救办法：** * 办法 A：在类内部注入自己（自身循环依赖，Spring 2.6+ 默认禁用，需开启 `allow-circular-references`）。
    
    - 办法 B：使用 `AopContext.currentProxy()` 获取当前代理对象再进行调用（需开启 `@EnableAspectJAutoProxy(exposeProxy = true)`）。
        

### 3. 异常被内部 try-catch 吞掉

- **原理分析：** 代理对象之所以能触发回滚，是因为它捕捉到了目标方法**向上抛出**的异常。如果你在方法内部把异常 `try-catch` 捕获了，并且在 `catch` 块里只打印了日志或者返回了错误码，而没有重新往外抛出，代理对象就会认为该方法执行得一帆风顺，直接执行 `commit`。
    
- **自救办法：** 在 `catch` 块中，必须将异常重新抛出（如 `throw new RuntimeException(e);`），或者手动通过代码回滚：`TransactionAspectSupport.currentTransactionStatus().setRollbackOnly();`。
    

### 4. 错误的异常回滚类型（rollbackFor）

- **原理分析：** Spring 事务默认**只在遇到 `RuntimeException`（运行时异常）和 `Error` 时**才会触发回滚。如果你的方法抛出的是 **Checked Exception（受检异常/编译时异常）**，比如 `IOException`、`SQLException` 或自定义的 `Exception`，Spring 默认是**不会**帮你回滚的。
    
- **自救办法：** 显式指定回滚的异常范围，生产环境强烈建议标配：`@Transactional(rollbackFor = Exception.class)`。
    

### 5. 所在的类没有被 Spring 管理

- **原理分析：** 如果你写了一个漂亮的 Service 类，配了完美的事务，但是**忘记在类名上加 `@Service`、`@Component` 或 `@Controller`**。Spring 容器在启动时根本没有把这个类实例化成一个 Bean，自然也就无法为它生成代理对象。
    
- **自救办法：** 检查类上是否漏掉了 Spring 核心组件注解。
    

### 6. 开启了新线程调用（多线程跨越）

- **原理分析：** Spring 的事务管理器（TransactionManager）底层是通过 `ThreadLocal` 来存储和传递数据库连接（Connection）的。这意味着**事务是和当前线程严格绑定的**。如果你在 `@Transactional` 方法内部开启了新线程（如 `new Thread()`、使用线程池、或者调用了 `@Async` 异步方法），新线程是拿不到主线程的事务连接的，它们执行的 SQL 将属于完全独立的非事务连接。
    
- **自救办法：** 尽量不要在事务方法内部做多线程的写操作。如果必须做数据一致性控制，考虑在主线程外层做分布式事务编排，或利用编程式事务。
    

### 7. 错误的事务传播行为（Propagation）

- **原理分析：** `@Transactional` 的 `propagation` 属性决定了事务的传递方式。如果你一不小心把它配成了以下几种，事务就不会按预期工作：
    
    - `NOT_SUPPORTED`：以非事务方式运行，如果当前存在事务，则把当前事务挂起。
        
    - `NEVER`：以非事务方式运行，如果当前存在事务，直接抛出异常。
        
    - `SUPPORTS`：如果当前没有事务，它自己也就不开启事务（单独调用时无事务）。
        
- **自救办法：** 默认的 `REQUIRED` 通常是最佳选择。修改传播行为前务必彻底搞懂业务边界。
    

### 8. 方法被 final 或 static 修饰

- **原理分析：** Spring AOP 默认使用 **CGLIB** 来生成动态代理类（通过继承目标类并重写其方法）。
    
    - 如果方法被 `final` 修饰，子类无法重写该方法，代理就无法生效。
        
    - 如果方法被 `static` 修饰，它是属于类的静态方法，不属于对象实例，AOP 同样无法拦截。
        
- **自救办法：** 事务方法千万不要加 `final` 或 `static` 关键字。
    

### 9. 数据库引擎本身不支持事务

- **原理分析：** 这是最底层的硬伤。Spring 的事务再强大，最终也是要在执行 `connection.commit()` 或 `rollback()` 时下发给数据库。如果你的 MySQL 数据库表使用的是旧版的 **MyISAM** 存储引擎（它压根不支持事务），哪怕 Spring 框架把全套代理演得再逼真，数据依然会直接写入，无法回滚。
    
- **自救办法：** 确保你的数据库表使用的是支持事务的引擎（如 MySQL 的 **InnoDB**）。
    

### 📥 总结速查表

| **失效原因**              | **核心本质**                 | **修复方案**                           |
| --------------------- | ------------------------ | ---------------------------------- |
| **非 public 方法**       | Spring 源码层面机制过滤          | 改为 `public`                        |
| **类内部自调用**            | 绕过了 AOP 代理对象             | 注入自身或使用 `AopContext`               |
| **内部 try-catch 吞异常**  | 代理对象感知不到报错               | `catch` 后重新 `throw` 异常             |
| **触发了 Checked 异常**    | 默认只回滚 `RuntimeException` | 配置 `rollbackFor = Exception.class` |
| **类未注册为 Bean**        | 没有生成代理对象的基质              | 加上 `@Service` 或 `@Component`       |
| **跨线程异步调用**           | `ThreadLocal` 无法跨线程共享连接  | 避免在子线程中做写操作                        |
| **传播行为配置错误**          | 主动挂起或禁用了事务机制             | 核对并改回 `REQUIRED`                   |
| **方法加了 final/static** | CGLIB 代理类无法重写该方法         | 去掉 `final` 或 `static`              |
| **MyISAM 引擎**         | 物理底层硬件不支持事务              | 表引擎转换为 `InnoDB`                    |
