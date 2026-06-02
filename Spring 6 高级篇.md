## 7.单元测试：JUnit
笔记小结：

> 1. 概述：JUnit 是一个 Java 编程语言的**单元测试框架**
> 2. 基本用例：
>
> 步骤一：引入依赖：Junit支持依赖、Junit5测试
>
> 步骤二：配置文件：配置基本的类扫描
>
> 步骤三：演示：使用**@SpringJUnitConfig(locations = “classpath:beans.xml”)**注解，会自动的加载配置文件并且创建类
>

### 7.1概述
JUnit 是一个 Java 编程语言的单元测试框架，它由 Erich Gamma 和 Kent Beck 创建，逐渐成为 Java 语言的标准单元测试框架之一。JUnit 提供了一些注解和断言方法，可以帮助开发者编写和运行测试用例。它还可以与构建工具（如 Maven 和 Gradle）集成，便于持续集成和自动化测试

### 7.2基本用例
#### 7.2.1 JUnit5
步骤一：引入依赖

```xml
<!-- Spring 测试框架，提供Spring应用的测试支持，包括依赖注入和事务管理等测试功能 -->
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-test</artifactId>
    <version>6.0.2</version>
</dependency>

<!-- JUnit Jupiter API，JUnit 5的核心API，提供现代Java单元测试的基础注解和功能 -->
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter-api</artifactId>
    <version>5.9.0</version>
</dependency>
```

步骤二：配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xmlns:context="http://www.springframework.org/schema/context"
  xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
  http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">
  <context:component-scan base-package="com.atguigu.spring6.bean"/>
</beans>
```

说明：

> 配置类扫描，用于自动装配Bean
>

步骤三：创建类

```java
import org.springframework.stereotype.Component;
@Component
public class User {
    /**
     * 无参构造方法
     * 当Spring容器初始化该Bean时，会调用此构造方法
     * 构造方法执行时会打印"run user"信息，用于观察Bean的创建时机
     */
    public User() {
        System.out.println("run user");
    }
}
```

步骤四：演示

```java
// Spring JUnit 5测试类，指定配置文件为classpath下的beans.xml
@SpringJUnitConfig(locations = "classpath:beans.xml")
public class SpringJUnit5Test {

    // 自动注入User类型的Bean
    @Autowired
    private User user;

    // 测试方法，用于验证User对象
    @Test
    public void testUser(){
        System.out.println(user);
    }
}
```

说明：

> + @SpringJUnitConfig注解是用于指定Spring的配置文件。在测试方法运行前，Spring会先根据该配置文件创建出需要的Bean，然后注入到测试类中。这样就可以在测试类中直接使用Spring的Bean进行测试了
> + 类是于：
> + 本段代码是通过`ApplicationContext`接口的类路径xml上下文`ClassPathXmlApplicationContext`进行加载
>

```java
ApplicationContext context = new ClassPathXmlApplicationContext("xxx.xml");
Xxxx xxx = context.getBean(Xxxx.class);
```

#### 7.2.2 JUnit4
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1754442677407-c1c1d63d-098a-48f1-b3d9-bdbfbade509d.png)

## 8.事务
笔记小结：

> 1. 概述： 
>     1. 定义：事务就是包含一个或多个数据操作，**要么全部成功要么全部失败**
>     2. 特性： 
>         * **原子性**：要么全部成功，要么全部失败
>         * **一致性**：事务执行失败后，数据库的数据是不改变的
>         * **隔离性**：多个事务之间，相互隔离
>         * **持久性**：事务完成后，数据库内的数据应该正常持久化存储
>     3. **编程**式事务：使用**代码的方式**进行事务的处理
>     4. **声明**式事务：使用**注解的方式**进行事务的处理。注解**@Transactional**
> 2. 基于注解的声明式事务：请**详细查看此小节**
> 3. 基于XML的声明式事务（了解）：通过**XML** 的方式来实现切面的定义和应用
>

### 8.1概述
#### 8.1.1定义
事务（Transaction）是指一个或多个操作序列组成的逻辑工作单元，这些操作要么全部成功，要么全部失败回滚。在数据库中，一个事务包含一个或多个数据操作，如增加、删除、修改等，同时，这些操作要么全部成功，要么全部失败，不能只有部分操作成功或失败

#### 8.1.2特性
1. 原子性（Atomicity）：事务是一个不可分割的工作单元，要么全部成功，要么全部失败，不允许出现部分成功部分失败的情况。
2. 一致性（Consistency）：事务执行前后，数据库的状态应该保持一致，如果一个事务执行失败，那么数据库应该恢复到执行前的状态。
3. 隔离性（Isolation）：多个事务之间应该互相隔离，事务之间不能互相干扰，避免脏读、不可重复读、幻读等问题。
4. 持久性（Durability）：事务完成后，对数据库的修改应该持久化保存，即使系统故障或崩溃，数据也不应该丢失。

#### 8.2.3编程式事务
编程式事务是通过在代码中编写事务管理代码来实现的，需要手动控制事务的开始、提交和回滚，需要在每个需要事务管理的方法中编写相关代码，这样代码**耦合度**高，且事务管理代码重复出现，不便于维护

```java
// 获取数据库连接（具体获取方式省略）
Connection conn = ...;

try {
    // 关闭自动提交，开启事务管理
    conn.setAutoCommit(false);
    
    // 此处执行数据库操作（如CRUD）
    // ...
    
    // 所有操作成功后提交事务
    conn.commit();
    
} catch(Exception e) {
    // 发生异常时回滚事务，撤销所有操作
    conn.rollback();
    
} finally {
    // 无论事务成功或失败，最终关闭数据库连接释放资源
    conn.close();
}
```

#### 8.2.4声明式事务
声明式事务则是通过AOP的方式实现，将事务管理代码从业务逻辑中分离出来，通过在配置文件中声明事务管理，实现事务的自动管理，开发人员只需要在需要事务管理的方法上添加注解或者配置即可，大大简化了代码的编写和维护工作

```java
/**
 * 用户服务接口，定义用户相关操作的规范
 */
public interface UserService {
    /**
     * 更新用户信息
     * @param user 待更新的用户对象
     */
    void updateUser(User user);
}

/**
 * UserService接口的实现类，提供用户相关服务的具体实现
 * @Service注解标记此类为Spring管理的服务层组件
 * @Transactional注解表示此类中的方法将在事务管理下执行
 */
@Service
@Transactional
public class UserServiceImpl implements UserService {
    
    /**
     * 自动注入UserDao对象，用于数据库操作
     */
    @Autowired
    private UserDao userDao;

    /**
     * 实现更新用户信息的方法
     * 调用数据访问层的update方法完成实际的数据库更新操作
     * 由于类上标注了@Transactional，此方法执行将在事务中进行
     * @param user 待更新的用户对象
     */
    @Override
    public void updateUser(User user) {
        userDao.update(user);
    }
}
```

说明：

> + 在这个例子中，`@Transactional` 注解被用于服务实现类的类级别上。这意味着，当调用 `updateUser` 方法时，Spring 将会创建一个事务，方法执行结束时，事务将被提交或回滚（如果出现异常）
> + 这样，在调用 `updateUser` 方法时，我们无需显式地开启和提交事务，Spring 框架将会自动处理事务的提交和回滚
>

### 8.2基于注解的声明式事务
笔记小节：

> 1. 基本用例：
>
> 步骤一：添加**tx命名空间**、添加**事务管理器**、**开启**事务的注解驱动
>
> 步骤二：**添加**事务**注解**：@Transactional
>
> 1. 事务属性：
>     1. **只读**：@Transactional(**readOnly** = true)，告诉数据库，此操作只能读不能改写
>     2. **超时**：@Transactional(**timeout** = 3)，当此操作超时时，做出提示
>     3. **回滚策略**： 
>         * rollbackFor属性：当事务方法**抛出指定类型**的异常时，事务会**回滚**
>         * rollbackForClassName属性：与rollbackFor属性相似，但是**指定异常类型**时**使用字符**串的方式
>         * noRollbackFor属性：指定当事务方法抛出指定类型的异常时，事务**不回滚**
>         * noRollbackForClassName属性：与noRollbackFor属性相似，但是**指定异常类型**时**使用字符串**的方式
>     4. **隔离级别**： 
>         * `READ_UNCOMMITTED`：表示一个事务可以读取另一个未提交事务的数据。此级别会出现脏读、不可重复读、幻读的问题，一般不建议使用。
>         * `READ_COMMITTED`：表示一个事务只能读取另一个已经提交的事务的数据，可以避免脏读问题，但不**可重复读和幻读问题**仍可能发生。
>         * `REPEATABLE_READ`：表示一个事务在执行期间可以多次读取同一行数据，可以避免脏读和不可重复读问题，但**幻读问题仍可能发生**。
>         * `SERIALIZABLE`：表示一个事务在执行期间对所涉及的所有数据加锁，避免了脏读、不可重复读、幻读问题，但**并发性能非常差**。
>     5. **传播行为**：事务的传播行为是指在多个事务方法相互调用时，控制事务如何传播和影响彼此的行为的规则
>     6. **全注解配置事务**：@EnableTransactionManagement **开启注解式事务管理**
>

#### 8.2.1基本用例-实现注解式的声明事务
步骤一：添加配置文件中的tx命名空间以及配置

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!-- Spring配置文件，定义Bean和事务管理相关配置 -->
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xmlns:context="http://www.springframework.org/schema/context"  <!-- 上下文命名空间，用于组件扫描等 -->
  xmlns:tx="http://www.springframework.org/schema/tx"  <!-- 事务命名空间，用于事务管理配置 -->
  xsi:schemaLocation="http://www.springframework.org/schema/beans
  http://www.springframework.org/schema/beans/spring-beans.xsd  <!-- Spring Bean的Schema约束 -->
  http://www.springframework.org/schema/context
  http://www.springframework.org/schema/context/spring-context.xsd  <!-- 上下文的Schema约束 -->
  http://www.springframework.org/schema/tx
  http://www.springframework.org/schema/tx/spring-tx.xsd">  <!-- 事务的Schema约束 -->

  <!-- 配置事务管理器，使用Spring提供的DataSourceTransactionManager -->
  <!-- 用于管理基于数据源的事务，依赖于数据源(druidDataSource) -->
  <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    <property name="dataSource" ref="druidDataSource"/>  <!-- 注入数据源，ref引用已定义的数据源Bean -->
  </bean>

  <!-- 启用注解驱动的事务管理 -->
  <!-- 指定使用名为transactionManager的事务管理器，使得@Transactional注解生效 -->
  <tx:annotation-driven transaction-manager="transactionManager" />
</beans>
```

说明：

> 1. 添加tx命名空间
> 2. 添加事务管理器
> 3. 开启事务的注解驱动
>

步骤二：添加事务注解

```java
/**
 * 购买图书的业务方法，包含事务管理
 * 使用@Transactional注解标记，表明此方法在事务中执行
 * 确保库存扣减和余额扣减操作要么同时成功，要么同时失败
 * 
 * @param bookId 图书ID
 * @param userId 用户ID
 */
@Transactional
public void buyBook(Integer bookId, Integer userId) {
    // 1. 根据图书ID查询图书价格
    Integer price = bookDao.getPriceByBookId(bookId);

    // 2. 减少图书库存
    bookDao.updateStock(bookId);

    // 3. 扣减用户余额
    bookDao.updateBalance(userId, price);

    // 若所有操作成功，事务自动提交；若发生异常，事务自动回滚
}
```

