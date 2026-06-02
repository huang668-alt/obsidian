这份关于 **Spring 依赖注入（DI）失效原理** 的技术笔记已为你重新梳理，优化了排版结构与视觉层级，更方便快速复习与查阅。

## 💡 核心大局观

Spring 依赖注入（DI）能够生效，完全依赖于 **IoC 容器健全的对象生命周期流水线**。一旦代码在某个瞬间**脱离了容器的托管**，或者**颠倒了执行时序**，`@Autowired` 或 `@Resource` 就会瞬间瘫痪，导致常见的 `NullPointerException`。

## 🔍 五大经典失效场景深度剖析

### 1. 致命硬伤：对象是手动 `new` 出来的

- **典型现象：** 在 `Controller` 里注入了 `ServiceA`，但在方法内部通过 `ServiceB b = new ServiceB();` 手动创建了实例，结果发现 `b` 内部依赖的 `Dao` 全都为 `null`。
    
- **底层原理：** Spring 派出的属性注入机器人（如 `AutowiredAnnotationBeanPostProcessor`）只服务于容器内的 Bean。使用 `new` 关键字创建的对象直奔 JVM 堆内存，**完全绕过了 IoC 容器的生命周期**，容器甚至不知道它的存在，自然不会对其进行依赖组装。
    

> **自救指南：** > 只要类内部需要使用 `@Autowired`，该类自身就必须交给 Spring 管理（加上 `@Component`、`@Service` 等），并且必须从容器中获取该对象，**严禁手动 `new`**。

### 2. 静态变量魔咒：注入到了 `static` 属性上

- **典型现象：**
	
``` java
@Autowired 
private static UserService userService; // ❌ 注入失败，使用时报空指针
```
    
- **底层原理：** Spring 的依赖注入是**基于实例（Instance）**的，通过反射为某个具体对象的堆内存字段赋值。而 `static` 修饰的变量属于**类（Class）元数据**，存储在方法区（元空间）。Spring 容器在解析注解时，其内置的后置处理器默认会**直接跳过静态属性**。
    

> **自救指南：** > 若工具类非要使用 Bean，可通过**构造函数注入**间接赋值：

``` java

private static UserService userService;

@Autowired
public MyUtils(UserService userService) {
MyUtils.userService = userService;
}


```
---
### 3. 时序错位：在构造函数中过早调用
* **典型现象：**

``` java
@Component
public class OrderService {
    @Autowired
    private UserService userService;

    public OrderService() {
        userService.sayHello(); // ❌ 报错：NullPointerException！
    }
}

```
- **底层原理：** 这是一个典型的**生命周期时序错误**。IoC 容器的流水线严格遵循以下先后顺序：
    
    $$\text{一阶段：实例化 (Instantiation，调用构造函数)} \longrightarrow \text{二阶段：属性填充 (Populate，执行注入)}$$
    
    在构造函数执行时，对象还处于“胚胎”阶段，Spring 还没来得及注入 `userService`，此时调用必然为 `null`。
    

> **自救指南：** > 将初始化逻辑移入 **`@PostConstruct`** 注解的方法中，该方法会在属性填充彻底完成**之后**才被容器触发。

### 4. 盲区孤儿：类没有被 Spring 容器扫描到

- **典型现象：** 注解写得很完美，代码逻辑也正确，但项目启动时报错找不到 Bean，或者注入的对象死活为空。
    
- **底层原理：** Spring Boot 默认的 `@ComponentScan` 扫描范围是**主启动类（XxxApplication）所在的包及其子包**。如果你的组件写在了定义域之外的独立包中，容器启动时根本无法为其绘制 `BeanDefinition` 蓝图。没有图纸，工厂就无法生产该 Bean，导致单例池中查无此人。
    

> **自救指南：** > 严格遵守约定优于配置，将组件写在启动类的子包下；或者在启动类上显式扩大扫描范围：
> 
> `@ComponentScan(basePackages = {"com.example.app", "com.example.utils"})`。

### 5. 安全网破裂：无法应对的循环依赖

- **典型现象：** 组件 A 注入 B，B 注入 A，项目启动直接抛出 `BeanCurrentlyInCreationException` 异常，导致注入流程彻底瘫痪。
    
- **底层原理：** 尽管 Spring 的**三级缓存**能完美解决单例 Bean 的普通循环依赖，但在以下两种场景下会宣告失效：
    
    - **构造器循环依赖：** A 和 B 均通过构造函数强行注入对方。在“一阶段：实例化”时，双方由于拿不到对方的实例，连“提前暴露原始对象”进入三级缓存的机会都没有，直接死锁。
        
    - **多例（Prototype）循环依赖：** 容器不会缓存 Prototype 作用域的 Bean。当 A（多例）找 B，B（多例）又找 A 时，容器会陷入无限 new 对象的死循环，直至触发安全防线崩溃。
        

> **自救指南：** > 
> 	 尽量使用 `setter` 注入或属性注入，避免构造器循环依赖。
> 	
> 	在循环依赖的任意一端加上 **`@Lazy`** 注解。Spring 会先注入一个懒加载的代理外壳，待真正使用时再动态解开，以此打破死锁链条。


## 📊 避坑速查表

|**失效场景**|**核心本质原因**|**快速修复方案**|
|---|---|---|
|**手动 `new` 对象**|脱离了 IoC 容器管辖，无法触发后置处理器|必须将该类注册为 Bean 并由容器注入|
|**`static` 变量赋值**|依赖注入基于对象实例，静态属性属于类元数据|使用构造风注入或 `@PostConstruct` 间接赋值|
|**构造函数中调用**|此时生命周期正处于实例化，属性尚未开始填充|将调用逻辑移至 `@PostConstruct` 方法中|
|**超出包扫描范围**|容器未能读取到该类，未生成 `BeanDefinition`|调整包结构，或显式配置 `@ComponentScan`|
|**构造器/多例循环**|突破了三级缓存的防御极限，导致实例化死锁|改变注入方式，或配合使用 `@Lazy` 注解|

## 🎯 终极防翻车口诀

> **手动 `new` 的不归管，静态属性跳过看；**
> 
> **构造函数时序早，超出扫描变孤儿；**
> 
> **构造循环缓存破，看清原理不翻车。**