如果把传统的 Java 开发比作“个体户手工小作坊”**（谁需要对象谁就自己 `new`），那么 Spring IoC（Inversion of Control，控制反转）容器就是一家**“现代化全自动智能工厂”。

你不再需要自己去采购原材料（`new` 对象）、组装零件（手动注入依赖），而是把你的类做成“设计图纸”交给工厂，工厂开足马力帮你把全套对象生产好、组装好，并在你需要的时候送到你手上。

要真正吃透 Spring IoC 容器的原理，我们需要拆解它的**核心组件、核心图纸（BeanDefinition）**以及**一条近乎严苛的对象生产流水线（生命周期）**。

## 一、 核心概念：控制反转（IoC）与依赖注入（DI）

- **控制反转（IoC）：** 这是一种**设计思想**。原本对象的创建、销毁、依赖组装的权利在你（开发者）手里，现在这个控制权被“反转”到了 Spring 容器手里。
    
- **依赖注入（DI）：** 这是 IoC 的**具体实现手段**。容器在运行期间，动态地将某个对象所需要的外部依赖，通过反射的方式强行注入进去（比如给 A 对象的 `b` 属性赋值）。
    

## 二、 容器的双层架构：两大家族接口

Spring 容器并不是一整块代码，它有着明确的层次设计，最核心的是两个接口：

### 1. 基层核心：`BeanFactory`

- **定位：** Spring 框架最底层的核心工厂。
    
- **特点：** 采用懒加载（Lazy-load）机制。只有当你真正调用 `getBean()` 去向它要一个对象时，它才现场帮你 `new` 出来。它的功能很纯粹，就是负责最基础的对象的实例化和依赖注入。
    

### 2. 高级门面：`ApplicationContext`

- **定位：** 派生自 `BeanFactory` 的高级子接口，也是我们在开发中直接打交道的“大容器”。
    
- **特点：** 采用**预加载**机制。容器一启动，默认就把所有单例（Singleton）的 Bean 全部实例化好。除了具备 `BeanFactory` 的所有功能外，它还集成了**国际化（MessageSource）**、**事件发布机制（ApplicationEventPublisher）**、资源加载（ResourcePatternResolver）等大厂标配的企业级高级功能。
    

## 三、 核心图纸：`BeanDefinition`

在 Spring 容器真正开始干活之前，它必须先知道它要生产什么。这就全靠 **`BeanDefinition`（Bean定义信息）**。

无论你是用 XML 配置的 `<bean>`，还是用注解写的 `@Component`，亦或是用 `@Bean` 声明的方法，Spring 都会利用解析器（如 `AnnotatedBeanDefinitionReader`）把它们统一翻译成一个个 `BeanDefinition` 对象存入一个 Map 中。

> **`BeanDefinition` 里面存了什么？**
> 
> - 这个类的全路径名（`className`）。
>     
> - 它的作用域（`scope`）：是单例（Singleton）还是多例（Prototype）。
>     
> - 它是不是懒加载（`isLazy`）。
>     
> - 它依赖了哪些别的 Bean（`propertyValues`）。
>     
> - 它的销毁和初始化方法叫什么名字。
>     

## 四、 核心工作流：对象的生命周期（生产流水线）

这是面试的核心重灾区。一个普通的 Java 类，是如何在 IoC 容器中一步步蜕变成一个功能强大的 Bean 的？整个流程可以归纳为以下四大战役：

### 1. 实例化 (Instantiation)

- **做什么**：Spring 容器根据 Bean 的定义（XML 或注解），利用 Java 反射机制在堆内存中为 Bean **分配空间，创建实例对象**。
    
- **对应操作**：相当于执行了 `new MyBean()`。此时对象仅仅是个“空壳”，属性都还是默认值。
    

### 2. 属性赋值 (Population of properties)

- **做什么**：Spring 将配置好的属性值和依赖的对象注入到 Bean 中。
    
- **对应操作**：执行依赖注入（DI），比如处理 `@Autowired`、`@Value` 注解，调用相应的 `setter` 方法。
    

### 3. 初始化 (Initialization)

这是生命周期中行为最丰富的一步，Spring 提供了大量的扩展点供开发者干预 Bean 的行为。