说明：

> + Service层表示业务逻辑层，注解通常添加在业务层的功能上，可达到事务管理的目的
> + @Transactional注解标识在方法上，只会影响此方法被事务管理利器进行管理。@Transactional注解标识在类上，则会影响该类中的所有方法被事务管理利器进行管理
>

步骤三：演示

说明：

> 若在操作事务中，出现错误，则Spring框架会帮助我们自动回滚数据
>

#### 8.2.2事务属性：只读
对一个查询操作来说，如果我们把它设置成只读，就能够明确告诉数据库，这个操作不涉及写操作。这样数据库就能够针对查询操作来进行优化

```java
/**
 * 购买图书的业务方法
 * 
 * 注意：此处@Transactional注解设置了readOnly = true，这是不合适的
 * readOnly=true通常用于纯查询操作，会告诉数据库这是一个只读事务
 * 而本方法包含更新操作（修改库存和余额），可能导致事务异常或更新失败
 */
@Transactional(readOnly = true)
public void buyBook(Integer bookId, Integer userId) {
    // 1. 根据图书ID查询图书价格
    Integer price = bookDao.getPriceByBookId(bookId);

    // 2. 减少图书库存（更新操作）
    bookDao.updateStock(bookId);

    // 3. 扣减用户余额（更新操作）
    bookDao.updateBalance(userId, price);

    // 警告：由于设置了readOnly=true，这些更新操作可能不会生效
    // 建议将readOnly属性移除或设置为false
}
```

说明：

> 1. 此时，已为事务添加了事务只读的属性，因此进行增删改操作会抛出异常：
>

```xml
Caused by: java.sql.SQLException: Connection is read-only. Queries leading to data modification are not allowed
```

#### 8.2.3事务属性：超时
事务在执行过程中，有可能因为遇到某些问题，导致程序卡住，从而长时间占用数据库资源。而长时间占用资源，大概率是因为程序运行出现了问题（可能是Java程序或MySQL数据库或网络连接等等）。此时这个很可能出问题的程序应该被回滚，撤销它已做的操作，事务结束，把资源让出来，让其他正常程序可以执行

```java
/**
 * 购买图书的业务方法，设置了事务超时时间
 * 
 * @Transactional(timeout = 3) 表示事务最长执行时间为3秒
 * 如果方法执行时间超过3秒，事务将自动回滚并抛出TransactionTimedOutException
 */
@Transactional(timeout = 3)
public void buyBook(Integer bookId, Integer userId) {
    try {
        // 模拟耗时操作，休眠5秒
        // 此操作会导致事务执行时间超过设置的3秒超时时间
        TimeUnit.SECONDS.sleep(5);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }

    // 查询图书价格
    Integer price = bookDao.getPriceByBookId(bookId);

    // 减少图书库存
    bookDao.updateStock(bookId);

    // 扣减用户余额
    bookDao.updateBalance(userId, price);

    // 实际执行结果：由于休眠5秒超过了3秒的超时设置
    // 事务会在超时后自动回滚，后续的数据库操作不会生效
}
```

说明：

> 1. 此时，已为事务添加了事务超时的属性，因此进行查改操作超出单位时间则会抛出异常：
>

```plain
org.springframework.transaction.TransactionTimedOutException: Transaction timed out: deadline was Fri Jun 04 16:25:39 CST 2022
```

#### 8.2.4事务属性：回滚策略
事务属性中的回滚策略指的是当事务出现异常时，应该如何处理事务的提交或回滚

属性分类：

1. rollbackFor属性：当事务方法抛出指定类型的异常时，事务会回滚

```java
@Transactional(rollbackFor = {SQLException.class, IOException.class})
```

2. rollbackForClassName属性：与rollbackFor属性相似，但是指定异常类型时使用字符串的方式

```java
@Transactional(rollbackForClassName = {"java.sql.SQLException", "java.io.IOException"})
```

3. noRollbackFor属性：指定当事务方法抛出指定类型的异常时，事务不回滚。

```java
@Transactional(noRollbackFor = {NullPointerException.class, IllegalArgumentException.class})
```

4. noRollbackForClassName属性：与noRollbackFor属性相似，但是指定异常类型时使用字符串的方式

```java
@Transactional(noRollbackForClassName = {"java.lang.NullPointerException", "java.lang.IllegalArgumentException"})
```

基本用例：

```java
/**
 * 购买图书的业务方法，设置了特定异常不回滚事务
 * 
 * @Transactional(noRollbackFor = ArithmeticException.class) 表示
 * 当方法中抛出ArithmeticException（算术异常）时，事务不会回滚
 * 其他类型的异常仍会触发事务回滚
 */
@Transactional(noRollbackFor = ArithmeticException.class)
public void buyBook(Integer bookId, Integer userId) {
    // 查询图书价格
    Integer price = bookDao.getPriceByBookId(bookId);

    // 减少图书库存
    bookDao.updateStock(bookId);

    // 扣减用户余额
    bookDao.updateBalance(userId, price);

    // 此处会抛出ArithmeticException（除零异常）
    // 由于注解中设置了noRollbackFor = ArithmeticException.class
    // 所以即使发生此异常，前面的数据库更新操作也不会回滚
    System.out.println(1/0);
}
```

说明：

> 此时，@Transactional注解配置的属性为`noRollbackFor`，因此当出现`ArithmeticException.class`此类异常后，事务不会进行回滚
>

#### 8.2.5事务属性：隔离级别
事务隔离级别是指在多个事务并发执行时，为了保证事务之间的数据一致性，数据库采用的一种隔离机制

| 隔离级别 | 脏读 | 不可重复读 | 幻读 |
| --- | --- | --- | --- |
| READ UNCOMMITTED | 有 | 有 | 有 |
| READ COMMITTED | 无 | 有 | 有 |
| REPEATABLE READ | 无 | 无 | 有 |
| SERIALIZABLE | 无 | 无 | 无 |


说明：

> + READ_UNCOMMITTED：表示一个事务可以读取另一个未提交事务的数据。此级别会出现脏读、不可重复读、幻读的问题，一般不建议使用。
> + READ_COMMITTED：表示一个事务只能读取另一个已经提交的事务的数据，可以避免脏读问题，但不可重复读和幻读问题仍可能发生。
> + REPEATABLE_READ：表示一个事务在执行期间可以多次读取同一行数据，可以避免脏读和不可重复读问题，但幻读问题仍可能发生。
> + SERIALIZABLE：表示一个事务在执行期间对所涉及的所有数据加锁，避免了脏读、不可重复读、幻读问题，但并发性能非常差。
>

基本用法

```java
// 使用默认的事务隔离级别
// 具体隔离级别由底层数据库决定（大多数数据库默认是READ_COMMITTED）
@Transactional(isolation = Isolation.DEFAULT)

// 读未提交：允许事务读取其他未提交事务的数据
// 可能导致脏读、不可重复读和幻读问题，隔离级别最低
@Transactional(isolation = Isolation.READ_UNCOMMITTED)

// 读已提交：只能读取其他事务已提交的数据
// 可避免脏读，但仍可能出现不可重复读和幻读
@Transactional(isolation = Isolation.READ_COMMITTED)

// 可重复读：确保同一事务中多次读取同一数据结果一致
// 可避免脏读和不可重复读，但可能出现幻读
@Transactional(isolation = Isolation.REPEATABLE_READ)

// 串行化：最高隔离级别，通过强制事务串行执行
// 可避免脏读、不可重复读和幻读，但会严重影响性能
@Transactional(isolation = Isolation.SERIALIZABLE)
```

#### 8.2.6事务属性：传播行为
事务的传播行为是指在多个事务方法相互调用时，控制事务如何传播和影响彼此的行为的规则。在Spring框架中，事务的传播行为由`Propagation`枚举类定义，常用的传播行为包括如下：

1. **REQUIRED（默认）：如果当前存在一个事务，则加入该事务；如果当前没有事务，则创建一个新的事务。换句话说，没有就新建，有就加入**
2. SUPPORTS：如果当前存在一个事务，则加入该事务；如果当前没有事务，则以非事务的方式继续运行。换句话说，有就加入，没有就不管了
3. MANDATORY：如果当前存在一个事务，则加入该事务；如果当前没有事务，则抛出异常。换句话说，有就加入，没有就抛异常
4. **REQUIRES_NEW：创建一个新的事务，如果当前存在事务，则将当前事务挂起。换句话说，不管有没有，直接开启一个新事务，开启的新事务和之前的事务不存在嵌套关系，之前事务被挂起**
5. NOT_SUPPORTED：以非事务方式运行，如果当前存在事务，则将当前事务挂起。换句话说，不支持事务，存在就挂起
6. NEVER：以非事务方式运行，如果当前存在事务，则抛出异常。换句话说，不支持事务，存在就抛异常
7. NESTED：如果当前存在一个事务，则在嵌套事务内执行；如果当前没有事务，则按照 PROPAGATION_REQUIRED 执行。换句话说，有事务的话，就在这个事务里再嵌套一个完全独立的事务，嵌套的事务可以独立的提交和回滚。没有事务就和REQUIRED一样

```java
/**
 * 使用REQUIRED传播行为的事务方法
 * 
 * @Transactional(propagation = Propagation.REQUIRED) 表示：
 * 1. 如果当前存在事务，则加入该事务
 * 2. 如果当前没有事务，则创建一个新的事务
 * 
 * 这是Spring事务传播行为的默认值，也是最常用的传播行为
 * 适用于大多数业务场景，确保方法在事务环境中执行
 */
@Transactional(propagation = Propagation.REQUIRED)
public void transactionalMethod() {
    // 方法业务逻辑
    // 所有操作将在同一个事务中执行，要么全部成功提交，要么全部失败回滚
}
```

#### 8.2.7全注解配置事务（重点）
步骤一：添加配置类

