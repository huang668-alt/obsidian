
**Spring 事务传播机制（Propagation）极详细讲解**

Spring 事务传播机制是 Spring 事务管理中**最核心、最重要**的概念之一。它定义了**当一个事务方法被另一个事务方法调用时，事务应该如何传播**（是加入当前事务，还是开启新事务等）。

---

### 1. 事务传播机制是什么？

- **传播行为（Propagation Behavior）**：用于控制当前方法在**有事务**和**无事务**的情况下，如何处理事务。
- Spring 通过 `@Transactional(propagation = Propagation.XXX)` 来设置。
- **默认传播机制**：`Propagation.REQUIRED`

---

### 2. Spring 支持的 7 种事务传播机制

以下按重要程度排序详细讲解：

| 传播机制              | 说明                                                                 | 有事务时          | 无事务时          | 典型使用场景                     |
|-----------------------|----------------------------------------------------------------------|-------------------|-------------------|----------------------------------|
| **REQUIRED**（默认）  | **最常用**。如果当前存在事务，则加入该事务；如果没有，则新建一个事务 | 加入当前事务      | 新建事务          | 大多数业务方法                   |
| **REQUIRES_NEW**      | **总是新建一个独立事务**。如果当前存在事务，则挂起当前事务           | 挂起当前事务，新建 | 新建事务          | 日志记录、审计、需要独立提交的场景 |
| **SUPPORTS**          | 支持当前事务。如果当前存在事务，则加入；如果没有，则以非事务方式执行 | 加入当前事务      | 非事务方式执行    | 查询方法（可有可无事务）         |
| **NOT_SUPPORTED**     | **不支持事务**。如果当前存在事务，则挂起当前事务，以非事务方式执行   | 挂起当前事务      | 非事务方式执行    | 耗时查询、第三方调用             |
| **MANDATORY**         | **必须有事务**。如果当前没有事务，则抛出异常                         | 加入当前事务      | 抛出异常          | 必须在事务中执行的核心操作       |
| **NEVER**             | **必须没有事务**。如果当前存在事务，则抛出异常                       | 抛出异常          | 非事务方式执行    | 绝对不能在事务中执行的方法       |
| **NESTED**            | **嵌套事务**。如果当前存在事务，则在嵌套事务中执行；没有则新建       | 在嵌套事务中执行  | 新建事务          | 部分成功部分回滚的复杂业务       |

---

### 3. 每个传播机制详细说明 + 示例

#### 1. `Propagation.REQUIRED`（最常用）

```java
@Transactional(propagation = Propagation.REQUIRED)
public void methodA() {
    // 业务逻辑
    methodB();   // 调用 B
}

@Transactional(propagation = Propagation.REQUIRED)
public void methodB() { ... }
```

**行为**：
- 如果 methodA 有事务，methodB 加入 A 的事务（共用同一个事务）
- 如果 methodA 没有事务，则 methodB 新建事务

**适用**：绝大多数 Service 方法。

---

#### 2. `Propagation.REQUIRES_NEW`（独立事务）

```java
@Transactional
public void placeOrder() {
    // 下单主逻辑
    saveOrder();
    recordLog();   // 日志必须记录，即使下单失败
}

@Transactional(propagation = Propagation.REQUIRES_NEW)
public void recordLog() {
    // 记录操作日志
}
```

**关键**：`recordLog()` 的成功与否**不影响** `placeOrder()` 的事务。即使 `recordLog()` 抛异常，主事务也可以独立提交或回滚。

---

#### 3. `Propagation.SUPPORTS`

适合**只读查询**方法：

```java
@Transactional(propagation = Propagation.SUPPORTS, readOnly = true)
public User getUserById(Long id) { ... }
```

---

#### 4. `Propagation.NOT_SUPPORTED`

```java
@Transactional(propagation = Propagation.NOT_SUPPORTED)
public List<User> queryLargeData() { 
    // 大数据查询，不希望占用事务连接
}
```

---

#### 5. `Propagation.MANDATORY`

```java
@Transactional(propagation = Propagation.MANDATORY)
public void updateUserScore() {
    // 必须在事务中执行
}
```

如果调用方没有事务，会抛 `IllegalTransactionStateException`。

---

#### 6. `Propagation.NEVER`

```java
@Transactional(propagation = Propagation.NEVER)
public void sendEmail() {
    // 发送邮件，不能在事务中（避免长时间占用事务）
}
```

---

#### 7. `Propagation.NESTED`（嵌套事务）

**核心特点**：
- 使用 JDBC 的 **Savepoint** 机制实现。
- 内层事务回滚不会影响外层事务。
- 外层事务回滚会带动内层一起回滚。

```java
@Transactional
public void transferMoney() {
    // 主事务
    deductMoney();
    
    try {
        recordDetail();   // 嵌套事务
    } catch (Exception e) {
        // 明细记录失败不影响转账主逻辑
    }
}

@Transactional(propagation = Propagation.NESTED)
public void recordDetail() { ... }
```

**注意**：并不是所有数据库都支持 `NESTED`（MySQL InnoDB 支持，Oracle 支持）。

---

### 4. 事务传播机制总结表（面试必背）

| 传播级别          | 外层有事务          | 外层无事务       | 是否创建新事务 | 是否独立提交 |
|-------------------|---------------------|------------------|----------------|--------------|
| REQUIRED          | 加入                | 创建             | 可能           | 否           |
| REQUIRES_NEW      | 挂起 + 创建新事务   | 创建             | 是             | 是           |
| SUPPORTS          | 加入                | 无事务           | 否             | -            |
| NOT_SUPPORTED     | 挂起                | 无事务           | 否             | -            |
| MANDATORY         | 加入                | 异常             | 否             | -            |
| NEVER             | 异常                | 无事务           | 否             | -            |
| NESTED            | 嵌套（Savepoint）   | 创建             | 可能           | 部分         |

---

### 5. 最佳实践与注意事项

1. **默认使用 `REQUIRED`**，能满足大部分需求。
2. **日志、审计、安全记录** 建议使用 `REQUIRES_NEW`。
3. **长耗时查询** 使用 `NOT_SUPPORTED` 或 `SUPPORTS` + `readOnly=true`。
4. **嵌套事务** 使用 `NESTED` 要谨慎，优先考虑业务拆分。
5. **事务传播 + 异常处理**：
   - Spring 默认只对 **RuntimeException** 和 **Error** 回滚。
   - 想对 checked exception 回滚，需要设置 `rollbackFor = Exception.class`。
6. **避免在同一类中方法调用**（`this.methodB()`），会绕过代理，导致传播机制失效。解决办法：
   - 通过接口调用
   - 注入自己（`@Autowired` 本类）
   - 使用 `AopContext.currentProxy()`

---

**你想继续深入哪个部分？**

- 事务传播机制 + 异常回滚的组合场景详解
- `NESTED` 与 `REQUIRES_NEW` 的底层区别
- 事务传播源码分析（`TransactionAspectSupport`）
- 实际项目中的事务传播案例

随时告诉我，我可以继续展开得更细！