- **检查 Aware 接口**：如果 Bean 实现了某些 `Aware` 接口，Spring 会注入相关的框架资源。
    
    - `BeanNameAware` -> 注入当前 Bean 的名称。
        
    - `BeanFactoryAware` -> 注入当前的 BeanFactory 容器。
        
    - `ApplicationContextAware` -> 注入当前的上下文环境。
        
- **BeanPostProcessor 前置处理**：执行所有注册的 `BeanPostProcessor` 的 `postProcessBeforeInitialization()` 方法。AOP 代理通常就在这个阶段（或后置阶段）埋下伏笔。
    
- **执行初始化方法**：
    
    1. 如果 Bean 实现了 `InitializingBean` 接口，会调用 `afterPropertiesSet()` 方法。
        
    2. 如果配置了自定义的 `init-method`（如 `@Bean(initMethod = "...")` 或 XML 里的配置），则执行该自定义方法。
        
- **BeanPostProcessor 后置处理**：执行 `postProcessAfterInitialization()` 方法。**这是 AOP 生成动态代理对象的核心位置**。
    

> 💡 **核心提示**：经历了初始化阶段后，Bean 已经完全准备就绪，可以被应用程序正常使用了。

### 4. 销毁 (Destruction)

当容器关闭（如应用关闭或 Web 容器停止）时，Bean 会进入销毁流程，用于释放资源（如关闭数据库连接、线程池、文件流等）。

- **执行销毁方法**：
    
    1. 如果实现了 `DisposableBean` 接口，调用 `destroy()` 方法。
        
    2. 如果配置了自定义的 `destroy-method`（如 `@Bean(destroyMethod = "...")`），则执行该自定义方法。


> 💡 **流水线终点：** 最终存入全局单例池（`singletonObjects` 一级缓存）供你使用的，就是这个打磨完毕、功能完备、甚至被 AOP 包裹过的 **Bean 对象**。当容器关闭时，它还会负责调用销毁方法（如 `@PreDestroy`）。

## 🛠️ 原理精髓小总结

Spring IoC 的底层原理，本质上就是：

> **“通过 I/O 模块读取配置信息 $\rightarrow$ 转换为内存中的 `BeanDefinition` 蓝图 $\rightarrow$ 借助工厂模式（BeanFactory）集中管理 $\rightarrow$ 在 Bean 的生命周期中利用反射技术动态执行实例化、属性注入 $\rightarrow$ 最终通过后置处理器（BeanPostProcessor）织入 AOP 代理，实现业务对象的全自动托管。”**



**Spring Bean 的作用域（Scope）极详细讲解**

Bean 的作用域是 Spring IoC 容器中非常重要的概念，它决定了**Bean 实例的创建时机**和**存在范围**（生命周期长度）。

---


# Bean的作用域

### 1. 什么是 Bean Scope？

Scope 直译为“作用域”，它控制了：
- Bean 实例什么时候被创建
- Bean 实例被创建多少个
- Bean 实例的生命周期有多长
- Bean 在什么范围内可见（单例还是每次都新对象）

---

### 2. Spring 支持的所有作用域

| 作用域          | 注解写法                     | 说明                                      | 创建时机               | 适用场景                     | 线程安全 |
|-----------------|------------------------------|-------------------------------------------|------------------------|------------------------------|----------|
| **singleton**   | `@Scope("singleton")`        | **默认值**，整个容器只有一个实例         | 容器启动时（预加载）   | 大多数 Service、Repository   | 需要注意 |
| **prototype**   | `@Scope("prototype")`        | 每次获取（getBean）都创建一个新实例     | 获取时（懒加载）       | 有状态 Bean、DTO 等          | 通常安全 |
| **request**     | `@Scope("request")`          | 每个 HTTP 请求创建一个实例               | 请求开始时             | Web 环境，Request 相关数据   | 安全     |
| **session**     | `@Scope("session")`          | 每个 HTTP Session 创建一个实例           | Session 开始时         | 用户会话数据（如购物车）     | 安全     |
| **application** | `@Scope("application")`      | 整个 ServletContext（应用）只有一个实例 | 应用启动时             | Web 环境，全局共享数据       | 需要注意 |
| **websocket**   | `@Scope("websocket")`        | 每个 WebSocket 会话创建一个实例         | WebSocket 连接建立时   | WebSocket 应用               | 安全     |

**注意**：`request`、`session`、`application`、`websocket` 只有在**Web 环境**下才有效（需使用 `WebApplicationContext`）。

---

