## 1. 为什么要用 Spring Boot？（必问）
**要点**：简化配置、自动配置、起步依赖、内嵌服务器、无需独立部署WAR。

## 2. 说一下 `@SpringBootApplication` 注解
**要点**：它是三个注解的组合：
- `@EnableAutoConfiguration`（开启自动配置）
- `@ComponentScan`（组件扫描）
- `@SpringBootConfiguration`（本质是 `@Configuration`）

## 3. Spring Boot 自动配置原理（高频重点）
### 第 1 步：入口触发

启动类上的 `@SpringBootApplication` 是一个**组合注解**，它里面包含了 `@EnableAutoConfiguration`。

```
@SpringBootApplication  // 点进去看源码
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

`@EnableAutoConfiguration` 才是自动配置的真正开关。
## 第 2 步：加载工厂文件

`@EnableAutoConfiguration` 内部通过 `@Import(AutoConfigurationImportSelector.class)` 导入一个核心类。

`AutoConfigurationImportSelector` 会去读取所有 Jar 包下的：
`META-INF/spring.factories`

在这个文件中，有一个 key 叫做：

```
org.springframework.boot.autoconfigure.EnableAutoConfiguration
```

它的 value 是一大堆配置类的全限定名，例如：

```
org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration,
org.springframework.boot.autoconfigure.web.servlet.WebMvcAutoConfiguration,
...
```


> ⚠️ 注意：Spring Boot 2.7+ 开始推荐使用 `META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports`，但原理相同，面试可以说 `spring.factories`，面试官能理解。

---

## 第 3 步：条件注解过滤（核心中的核心）

如果所有配置类都生效，那会有成千上万个 Bean 被加载，显然不行。

Spring Boot 在每个自动配置类上，都加了**条件注解**，例如：

```
@ConditionalOnClass({ DataSource.class, EmbeddedDatabaseType.class })
@ConditionalOnMissingBean(type = "io.r2dbc.spi.ConnectionFactory")
public class DataSourceAutoConfiguration {
    // ...
}
```

常用条件注解：

|注解|含义|
|---|---|
|`@ConditionalOnClass`|classpath 中存在某个类才生效|
|`@ConditionalOnMissingBean`|容器中没有某个 Bean 才生效|
|`@ConditionalOnProperty`|配置文件中存在某个属性才生效|
|`@ConditionalOnWebApplication`|当前是 Web 环境才生效|

**举例**：  
你的项目中没有引入 `spring-jdbc` 依赖 → `DataSource.class` 不存在 → `DataSourceAutoConfiguration` 直接跳过，不加载。

这就是 **按需生效**。

---

## 第 4 步：最终生成 Bean

通过条件过滤后，剩下的自动配置类开始执行，内部通过 `@Bean` 等方法产生具体的 Bean 注入到 Spring 容器中。

而且大量配合：

@ConfigurationProperties(prefix = "spring.datasource")

把 `application.yml` 中的配置绑定到 Bean 上。

## 4. 如何禁用一个特定的自动配置类？
**要点**：使用 `@SpringBootApplication(exclude = {DataSourceAutoConfiguration.class})` 或 `@EnableAutoConfiguration` 的 exclude。

## 5. Spring Boot 的配置文件有哪些？区别？
**要点**：`application.properties` 和 `application.yml`。YAML 更结构化、支持列表；properties 更简单。

## 6. 如何修改 Spring Boot 的内嵌服务器端口？
**要点**：`server.port=8081` 配置在 `application.properties` 中。

## 7. Spring Boot 支持哪些内嵌服务器？
**要点**：Tomcat（默认）、Jetty、Undertow。可通过排除 tomcat 依赖并引入其他来实现。

## 8. 如何实现热部署？
**要点**：引入 `spring-boot-devtools`，支持自动重启。也可配合 IDEA 的自动编译。

## 9. Spring Boot Actuator 是什么？
**要点**：提供生产就绪功能，如健康检查、指标、信息端点。访问 `/actuator/health`、`/actuator/info`。

## 10. 如何定义一个自定义的 Starter？
**要点**：创建两个模块：
- autoconfigure 模块（配置类 + 条件注解）
- starter 模块（仅依赖 autoconfigure 和必要的核心库）

## 11. Spring Boot 中如何读取配置？
**要点**：
- `@Value("${my.key}")`
- `@ConfigurationProperties(prefix = "my")` 绑定到对象
- `Environment` 对象

## 12. `@SpringBootTest` 是用来做什么的？
**要点**：集成测试注解，启动完整的应用上下文进行测试。

## 13. Spring Boot 如何统一异常处理？
**要点**：
- `@ControllerAdvice` + `@ExceptionHandler`
- 或者实现 `ErrorController` / 继承 `BasicErrorController`

## 14. Spring Boot 如何切换不同环境的配置？
**要点**：多 profile 配置文件：
- `application-dev.yml`, `application-prod.yml`
- 启动参数：`--spring.profiles.active=dev`

## 15. Spring Boot 如何实现异步任务？
**要点**：
- 主类加 `@EnableAsync`
- 方法加 `@Async`
- 可配合 `ThreadPoolTaskExecutor`

---

## 给你一个偷分技巧（明天面试用）

如果面试官问你**不会**的题，不要说“我不会”，而是说：
> “这个我了解得不太深，但我用过类似的 XXX 场景，比如我们项目中用到了 YYY 来解决问题。”

另外，**自动配置原理**和 **`@SpringBootApplication`** 是最高频的，**一定要能讲 1-2 分钟**。

---

## 问题 1：Starter 是什么？

**一句话定义**：
> Starter 是 Spring Boot 的**一站式依赖描述符**，它把某个特定功能需要的**所有依赖 + 自动配置**打包在一起。

**举例**：
- 你想用 MySQL + JDBC → 引入 `spring-boot-starter-data-jdbc`
- 你想用 Web 开发 → 引入 `spring-boot-starter-web`

**不用 Starter 的麻烦**：你要自己找 `tomcat-embed`、`spring-webmvc`、`jackson` 等一堆依赖，还要自己配版本，还容易冲突。

---

## 问题 2：Starter 的核心原理（回答重点）

Starter 本质是一个 **空 Jar** 或 **轻量 Jar**，它不做业务，只做两件事：

| 角色 | 作用 |
|------|------|
| **依赖聚合** | 通过 Maven/Gradle 的 `pom.xml` 引入一批必需的依赖 |
| **引入自动配置** | 依赖对应的 `-autoconfigure` 模块（或内置自动配置） |

**典型结构**：
```
spring-boot-starter-xxx
├── pom.xml（引入 xxx-autoconfigure 及核心依赖）
└── (没有业务代码)
```

**流程图**：
```
你引入 spring-boot-starter-web
        ↓