```java
package com.atguigu.spring6.config;

import com.alibaba.druid.pool.DruidDataSource;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.jdbc.datasource.DataSourceTransactionManager;
import org.springframework.transaction.annotation.EnableTransactionManagement;

import javax.sql.DataSource;

/**
 * Spring的核心配置类，替代传统的XML配置文件
 */
// 标记当前类为配置类，相当于传统的applicationContext.xml
@Configuration 
// 指定Spring要扫描的包，会自动注册包下带有@Component等注解的类为Bean
@ComponentScan("com.atguigu.spring6") 
// 开启事务管理功能，使@Transactional注解生效
@EnableTransactionManagement 
public class SpringConfig {

    /**
     * 配置数据源Bean
     * @return 数据源对象
     */
    // 将方法返回的对象注册为Spring容器中的Bean，默认Bean名称为方法名
    @Bean 
    public DataSource getDataSource(){
        // 使用Druid连接池，Druid是阿里巴巴的开源数据库连接池，性能优秀
        DruidDataSource dataSource = new DruidDataSource(); 
        // 设置数据库驱动类名，MySQL 8.x使用com.mysql.cj.jdbc.Driver
        dataSource.setDriverClassName("com.mysql.cj.jdbc.Driver"); 
        // 设置数据库连接URL，包含数据库地址、端口、数据库名和连接参数
        dataSource.setUrl("jdbc:mysql://localhost:3306/spring?characterEncoding=utf8&useSSL=false"); 
        // 设置数据库用户名
        dataSource.setUsername("root"); 
        // 设置数据库密码
        dataSource.setPassword("123456"); 
        return dataSource; 
    }

    /**
     * 配置JdbcTemplate Bean，用于简化JDBC操作
     * @param dataSource 数据源对象，Spring会自动从容器中注入
     * @return JdbcTemplate对象
     */
    // 显式指定Bean的名称为jdbcTemplate
    @Bean(name = "jdbcTemplate") 
    public JdbcTemplate getJdbcTemplate(DataSource dataSource){
        JdbcTemplate jdbcTemplate = new JdbcTemplate(); 
        // 为JdbcTemplate设置数据源
        jdbcTemplate.setDataSource(dataSource); 
        return jdbcTemplate; 
    }

    /**
     * 配置事务管理器，用于管理事务
     * @param dataSource 数据源对象，Spring会自动从容器中注入
     * @return 事务管理器对象
     */
    @Bean 
    public DataSourceTransactionManager getDataSourceTransactionManager(DataSource dataSource){
        // 数据源事务管理器，需要与数据源关联
        DataSourceTransactionManager dataSourceTransactionManager = new DataSourceTransactionManager(); 
        // 设置事务管理器使用的数据源
        dataSourceTransactionManager.setDataSource(dataSource); 
        return dataSourceTransactionManager; 
    }
}
```

说明：

> 当使用全注解配置事务时，需要声明一个事务管理器，并使用注解@EnableTransactionManagement开启事务管理器
>

步骤二：演示

```java
@Test
public void testTxAllAnnotation(){
    ApplicationContext applicationContext = new AnnotationConfigApplicationContext(SpringConfig.class);
    BookController accountService = applicationContext.getBean("bookController", BookController.class);
    accountService.buyBook(1, 1);
}
```

#### 8.2.8底层原理
在 Java 中，使用 `@Transactional` 注解实现方法级别的事务管理是基于 Spring 框架的特性和 AOP（面向切面编程）的原理。

底层原理如下：

1. Spring 通过 AOP 功能拦截带有 `@Transactional` 注解的方法的执行。
2. 当方法被调用时，Spring 在运行时会为该方法创建一个代理对象。
3. 代理对象在方法执行前后会插入事务相关的逻辑。
4. 在方法开始时，事务管理器会开启一个新的数据库事务。
5. 如果方法执行成功（无异常抛出），事务管理器会提交事务，将对数据库的修改持久化到数据库。
6. 如果方法执行出现异常，事务管理器会回滚事务，撤销对数据库的修改，恢复到事务开始之前的状态。
7. 方法执行完成后，事务管理器会关闭事务。

通过 AOP 的方式，Spring 能够在方法执行前后对事务进行控制。它在方法执行前开启事务，在方法执行后提交或回滚事务，从而保证了数据的一致性和完整性。

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1754442572352-c66cf9c4-35d7-40fb-a637-6f16b9fbdfec.png)

需要注意的是，`@Transactional` 注解的生效还依赖于事务管理器的配置和使用环境的支持。Spring 框架提供了多种事务管理器的实现，如基于 JDBC 的事务管理器和基于 JTA 的事务管理器，可以根据具体的需求选择合适的事务管理器。同时，Spring 需要在配置文件中启用事务管理器，并确保被 `@Transactional` 注解修饰的方法是通过 Spring 容器获取的代理对象来调用的。这样才能使事务注解生效并正确地应用事务管理的功能。

#### 8.2.9<font style="color:rgb(15, 17, 21);">九种常见的让 </font>`@Transactional`<font style="color:rgb(15, 17, 21);"> 失效的情况</font>
##### <font style="color:rgb(15, 17, 21);">1. 注解应用于非 Public 方法</font>
**<font style="color:rgb(15, 17, 21);">原因</font>**<font style="color:rgb(15, 17, 21);">： Spring 默认使用基于代理的 AOP 来实现事务管理。在生成代理对象时，Spring 的</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">DefaultAopProxyFactory</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">会判断方法是否为 public。如果不是 public，则不会为该方法创建事务代理，导致注解失效。</font>

**<font style="color:rgb(15, 17, 21);">解决方案</font>**<font style="color:rgb(15, 17, 21);">：</font>

+ <font style="color:rgb(15, 17, 21);">将需要使用事务的方法声明为</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">public</font>`<font style="color:rgb(15, 17, 21);">。</font>
+ <font style="color:rgb(15, 17, 21);">如果必须用于非 public 方法，可以配置 AspectJ 模式（</font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">mode = AdviceMode.ASPECTJ</font>`<font style="color:rgb(15, 17, 21);">），但这会引入更复杂的配置和编译时/加载时织入，一般不推荐。</font>

```java
// 失效 ❌
@Transactional
private void createUser(User user) {
userDao.save(user);
// ... 其他操作
}

// 生效 ✅
@Transactional
public void createUser(User user) {
userDao.save(user);
// ... 其他操作
}
```

##### <font style="color:rgb(15, 17, 21);">2. 自调用（Self-Invocation）</font>
**<font style="color:rgb(15, 17, 21);">原因</font>**<font style="color:rgb(15, 17, 21);">： 这是最常见的原因之一。当一个类中的方法 A（无事务）调用同一个类中的方法 B（有</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">@Transactional</font>`<font style="color:rgb(15, 17, 21);">）时，事务不会生效。因为代理对象的方法 A 被调用时，</font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">this</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">指向的是目标对象本身，而不是代理对象，所以方法 B 的调用绕过了代理，自然也就没有了事务。</font>

**<font style="color:rgb(15, 17, 21);">解决方案</font>**<font style="color:rgb(15, 17, 21);">：</font>

+ <font style="color:rgb(15, 17, 21);">将方法 A 和方法 B 拆分到不同的 Service 类中。</font>
+ <font style="color:rgb(15, 17, 21);">通过 ApplicationContext 获取当前代理对象，然后调用其方法（不推荐，代码侵入性强）。</font>
+ <font style="color:rgb(15, 17, 21);">使用 </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">@EnableAspectJAutoProxy(exposeProxy = true)</font>`<font style="color:rgb(15, 17, 21);"> 并调用 </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">AopContext.currentProxy()</font>`<font style="color:rgb(15, 17, 21);">。</font>

```java
@Service
public class UserService {

    public void createUserAndLog() {
        // 这是自调用，insertUser上的@Transactional会失效 ❌
        this.insertUser(); // this 是真实对象，不是代理
    }

    @Transactional
    public void insertUser() {
        userDao.save(user);
    }
}

// 解决方案：拆分类 ✅
@Service
public class UserService {
    @Autowired
    private UserTransactionService userTransactionService;

    public void createUserAndLog() {
        userTransactionService.insertUser(); // 调用另一个Bean的方法，事务生效
    }
}

@Service
public class UserTransactionService {
    @Transactional
    public void insertUser() {
        // ...
    }
}
```

##### <font style="color:rgb(15, 17, 21);">3. 异常类型不正确或被捕获</font>
**<font style="color:rgb(15, 17, 21);">原因</font>**<font style="color:rgb(15, 17, 21);">：</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">@Transactional</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">默认只在抛出</font>**<font style="color:rgb(15, 17, 21);">未检查的异常</font>**<font style="color:rgb(15, 17, 21);">（即</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">RuntimeException</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">及其子类）和</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">Error</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">时回滚。如果抛出了</font>**<font style="color:rgb(15, 17, 21);">已检查异常</font>**<font style="color:rgb(15, 17, 21);">（如</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">Exception</font>`<font style="color:rgb(15, 17, 21);">），事务不会回滚。更常见的是，在方法内部使用</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">try-catch</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">捕获了异常，但没有重新抛出，导致事务管理器认为方法执行成功，从而提交事务。</font>

**<font style="color:rgb(15, 17, 21);">解决方案</font>**<font style="color:rgb(15, 17, 21);">：</font>

+ <font style="color:rgb(15, 17, 21);">根据需要，使用</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">@Transactional(rollbackFor = Exception.class)</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">指定在遇到特定异常时回滚。</font>
+ <font style="color:rgb(15, 17, 21);">确保异常被抛出到事务切面，不要在方法内部“吞掉”异常。</font>

```java
// 失效 ❌ (默认只回滚RuntimeException)
@Transactional
public void createUser(User user) throws Exception {
    userDao.save(user);
    throw new Exception("发生检查异常"); // 事务提交！
}

// 失效 ❌ (异常被吞没)
@Transactional
public void createUser(User user) {
try {
    userDao.save(user);
    int i = 1 / 0; // 会抛出RuntimeException
} catch (Exception e) {
    e.printStackTrace();
    // 没有重新抛出，事务提交！
}
}

// 生效 ✅
@Transactional(rollbackFor = Exception.class) // 指定检查异常也回滚
public void createUser(User user) throws Exception {
    userDao.save(user);
    throw new Exception("发生检查异常"); // 事务回滚
}

// 生效 ✅
@Transactional
public void createUser(User user) {
try {
    userDao.save(user);
    int i = 1 / 0;
} catch (Exception e) {
    e.printStackTrace();
    throw new RuntimeException(e); // 重新抛出运行时异常
}
}
```

##### <font style="color:rgb(15, 17, 21);">4. 数据库引擎不支持事务</font>
**<font style="color:rgb(15, 17, 21);">原因</font>**<font style="color:rgb(15, 17, 21);">： 如果你使用的数据库引擎（如 MySQL 的</font><font style="color:rgb(15, 17, 21);"> </font>**<font style="color:rgb(15, 17, 21);">MyISAM</font>**<font style="color:rgb(15, 17, 21);">）本身不支持事务，那么 Spring 无论如何配置事务管理都是无效的。</font>

**<font style="color:rgb(15, 17, 21);">解决方案</font>**<font style="color:rgb(15, 17, 21);">：</font>

+ <font style="color:rgb(15, 17, 21);">将数据库表引擎切换为支持事务的引擎，如 MySQL 的 </font>**<font style="color:rgb(15, 17, 21);">InnoDB</font>**<font style="color:rgb(15, 17, 21);">。</font>

##### <font style="color:rgb(15, 17, 21);">5. propagation 传播行为设置不当</font>
**<font style="color:rgb(15, 17, 21);">原因</font>**<font style="color:rgb(15, 17, 21);">： 传播行为定义了事务的边界。例如，</font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">Propagation.NOT_SUPPORTED</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">表示以非事务方式运行，如果当前存在事务则挂起它。</font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">Propagation.NEVER</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">表示不能存在事务，否则抛出异常。如果错误地设置了这些属性，方法就不会在事务中运行。</font>

**<font style="color:rgb(15, 17, 21);">解决方案</font>**<font style="color:rgb(15, 17, 21);">：</font>

+ <font style="color:rgb(15, 17, 21);">根据业务逻辑正确设置</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">propagation</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">属性。默认是</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">Propagation.REQUIRED</font>`<font style="color:rgb(15, 17, 21);">（如果当前没有事务，就新建一个；如果已存在，则加入）。</font>