### 3. singleton 作用域（最常用）

**特点**：
- 整个 Spring 容器中只存在**一个**实例
- 容器启动时（非懒加载）就创建
- 所有地方注入的都是同一个对象

**配置方式**：

```java
@Service                    // 默认就是 singleton
public class UserService { }

@Scope("singleton")         // 可以显式声明
@Component
public class UserService { }
```

**使用注意**：
- 如果 Bean 有状态（成员变量会被修改），必须考虑**线程安全**问题（加锁或使用 ThreadLocal）。
- 适合**无状态**的服务类（Service、Controller 等）。

---

### 4. prototype 作用域（多例）

**特点**：
- 每次从容器中获取（`getBean()` 或注入）都会创建一个**全新**的实例
- 容器不负责管理其生命周期（销毁回调不会被调用）

**配置方式**：

```java
@Scope("prototype")
@Component
public class PrototypeBean {
    private int count = 0;
    
    public void increment() {
        count++;
        System.out.println("当前计数: " + count);
    }
}
```

**测试代码**：

```java
PrototypeBean b1 = context.getBean(PrototypeBean.class);
b1.increment();   // 1

PrototypeBean b2 = context.getBean(PrototypeBean.class);
b2.increment();   // 1 （不是 2！）
```

**适用场景**：
- 有状态的 Bean（每个使用方需要独立状态）
- 大对象、耗时初始化对象（懒加载）
- 集成第三方非线程安全类

**重要注意**：
- 当 singleton Bean 注入 prototype Bean 时，默认只会注入**一次**（因为 singleton 只创建一次）。
- 解决方法：使用 **Lookup 方法注入** 或 **ObjectFactory / Provider**。

**Lookup 方法注入示例**：

```java
@Service
public class SingletonService {
    
    @Lookup                     // Spring 会动态实现这个方法
    public PrototypeBean getPrototypeBean() {
        return null;            // 返回值会被忽略
    }
    
    public void doSomething() {
        PrototypeBean bean = getPrototypeBean();  // 每次调用都新对象
    }
}
```

---

### 5. Web 相关作用域（request / session / application）

这些作用域依赖于 `RequestContextListener` 或 `RequestContextFilter`。

#### 5.1 request 作用域

```java
@Scope("request")
@Component
public class RequestScopedBean {
    private String requestId = UUID.randomUUID().toString();
}
```

每个 HTTP 请求都会创建一个独立的 Bean，请求结束时自动销毁。

#### 5.2 session 作用域

```java
@Scope("session")
@Component
public class ShoppingCart {     // 购物车典型例子
    private List<Item> items = new ArrayList<>();
}
```

同一个用户会话内共享同一个 Bean，不同用户是不同实例。

#### 5.3 application 作用域

相当于 ServletContext 级别的单例，比 singleton 范围更大（跨 Spring 容器）。

---

### 6. 如何自定义作用域（高级）

Spring 允许自定义 Scope：

```java
public class MyCustomScope implements Scope {
    // 实现 get()、remove()、registerDestructionCallback() 等方法
}
```

然后注册：

```java
beanFactory.registerScope("myScope", new MyCustomScope());
```

---

### 7. 作用域与代理（Scoped Proxy）

当 **singleton Bean 注入 request/session 作用域 Bean** 时，会出现问题（因为 singleton 生命周期长于 request）。

**解决方式**：使用作用域代理。

```java
@Scope(value = "session", proxyMode = ScopedProxyMode.TARGET_CLASS)
@Component
public class UserPreferences { }
```

- `ScopedProxyMode.TARGET_CLASS`：CGLIB 代理（推荐）
- `ScopedProxyMode.INTERFACES`：JDK 动态代理

---

### 8. 作用域总结与最佳实践

1. **默认使用 singleton**，性能最好。
2. **有状态的 Bean** 使用 `prototype`。
3. **Web 环境中用户相关数据** 使用 `session` / `request`。
4. **避免在 singleton 中注入 prototype**，除非使用 `@Lookup` 或 `ObjectProvider<T>`（Spring 4.3+ 推荐）：

```java
@Autowired
private ObjectProvider<PrototypeBean> prototypeProvider;

public void method() {
    PrototypeBean bean = prototypeProvider.getObject();  // 每次新对象
}
```

5. **线程安全**：singleton Bean 如果有可变状态，必须加同步控制。

---