它帮你引入 tomcat、spring-webmvc、jackson
        ↓
同时引入 spring-boot-autoconfigure（或内置）
        ↓
自动配置类判断 classpath 有这些类 → 生效
        ↓
你直接写 @Controller 就能用
```

---

## 问题 3：如何自定义一个 Starter？（高频追问）

面试官听完原理后，经常接着问：**“让你写一个 Starter，怎么做？”**

**标准 4 步**：

### 第 1 步：创建两个 Maven 模块
- `xxx-spring-boot-starter`（给用户引入的）
- `xxx-spring-boot-autoconfigure`（写配置逻辑的）

### 第 2 步：autoconfigure 模块中写配置类

```java
@Configuration
@ConditionalOnClass({MyService.class})  // 存在某个类才生效
@EnableConfigurationProperties(MyProperties.class)  // 绑定配置
public class MyAutoConfiguration {

    @Bean
    @ConditionalOnMissingBean  // 用户没自己定义才创建
    public MyService myService(MyProperties properties) {
        return new MyService(properties);
    }
}
```

### 第 3 步：绑定配置属性

```java
@ConfigurationProperties(prefix = "my.starter")
public class MyProperties {
    private String name = "default";
    private int timeout = 1000;
    // getter/setter
}
```

### 第 4 步：注册自动配置

在 `autoconfigure` 模块的 `META-INF/spring.factories` 中添加：
```properties
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
com.example.MyAutoConfiguration
```

> ⚠️ Spring Boot 2.7+ 可以用 `META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports`，每行一个配置类全名。

**最终用户使用**：
```xml
<dependency>
    <groupId>com.example</groupId>
    <artifactId>xxx-spring-boot-starter</artifactId>
</dependency>
```
然后在 `application.yml` 中配：
```yaml
my.starter:
  name: "custom"
  timeout: 5000
```

---

## 面试时讲这一句话总结（加分）

> "Starter 机制遵循**约定大于配置**的理念，让开发者只需要引入一个依赖，就能得到一个开箱即用的功能模块，底层结合自动配置和条件注解按需加载。"

---

## 对比记忆（防止混淆）

| 概念 | 作用 |
|------|------|
| **Starter** | 依赖聚合包，让用户引入更方便 |
| **自动配置** | 判断条件后生成 Bean 的逻辑 |
| **Conditional 注解** | 自动配置的开关，控制是否生效 |

**关系**：  
你引入 Starter → Starter 引入自动配置模块 → 自动配置 + 条件注解 → Bean 按需创建

---

## 最后给你一个模拟回答模板（背下来直接用）

> "Starter 是一种聚合依赖的机制。比如 `spring-boot-starter-web` 帮你引入了 Tomcat、Spring MVC 等，同时触发 `WebMvcAutoConfiguration` 自动配置。  
> 如果需要自定义 Starter，我会创建一个 `autoconfigure` 模块写 `@Configuration` 类，加上 `@ConditionalOnClass` 等注解，用 `@ConfigurationProperties` 绑定配置，最后在 `spring.factories` 中注册；再创建一个空的 `starter` 模块，只依赖那个 `autoconfigure` 模块。用户只需要依赖我的 Starter，配合配置文件就能直接使用。"

明天面试时，这个问题你可以讲 2-3 分钟，绝对让面试官满意。