<font style="color:rgb(15, 17, 21);">java</font>

```xml
// 此方法不会在事务中运行 ❌
@Transactional(propagation = Propagation.NOT_SUPPORTED)
public void createUser(User user) {
   userDao.save(user); // 无事务
}
```

##### <font style="color:rgb(15, 17, 21);">6. 未被 Spring 容器管理</font>
**<font style="color:rgb(15, 17, 21);">原</font>****<font style="color:rgb(15, 17, 21);">因</font>**<font style="color:rgb(15, 17, 21);">： 如果类的对象不是由 Spring IoC 容器创建和管理的（即没有使用 </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">@Component</font>`<font style="color:rgb(15, 17, 21);">, </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">@Service</font>`<font style="color:rgb(15, 17, 21);"> 等注解），那么它上面的 </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">@Transactional</font>`<font style="color:rgb(15, 17, 21);"> 注解不会被扫描和处理。</font>

**<font style="color:rgb(15, 17, 21);">解决方案</font>**<font style="color:rgb(15, 17, 21);">：</font>

+ <font style="color:rgb(15, 17, 21);">确保你的类是一个 Spring Bean。</font>

```java
// 失效 ❌ - 这是一个普通对象，不是Spring Bean
// @Service 注解被注释掉了
//@Service
public class UserService {
    @Transactional
    public void createUser(User user) {
        // ...
    }
}
```

##### <font style="color:rgb(15, 17, 21);">7. 在多线程环境下使用</font>
**<font style="color:rgb(15, 17, 21);">原因</font>**<font style="color:rgb(15, 17, 21);">： Spring 的事务信息是通过</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">ThreadLocal</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">存储的。这意味着事务上下文是线程绑定的。如果你在一个新线程中执行数据库操作，它将无法获取到原始线程的事务上下文，导致操作不在同一个事务中。</font>

**<font style="color:rgb(15, 17, 21);">解决方案</font>**<font style="color:rgb(15, 17, 21);">：</font>

+ <font style="color:rgb(15, 17, 21);">避免在事务方法中开启新线程进行数据库操作。如果必须这样做，需要考虑复杂的分布式事务解决方案。</font>

##### <font style="color:rgb(15, 17, 21);">8. 错误配置了事务管理器</font>
**<font style="color:rgb(15, 17, 21);">原因</font>**<font style="color:rgb(15, 17, 21);">： 如果你在代码中配置了多个数据源和事务管理器，但没有指定默认的，或者在使用</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">@Transactional</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">时没有通过</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">transactionManager</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">属性指定正确的管理器，也会导致失效。</font>

**<font style="color:rgb(15, 17, 21);">解决方案</font>**<font style="color:rgb(15, 17, 21);">：</font>

+ <font style="color:rgb(15, 17, 21);">确保数据源和事务管理器配置正确。</font>
+ <font style="color:rgb(15, 17, 21);">如果有多个事务管理器，使用 </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">@Transactional(value="txManagerName")</font>`<font style="color:rgb(15, 17, 21);"> 明确指定。</font>

```java
@Configuration
@EnableTransactionManagement
public class PersistenceConfig {

    @Bean
    public PlatformTransactionManager transactionManager(EntityManagerFactory emf) {
        // 必须配置一个事务管理器
        return new JpaTransactionManager(emf);
    }
}
```

##### <font style="color:rgb(15, 17, 21);">9. 在 Controller 层直接使用</font>
**<font style="color:rgb(15, 17, 21);">原因</font>**<font style="color:rgb(15, 17, 21);">： 虽然技术上可以将</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">@Transactional</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">放在 Controller 的方法上，但这是一种</font>**<font style="color:rgb(15, 17, 21);">非常糟糕的实践</font>**<font style="color:rgb(15, 17, 21);">。它会导致事务边界过大，延长数据库连接的持有时间，严重影响性能。并且，Controller 层的职责应该是处理 HTTP 请求和响应，业务逻辑（和与之对应的事务）应该放在 Service 层。</font>

**<font style="color:rgb(15, 17, 21);">解决方案</font>**<font style="color:rgb(15, 17, 21);">：</font>

+ **<font style="color:rgb(15, 17, 21);">严格遵守分层架构</font>**<font style="color:rgb(15, 17, 21);">：将事务注解始终用在 Service 层的方法上。</font>

```java
// 不推荐 ❌ (尽管可能有效，但属于错误用法)
@RestController
public class UserController {

    @Autowired
    private UserDao userDao;

    @PostMapping("/user")
    @Transactional // 事务放在Controller层
    public String createUser(@RequestBody User user) {
        userDao.save(user);
        return "success";
    }
}

// 推荐 ✅
@RestController
public class UserController {

    @Autowired
    private UserService userService; // 调用Service层

    @PostMapping("/user")
    public String createUser(@RequestBody User user) {
        userService.createUser(user); // Service方法上有@Transactional
        return "success";
    }
}

@Service
public class UserService {
    @Transactional // 事务在Service层
    public void createUser(User user) {
        // ...
    }
}
```

### 8.3基于XML声明式事务（了解）
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="
           http://www.springframework.org/schema/beans
           http://www.springframework.org/schema/beans/spring-beans.xsd
           http://www.springframework.org/schema/tx
           http://www.springframework.org/schema/tx/spring-tx.xsd
           http://www.springframework.org/schema/aop
           http://www.springframework.org/schema/aop/spring-aop.xsd
           http://www.springframework.org/schema/context
           http://www.springframework.org/schema/context/spring-context.xsd">

    <!-- 1. 配置数据源 -->
    <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
        <property name="driverClassName" value="com.mysql.cj.jdbc.Driver"/>
        <property name="url" value="jdbc:mysql://localhost:3306/spring?characterEncoding=utf8&useSSL=false"/>
        <property name="username" value="root"/>
        <property name="password" value="root"/>
    </bean>

    <!-- 2. 配置事务管理器 -->
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource"/>
    </bean>

    <!-- 3. 配置事务通知 -->
    <tx:advice id="txAdvice" transaction-manager="transactionManager">
        <tx:attributes>
            <tx:method name="get*" read-only="true"/>
            <tx:method name="query*" read-only="true"/>
            <tx:method name="find*" read-only="true"/>
            <tx:method name="save*" read-only="false" rollback-for="java.lang.Exception" propagation="REQUIRES_NEW"/>
            <tx:method name="update*" read-only="false" rollback-for="java.lang.Exception" propagation="REQUIRES_NEW"/>
            <tx:method name="delete*" read-only="false" rollback-for="java.lang.Exception" propagation="REQUIRES_NEW"/>
        </tx:attributes>
    </tx:advice>

    <!-- 4. 配置AOP切面 -->
    <aop:config>
        <aop:pointcut id="txPointcut" expression="execution(* com.atguigu.spring.tx.xml.service.impl.*.*(..))"/>
        <aop:advisor advice-ref="txAdvice" pointcut-ref="txPointcut"/>
    </aop:config>

    <!-- 5. 扫描Service组件 -->
    <context:component-scan base-package="com.atguigu.spring.tx.xml.service"/>

</beans>
```

注意：基于xml实现的**声明式事务**，必须引入aspectJ的依赖

```xml
<dependency>
  <groupId>org.springframework</groupId>
  <artifactId>spring-aspects</artifactId>
  <version>6.0.2</version>
</dependency>
```

## 9.资源操作：Resources
笔记小结：

> 1. 概述：在Spring框架中，`org.springframework.core.io.Resource`接口及其实现类是用于**处理资源**（如文件、类路径资源、URL等）的抽象层
> 2. Resources接口：Resource 接口是 Spring 资源访问策略的抽象，它本身并不提供任何资源访问实现，具体的**资源访问由该接口的实现类完成**。Resource 接口用于抽象**对低级资源的****<u>访问</u>**
> 3. Resources实现类： 
>     - `UrlResource`：此类用于表示**URL类型**的资源，可用于访问**网络资源**
>     - `ClassPathResource `：**ClassPath**Resource 用于表示**类路径**下的资源，可以通过指定资源的相对路径或绝对路径进行实例化
>     - `FileSystemResource `： **FileSystem**Resource 类用于访问**文件系统**资源
>     - `ServletContextResource`：略
>     - `InputStreamResource`：略
>     - `ByteArrayResource`：略
> 4. ResourceLoader接口：它定义了用于**加载**资源的统一访问方式。
> 5. ResourceLoader实现类： 
>     - `DefaultResourceLoader`: 默认的资源加载器，可用于**加载类路径、文件系统和URL资源**。
>     - `FileSystemResourceLoader`: 用于加载**文件系统**中的资源。
>     - `ClassPathResourceLoader`: 用于加载**类路径**下的资源。
>     - `ServletContextResourceLoader`: 用于加载**Web应用程序上下文**中的资源。
> 6. ResourceLoaderAware接口：用于将 ResourceLoader 实例**注入**到实现该接口的 Bean 中。
> 7. 动态获取Resource资源：直接利用**依赖注入**的方式，最大程度地简化了 Spring 资源访问
> 8. 确定资源访问路径： 
>     - 实现类指定访问路径 
>         * **ClassPathXML**ApplicationContext : 对应使用ClassPathResource进行资源访问。
>         * **FileSystemXml**ApplicationContext ： 对应使用FileSystemResource进行资源访问。
>         * **XmlWeb**ApplicationContext ： 对应使用ServletContextResource进行资源访问。
>     - 前缀指定访问路径: 
>         1. classpath前缀
>         2. classpath通配符
>         3. 通配符其他使用
>

### 9.1概述
在Spring框架中，`org.springframework.core.io.Resource`接口及其实现类是用于处理资源（如文件、类路径资源、URL等）的抽象层。它提供了一种统一的方式来访问和操作不同类型的资源，无论这些资源是在文件系统中、类路径下还是通过URL进行访问

### 9.2Resources接口
#### 9.2.1概述
Spring 的 Resource 接口位于 `org.springframework.core.io` 中。 旨在成为一个更强大的接口，用于抽象对低级资源的访问。

`Resource`接口在Spring中确实继承了`InputStreamSource`接口，并提供了更多的方法。而`InputStreamSource`接口只有一个方法

#### 9.2.2常用方法
+ `boolean exists()`：检查资源是否存在。
+ `boolean isReadable()`：检查资源是否可读。
+ `boolean isOpen()`：检查资源是否打开。
+ `URL getURL()`：获取资源的URL。
+ `URI getURI()`：获取资源的URI。
+ `File getFile()`：获取资源对应的文件。
+ `long contentLength()`：获取资源的长度。
+ `long lastModified()`：获取资源的最后修改时间。
+ `Resource createRelative(String relativePath)`：创建相对于当前资源的相对资源。
+ `String getFilename()`：获取资源的文件名。
+ `String getDescription()`：获取资源的描述信息。
+ `InputStream getInputStream()`：获取资源的输入流。

### 9.3Resources的实现类
#### 9.3.1概述
Resource 接口是 Spring 资源访问策略的抽象，它本身并不提供任何资源访问实现，具体的资源访问由该接口的实现类完成。每个实现类代表一种资源访问策略。Resource一般包括这些实现类：`UrlResource、ClassPathResource、FileSystemResource、ServletContextResource、InputStreamResource、ByteArrayResource`

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1754442572430-e0481ecc-c8a9-41a4-8ab9-923400d06d61.png)

#### 9.3.2UrlResource类
此类用于表示URL类型的资源，可以通过`URL`对象或`String`类型的URL路径进行实例化，用来访问网络资源。

基本用例：

```java
public class UrlResourceDemo {

    public static void loadAndReadUrlResource(String path){
        UrlResource url = null;
        try {
            url = new UrlResource(path);
            System.out.println(url.getFilename());
            System.out.println(url.getURI());
            System.out.println(url.getDescription());
            System.out.println(url.getInputStream().read());
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }
    public static void main(String[] args) {
        loadAndReadUrlResource("file:atguigu.txt");
    }
}
```

说明：

> + 当访问文件系统资源时，文件需要基于当前项目的根路径下来进行读取
>
> <!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1754442572434-0c9646a5-0908-421a-92d1-f9217308a455.png)
>

补充：

> + http:------该前缀用于访问基于HTTP协议的网络资源
> + ftp:------该前缀用于访问基于FTP协议的网络资源
> + file: ------该前缀用于从文件系统中读取资源
>

#### 9.3.3ClassPathResource 类
ClassPathResource 用于表示类路径下的资源，可以通过指定资源的相对路径或绝对路径进行实例化。相对于其他的 Resource 实现类，其主要优势是方便访问类加载路径里的资源，尤其对于 Web 应。，ClassPathResource 可自动搜索位于 classes 下的资源文件，无须使用绝对路径访问。

基本用例：

```java
public class ClassPathResourceDemo {

    public static void loadAndReadUrlResource(String path) throws Exception{
        ClassPathResource resource = new ClassPathResource(path);
        System.out.println("resource.getFileName = " + resource.getFilename());
        System.out.println("resource.getDescription = "+ resource.getDescription());
        InputStream in = resource.getInputStream();
        byte[] b = new byte[1024];
        while(in.read(b)!=-1) {
            System.out.println(new String(b));
        }
    }
    public static void main(String[] args) throws Exception {
        loadAndReadUrlResource("atguigu.txt");
    }
}
```

说明：

> + 当访问文件资源系统时，文件需要基于当前项目的类路径下来进行读取
>
> <!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1754442572351-a6d3d253-9a92-4220-9ad8-b8445cf48a7e.png)
>

#### 9.3.4FileSystemResource 类
Spring 提供的 FileSystemResource 类用于访问文件系统资源，使用 FileSystemResource 来访问文件系统资源并没有太大的优势，因为 Java 提供的 File 类也可用于访问文件系统资源。

```java
public class FileSystemResourceDemo {

    public static void loadAndReadUrlResource(String path) throws Exception{
        FileSystemResource resource = new FileSystemResource("atguigu.txt");
        System.out.println("resource.getFileName = " + resource.getFilename());
        System.out.println("resource.getDescription = "+ resource.getDescription());
        InputStream in = resource.getInputStream();
        byte[] b = new byte[1024];
        while(in.read(b)!=-1) {
            System.out.println(new String(b));
        }
    }

    public static void main(String[] args) throws Exception {
        loadAndReadUrlResource("atguigu.txt");
    }
}
```

#### 9.3.5ServletContextResource类
这是`ServletContext`资源的Resource实现，它解释相关Web应用程序根目录中的相对路径。它始终支持流(stream)访问和URL访问，但只有在扩展Web应用程序存档且资源实际位于文件系统上时才允许java.io.File访问。无论它是在文件系统上扩展还是直接从JAR或其他地方（如数据库）访问，实际上都依赖于Servlet容器。

#### 9.3.6InputStreamResource类
`InputStreamResource `是给定的输入流的Resource实现。它的使用场景在没有特定的资源实现的时候使用(感觉和@Component 的适用场景很相似)。与其他Resource实现相比，这是已打开资源的描述符。 因此，它的`isOpen()`方法返回true。如果需要将资源描述符保留在某处或者需要多次读取流，请不要使用它。

#### 9.6.7ByteArrayResource类
字节数组的Resource实现类。通过给定的数组创建了一个`ByteArrayInputStream`。它对于从任何给定的字节数组加载内容非常有用，而无需求助于单次使用的`InputStreamResource`。

### 9.4ResourceLoader接口
#### 9.4.1概述
`ResourceLoader`接口是Spring框架中的一个核心接口，它定义了用于**加载**资源的统一访问方式。它提供了一种统一的方法，可以加载各种类型的资源，例如文件、类路径资源、URL资源等。

#### 9.4.2常用方法
+ `Resource getResource(String location)`: 根据给定的资源位置（location）获取一个`Resource`对象。资源位置可以是文件路径、类路径、URL等。具体的资源加载策略由`ResourceLoader`的具体实现类决定。
+ `ClassLoader getClassLoader()`: 获取用于加载类的`ClassLoader`对象。这对于加载类路径下的资源非常有用。

#### 9.4.3实现类
Spring框架提供了多个实现`ResourceLoader`接口的类，包括：

+ `DefaultResourceLoader`: 默认的资源加载器，可用于加载类路径、文件系统和URL资源。
+ `FileSystemResourceLoader`: 用于加载文件系统中的资源。
+ `ClassPathResourceLoader`: 用于加载类路径下的资源。
+ `ServletContextResourceLoader`: 用于加载Web应用程序上下文中的资源。

#### 9.4.4基本用例
```java
public static void main(String[] args) {
    ApplicationContext ctx = new ClassPathXmlApplicationContext();
    Resource res = ctx.getResource("atguigu.txt");
    System.out.println(res.getFilename());
}
```

说明：

> Spring将采用和ApplicationContext相同的策略来访问资源。也就是说，如果ApplicationContext是ClassPathXmlApplicationContext，res就是ClassPathResource实例
>

```java
public static void main(String[] args) {
    ApplicationContext ctx = new FileSystemXmlApplicationContext();
    Resource res = ctx.getResource("atguigu.txt");
    System.out.println(res.getFilename());
}
```

说明：

> Spring将采用和ApplicationContext相同的策略来访问资源。也就是说，果ApplicationContext是FileSystemXmlApplicationContext，res就是FileSystemResource实例；
>

```java
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
import org.springframework.core.io.Resource;

public class ResourceLoaderExample {

    public static void main(String[] args) {
        ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
        Resource resource1 = context.getResource("classpath:config.properties");
        System.out.println("Resource 1: " + resource1.getClass().getSimpleName());
        Resource resource2 = context.getResource("file:/path/to/file.txt");
        System.out.println("Resource 2: " + resource2.getClass().getSimpleName());
        Resource resource3 = context.getResource("http://www.example.com");
        System.out.println("Resource 3: " + resource3.getClass().getSimpleName());
    }
}
```

说明：

> 使用 ApplicationContext 的 getResource() 方法获取了三个不同类型的资源，使用不同的前缀指定了使用不同的 Resource 实现类
>

### 9.5ResourceLoaderAware接口
#### 9.5.1概述
##### 9.5.1.1含义
ResourceLoaderAware 是一个 Spring Bean 接口，用于将 ResourceLoader 实例注入到实现该接口的 Bean 中。它定义了一个 setResourceLoader(ResourceLoader resourceLoader) 方法，Spring 容器在启动时将 ResourceLoader 实例作为参数传递给该方法。

##### 9.5.1.2作用
实现ResourceLoaderAware接口的类可以获取Spring容器的ResourceLoader实例，从而利用Spring容器提供的资源加载功能

#### 9.5.2基本用例
步骤一：创建Bean

```java
package com.atguigu.spring6.resouceloader;

import org.springframework.context.ResourceLoaderAware;
import org.springframework.core.io.ResourceLoader;

public class TestBean implements ResourceLoaderAware {
    private ResourceLoader resourceLoader;
    public void setResourceLoader(ResourceLoader resourceLoader) {
        this.resourceLoader = resourceLoader;
    }
    public ResourceLoader getResourceLoader(){
        return this.resourceLoader;
    }

}
```

说明：

> + 实现ResourceLoaderAware接口必须实现`setResourceLoader`方法，
>

步骤二:配置Bean

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

  <bean id="testBean" class="com.atguigu.spring6.resouceloader.TestBean"></bean>
</beans>
```

步骤三：演示

```java
public static void main(String[] args) {

    ApplicationContext ctx = new ClassPathXmlApplicationContext("bean.xml");
    TestBean testBean = ctx.getBean("testBean",TestBean.class);

    ResourceLoader resourceLoader = testBean.getResourceLoader();
    System.out.println("Spring容器将自身注入到ResourceLoaderAware Bean 中 ？ ：" + (resourceLoader == ctx));

    Resource resource = resourceLoader.getResource("atguigu.txt");
    System.out.println(resource.getFilename());
    System.out.println(resource.getDescription());
}
```

说明：

> 因为`ApplicationContext`接口的实现类`ClassPathXmlApplicationContext`实现了`ResourceLoader`接口，因此`ClassPathXmlApplicationContext`实例对象，也具备"ResourceLoader"的功能。因此，实现了`ResourceLoaderAware`此接口的对象，就可以利用Spring容器提供的资源加载功能
>

### 9.6动态获取Resource资源
#### 9.6.1概述
##### 9.6.1.1含义
Spring 框架不仅充分利用了策略模式来简化资源访问，而且还将策略模式和 IoC 进行充分地结合，最大程度地简化了 Spring 资源访问。当应用程序中的 Bean 实例需要访问资源时，Spring 有更好的解决方法：直接利用依赖注入

##### 9.6.1.2作用
对于通过在代码中获取 Resource 实例，当程序获取 Resource 实例时，总需要提供 Resource 所在的位置，不管通过 FileSystemResource 创建实例，还是通过 ClassPathResource 创建实例，或者通过 ApplicationContext 的 getResource() 方法获取实例，都需要提供资源位置。这意味着：资源所在的物理位置将被耦合到代码中，如果资源位置发生改变，则必须改写程序。因此，通常建议采用依赖注入的方法，让 Spring 为 Bean 实例**依赖注入**资源。

#### 9.6.2基本用例
步骤一： 创建依赖注入类，定义属性和方法

```java
public class ResourceBean {

    private Resource res;

    public void setRes(Resource res) {
        this.res = res;
    }
    public Resource getRes() {
        return res;
    }

    public void parse(){
        System.out.println(res.getFilename());
        System.out.println(res.getDescription());
    }
}
```

步骤二：创建spring配置文件，配置依赖注入

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

  <bean id="resourceBean" class="com.atguigu.spring6.resouceloader.ResourceBean" >
    <property name="res" value="classpath:atguigu.txt"/>
  </bean>
  
</beans>
```

步骤三：演示

```java
public static void main(String[] args) {
    ApplicationContext ctx =
    new ClassPathXmlApplicationContext("bean.xml");
    ResourceBean resourceBean = ctx.getBean("resourceBean",ResourceBean.class);
    resourceBean.parse();
}
```

### 9.7确定资源访问路径
#### 9.7.1概述
不管以怎样的方式创建ApplicationContext实例，都需要为ApplicationContext指定配置文件，Spring允许使用一份或多分XML配置文件。当程序创建ApplicationContext实例时，通常也是以Resource的方式来访问配置文件的，所以ApplicationContext完全支持ClassPathResource、FileSystemResource、ServletContextResource等资源访问方式。

#### 9.7.2实现类指定访问路径
（1）ClassPathXMLApplicationContext : 对应使用ClassPathResource进行资源访问。

（2）FileSystemXmlApplicationContext ： 对应使用FileSystemResource进行资源访问。

（3）XmlWebApplicationContext ： 对应使用ServletContextResource进行资源访问。

详细步骤，请查看Resources的实现类

#### 9.7.3前缀指定访问路径
##### 9.7.3.1classpath前缀
```java
public class Demo1 {

    public static void main(String[] args) {

        ApplicationContext ctx =
        new ClassPathXmlApplicationContext("classpath:bean.xml");
        System.out.println(ctx);
        Resource resource = ctx.getResource("atguigu.txt");
        System.out.println(resource.getFilename());
        System.out.println(resource.getDescription());
    }
}
```

说明：

> 在使用ApplicationContext中，指定路径时，使用**classpath:**为前缀
>

##### 9.7.3.2classpath通配符
```java
ApplicationContext ctx = new ClassPathXmlApplicationContext("classpath*:bean.xml");
```

说明：

> 在使用ApplicationContext中，指定路径时，使用classpath*为前缀，表示Spring将会搜索类加载路径下所有满足该规则的配置文件。例如:bean.xml、beans.xml
>

##### 9.7.3.3通配符其他使用
```java
ApplicationContext ctx = new ClassPathXmlApplicationContext("classpath:bean*.xml");
```

说明：

> 一次性加载多个配置文件的方式：指定配置文件时使用通配符
>

```java
ApplicationContext ctx = new ClassPathXmlApplicationContext("classpath*:bean*.xml");
```

说明：

> Spring允许将classpath*:前缀和通配符结合使用
>

## 10.国际化：i18n
笔记小结：

> 1. 概述：将软件、应用程序或网站设计为能够**适应不同语言**、**文化和地区**的需求
> 2. Java国际化：
>     - **ResourceBundle**类：是**国际化**（Internationalization）的**工具类**，用于加载和管理不同语言环境下的资源文件
>     - 配置文件命名规则：`basename_language_country.properties`，例如`messages_en_CB.properties`
>     - 基本用例：
>         1. 创建资源配置文件
>         2. 通过ResourceBundle类获取
> 3. Spring6国际化
>
> 步骤一：创建properties资源配置文件
>
> 步骤二：创建spring配置文件，配置MessageSource
>
> 步骤三：通过注入到Spring6的配置文件获取资源配置文件中的键
>

```java
public static void main(String[] args) {
    ResourceBundle resourceBundle = ResourceBundle.getBundle("messages", new Locale("en", "GB"));
    String string = resourceBundle.getString("test");
    System.out.println(string);
}
```

```java
public static void main(String[] args) {
    ApplicationContext context = new ClassPathXmlApplicationContext("beans.xml");
    Object[] objs = new Object[]{"atguigu",new Date().toString()};
    String str=context.getMessage("www.atguigu.com", objs, Locale.CHINA);
    System.out.println(str);
}
```

### 10.1概述
国际化（Internationalization），通常缩写为 i18n（i + 18个字母 + n），是指将软件、应用程序或网站设计为能够适应不同语言、文化和地区的需求。通常来讲，软件中的国际化是通过配置文件来实现的，假设要支撑两种语言，那么就需要两个版本的配置文件。

### 10.2Java国际化
#### 10.2.1createConstant方法
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1754442572422-6d144db3-d5f5-41ef-aa85-5cc549b7161f.png)

说明：

> java.util.Locale用于指定当前用户所属的语言环境等信息，java.util.ResourceBundle用于查找绑定对应的资源文件。Locale包含了language信息和country信息，Locale创建默认locale对象时使用的静态方法：
>

#### 10.2.2ResourceBundle类
ResourceBundle是Java提供的一个用于国际化（Internationalization）的工具类，用于加载和管理不同语言环境下的资源文件

#### 10.2.3配置文件命名规则
```xml
basename_language_country.properties
```

说明：

> 必须遵循以上的命名规则，java才会识别。其中，basename是必须的，语言和国家是可选的。这里存在一个优先级概念，如果同时提供了messages.properties和messages_zh_CN.propertes两个配置文件，如果提供的locale符合en_CN，那么优先查找messages_en_CN.propertes配置文件，如果没查找到，再查找messages.properties配置文件。最后，提示下，所有的配置文件必须放在classpath中，一般放在resources目录下
>

### 10.3基本用例
步骤一：在resources资源文件中，创建配置文件

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1754442572649-98c48dec-4659-4dd2-8efb-d5c9f4cbb330.png)

说明：

> Resource Bundle’messages’为系统自动生成，因为所创建的properties文件均符合Java国际化命名规则
>

步骤二：编写不同语言配置文件

```xml
test=123

test=456
```

步骤三：演示

```xml
public class ResourceBundleExample {

    public static void main(String[] args) {
        
        ResourceBundle resourceBundle = ResourceBundle.getBundle("messages", new Locale("en", "GB"));
        
        
        String string = resourceBundle.getString("test");
        
        System.out.println(string);
    }
}
```

说明：

> ResourceBundle类用于管理国际化中不同语言环境下的资源文件、Locale用于指定当前用户所属的语言环境
>

### 10.4Spring6国际化
**步骤一：创建资源文件**

**国际化文件命名格式：基本名称 _ 语言 _ 国家.properties**

**{0},{1}这样内容，就是动态参数**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1754442572649-8bd9cd2c-e354-4a0d-9f6e-6c71ac44a22e.png)

**（1）创建atguigu_en_US.properties**

```xml
www.atguigu.com=welcome {0},时间:{1}
```

**（2）创建atguigu_zh_CN.properties**

```xml
www.atguigu.com=欢迎 {0},时间:{1}
```

**步骤二： 创建spring配置文件，配置MessageSource**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

  <bean id="messageSource"
    class="org.springframework.context.support.ResourceBundleMessageSource">
    <property name="basenames">
      <list>
        <value>atguigu</value>
      </list>
    </property>
    <property name="defaultEncoding">
      <value>utf-8</value>
    </property>
  </bean>
</beans>
```

说明：

> 在Spring6中，将ResourceBundleMessageSource的id命名为messageSource，可以减少不必要的配置和显式引用，让Spring6实现自动注入
>

**步骤三：创建测试类**

```xml
package com.atguigu.spring6.javai18n;

import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
import java.util.Date;
import java.util.Locale;

public class Demo2 {

    public static void main(String[] args) {

        ApplicationContext context = new ClassPathXmlApplicationContext("beans.xml");


        Object[] objs = new Object[]{"atguigu",new Date().toString()};

        String str=context.getMessage("www.atguigu.com", objs, Locale.CHINA);
        System.out.println(str);
    }
}
```

## 11.数据校验：Validation
### 11.1概述
数据校验（Validation）是在应用程序中验证用户输入数据的过程，确保数据符合预期的规则和约束。在Java中，常用的数据校验框架是Validation API，它是Java EE的一部分，并且在Spring框架中也有集成和支持。

### 11.2基本用例-通过Validator接口实现校验
说明：

> 通过`Validator`接口实现数据校验是另一种使用Validation API进行数据校验的方式。`Validator`接口提供了更灵活的校验机制，可以自定义校验逻辑和错误消息。
>

步骤一： 引入相关依赖

```xml
<dependencies>
  <dependency>
    <groupId>org.hibernate.validator</groupId>
    <artifactId>hibernate-validator</artifactId>
    <version>7.0.5.Final</version>
  </dependency>

  <dependency>
    <groupId>org.glassfish</groupId>
    <artifactId>jakarta.el</artifactId>
    <version>4.0.1</version>
  </dependency>
</dependencies>
```

步骤二：创建实体类

```java
package com.atguigu.spring6.validation.method1;

public class Person {
    private String name;
    private int age;

    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
    public int getAge() {
        return age;
    }
    public void setAge(int age) {
        this.age = age;
    }
}
```

步骤三：实现Validator接口

```java
public class PersonValidator implements Validator {

    @Override
    public boolean supports(Class<?> clazz) {
        // 这个 Validator 只处理 Person 类型的对象
        return Person.class.equals(clazz);
    }
    
    @Override
    public void validate(Object object, Errors errors) {
        // 校验 name 是否为空
        ValidationUtils.rejectIfEmpty(errors, "name", "name.empty");
        
        Person p = (Person) object;
        
        // 校验年龄
        if (p.getAge() < 0) {
            errors.rejectValue("age", "error.value.lt0");
        } else if (p.getAge() > 110) {
            errors.rejectValue("age", "error.value.too.old");
        }
    }
}
```

说明：

> + 实现Validator接口，重写supports和validate方法，实现逻辑
> + ValidationUtils，是Spring封装的校验工具类，帮助快速实现校验
>

步骤四：演示

```java
public static void main(String[] args) {
    // 创建person对象
    Person person = new Person();
    person.setName("lucy");
    person.setAge(-1);

    // 创建Person对应的DataBinder
    DataBinder binder = new DataBinder(person);

    // 绑定校验
    binder.setValidator(new PersonValidator());

    // 开始校验
    binder.validate();

    // 输出结果
    BindingResult results = binder.getBindingResult();
    System.out.println(results.getAllErrors());
}
```

说明：

> 通过使用`DataBinder`校验器，可以方便地对对象进行数据校验，并获取校验结果
>

### 11.3基本用例-通过Bean Validation注解实现校验
说明：

> Bean Validation 是 Java 中一种基于注解的数据校验规范，可以通过在实体类的属性上添加相应的注解来定义数据校验规则
>

常用注解：

1. `@NotNull`: 检查字段值不为null。
2. `@NotEmpty`: 检查字符串、集合或数组的值不为空。
3. `@NotBlank`: 检查字符串的值不为空或不只包含空格字符。
4. `@Min(value)`: 检查字段值大于或等于指定的最小值。
5. `@Max(value)`: 检查字段值小于或等于指定的最大值。
6. `@Size(max, min)`: 检查字段值的大小在指定范围内。
7. `@Email`: 检查字段值符合电子邮件的格式要求。
8. `@Pattern(regex)`: 检查字段值符合指定的正则表达式。

步骤一：创建配置文件类

```java
@Configuration
@ComponentScan("org.example")
public class ValidationConfig {

    @Bean
    public LocalValidatorFactoryBean validator() {
        return new LocalValidatorFactoryBean();
    }
}
```

说明：

> + `LocalValidatorFactoryBean` 是 Spring 框架提供的一个实现了 `ValidatorFactory` 接口的类，用于创建和管理 `Validator` 实例。
> + 配置ComponentScan扫描规则，是为了开启包扫描，便于Spring框架的Bean注入
>

步骤二：创建实体类，使用注解定义校验规则

```java
@Data
public class User {

    @NotNull
    private String name;

    @Min(0)
    @Max(120)
    private int age;
}
```

注意：

> @Max和@Min注解，进行数据校验之前，需要导入依赖。依赖请查看通过Validator接口，进行查看
>

步骤三：创建校验器

1.使用jakarta.validation.Validator校验

```java
@Service
public class ValidatorServiceOne {

    // 导入 jakarta.validation.Validator
    @Autowired
    private Validator validator;

    public Boolean vailAge(Person person) {
        // 使用Validator的vaildate方法进行数据验证
        Set<ConstraintViolation<Person>> validate = validator.validate(person);
        return validate.isEmpty();
    }
}
```

说明：

> 在校验过程中，`Validator` 会依次检查用户对象的各个属性，验证其是否满足指定的约束条件。如果某个属性的值不符合约束条件，就会生成一个 `ConstraintViolation` 对象，其中包含了违反约束的详细信息，如属性名称、违反的约束类型、错误消息等。最后，这些 `ConstraintViolation` 对象被收集到一个 `Set` 集合中返回。
>

2.使用org.springframework.validation.Validator校验

```java
@Service
public class ValidatorServiceTwo {

    // 导入  org.springframework.validation.Validator;
    @Autowired
    private Validator validator; 

    public boolean validaPersonByValidator(User user) {
        // 创建 BindException 对象，用于收集校验错误信息
        BindException bindException = new BindException(user, user.getName());

        // 使用 Validator 对象对用户对象进行校验，将校验结果收集到 bindException 中
        validator.validate(user, bindException);

        // 判断 bindException 中是否存在校验错误信息
        return bindException.hasErrors();
    }
}
```

说明：

> `BindException` 是 `org.springframework.validation` 包中的一个类，它继承自 `org.springframework.validation.Errors` 接口，用于收集校验错误信息。通过创建 `BindException` 对象，可以将校验结果收集到其中，方便后续对错误信息的处理。
>

注意：

> 使用Validator时，需要导入依赖，同通过Validator接口实现一样
>

步骤四：演示

1.使用jakarta.validation.Validator校验

```java
@Test
public void testMyService1() {
    // 创建 Spring 容器，只加载 ValidationConfig 里的配置（包含 Validator、MyService1 等 bean）
    ApplicationContext context = new AnnotationConfigApplicationContext(ValidationConfig.class);
    
    // 从容器中获取 MyService1 这个 bean
    MyService1 myService = context.getBean(MyService1.class);
    
    // 创建一个测试用的 User 对象，并故意设置一个不合法的年龄（负数）
    User user = new User();
    user.setAge(-1);
    
    // 调用 MyService1 里的 validator 方法，校验这个 user 是否合法
    // （通常返回 true=通过校验，false=有错误）
    boolean validator = myService.validator(user);
    
    // 打印校验结果，方便观察测试是否按预期失败
    System.out.println(validator);
}
```

2.使用org.springframework.validation.Validator校验

```java
@Test
public void testMyService2() {
    // 启动 Spring 容器，加载 ValidationConfig 中的配置（包含 Validator 和 MyService2）
    ApplicationContext context = new AnnotationConfigApplicationContext(ValidationConfig.class);
    
    // 获取 MyService2 这个 bean
    MyService2 myService = context.getBean(MyService2.class);
    
    // 创建测试用的 User 对象
    User user = new User();
    user.setName("lucy");
    user.setAge(130);   // 先设一个超大年龄
    user.setAge(-1);    // 再覆盖成负数（最后生效的是 -1）

    // 调用 MyService2 的校验方法（名字写成了 validaPersonByValidator，可能是 typo）
    // 通常返回 true=校验通过，false=有错误
    boolean validator = myService.validaPersonByValidator(user);
    
    // 打印结果，预期应该是 false（因为年龄 -1 不合法）
    System.out.println(validator);
}
```

### 11.4基本用例-通过方法实现校验
说明：

> 基于方法的校验是指在特定的方法上使用校验注解来验证方法的输入参数、返回值或方法执行过程中的状态。Spring 提供了 `@Validated` 注解和校验注解（例如 `@NotNull`、`@NotBlank`、`@Min`、`@Max` 等）来实现基于方法的校验。
>

步骤一：创建配置文件类

```java
/**
 * Spring 配置类，专门用于启用方法参数/返回值校验（@Validated + Bean Validation）
 */
@Configuration
@ComponentScan("org.example")  // 扫描 org.example 包及其子包，自动注册 @Component、@Service 等 bean
public class ValidationConfig {

    /**
     * 注册 MethodValidationPostProcessor 这个 Bean
     * 作用：让 Spring 在调用带有 @Validated 的方法时，自动校验方法参数和返回值
     * 
     * 典型使用场景：
     *   - Controller 方法参数加 @Valid / @Validated
     *   - Service 方法参数/返回值加 @Validated + 约束注解（如 @NotNull、@Min）
     * 
     * 没有这个 Bean，@Validated 在方法级别不会生效（只对类级别部分生效）
     */
    @Bean
    public MethodValidationPostProcessor validationPostProcessor() {
        return new MethodValidationPostProcessor();
    }
}
```

说明：

> `MethodValidationPostProcessor` 是 Spring 提供的一个后置处理器，用于在方法调用时进行参数校验。它可以自动拦截被 `@Validated` 注解标记的方法，并对方法的参数进行校验。
>

步骤二：创建实体类，使用注解定义校验规则

```java
@Data
public class User {

    @NotNull
    private String name;

    @Min(0)
    @Max(120)
    private int age;

    @Pattern(regexp = "^1(3|4|5|7|8)\\d{9}$",message = "手机号码格式错误")
    @NotBlank(message = "手机号码不能为空")
    private String phone;
}
```

步骤三：创建校验器

```java
@Service
@Validated // 类级别加 @Validated → 启用方法参数/返回值校验（需要配合 MethodValidationPostProcessor）
public class MyService {

    /**
     * 方法参数校验示例
     * 
     * @NotNull  → user 参数不能为 null（来自 jakarta.validation.constraints）
     * @Valid    → 对 user 对象内部的字段进行级联校验（如 User 类里的 @NotBlank、@Min 等注解会生效）
     * 
     * 调用此方法时，如果 user 为 null 或 user 内部字段不合法 → 会抛 MethodArgumentNotValidException / ConstraintViolationException
     */
    public String testParams(@NotNull @Valid User user) {
        return user.toString();     // 业务逻辑：简单返回 user 的 toString
    }
}
```

说明：

> 如果校验不通过，将抛出 ConstraintViolationException 异常、如果校验通过，执行其他业务逻辑
>

步骤四：演示

```java
@Test
public void testMyService1() {
    ApplicationContext context = new AnnotationConfigApplicationContext(ValidationConfig.class);
    MyService myService = context.getBean(MyService.class);
    User user = new User();
    user.setAge(-1);
    myService.testParams(user);
}
```

### 11.5基本用例-通过自定义校验
步骤一：自定义校验注解

```java
@Target({ElementType.METHOD, ElementType.FIELD, ElementType.ANNOTATION_TYPE, ElementType.CONSTRUCTOR, ElementType.PARAMETER})
@Retention(RetentionPolicy.RUNTIME)
// @Target 和 @Retention 来定义注解的使用范围和生命周期
@Documented
// @Constraint 标记该注解为校验注解，并指定对应的校验器类
@Constraint(validatedBy = {CannotBlankValidator.class})
public @interface CannotBlank {
    //默认错误消息
    String message() default "不能包含空格";

    //分组
    Class<?>[] groups() default {};

    //负载
    Class<? extends Payload>[] payload() default {};

    //指定多个时使用
    @Target({ElementType.METHOD, ElementType.FIELD, ElementType.ANNOTATION_TYPE, ElementType.CONSTRUCTOR, ElementType.PARAMETER, ElementType.TYPE_USE})
    @Retention(RetentionPolicy.RUNTIME)
    @Documented
    @interface List {
        CannotBlank[] value();
    }
}
```

说明：

> 此注解的内容实现，可以参考现有注解自带部分
>

补充：

> + `@Target` 注解用于指定注解的使用范围，包括方法、字段、注解类型、构造函数、参数。
> + `@Retention` 注解用于指定注解的生命周期，即在运行时仍然保留注解信息。
> + `@Documented` 注解用于指示该注解应该被包含在生成的文档中。
> + `@Constraint` 注解标记该注解为校验注解，并指定对应的校验器类 `CannotBlankValidator`。
> + `message()` 方法定义了默认的错误消息，当校验失败时使用该消息。
> + `groups()` 方法用于分组校验，可以指定校验器在哪个分组生效。
> + `payload()` 方法指定负载类型，可在自定义校验器中使用。
> + `@List` 注解用于指定多个 `@CannotBlank` 注解时使用，可以对多个注解进行组合。
>

注意：

> 注解本身并不会实现具体的校验逻辑，而是通过校验器类 `CannotBlankValidator` 来实现校验的具体逻辑。
>

步骤二：编写校验类

```java
public class CannotBlankValidator implements ConstraintValidator<CannotBlank, String> {

    // initialize() 方法用于初始化校验器，在校验之前进行一些初始化操作。可以获取注解中的属性值，并进行相应的处理
    @Override
    public void initialize(CannotBlank constraintAnnotation) {
    }

    // isValid() 方法是校验的核心逻辑，用于判断被校验的值是否符合自定义的校验规则
    @Override
    public boolean isValid(String value, ConstraintValidatorContext context) {
        //null时不进行校验
        if (value != null && value.contains(" ")) {
            //获取默认提示信息
            String defaultConstraintMessageTemplate = context.getDefaultConstraintMessageTemplate();
            System.out.println("default message :" + defaultConstraintMessageTemplate);
            //禁用默认提示信息
            context.disableDefaultConstraintViolation();
            //设置提示语
            context.buildConstraintViolationWithTemplate("can not contains blank").addConstraintViolation();
            return false;
        }
        return true;
    }
}
```

说明：

> + 创建自定义的校验器类：实现 ConstraintValidator 接口，并指定要校验的注解类型和校验的值类型。在校验逻辑中，实现自定义的校验规则，根据需要返回校验结果
> + 在 `isValid()` 方法中，有两个参数： 
>     - `value`: 被校验的值，即需要进行校验的字段或方法参数的值。
>     - `context`: 校验器的上下文对象，用于控制校验过程中的一些行为和设置。
> + 在实现 `isValid()` 方法时，根据自定义的校验规则，可以使用 `value` 参数获取被校验的值，并根据业务逻辑进行校验。如果校验成功，返回 `true` 表示校验通过；如果校验失败，返回 `false` 表示校验不通过。
>

步骤三：演示

```java
public static void main(String[] args) {
    ApplicationContext applicationContext = new AnnotationConfigApplicationContext(ValidationConfig.class);
    ValidatorService bean = (ValidatorService) applicationContext.getBean("validatorService");
    Person person = new Person();
    person.setAge(12);
    person.setMessage("fd s  f");
    String s = bean.vailAge(person);
    System.out.println(s);
}
```

说明：

> 此步骤只是通过自定义注解实现自定义规则来进行校验，具体如何实现校验还需要参考`<u>实现</u>`校验的方法
>

## 12.提前编译：AOT
笔记小结：

> 1. 概述： 
>     - 含义：提前编译（Ahead of Time Compilation，AOT）是一种将**程序代码**在运行之前**编译为机器代码**的技术。与传统的即时编译（Just-in-Time Compilation，JIT）相反，即时编译是在运行时将程序代码逐行解释为机器代码并执行。
>     - Graalvm：Spring6 支持的 AOT 技术，这个 GraalVM 就是底层的支持，Spring 也对 GraalVM 本机映像提供了一流的支持。GraalVM 是一种高性能 JDK，旨在加速用 Java 和其他 JVM 语言编写的应用程序的执行，同时还为 JavaScript、Python 和许多其他流行语言提供运行时。
>     - Native Image：目前业界除了这种在JVM中进行AOT的方案，还有另外一种实现Java AOT的思路，那就是直接**摒弃JVM**，和C/C++一样通过编译器直接将代码编译成机器代码，然后运行。这无疑是一种直接颠覆Java语言设计的思路，那就是GraalVM Native Image。
> 2. Native Image构建过程 
>     - GraalVM安装
>     - 安装C++的编译环境
>     - 编写代码，构建Native Image
>

### 12.1概述
#### 12.1.1含义
提前编译（Ahead of Time Compilation，AOT）是一种将程序**代码**在**运行**之**前编译**为机器代码的技术。与传统的即时编译（Just-in-Time Compilation，JIT）相反，即时编译是在运行时将程序代码逐行解释为机器代码并执行。

**AOT优点**

1. 更**快**的**启动时间**：由于代码已经编译为机器码，无需在运行时进行编译和解释，因此可以直接执行，减少了启动时间。
2. 更**高**的**执行性能**：AOT 编译可以进行全面的静态优化，包括代码优化、内联、消除不必要的检查等，这些优化在编译时进行，可以提供更高的执行性能。
3. 更**好**的**安全性**：AOT 编译可以将源代码编译为机器码，隐藏了源代码的逻辑，提供了更好的安全性。

**AOT缺点**

1. 较大的编译输出：AOT 编译将源代码或中间代码完全编译为机器码，因此编译输出通常比源代码或字节码更大，占用更多的存储空间。
2. 编译时间延迟：AOT 编译需要在程序运行之前进行，因此会增加开发和构建的时间成本，特别是对于较大的代码库或复杂的项目。
3. 缺乏运行时优化：由于 AOT 编译是在程序运行之前进行的，它无法根据实际运行时的上下文信息进行优化，无法动态适应变化的执行环境。

#### 12.1.2JIT与AOT区别
**JIT， Just-in-time,动态(即时)编译，边运行边编译；**

在程序运行时，根据算法计算出热点代码，然后进行 JIT 实时编译，这种方式吞吐量高，有运行时性能加成，可以跑得更快，并可以做到**动态生成代码**等，但是相对**启动速度较慢**，并需要一定时间和调用频率才能触发 JIT 的分层机制。JIT 缺点就是编译需要占用运行时资源，会导致进程卡顿。

**AOT，Ahead Of Time，指运行前编译，预先编译;**

AOT 编译能直接将源代码转化为机器码，**内存占用低**，**启动速度快**，可以无需 runtime 运行，直接将 runtime 静态链接至最终的程序中，但是无运行时性能加成，不能根据程序运行情况做进一步的优化，AOT 缺点就是在程序运行前编译会使程序安装的时间增加。

总的来说：JIT即时编译指的是在程序的运行过程中，将字节码转换为可在硬件上直接运行的机器码，并部署至托管环境中的过程。而 AOT 编译指的则是，在程序运行之前，便将字节码转换为机器码的过程。

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1754442572765-82123ade-f6fb-435f-affc-a6f3a591af63.png)

说明：

> .java -> .class -> (使用jaotc编译工具) -> .so（程序函数库,即编译好的可以供其他程序使用的代码和数据）
>

#### 12.1.3Graalvm
Spring6 支持的 AOT 技术，这个 GraalVM 就是底层的支持，Spring 也对 GraalVM 本机映像提供了一流的支持。GraalVM 是一种高性能 JDK，旨在加速用 Java 和其他 JVM 语言编写的应用程序的执行，同时还为 JavaScript、Python 和许多其他流行语言提供运行时。 GraalVM 提供两种运行 Java 应用程序的方法：在 HotSpot JVM 上使用 Graal 即时 (JIT) 编译器或作为提前 (AOT) 编译的本机可执行文件。 GraalVM 的多语言能力使得在单个应用程序中混合多种编程语言成为可能，同时消除了外语调用成本。GraalVM 向 HotSpot Java 虚拟机添加了一个用 Java 编写的高级即时 (JIT) 优化编译器。

GraalVM 具有以下特性：

（1）一种**高级**优化**编译器**，它生成更快、更精简的代码，需要更少的计算资源

（2）AOT 本机图像编译提前将 Java 应用程序编译为本机二进制文件，立即启动，**无需预热**即可实现最高性能

（3）Polyglot 编程在单个应用程序中利用流行语言的最佳功能和库，无需额外开销

（4）高级工具在 Java 和**多**种**语言**中调试、监视、分析和优化资源消耗

总的来说对云原生的要求不算高短期内可以继续使用 2.7.X 的版本和 JDK8，不过 Spring 官方已经对 Spring6 进行了正式版发布。

#### 12.1.4Native Image
目前业界除了这种在JVM中进行AOT的方案，还有另外一种实现Java AOT的思路，那就是直接**摒弃JVM**，和C/C++一样通过编译器直接将代码编译成机器代码，然后运行。这无疑是一种直接颠覆Java语言设计的思路，那就是GraalVM Native Image。它通过C语言实现了一个超微缩的运行时组件 —— Substrate VM，基本实现了JVM的各种特性，但足够轻量、可以被轻松内嵌，这就让Java语言和工程摆脱JVM的限制，能够真正意义上实现和C/C++一样的AOT编译。这一方案在经过长时间的优化和积累后，已经拥有非常不错的效果，基本上成为Oracle官方首推的Java AOT解决方案。  
Native Image 是一项创新技术，可将 Java 代码编译成**独立的本机可执行文件**或**本机共享库**。在构建本机可执行文件期间处理的 Java 字节码包括所有应用程序类、依赖项、第三方依赖库和任何所需的 JDK 类。生成的自包含本机可执行文件特定于不需要 JVM 的每个单独的操作系统和机器体系结构。

### 12.2Native Image构建过程
#### 12.2.1GraalVM安装
##### 步骤一：下载GraalVM
进入官网下载：https://www.graalvm.org/downloads/

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1754442572844-4187b0b2-f1f6-4f55-9821-6fea8370d204.png)

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1754442572820-1b1c3278-cca1-4708-8e9d-2d35b69e742b.png)

##### 步骤二：配置环境变量
**添加GRAALVM_HOME**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1754442572919-2a43beed-3cf1-4030-a9f3-2a352300589c.png)

**把JAVA_HOME修改为graalvm的位置**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1754442572932-d9eabb49-85ce-445a-ba3e-1955978e4ff3.png)

**把Path修改位graalvm的bin位置**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1754442572982-ca0dbeb2-658d-4cfd-9d21-b9c06aa30126.png)

**使用命令查看是否安装成功**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1754442573066-9a41e203-883e-4d12-a38e-c17dd3019b2b.png)

##### 步骤三：安装native-image插件
**使用命令 gu install native-image下载安装**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1754442573147-6a560403-dfb0-4120-af66-69c052cbec12.png)

#### 12.2.2安装C++的编译环境
##### 步骤一：下载Visual Studio安装软件
https://visualstudio.microsoft.com/zh-hans/downloads/

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1754442573377-33685fcf-280a-473e-9b6e-024f4061068f.png)

##### 步骤二：安装Visual Studio
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1754442573391-c46ba180-d5ac-43c6-9f61-1a961640ab71.png)

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1754442573283-cd3fb8ba-935a-4ca0-9676-cbc962af9f40.png)

##### 步骤三：添加Visual Studio环境变量
配置INCLUDE、LIB和Path

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1754442573365-f9c3a7aa-0155-4076-97db-6c5b421cc235.png)

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1754442573427-65385839-153c-4a23-abb5-dc2939fe32a8.png)

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1754442573482-13e9ac50-8f45-4386-b330-5c81bafd51c9.png)

##### 步骤四：打开工具，在工具中操作
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1754442573601-4bb9b97e-e092-43f2-9069-e492790ece7a.png)

#### 12.2.3编写代码，构建Native Image
##### 步骤一：编写Java代码
```xml
public class Hello {

    public static void main(String[] args) {
        System.out.println("hello world");
    }
}
```

##### 步骤二：复制文件到目录，执行编译
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1754442573750-43d7b2fa-012f-4e36-ba42-c9ee1225c1a0.png)

##### 步骤三：Native Image 进行构建
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1754442573757-c1ab98b9-e86b-42e8-8c11-fc3c5441d9c6.png)

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1754442573825-5be4ed78-cc78-4b36-992c-c6c8688fc841.png)

##### 步骤四：查看构建的文件
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1754442573796-482b7a03-1f99-463b-bafa-6978542d487f.png)

##### 步骤五：执行构建的文件
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1754442573834-66cd3a29-daf2-483a-b445-c348bfb1d026.png)

可以看到这个Hello最终打包产出的二进制文件大小为11M，这是包含了SVM和JDK各种库后的大小，虽然相比C/C++的二进制文件来说体积偏大，但是对比完整JVM来说，可以说是已经是非常小了。

相比于使用JVM运行，Native Image的速度要快上不少，cpu占用也更低一些，从官方提供的各类实验数据也可以看出Native Image对于启动速度和内存占用带来的提升是非常显著的：

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1754442574177-8e17f039-5eb7-4bbf-b20c-1d595b0e6756.png)

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1754442574290-c2f2830b-b1a9-43aa-8256-de43442dd946.png)

## 知识加油站
### 1.ResourceLoader和Resource的实现类的区别
`ResourceLoader`和`Resource`接口的实现类主要有以下区别：

1. `ResourceLoader`的实现类主要用于**加载**资源，提供了加载资源的方法，例如`ClassPathResourceLoader`用于加载类路径下的资源，`UrlResourceLoader`用于加载URL资源，`FileSystemResourceLoader`用于加载文件系统中的资源等。
2. `Resource`的实现类主要用于**访问和操作**资源，提供了对资源的具体操作方法，例如`ClassPathResource`用于访问类路径下的资源，`UrlResource`用于访问URL资源，`FileSystemResource`用于访问文件系统中的资源等。
3. `ResourceLoader`的实现类通常是Spring框架**内部使用**的，用于提供资源加载的功能，不直接在应用程序中使用。
4. `Resource`的实现类可以在**应用程序中使用**，通过它可以获取资源的信息和内容，进行读取和操作。

总的来说，`ResourceLoader`的实现类主要负责**加载**资源，而`Resource`的实现类主要负责**访问和操作**资源。它们在框架中扮演不同的角色，但都是用于处理和管理应用程序中的资源。

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1754442574149-38e0fb42-e830-4897-b914-876ff6af9fbe.png)

说明：

> Resource接口是Spring中访问资源的抽象，Resource接口本身只提供了规定，下面有很多的实现类。都是从实际的系统底层资源进行抽象的资源描述符。一般来说在Spring中是将资源描述为URL格式和Ant风格带通配符的资源地址。
>

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1754442574165-f9678fd2-d217-4e28-9577-1a08f007c836.png)

说明：

> `ResourceLoader`接口，顾名思义，设计的目的是为了快速返回（也就是加载）Resource实例的对象
>



