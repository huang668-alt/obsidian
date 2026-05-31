## 一、Spring Boot 3-核心特性
### 第 1 章 SpringBoot3-快速入门
#### 1.1 简介
##### 1.1.1 前置知识
+ Java17
+ Spring、SpringMVC、MyBatis
+ Maven、IDEA

##### 1.1.2 环境要求
| **环境&工具** | **版本** |
| --- | --- |
| SpringBoot | 3.1.3+ |
| IDEA | 2022.3.3+ |
| Java | 17+ |
| Maven | 3.8.1+ |
| Tomcat | 10.1.12+ |
| Servlet | 5.0.0+ |
| GraalVM Community | 22.3+ |
| Native Build Tools | 0.9.19+ |


##### 1.1.3 SpringBoot 是什么
SpringBoot 帮我们简单、快速地创建一个独立的、生产级别的 **Spring 应用****（说明：SpringBoot 底层是 Spring）**。

大多数 SpringBoot 应用只需要编写少量配置即可快速整合 Spring 平台以及第三方技术。

**特性**：

+ `快速创建`独立 Spring 应用。
    - SSM：导包、配置、启动运行。
+ 直接`嵌入` Tomcat、Jetty 或 Undertow（无需部署 war 包）【Servlet 容器】。
    - Linux、Java、Tomcat、MySQL：war 放到 Tomcat 的 webapps 下。
    - jar、Java 环境：java -jar。
+ **重点**：提供可选的 `starter`，简化应用**整合**。
    - **场景启动器（starter）**：web、json、邮件、oss（对象存储）、异步、定时任务、缓存…
    - 导很多包，控制好版本。
    - 为每一种场景准备了一个依赖：**web-starter**、**mybatis-starter**。
+ **重点**：`按需自动配置` Spring 以及第三方库。
    - 如果这些场景要使用（生效）。这个场景的所有配置都会自动配置好。
    - **约定大于配置**：每个场景都有很多默认配置。
    - 自定义：配置文件中修改几项就可以。
+ 提供`生产级特性`：如监控指标、健康检查、外部化配置等。
    - 监控指标、健康检查（k8s）、外部化配置。
+ 无代码生成、`无 xml`。

总结：简化开发，简化配置，简化整合，简化部署，简化监控，简化运维。

#### 1.2 快速体验
> 场景：浏览器发送 /hello 请求，返回"Hello, Spring Boot 3!"
>

##### 1.2.1 开发流程
###### 1.2.1.1 创建项目
maven 项目

```xml
<!-- 所有 Spring Boot 项目都必须继承自 spring-boot-starter-parent -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>3.1.3</version>
</parent>
```

###### 1.2.1.2 导入场景
场景启动器

```xml
<dependencies>
  <!--web开发的场景启动-->
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
  </dependency>
</dependencies>
```

###### 1.2.1.3 主程序
``` java
@SpringBootApplication //这是⼀个SpringBoot应⽤
public class MainApplication {
    public static void main(String[] args) {
        SpringApplication.run(MainApplication.class,args);
    }
}
```

###### 1.2.1.4 业务
``` java
@RestController
public class HelloController {
    @GetMapping("/hello")
    public String hello(){
        return "Hello,Spring Boot 3!";
    }
}
```

###### 1.2.1.5 测试
默认启动访问：localhost:8080
###### 1.2.1.6 打包
``` xml
<build>
  <plugins>
    <!--    SpringBoot应⽤打包插件-->
    <plugin>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-maven-plugin</artifactId>
    </plugin>
  </plugins>
</build>
```

`mvn clean package` 把项目打成可执行的 jar 包。

`java -jar boot3-01-demo-1.0-SNAPSHOT.jar` 启动项目。

##### 1.2.2 特性小结
###### 1.2.2.1 简化整合
导入相关的场景，拥有相关的功能的场景启动器。

默认支持的所有场景：https://docs.spring.io/spring-boot/docs/current/reference/html/using.html#using.build-systems.starters

+ 官方提供的场景：命名为 `spring-boot-starter-*`。
+ 第三方提供场景：命名为 `*-spring-boot-starter`。

场景一导入，万物皆就绪。

###### 1.2.2.2 简化开发
无需编写任何配置，直接开发业务。

###### 1.2.2.3 简化配置
`application.properties`：

+ 集中式管理配置，只需要修改这个文件就行。
+ 配置基本都有默认值。
+ 能写的所有配置都在：https://docs.spring.io/spring-boot/docs/current/reference/html/application-properties.html#appendix.application-properties

###### 1.2.2.4 简化部署
打包为可执行的 jar 包。

Linux 服务器上有 Java 环境。

###### 1.2.2.5 简化运维
修改配置（外部放一个 application.properties 文件）、监控、健康检查…

###### 1.2.2.6 Spring Initializr 创建向导
一键创建好整个项目结构。

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1755192535644-35b666bc-bc91-4d08-b2e7-a2aee58999d6.png)

#### 1.3 应用分析
##### 1.3.1 依赖管理机制
思考：

1、为什么导入 `starter-web `所有相关依赖都导入进来？

+ 开发什么场景，导入什么**场景启动器**。
+ **maven 依赖传递原则**。**A-B-C：A 就拥有 B 和 C**。
+ 导入场景启动器，场景启动器自动把这个场景的所有核心依赖全部导入进来。

2、为什么版本号都不用写？

+ 每个 boot 项目都有一个父项目 `spring-boot-starter-parent`。
+ parent 的父项目是 `spring-boot-dependencies`。
+ 父项目**版本仲裁中心**，把所有常见的 jar 的依赖版本都声明好了。
+ 比如：`mysql-connector-j`。

3、自定义版本号。

+ 利用 maven 的就近原则。
    - 直接在当前项目 `properties` 标签中声明父项目用的版本属性的 key。
    - 直接在**导入依赖的时候声明版本**。

4、第三方的 jar 包。

+ boot 父项目没有管理的需要自行声明好。

``` xml
<!-- https://mvnrepository.com/artifact/com.alibaba/druid -->
<dependency>
  <groupId>com.alibaba</groupId>
  <artifactId>druid</artifactId>
  <version>1.2.16</version>
</dependency>
```

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1755192534750-eb81356f-f0bb-47df-85ea-e37c61c86347.png)

##### 1.3.2 自动配置机制
###### 1.3.2.1 初步理解
+ **自动配置**的 Tomcat、SpringMVC 等。
    - **导入场景**，容器中就会自动配置好这个场景的核心组件。
    - 以前：DispatcherServlet、ViewResolver、CharacterEncodingFilter…
    - 现在：自动配置好的这些组件。
    - 验证：**容器中有了什么组件，就具有什么功能**。

```java
@SpringBootApplication
public class MainApplication{
    public static void main(String[] args){
        局部变量类型的⾃动推断
        var ioc = SpringApplication.run(MainApplication.class, args);
        //1,获取容器中所有组件的名字
        String[] beanNames = ioc.getBeanDefinitionNames();
        //2,挨个遍历：
        // dispatcherServlet,beanNameViewResolver,characterEncodingFilte,multipartResolver
        // SpringBoot
        //把以前配置的核⼼组件现在都给我们⾃动配置好了
        for (String beanName : beanNames){
            System.out.println("beanName = " + beanName);
        }
    }
}
```

+ **默认的包扫描规则**。
    - `@SpringBootApplication` 标注的类就是主程序类。
    - **SpringBoot 只会扫描主程序所在的包及其下面的子包，自动的 component-scan 功能**。
    - **自定义扫描路径**。
        * @SpringBootApplication(scanBasePackages = “com.myxh.springboot”)
        * `@ComponentScan("com.myxh.springboot")` 直接指定扫描的路径。
+ **配置默认值**。
    - **配置文件**的所有配置项是和某个**类的对象**值进行一一绑定的。
    - 绑定了配置文件中每一项值的类：**属性类**。
    - 比如：
        * `ServerProperties` 绑定了所有 Tomcat 服务器有关的配置。
        * `MultipartProperties` 绑定了所有文件上传相关的配置。
        * 参照官方文档 https://docs.spring.io/spring-boot/docs/current/reference/html/application-properties.html#appendix.application-properties.server， 或者参照绑定的**属性类**。
+ 按需加载自动配置。
    - 导入场景 `spring-boot-starter-web`。
    - 场景启动器除了会导入相关功能依赖，导入一个 `spring-boot-starter`，是所有 `starter` 的 `starter`，基础核心 starter。
    - `spring-boot-starter` 导入了一个包 `spring-boot-autoconfigure`。包里面都是各种场景的 `AutoConfiguration`**自动配置类**。
    - 虽然全场景的自动配置都在 `spring-boot-autoconfigure` 这个包，但是不是全都开启的。
        * 导入哪个场景就开启哪个自动配置。

总结：导入场景启动器、触发 `spring-boot-autoconfigure` 这个包的自动配置生效、容器中就会具有相关场景的功能。

###### 1.3.2.2 完整流程
> 思考：
>
> 1、**SpringBoot 怎么实现导一个 starter、写一些简单配置，应用就能跑起来，无需关心整合**。
>
> 2、为什么 Tomcat 的端口号可以配置在 **application.properties** 中，并且 **Tomcat** 能启动成功？
>
> 3、导入场景后哪些**自动配置能生效**？
>

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1755192534694-80a88708-5985-4752-9bcc-32e39127d68d.png)

**自动配置流程细节梳理**：

**1**、导入 `starter-web`：导入了 web 开发场景。

+ 1、场景启动器导入了相关场景的所有依赖：`starter-json`、`starter-tomcat`、`springmvc`。
+ 2、每个场景启动器都引入了一个 `spring-boot-starter`，核心场景启动器。
+ 3、**核心场景启动器**引入了 `spring-boot-autoconfigure` 包。
+ 4、`spring-boot-autoconfigure` 里面囊括了所有场景的所有配置。
+ 5、只要这个包下的所有类都能生效，那么相当于 SpringBoot 官方写好的整合功能就生效了。
+ 6、SpringBoot 默认却扫描不到 `spring-boot-autoconfigure` 下写好的所有**配置类**。（这些**配置类**做了整合操作），**默认只扫描主程序所在的包**。

**2**、**主程序**：`@SpringBootApplication`。

+ 1、`@SpringBootApplication` 由三个注解组成`@SpringBootConfiguration`、`@EnableAutoConfiguration`、`@ComponentScan`。
+ 2、SpringBoot 默认只能扫描自己主程序所在的包及其下面的子包，扫描不到 `spring-boot-autoconfigure` 包中官方写好的**配置类**。
+ 3、`@EnableAutoConfiguration`：SpringBoot **开启自动配置的核心**。
    - ① 是由 `@Import(AutoConfigurationImportSelector.class)` 提供功能：批量给容器中导入组件。
    - ② SpringBoot 启动会默认加载 146 个配置类。
    - ③ 这 **146 个配置类**来自于 `spring-boot-autoconfigure` 下 `META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports` 文件指定的。
    - ④ 项目启动的时候利用 @Import 批量导入组件机制把 `autoconfigure` 包下的 146 `xxxAutoConfiguration` 类导入进来（**自动配置类**）。
+ 4、按需生效：
    - 虽然导入了 `146` 个自动配置，并不是这 `146` 个自动配置类都能生效。
    - 每一个自动配置类，都有条件注解 `@ConditionalOnXxx`，只有条件成立，才能生效。

**3**、`xxxAutoConfiguration`**自动配置类**。

+ 1、**给容器中使用 @Bean 放一堆组件**。
+ 2、每个**自动配置类**都可能有这个注解 `@EnableConfigurationProperties(ServerProperties.class)`，用来把配置文件中配的指定前缀的属性值封装到 `xxxProperties`**属性类**中。
+ 3、以 Tomcat 为例：把服务器的所有配置都是以 `server` 开头的。配置都封装到了属性类中。
+ 4、给**容器**中放的所有**组件**的一些**核心参数**，都来自于 `xxxProperties`。`xxxProperties`**都是和配置文件绑定**。
+ 5、**只需要改配置文件的值，核心组件的底层参数都能修改**。

**4**、写业务，全程无需关心各种整合（底层这些整合写好了，而且也生效了）。

**核心流程总结**：

1、导入 `starter`，就会导入 `autoconfigure` 包。

2、`autoconfigure` 包里面 有一个文件 `META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports`,里面指定的所有启动要加载的自动配置类。

3、@EnableAutoConfiguration 会自动的把上面文件里面写的所有**自动配置类都导入进来**。**xxxAutoConfiguration 是有条件注解进行按需加载**。

4、`xxxAutoConfiguration` 给容器中导入一堆组件，组件都是从 `xxxProperties` 中提取属性值。

5、`xxxProperties` 又是和**配置文件**进行了绑定。

**效果**：导入 `starter`、修改配置文件，就能修改底层行为。

###### 1.3.2.3 如何学好 SpringBoot
框架的框架、底层基于 Spring。能调整每一个场景的底层行为。100%项目一定会用到**底层自定义**。

摄影：

+ 傻瓜：自动配置好。
+ **单反**：焦距、光圈、快门、感光度…
+ 傻瓜+**单反**：

1、理解**自动配置原理**。

+ ① **导入 starter** -> **生效 xxxAutoConfiguration** -> **组件** -> **xxxProperties** -> **配置文件**。

2、理解**其他框架底层**。

+ ① 拦截器。

3、可以随时**定制化任何组件**。

+ ① **配置文件**。
+ ② **自定义组件**。

普通开发：`导入 starter`，Controller、Service、Mapper、偶尔修改配置文件。

**高级开发**：自定义组件、自定义配置、自定义 starter。

核心：

+ 这个场景自动配置导入了哪些组件，能不能 Autowired 进来使用。
+ 能不能通过修改配置改变组件的一些默认参数。
+ 需不需要自己完全定义这个组件。
+ **场景定制化**。

**最佳实战**：

+ **选场景**，导入到项目。
    - 官方：starter。
    - 第三方：去仓库搜。
+ **写配置，改配置文件关键项**。
    - 数据库参数（连接地址、账号密码…）。
+ 分析这个场景导入了**哪些能用的组件**。
    - **自动装配**这些组件进行后续使用。
    - 不满意 SprngBoot 提供的自动配好的默认组件。
        * **定制化**。
        * 改配置。
        * 自定义组件。

整合 redis：

+ **选场景**：`spring-boot-starter-data-redis`。
    - 场景 AutoConfiguration 就是这个场景的自动配置类。
+ 写配置：
    - 分析到这个场景的自动配置类开启了哪些属性绑定关系。
    - `@EnableConfigurationProperties(RedisProperties.class)`。
    - 修改 redis 相关的配置。
+ 分析组件：
    - 分析到 `RedisAutoConfiguration` 给容器中放了 `StringRedisTemplate`。
    - 给业务代码中自动装配 `StringRedisTemplate`。
+ 定制化：
    - 修改配置文件。
    - 自定义组件，自己给容器中放一个 `StringRedisTemplate`。

#### 1.4 核心技能
##### 1.4.1 常用注解
> SpringBoot 摒弃 XML 配置方式，改为**全注解驱动**。
>

###### 1.4.1.1 组件注册
**@Configuration**、**@SpringBootConfiguration**

**@Bean**、**@Scope**

**@Controller**、 **@Service**、**@Repository**、**@Component**

**@Import**

**@ComponentScan**

步骤：

**1**、**@Configuration 编写一个配置类**。

**2**、**在配置类中，自定义方法给容器中注册组件。配合@Bean**。

**3**、**或使用@Import 导入第三方的组件**。

###### 1.4.1.2 条件注解
> 如果注解指定的**条件成立**，则触发指定行为。
>

**@ConditionalOnXxx**

**@ConditionalOnClass：如果类路径中存在这个类，则触发指定行为**。

**@ConditionalOnMissingClass：如果类路径中不存在这个类，则触发指定行为**。

**@ConditionalOnBean：如果容器中存在这个 Bean（组件），则触发指定行为**。

**@ConditionalOnMissingBean：如果容器中不存在这个 Bean（组件），则触发指定行为**。

> 场景：
>
> + 如果存在 **FastsqlException** 这个类，给容器中放一个 **Cat** 组件，命名 cat1。
> + 否则，就给容器中放一个 **Dog** 组件，命名 dog1。
> + 如果系统中有 **dog1** 这个组件，就给容器中放一个 User 组件，名 zhangsan。
> + 否则，就放一个 User，名叫 lisi。
>

**@ConditionalOnBean（value=组件类型，name=组件名字）：判断容器中是否有这个类型的组件，并且名字是指定的值**。

@ConditionalOnRepositoryType (org.springframework.boot.autoconfigure.data)

@ConditionalOnDefaultWebSecurity (org.springframework.boot.autoconfigure.security)

@ConditionalOnSingleCandidate (org.springframework.boot.autoconfigure.condition)

@ConditionalOnWebApplication (org.springframework.boot.autoconfigure.condition)

@ConditionalOnWarDeployment (org.springframework.boot.autoconfigure.condition)

@ConditionalOnJndi (org.springframework.boot.autoconfigure.condition)

@ConditionalOnResource (org.springframework.boot.autoconfigure.condition)

@ConditionalOnExpression (org.springframework.boot.autoconfigure.condition)

**@ConditionalOnClass (org.springframework.boot.autoconfigure.condition)**

@ConditionalOnEnabledResourceChain (org.springframework.boot.autoconfigure.web)

**@ConditionalOnMissingClass (org.springframework.boot.autoconfigure.condition)**

@ConditionalOnNotWebApplication (org.springframework.boot.autoconfigure.condition)

@ConditionalOnProperty (org.springframework.boot.autoconfigure.condition)

@ConditionalOnCloudPlatform (org.springframework.boot.autoconfigure.condition)

**@ConditionalOnBean (org.springframework.boot.autoconfigure.condition)**

**@ConditionalOnMissingBean (org.springframework.boot.autoconfigure.condition)**

@ConditionalOnMissingFilterBean (org.springframework.boot.autoconfigure.web.servlet)

@Profile (org.springframework.context.annotation)

@ConditionalOnInitializedRestarter (org.springframework.boot.devtools.restart)

@ConditionalOnGraphQlSchema (org.springframework.boot.autoconfigure.graphql)

@ConditionalOnJava (org.springframework.boot.autoconfigure.condition)

###### 1.4.1.3 属性绑定
**@ConfigurationProperties：声明组件的属性和配置文件哪些前缀开始项进行绑定**。

**@EnableConfigurationProperties：快速注册注解**：

+ **场景**：SpringBoot 默认只扫描自己主程序所在的包。如果导入第三方包，即使组件上标注了 @Component、@ConfigurationProperties 注解也没用。因为组件都扫描不进来，此时使用这个注解就可以快速进行属性绑定并把组件注册进容器。

> 将容器中任意**组件（Bean）的属性值**和**配置文件**的配置项的值**进行绑定**。
>
> 1、**给容器中注册组件（@Component、@Bean）**。
>
> 2、**使用 @ConfigurationProperties 声明组件和配置文件的哪些配置项进行绑定**。
>

##### 1.4.2 YAML 配置文件
> **痛点**：SpringBoot 集中化管理配置，**application.properties**。
>
> **问题**：配置多以后难阅读和修改，**层级结构辨识度不高**。
>

> YAML 是 “YAML Ain’t a Markup Language”（YAML 不是一种标记语言）。在开发的这种语言时，YAML 的意思其实是：“Yet Another Markup Language”（是另一种标记语言）。
>
> + 设计目标，就是**方便人类读写**。
> + **层次分明**，更适合做配置文件。
>
> 使用 **.yaml** 或 **.yml** 作为文件后缀。
>

###### 1.4.2.1. 基本语法
+ **大小写敏感**。
+ 使用**缩进表示层级关系**，**k: v，使用空格分割 k, v**。
+ 缩进时不允许使用 Tab 键，只允许**使用空格**。
+ 缩进的空格数目不重要，只要**相同层级**的元素**左侧对齐**即可。
+ **# 表示注释**，从这个字符一直到行尾，都会被解析器忽略。

支持的写法：

+ **对象**：**键值对**的集合，例如：映射（map）、 哈希（hash）、 字典（dictionary）。
+ **数组**：一组按次序排列的值，例如：序列（sequence）、 列表（list）。
+ **纯量**：单个的、不可再分的值，例如：字符串、数字、bool、日期。

###### 1.4.2.2 示例
```java
@Component
//和配置⽂件person前缀的所有配置进⾏绑定
@ConfigurationProperties(prefix = "person")
//@NoArgsConstructor //⾃动⽣成⽆参构造器
//@AllArgsConstructor //⾃动⽣成全参构造器

//⾃动⽣成JavaBean属性的getter/setter方法
@Data
public class Person{
    private String name;
    private Integer age;
    private Date birthDay;
    private Boolean like;
    //嵌套对象
    private Child child;
    //数组（里面是对象）
    private List<Dog> dogs;
    //表示Map
    private Map<String, Cat> cats;
}
```

```java
@Component
//@NoArgsConstructor
//@AllArgsConstructor
@Data
public class Child{
    private String name;
    private Integer age;
    private Date birthDay;
    //数组
    private List<String> text;
}
```

```java
@Component
//@NoArgsConstructor
//@AllArgsConstructor
@Data
public class Dog{
    private String name;
    private Integer age;
}
```

```java
@Component
//@NoArgsConstructor
//@AllArgsConstructor
@Data
public class Cat
{
    private String name;
    private Integer age;
}
```

properties 表示法。

```java
# properties 表示复杂对象
person.name=张三
person.age=35
person.birthDay=1988/01/01 00:00:00
person.like=true
person.child.name=李四
person.child.age=12
person.child.birthDay=2011/01/01
person.child.text[0]=hello
person.child.text[1]=world
person.dogs[0].name=小黑
person.dogs[0].age=2
person.dogs[1].name=小白
person.dogs[1].age=1
person.cats.cat1.name=小蓝
person.cats.cat1.age=2
person.cats.cat2.name=小灰
person.cats.cat2.age=1
```

yaml 表示法。

```java
# 认为下面是一个文档
# 
---
person:
  name: 张三
  age: 35
  birth-day: 1988/01/01 00:00:00
  like: true

  child:
    name: 李四
    age: 12
    birth-day: 2011/01/01
    
    text:
      - "he\nllo"
      - 'wor\nld'
      - | # | 开头，大文本写在下层，保留文本格式，换行符正确显示。
        cats:
          cat1:
            name: 小蓝
            age: 2       
          cat2: {name: 小灰,age: 1}
      - > # > 开头，大文本写在下层，折叠换行符。
        cats:
        cat1:
        name: 小蓝
        age: 2        
        cat2: {name: 小灰,age: 1}
  dogs:
    - name: 小黑
      age: 2
    - name: 小白
      age: 1
  cats:
    cat1:
      name: 小蓝
      age: 2
    cat2: { name: 小灰, age: 1 }
```

###### 1.4.2.3 细节
+ birthDay 推荐写为 birth-day。
+ **文本**：
    - **单引号**不会转义【\n 则为普通字符串显示】。
    - **双引号**会转义【\n 会显示为**换行符**】。
+ **大文本**：
    - `|` 开头，大文本写在下层，**保留文本格式，换行符正确显示**。
    - `>` 开头，大文本写在下层，折叠换行符。

**多文档合并**：

+ 使用`---`可以把多个 yaml 文档合并在一个文档中，每个文档区依然认为内容独立。

###### 1.4.2.4. 小技巧：lombok
> 简化 JavaBean 开发。自动生成构造器、getter/setter、自动生成 Builder 模式等。
>

```java
<dependency>
  <groupId>org.projectlombok</groupId>
  <artifactId>lombok</artifactId>
  <scope>compile</scope>
</dependency>
```

使用 `@Data`等注解。

##### 1.4.3 日志配置
规范：项目开发不要编写 `System.out.println()`，应该用**日志**记录信息。

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1755192534756-d020834c-604c-4503-b9e3-c8d5a4721557.png)

###### 1.4.3.1 简介
1、Spring 使用 `commons-logging` 作为内部日志，但底层日志实现是开放的。可对接其他日志框架。

+ ① spring5 及以后 commons-logging 被 spring 直接自己实现了。

2、支持 `jul`，`log4j2`，`logback`。SpringBoot 提供了默认的控制台输出配置，也可以配置输出为文件。

3、`logback` 是默认使用的。

4、虽然**日志框架很多**，但是不用担心，使用 SpringBoot 的**默认配置就能工作的很好**。

**SpringBoot 怎么把日志默认配置好的**。

1、每个 `starter` 场景，都会导入一个核心场景 `spring-boot-starter`。

2、核心场景引入了日志的所用功能 `spring-boot-starter-logging`。

3、默认使用了 `logback + slf4j` 组合作为默认底层日志。

4、`日志是系统一启动就要用`，`xxxAutoConfiguration` 是系统启动好了以后放好的组件，后来用的。

5、日志是利用**监听器机制**配置好的。`ApplicationListener`。

6、日志所有的配置都可以通过修改配置文件实现。以 `logging` 开始的所有配置。

###### 1.4.3.2 日志格式
```java
2023-09-14 20:24:43.709  INFO 96528 --- [main] o.s.b.w.e.t.TomcatWebServer              : Tomcat initialized with port(s): 8080 (http)
2023-09-14 20:24:43.712  INFO 96528 --- [main] o.a.c.c.AprLifecycleListener             : Loaded Apache Tomcat Native library [2.0.5] using APR version [1.7.4].
```

默认输出格式：

+ 时间和日期：毫秒级精度。
+ 日志级别：`ERROR`, `WARN`, `INFO`, `DEBUG`, `TRACE`。
+ 进程 ID。
+ ---：消息分割符。
+ 线程名：使用[]包含。
+ Logger 名：通常是产生日志的**类名**。
+ 消息：日志记录的内容。

注意：logback 没有 `FATAL` 级别，对应的是 `ERROR`。

默认值：参照：`spring-boot` 包 `additional-spring-configuration-metadata.json` 文件。

默认输出格式值：`%clr(%d{${LOG_DATEFORMAT_PATTERN:-yyyy-MM-dd'T'HH:mm:ss.SSSXXX}}){faint} %clr(${LOG_LEVEL_PATTERN:-%5p}) %clr(${PID:- }){magenta} %clr(---){faint} %clr([%15.15t]){faint} %clr(%-40.40logger{39}){cyan} %clr(:){faint} %m%n${LOG_EXCEPTION_CONVERSION_WORD:-%wEx}`。

可修改为：`%d{yyyy-MM-dd HH:mm:ss.SSS} %-5level [%thread] %logger{15} ===> %msg%n`。

###### 1.4.3.3 记录日志
```java
Logger logger = LoggerFactory.getLogger(getClass());
```

或者使用 Lombok 的@Slf4j 注解。

###### 1.4.3.4 日志级别
+ 由低到高：`ALL, TRACE, DEBUG, INFO, WARN, ERROR, FATAL, OFF`。
    - **只会打印指定级别及以上级别的日志**。
    - ALL：打印所有日志。
    - TRACE：追踪框架详细流程日志，一般不使用。
    - DEBUG：开发调试细节日志。
    - INFO：关键、感兴趣信息日志。
    - WARN：警告但不是错误的信息日志，比如：版本过时。
    - ERROR：业务错误日志，比如出现各种异常。
    - FATAL：致命错误日志，比如 jvm 系统崩溃。
    - OFF：关闭所有日志记录。
+ 不指定级别的所有类，都使用 root 指定的级别作为默认级别。
+ SpringBoot 日志**默认级别是 INFO**。

1、在 `application.properties/yaml` 中配置 `logging.level.<logger-name>=<level>`指定日志级别。

2、`level` 可取值范围：`TRACE, DEBUG, INFO, WARN, ERROR, FATAL, OFF`，定义在 `LogLevel` 类中。

3、root 的 `logger-name` 叫 `root`，可以配置 `logging.level.root=warn`，代表所有未指定日志级别都使用 root 的 warn 级别。

###### 1.4.3.5 日志分组
比较有用的技巧是：

将相关的 `logger` 分组在一起，统一配置。SpringBoot 也支持。比如：Tomcat 相关的日志统一设置。

```java
logging.group.tomcat=org.apache.catalina,org.apache.coyote,org.apache.tomcat
logging.level.tomcat=trace
```

SpringBoot 预定义两个组。

| Name | Loggers |
| --- | --- |
| web | `org.springframework.core.codec`<br/>, `org.springframework.http`<br/>, `org.springframework.web`<br/>, `org.springframework.boot.actuate.endpoint.web`<br/>, `org.springframework.boot.web.servlet.ServletContextInitializerBeans` |
| sql | `org.springframework.jdbc.core`<br/>, `org.hibernate.SQL`<br/>, `org.jooq.tools.LoggerListener` |


###### 1.4.3.6 文件输出
SpringBoot 默认只把日志写在控制台，如果想额外记录到文件，可以在 `application.properties` 中添加 `logging.file.name` 或 `logging.file.path` 配置项。

| `logging.file.name` | `logging.file.path` | 示例 | 效果 |
| --- | --- | --- | --- |
| 未指定 | 未指定 | 无 | 仅控制台输出。 |
| **指定** | 未指定 | `my.log` | 写入指定文件。可以`加路径`<br/>。 |
| 未指定 | **指定** | `./log` | 写入指定目录，文件名为 `spring.log`<br/>。 |
| **指定** | **指定** | 无 | 以 `logging.file.name`<br/> 为准。 |


###### 1.4.3.7 文件归档与滚动切割
> 归档：每天的日志单独存到一个文档中。
>
> 切割：每个文件 10MB，超过大小切割成另外一个文件。
>

1、每天的日志应该独立分割出来存档。如果使用 `logback`（SpringBoot 默认整合），可以通过 `application.properties/yaml` 文件指定日志滚动规则。

2、如果是其他日志系统，需要自行配置（添加 `log4j2.xml` 或 `log4j2-spring.xml`）。

3、支持的滚动规则设置如下。

| 配置项 | 描述 |
| --- | --- |
| `logging.logback.rollingpolicy.file-name-pattern` | 日志存档的文件名格式（默认值：`${LOG_FILE}.%d{yyyy-MM-dd}.%i.gz`<br/>）。 |
| `logging.logback.rollingpolicy.clean-history-on-start` | 应用启动时是否清除以前存档（默认值：`false`<br/>）。 |
| `logging.logback.rollingpolicy.max-file-size` | 存档前，每个日志文件的最大大小（默认值：`10MB`<br/>）。 |
| `logging.logback.rollingpolicy.total-size-cap` | 日志文件被删除之前，可以容纳的最大大小（默认值：`0B`<br/>）。设置 `1GB`<br/> 则磁盘存储超过 1GB 日志后就会删除旧日志文件。 |
| `logging.logback.rollingpolicy.max-history` | 日志文件保存的最大天数(默认值：`7`<br/>)。 |


###### 1.4.3.8 自定义配置
通常配置 `application.properties` 就够了。当然也可以自定义。比如：

| 日志系统 | 自定义 |
| --- | --- |
| Logback | `logback-spring.xml`<br/>, `logback-spring.groovy`<br/>, `logback.xml`<br/>, `logback.groovy` |
| Log4j2 | `log4j2-spring.xml or log4j2.xml` |
| JDK (Java Util Logging) | `logging.properties` |


如果可能，建议在日志配置中使用 `-spring` 变量（例如，`logback-spring.xml` 而不是 `logback.xml`）。如果使用标准配置文件，spring 无法完全控制日志初始化。

最佳实战：自己要写配置，配置文件名加上 `xxx-spring.xml`。

###### 1.4.3.9 切换日志组合
```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-web</artifactId>
</dependency>

<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter</artifactId>
  <exclusions>
    <exclusion>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-logging</artifactId>
    </exclusion>
  </exclusions>
</dependency>

<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-log4j2</artifactId>
</dependency>
```

log4j2 支持 yaml 和 json 格式的配置文件。

| 格式 | 依赖 | 文件名 |
| --- | --- | --- |
| YAML | com.fasterxml.jackson.core:jackson-databind、com.fasterxml.jackson.dataformat:jackson-dataformat-yaml | log4j2.yaml 或 log4j2.yml |
| JSON | com.fasterxml.jackson.core:jackson-databind | log4j2.json 或 log4j2.jsn |


###### 1.4.3.10 最佳实战
1、导入任何第三方框架，先排除它的日志包，因为 Boot 底层控制好了日志。

2、修改 `application.properties` 配置文件，就可以调整日志的所有行为。如果不够，可以编写日志框架自己的配置文件放在类路径下就行，比如 `logback-spring.xml`，`log4j2-spring.xml`。

3、如需对接**专业日志系统**，也只需要把 logback 记录的**日志**灌倒 **kafka** 之类的中间件，这和 SpringBoot 没关系，都是日志框架自己的配置，**修改配置文件即可**。

4、**业务中使用 slf4j-api 记录日志，不要再用 System.out.println() 了**

****

****

****

****

### 第 2 章 SpringBoot3-Web 开发
> SpringBoot 的 Web 开发能力，由 **SpringMVC** 提供。
>

#### 2.1 WebMvcAutoConfiguration 原理
##### 2.1.1 生效条件
```java
// 在这些自动配置之后
@AutoConfiguration(after = { DispatcherServletAutoConfiguration.class,
                             TaskExecutionAutoConfiguration.class,
                             ValidationAutoConfiguration.class })
// 如果是 Web 应用就生效，类型有 SERVLET、REACTIVE（响应式 Web）
@ConditionalOnWebApplication(type = Type.SERVLET)
@ConditionalOnClass({ Servlet.class, DispatcherServlet.class, WebMvcConfigurer.class })
// 容器中没有这个 Bean，才生效，默认就是没有
@ConditionalOnMissingBean(WebMvcConfigurationSupport.class)
// 优先级
@AutoConfigureOrder(Ordered.HIGHEST_PRECEDENCE + 10)
@ImportRuntimeHints(WebResourcesRuntimeHints.class)
public class WebMvcAutoConfiguration{
}
```

##### 2.1.2 效果
1、放了两个 Filter：

+ ① `**HiddenHttpMethodFilter**`：页面表单提交 Rest 请求（GET、POST、PUT、DELETE）。
+ ② `**FormContentFilter**`：表单内容 Filter，GET（数据放 URL 后面）、POST（数据放请求体）请求可以携带数据，PUT、DELETE 的请求体数据会被忽略。

2、给容器中放了 `WebMvcConfigurer` 组件：给 SpringMVC 添加各种定制功能。

+ ① 所有的功能最终会和配置文件进行绑定。
+ ② WebMvcProperties：`spring.mvc` 配置文件。
+ ③ WebProperties：`spring.web` 配置文件。

```java
@Configuration(proxyBeanMethods = false)
// 额外导入了其他配置
@Import(EnableWebMvcConfiguration.class)
@EnableConfigurationProperties({ WebMvcProperties.class, WebProperties.class })
@Order(0)
public static class WebMvcAutoConfigurationAdapter implements WebMvcConfigurer, 
                                                              ServletContextAware{

}
```

##### 2.1.3 WebMvcConfigurer 接口
提供了配置 SpringMVC 底层的所有组件入口。

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1755192535194-9c7cc0d4-14b8-421e-ac06-212e24969554.png)

##### 2.1.4 静态资源规则源码
```java
@Override
public void addResourceHandlers(ResourceHandlerRegistry registry){
if (!this.resourceProperties.isAddMappings()){
    logger.debug("Default resource handling disabled");

    return;
}
addResourceHandler(registry, this.mvcProperties.getWebjarsPathPattern(),
                   "classpath:/META-INF/resources/webjars/");
addResourceHandler(registry, this.mvcProperties.getStaticPathPattern(),
                   (registration) -> {
                       registration.addResourceLocations(this.resourceProperties.getStaticLocations());
                       if (this.servletContext != null)
                       {
                           ServletContextResource resource = new ServletContextResource(this.servletContext, SERVLET_LOCATION);
                           registration.addResourceLocations(resource);
                       }
                   });
}
```

1、规则一：访问 `/webjars/**` 路径就去 `classpath:/META-INF/resources/webjars/` 下找资源。

+ ① Maven 导入依赖。

2、规则二：访问 `/**` 路径就去`静态资源默认的四个位置找资源`。

+ ① `classpath:/META-INF/resources/`
+ ② `classpath:/resources/`
+ ③ `classpath:/static/`
+ ④ `classpath:/public/`

3、规则三：**静态资源默认都有缓存规则的设置**。

+ ① 所有缓存的设置，直接通过**配置文件**：`spring.web`。
+ ② cachePeriod：缓存周期，多久不用找服务器要新的，默认没有缓存周期，以秒为单位。
+ ③ cacheControl：**HTTP 缓存控制**，https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Caching 。
+ ④ **useLastModified**：是否使用最后一次修改，配合 HTTP Cache 规则。

> 如果浏览器访问了一个静态资源 **index.js**，如果服务这个资源没有发生变化，下次访问的时候就可以直接让浏览器用自己缓存中的东西，而不用给服务器发请求。
>

```java
registration.setCachePeriod(getSeconds(this.resourceProperties.getCache().getPeriod()));
registration.setCacheControl(this.resourceProperties.getCache().getCachecontrol().toHttpCacheControl());
registration.setUseLastModified(this.resourceProperties.getCache().isUseLastModified());
```

##### 2.1.5 EnableWebMvcConfiguration 源码
```java
/*
SpringBoot 给容器中放 WebMvcConfigurationSupport 组件
如果自己放了 WebMvcConfigurationSupport 组件，SpringBoot 的 WebMvcAutoConfiguration 都会失效
*/
@Configuration(proxyBeanMethods = false)
@EnableConfigurationProperties(WebProperties.class)
public static class EnableWebMvcConfiguration extends DelegatingWebMvcConfiguration
                                              implements ResourceLoaderAware{
}
```

1、`HandlerMapping`：根据请求路径 `/xxx` 找那个 handler 能处理请求。

+ ① `WelcomePageHandlerMapping`：
    - (1) 访问 `/\*\*` 路径下的所有请求，都在以前四个静态资源路径下找，欢迎页也一样。
    - (2) 找 `index.html`：只要静态资源的位置有一个 `index.html` 页面，项目启动默认访问。

##### 2.1.6 为什么容器中放一个 WebMvcConfigurer 就能配置底层行为
1、WebMvcAutoConfiguration 是一个自动配置类，它里面有一个 `EnableWebMvcConfiguration`。

2、`EnableWebMvcConfiguration` 继承于 `DelegatingWebMvcConfiguration`，这两个都生效。

3、`DelegatingWebMvcConfiguration` 利用依赖注入把容器中所有 `WebMvcConfigurer` 注入进来。

4、别人调用 `DelegatingWebMvcConfiguration` 的方法配置底层规则，而它调用所有 `WebMvcConfigurer` 的配置底层方法。

##### 2.1.7 WebMvcConfigurationSupport
提供了很多的默认设置。

判断系统中是否有相应的类：如果有，就加入相应的 `HttpMessageConverter`

```java
jackson2Present = ClassUtils.isPresent("com.fasterxml.jackson.databind.ObjectMapper", classLoader) &&
  ClassUtils.isPresent("com.fasterxml.jackson.core.JsonGenerator", classLoader);
jackson2XmlPresent = ClassUtils.isPresent("com.fasterxml.jackson.dataformat.xml.XmlMapper", classLoader);
jackson2SmilePresent = ClassUtils.isPresent("com.fasterxml.jackson.dataformat.smile.SmileFactory", classLoader);
```

#### 2.2 Web 场景
##### 2.2.1 自动配置
1、整合 web 场景。

```java
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

2、引入了 `autoconfigure` 功能。

3、`@EnableAutoConfiguration` 注解使用 `@Import(AutoConfigurationImportSelector.class)` 批量导入组件。

4、加载 `META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports` 文件中配置的所有组件。

5、所有自动配置类如下。

```java
org.springframework.boot.autoconfigure.web.client.RestTemplateAutoConfiguration
org.springframework.boot.autoconfigure.web.embedded.EmbeddedWebServerFactoryCustomizerAutoConfiguration
// ==============以下是响应式 Web 场景==============
org.springframework.boot.autoconfigure.web.reactive.HttpHandlerAutoConfiguration
org.springframework.boot.autoconfigure.web.reactive.ReactiveMultipartAutoConfiguration
org.springframework.boot.autoconfigure.web.reactive.ReactiveWebServerFactoryAutoConfiguration
org.springframework.boot.autoconfigure.web.reactive.WebFluxAutoConfiguration
org.springframework.boot.autoconfigure.web.reactive.WebSessionIdResolverAutoConfiguration
org.springframework.boot.autoconfigure.web.reactive.error.ErrorWebFluxAutoConfiguration
org.springframework.boot.autoconfigure.web.reactive.function.client.ClientHttpConnectorAutoConfiguration
org.springframework.boot.autoconfigure.web.reactive.function.client.WebClientAutoConfiguration
// ===============================================
org.springframework.boot.autoconfigure.web.servlet.DispatcherServletAutoConfiguration
org.springframework.boot.autoconfigure.web.servlet.ServletWebServerFactoryAutoConfiguration
org.springframework.boot.autoconfigure.web.servlet.error.ErrorMvcAutoConfiguration
org.springframework.boot.autoconfigure.web.servlet.HttpEncodingAutoConfiguration
org.springframework.boot.autoconfigure.web.servlet.MultipartAutoConfiguration
org.springframework.boot.autoconfigure.web.servlet.WebMvcAutoConfiguration
```

6、绑定了配置文件的一堆配置项。

+ ① SpringMVC 的所有配置 `spring.mvc`。
+ ② Web 场景通用配置 `spring.web`。
+ ③ 文件上传配置 `spring.servlet.multipart`。
+ ④ 服务器的配置 `server`，比如：编码方式

##### 2.2.2 默认效果
默认配置：

1、包含了 `ContentNegotiatingViewResolver` 和 `BeanNameViewResolver` 组件，**方便视图解析**。

2、**默认的静态资源处理机制**：静态资源放在 `static` 文件夹下即可直接访问。

3、**自动注册**了 `Converter`, `GenericConverter`, `Formatter` 组件，适配常见**数据类型转换**和**格式化需求**。

4、**支持**`HttpMessageConverters`，可以**方便返回**`json` 等**数据类型**。

5、**注册**`MessageCodesResolver`，方便**国际化**及错误消息处理。

6、**支持静态**`index.html`。

7、**自动使用**`ConfigurableWebBindingInitializer`，实现**消息处理、数据绑定、类型转化、数据校验**等功能。

> **重要**：
>
> + 如果想保持 **boot mvc 的默认配置**，并且自定义更多的 mvc 配置，比如：**interceptors**, **formatters**, **view controllers** 等。可以使用 **@Configuration** 注解添加一个 **WebMvcConfigurer** 类型的配置类，并且不要标注 **@EnableWebMvc**。
> + 如果想保持 boot mvc 的默认配置，但要自定义核心组件实例，比如：**RequestMappingHandlerMapping**, **RequestMappingHandlerAdapter**, 或 **ExceptionHandlerExceptionResolver**，给容器中放一个 **WebMvcRegistrations** 组件即可。
> + 如果想全面接管 Spring MVC，**@Configuration** 标注一个配置类，并加上 **@EnableWebMvc** 注解，实现 **WebMvcConfigurer** 接口。
>

#### 2.3 静态资源
##### 2.3.1 默认规则
###### 2.3.1.1 静态资源映射
静态资源映射规则在 `WebMvcAutoConfiguration` 中进行了定义：

1、`/webjars/**` 的所有路径资源都在 `classpath:/META-INF/resources/webjars/`。

2、`/**` 的所有路径资源都在 `classpath:/META-INF/resources/`、`classpath:/resources/`、`classpath:/static/`、`classpath:/public/`。

3、所有静态资源都定义了`缓存规则`。【浏览器访问过一次，就会缓存一段时间】，但此功能参数无默认值。

+ ① `period`：缓存间隔，默认 0 秒。
+ ② `cacheControl`：缓存控制，默认无。
+ ③ `useLastModified`：是否使用 lastModified 头，默认 false。

###### 2.3.1.2 静态资源缓存
如前面所述

1、所有静态资源都定义了`缓存规则`。【浏览器访问过一次，就会缓存一段时间】，但此功能参数无默认值。

+ ① `<font style="color:#DF2A3F;">period</font>`：缓存间隔，默认 0 秒。
+ ② `<font style="color:#DF2A3F;">cacheControl</font>`：缓存控制，默认无。
+ ③ `<font style="color:#DF2A3F;">useLastModified</font>`：是否使用 lastModified 头，默认 false。

###### 2.3.1.3 欢迎页
欢迎页规则在 `WebMvcAutoConfiguration` 中进行了定义：

1、在**静态资源**目录下找 `index.html` 模板页。

2、没有就在 `templates` 下找 `index.html` 模板页。

###### 2.3.1.4 Favicon
1、在静态资源目录下找 `favicon.ico`。

###### 2.3.1.5 缓存实验
```java
server.port=8080

# 1、spring.web：
# ① 配置国际化的区域信息
# ② 静态资源策略（开启、处理链、缓存）

# 开启静态资源映射规则
spring.web.resources.add-mappings=true

# 设置缓存
spring.web.resources.cache.period=3600

# 缓存详细合并项控制，覆盖 period 配置
# 浏览照第一次请求服务器，服务器告诉浏览器此资源缓存 7200 秒，7200 秒以内的所有此资源访问不用发给服务器请求，7200 秒以后发请求给服务器
spring.web.resources.cache.cachecontrol.max-age=7200

# 共享缓存
spring.web.resources.cache.cachecontrol.cache-public=true

# 使用资源 last-modified 时间，来对比服务器和浏览照的资源是否相同没有变化，相同返回 304
spring.web.resources.cache.use-last-modified=true
```

##### 2.3.2 自定义静态资源规则
> 自定义静态资源路径、自定义缓存规则。
>

###### 2.3.2.1 配置方式
`spring.mvc`：静态资源访问前缀路径。

`spring.web`：

+ 静态资源目录。
+ 静态资源缓存策略。

```java
# 2、spring.mvc
# ① 自定义 webjars 路径前缀
spring.mvc.webjars-path-pattern=/webjars/**

# ② 静态资源访问路径前缀
spring.mvc.static-path-pattern=/static/**
```

###### 2.3.2.2 代码方式
> 容器中只要有一个 WebMvcConfigurer 组件，配置的底层行为都会生效。
>
> @EnableWebMvc，禁用 boot 的默认配置。
>

```java
// 禁用 Spring Boot 的默认配置
// @EnableWebMvc
// 这是一个配置类，给容器中放一个 WebMvcConfigurer 组件，就能自定义底层
@Configuration
public class MyConfig implements WebMvcConfigurer
{
    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry)
    {
        // 保留默认规则
        WebMvcConfigurer.super.addResourceHandlers(registry);

        // 新增自定义规则
        registry.addResourceHandler("/static/**")
                .addResourceLocations("classpath:/image/, classpath:/static/")
                .setCacheControl(CacheControl.maxAge(7200, TimeUnit.SECONDS));
    }
}
```

```java
// 禁用 Spring Boot 的默认配置
// @EnableWebMvc
// 这是一个配置类，给容器中放一个 WebMvcConfigurer 组件，就能自定义底层
@Configuration
public class MyConfig
{
    @Bean
    public WebMvcConfigurer webMvcConfigurer()
    {
        return new WebMvcConfigurer()
        {
            /**
             * 配置静态资源
             * @param registry 注册表
             */
            @Override
            public void addResourceHandlers(ResourceHandlerRegistry registry)
            {
                // 保留默认规则
                WebMvcConfigurer.super.addResourceHandlers(registry);

                // 新增自定义规则
                registry.addResourceHandler("/static/**")
                        .addResourceLocations("classpath:/image/, classpath:/static/")
                        .setCacheControl(CacheControl.maxAge(7200, TimeUnit.SECONDS));
            }
        };
    }
}
```

#### 2.4 路径匹配
> **Spring5.3** 之后加入了更多的**请求路径匹配**的实现策略。
>
> 以前只支持 **AntPathMatcher** 策略, 现在提供了 **PathPatternParser** 策略，并且可以指定到底使用那种策略。
>

##### 2.4.1 Ant 风格路径用法
Ant 风格的路径模式语法具有以下规则：

+ `\*`：表示**任意数量的字符**。
+ `?`：表示**任意一个字符**。
+ `\*\*`：表示**任意数量的目录**。
+ `{}`：表示一个命名的模式**占位符**。
+ `[]`：表示`字符集合`，例如`[a-z]`表示小写字母。

例如：

+ `\*.html` 匹配任意名称，扩展名为 `.html` 的文件。
+ /`folder1/\*/\*.java` 匹配在 `folder1` 目录下的任意两级目录下的 `.java` 文件。
+ `/folder2/\*\*/\*.jsp` 匹配在 `folder2` 目录下任意目录深度的 `.jsp` 文件。
+ `/{type}/{id}.html` 匹配任意文件名为 `{id}.html`，在任意命名的 `{type}` 目录下的文件。

注意：Ant 风格的路径模式语法中的特殊字符需要转义，例如：

+ 要匹配文件路径中的星号，则需要转义为`\\\\*`。
+ 要匹配文件路径中的问号，则需要转义为 `\\\\?`。

##### 2.4.2 模式切换
> **AntPathMatcher** 与 **PathPatternParser**。
>
> **PathPatternParser** 在 jmh 基准测试下，有 6~8 倍吞吐量提升，降低 30%~40% 空间分配率。
>
> **PathPatternParser** 兼容 **AntPathMatcher** 语法，并支持更多类型的路径模式。
>
> **PathPatternParser** “******” **多段匹配**的支持**仅允许在模式末尾使用**。
>

```java
@Slf4j
@RestController
public class HelloController
{
    /**
     * 默认使用新版 PathPatternParser 进行路径匹配
     * 不能匹配 ** 在中间的情况，其他情况和 antPathMatcher语法兼容
     * @param request 请求
     * @param path 路径
     * @return uri
     */
    @GetMapping("/a*/b?/**/{p1:[a-f]+}/**")
    public String hello(HttpServletRequest request, @PathVariable("p1") String path)
    {
        log.info("路径变量 p1：{}", path);
        String uri = request.getRequestURI();
        return uri;
    }
}
```

总结：

+ 使用默认的路径匹配规则，是由 `PathPatternParser` 提供的。
+ 如果路径中间需要有 **，替换成 ant 风格路径。

#### 2.5 内容协商
> 一套系统适配多端数据返回。  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1755192535150-89f1a6bb-cc12-494a-8461-5be9015a4193.png)
>

##### 2.5.1 多端内容适配
###### 2.5.1.1 默认规则
1、**SpringBoot 多端内容适配**。

+ ① **基于请求头内容协商**：（默认开启）
    - (1)客户端向服务端发送请求，携带 HTTP 标准的 **Accept 请求头**。
        * [1] **Accept**: `application/json`、`text/xml`、`text/yaml`。
        * [2] 服务端根据客户端**请求头期望的数据类型**进行**动态返回**。
    - ② **基于请求参数内容协商**：（**需要开启**）
        * [1] 发送请求 `GET /projects/spring-boot?format=json`。
        * [2] 匹配到 `@GetMapping("/projects/spring-boot")`。
        * [3] 根据**参数协商**，优先返回 json 类型数据 **【需要开启参数匹配设置】**。
        * [4] 发送请求 `GET /projects/spring-boot?format=xml`，优先返回 xml 类型数据。

###### 2.5.1.2 效果演示
> 请求同一个接口，可以返回 json 和 xml 不同格式数据。
>

1、引入支持写出 xml 内容依赖。

```java
<dependency>
  <groupId>com.fasterxml.jackson.dataformat</groupId>
  <artifactId>jackson-dataformat-xml</artifactId>
</dependency>
```

2、标注注解。

```java
@JacksonXmlRootElement  // 可以写出为 xml 文档
@Component
//@ConfigurationProperties(prefix = "user")  和配置文件 person 前缀的所有配置进行绑定
//@NoArgsConstructor  自动生成无参构造器
//@AllArgsConstructor 自动生成全参构造器
// 自动生成 JavaBean 属性的 getter/setter
@Data
public class User
{
    private Long id;
    private String userName;
    private String password;
    private Integer age;
    private String email;
    private String role;
}
```

3、开启基于请求参数的内容协商。

```java
# 开启基于请求参数的内容协商功能，默认参数名：format，默认此功能不开启
spring.mvc.contentnegotiation.favor-parameter=true

# 指定内容协商时使用的参数名，默认是 format
spring.mvc.contentnegotiation.parameter-name=type
```

4、效果。

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1755192535475-8f7c6aeb-7e2c-4a3b-9930-db116544cb4c.png)

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1755192535489-645e6698-3611-4ee0-a579-198eb5027f57.png)

###### 2.5.1.3 配置协商规则与支持类型
1、修改**内容协商方式**。

```java
# 开启基于请求参数的内容协商功能，默认参数名：format，默认此功能不开启
spring.mvc.contentnegotiation.favor-parameter=true

# 指定内容协商时使用的参数名，默认是 format
spring.mvc.contentnegotiation.parameter-name=type
```

2、大多数 MediaType 都是开箱即用的。也可以**自定义内容类型**，例如：

```java
# 增加一种新的内容类型
spring.mvc.contentnegotiation.media-types.yaml=text/yaml
```

##### 2.5.2 自定义内容返回
###### 2.5.2.1 增加 yaml 返回支持
导入依赖。

```java
<dependency>
  <groupId>com.fasterxml.jackson.dataformat</groupId>
  <artifactId>jackson-dataformat-yaml</artifactId>
</dependency>
```

把对象写出成 YAML。

```java
package com.myxh.springboot.web.controller;

import com.myxh.springboot.web.bean.User;

public class HelloController{
    public static void main(String[] args) throws JsonProcessingException{
        User user = new User();
        user.setId(1L);
        user.setUserName("MYXH");
        user.setPassword("520.ILY!");
        user.setAge(21);
        user.setEmail("1735350920@qq.com");

        YAMLFactory factory = new YAMLFactory().disable(YAMLGenerator.Feature.WRITE_DOC_START_MARKER);
        ObjectMapper mapper = new ObjectMapper(factory);

        String userYaml = mapper.writeValueAsString(user);
        System.out.println("userYaml = " + userYaml);
    }
}
```

编写配置。

```java
# 增加一种新的内容类型
spring.mvc.contentnegotiation.media-types.yaml=text/yaml
```

增加 `HttpMessageConverter` 组件，专门负责把对象写出为 yaml 格式。

```java
/**
 * @author MYXH
 * @date 2023/9/18
 */
// 禁用 Spring Boot 的默认配置
// @EnableWebMvc
// 这是一个配置类，给容器中放一个 WebMvcConfigurer 组件，就能自定义底层
@Configuration
public class MyConfig{
    @Bean
    public WebMvcConfigurer webMvcConfigurer(){
        return new WebMvcConfigurer(){
            /**
             * 配置一个能把对象转为 yaml 的 messageConverter
             * @param converters 最初是转换器的空列表
             */
            @Override
            public void configureMessageConverters(List<HttpMessageConverter<?>> converters){
                converters.add(new MyYamlHttpMessageConverter());
            }
        };
    }
}
```

###### 2.5.2.2 思考：如何增加其他
+ 配置媒体类型支持:
    - `spring.mvc.contentnegotiation.media-types.yaml=text/yaml`。
+ 编写对应的 `HttpMessageConverter`，要告诉 Boot 这个支持的媒体类型。
    - 按照 HttpMessageConverter 的示例写法。
+ 把 MessageConverter 组件加入到底层。
    - 容器中放一个 `WebMvcConfigurer` 组件，并配置底层的 `MessageConverter`。

###### 2.5.2.3 HttpMessageConverter 的示例写法
```java
/**
 * @author MYXH
 * @date 2023/9/18
 * @description
 * 自定义的 YAML HTTP 消息转换器
 * 用于将对象转换为 YAML 格式的内容
 */
public class MyYamlHttpMessageConverter extends AbstractHttpMessageConverter<Object>{
    // 将对象转换为 YAML 的 ObjectMapper
    private final ObjectMapper objectMapper;

    public MyYamlHttpMessageConverter(){
        // 告诉 SpringBoot 这个 MessageConverter 支持哪种媒体类型
        super(new MediaType("text", "yaml", StandardCharsets.UTF_8),
              new MediaType("text", "yml", StandardCharsets.UTF_8));

        // 创建 YAMLFactory 并禁用写入文档起始标记
        YAMLFactory factory = new YAMLFactory().
        disable(YAMLGenerator.Feature.WRITE_DOC_START_MARKER);
        this.objectMapper = new ObjectMapper(factory);
    }
    /**
     * 判断该转换器是否支持将指定类型的对象转换为 YAML
     *
     * @param clazz 要测试支持的类
     * @return 如果支持转换，则返回 true；否则返回 false
     */
    @Override
    protected boolean supports(Class<?> clazz){
        // 只要是对象类型，不是基本类型和数组类型，都支持
        return !clazz.isPrimitive() && !clazz.isArray();
    }
    /**
     * 从 HTTP 输入消息中读取数据并将其转换为指定类型的对象
     * {@code @RequestBody} 把对象怎么读进来
     *
     * @param clazz        要返回的对象类型
     * @param inputMessage 要从中读取的 HTTP 输入消息
     * @return 转换后的对象
     * @throws IOException                     IO 异常
     * @throws HttpMessageNotReadableException Http 消息不可读异常
     */
    @Override
    protected Object readInternal(Class<?> clazz, HttpInputMessage inputMessage) throws IOException, HttpMessageNotReadableException{
        return objectMapper.readValue(inputMessage.getBody(), clazz);
    }
    /**
     * 将指定对象写入 HTTP 输出消息
     * {@code @ResponseBody} 把对象怎么写出去
     *
     * @param methodReturnValue 要写入输出消息的对象
     * @param outputMessage     要写入的 HTTP 输出消息
     * @throws IOException                     IO 异常
     * @throws HttpMessageNotWritableException Http 消息不可写入异常
     */
    @Override
    protected void writeInternal(Object methodReturnValue, HttpOutputMessage outputMessage) throws IOException, HttpMessageNotWritableException{
        // 使用 try-with-resources 语句以自动关闭流
        try (OutputStream os = outputMessage.getBody()){
            this.objectMapper.writeValue(os, methodReturnValue);
        }
    }
}
```

##### 2.5.3 内容协商原理 HttpMessageConverter
> **HttpMessageConverter** 怎么工作？合适工作？
>
> 定制 **HttpMessageConverter** 来实现多端内容协商。
>
> 编写 **WebMvcConfigurer** 提供的 **configureMessageConverters** 底层，修改底层的 **MessageConverter**。
>

###### 2.5.3.1 @ResponseBody 由 HttpMessageConverter 处理
> 标注了 **@ResponseBody** 的返回值 将会由支持它的 **HttpMessageConverter** 写给浏览器。
>

1、如果 controller 方法的返回值标注了 `@ResponseBody` 注解。

+ ① 请求进来先来到 `DispatcherServlet` 的 `doDispatch()` 进行处理。
+ ② 找到一个 `HandlerAdapter` 适配器，利用适配器执行目标方法。
+ ③ `RequestMappingHandlerAdapter` 来执行，调用 `invokeHandlerMethod()` 来执行目标方法。
+ ④ 目标方法执行之前，准备好两个东西。
    - (1) `HandlerMethodArgumentResolver`：参数解析器，确定目标方法每个参数值。
    - (2) `HandlerMethodReturnValueHandler`：返回值处理器，确定目标方法的返回值改怎么处理。
+ ⑤ `RequestMappingHandlerAdapter` 里面的 `invokeAndHandle()` 真正执行目标方法。
+ ⑥ 目标方法执行完成，会返回**返回值对象**。
+ ⑦ **找到一个合适的返回值处理器**`HandlerMethodReturnValueHandler`。
+ ⑧ 最终找到 `RequestResponseBodyMethodProcessor` 能处理标注了 `@ResponseBody` 注解的方法。
+ ⑨ `RequestResponseBodyMethodProcessor` 调用 `writeWithMessageConverters`，利用 `MessageConverter` 把返回值写出去。

> 上面解释：**@ResponseBody** 由 **HttpMessageConverter** 处理。
>

2、`HttpMessageConverter` 会**先进行内容协商**。

+ ① 遍历所有的 `MessageConverter` 看谁支持这种**内容类型的数据**。
+ ② 默认 `MessageConverter` 有以下 10 个。

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1755192535521-7573e9fb-de13-47ad-a266-458250274dbd.png)

+ ③ 最终因为要 `json` 所以 `MappingJackson2HttpMessageConverter` 支持写出 json。
+ ④ jackson 用 `ObjectMapper` 把对象写出去。

###### 2.5.3.2 WebMvcAutoConfiguration 提供几种默认 HttpMessageConverters
+ `EnableWebMvcConfiguration` 通过 `addDefaultHttpMessageConverters` 添加了默认的 `MessageConverter`，如下：
    - `ByteArrayHttpMessageConverter`：支持字节数据读写。
    - `StringHttpMessageConverter`：支持字符串读写。
    - `ResourceHttpMessageConverter`：支持资源读写。
    - `ResourceRegionHttpMessageConverter`: 支持分区资源写出。
    - `AllEncompassingFormHttpMessageConverter`：支持表单 xml/json 读写。
    - `MappingJackson2HttpMessageConverter`：支持请求响应体 Json 读写。

默认 `MessageConverter` 有以下 8 个：

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1755192535607-5db6d706-d3e1-4cf9-8dbb-6abe9357c149.png)

> 系统提供默认的 MessageConverter 功能有限，仅用于 json 或者普通返回数据。额外增加新的内容协商功能，必须增加新的 **HttpMessageConverter**。
>

#### 2.6 模板引擎
> 由于 **SpringBoot** 使用了**嵌入式 Servlet 容器**。所以 **JSP** 默认是**不能使用的**。
>
> 如果需要**服务端页面渲染**，优先考虑使用**模板引擎**。
>

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1755192536086-3f837723-0f9e-454f-a845-0683626d5320.png)

**模板引擎**页面默认放在 **src/main/resources/templates**。

**SpringBoot** 包含以下模板引擎的自动配置。

+ FreeMarker
+ Groovy
+ **Thymeleaf**
+ Mustache

**Thymeleaf 官网**：https://www.thymeleaf.org/ 。

##### 2.6.1 Thymeleaf 整合
```java
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```

自动配置原理：

1、开启了 `org.springframework.boot.autoconfigure.thymeleaf.ThymeleafAutoConfiguration` 自动配置。

2、属性绑定在 `ThymeleafProperties` 中，对应配置文件 `spring.thymeleaf` 内容。

3、所有的模板页面默认在 `classpath:/templates/` 文件夹下。

4、默认效果。

+ ① 所有的模板页面在 `classpath:/templates/` 下面找。
+ ② 找后缀名为 `.html` 的页面。

##### 2.6.2 基础语法
###### 2.6.2.1 核心用法
`th:xxx`：**动态渲染指定的 html 标签属性值、或者 th 指令（遍历、判断等）**。

+ `th:text`：标签体内文本值渲染。
    - `th:utext`：不会转义，显示为 html 原本的样子。
+ `th:属性`：标签指定属性渲染。
+ `th:attr`：标签任意属性渲染。
+ `th:if`、`th:each``...` ：其他 th 指令。
+ 例如：

```java
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
  <head>
    <meta charset="UTF-8" />
    <title>欢迎页</title>
  </head>

  <body>
    <h1>你好，<span th:utext="${message}"></span></h1>

    <hr />

    <h2>1、Thymeleaf 基础语法</h2>

    <h3>1.1 th:text：替换标签体的内容，会转义</h3>
    <p th:text="${message}"></p>

    <h3>
      1.2 th:utext: 替换标签体的内容，不会转义 html 标签，真正显示为 html
      该有的样式
    </h3>
    <p th:utext="${message}"></p>

    <h3>1.3 th:任意 html 属性，动态替换任意属性的值</h3>
    <img
      th:src="@{${imageUrl}}"
      src="大户爱.png"
      style="width:300px;"
      alt="大户爱"
    />

    <h3>1.4 th:attr：任意属性指定</h3>
    <img
      th:attr="src=@{${imageUrl}},style=${style}"
      src="大户爱.png"
      alt="大户爱"
    />

    <h3>1.5 th：其他指令</h3>
    <img
      th:src="@{${imageUrl}}"
      th:style="${style}"
      th:if="${show}"
      src="大户爱.png"
      alt="大户爱"
    />

    <h3>1.6 @{} 专门用来取各种路径</h3>
    <img
      th:src="@{${imageUrl}}"
      src="大户爱.png"
      style="width:300px;"
      alt="大户爱"
    />

    <h3>1.7 字符串转换为大写</h3>
    <p th:text="${#strings.toUpperCase(name)}"></p>

    <h3>1.8 字符串拼接</h3>
    <p th:text="${'前缀'+name+'后缀'}"></p>
    <p th:text="|前缀${name}后缀|"></p>
  </body>
</html>
```

`表达式`：**用来动态取值**。

+ `$\{\}`：**变量取值；使用 model 共享给页面的值都直接用 ${}**。
+ `@{}`：**url 路径**。
+ `#{}`：国际化消息。
+ `~{}`：片段引用。
+ `\*{}`：变量选择：需要配合 th:object 绑定对象。

`系统工具&内置对象`：**详细文档**。

+ `param`：请求参数对象。
+ `session`：session 对象。
+ `application`：application 对象。
+ `#execInfo`：模板执行信息。
+ `#messages`：国际化消息。
+ `#uris`：uri/url 工具。
+ `#conversions`：类型转换工具。
+ `#dates`：日期工具，是 `java.util.Date` 对象的工具类。
+ `#calendars`：类似 #dates，只不过是 `java.util.Calendar` 对象的工具类。
+ `#temporals`：JDK8+ `java.time` API 工具类。
+ `#numbers`：数字操作工具。
+ `#strings`：字符串操作。
+ `#objects`：对象操作。
+ `#bools`：bool 操作。
+ `#arrays`：array 工具。

-`#lists`：list 工具。

+ `#sets`：set 工具。
+ `#maps`：map 工具。
+ `#aggregates`：集合聚合工具（sum、avg）。
+ `#ids`：id 生成工具。

###### 2.6.2.2 语法示例
**表达式**：

+ 变量取值：`${...}`
+ url 取值：`@{...}`
+ 国际化消息：`#{...}`
+ 变量选择：`\*{...}`
+ 片段引用: `~{...}`

**常见**：

+ 文本：`'one text'`，`'another one!'`，…
+ 数字：`0`, `12`, `3.0`, `15.6`, …
+ 布尔：`true`、`false`
+ null: `null`
+ 变量名：`one`，`sometext`，`main` …

**文本操作**：

+ 拼串：`+`
+ 文本替换：`| The name is ${name} |`

**布尔操作**：

+ 二进制运算：`and`, `or`
+ 取反：`!`, `not`

**比较运算**：

+ 比较：`<`，`>`，`<=`，`>=`（`lt`，`gt`，`le`,`ge`）
+ 等值运算：`==`，`!=`（`eq`，`ne`）

**条件运算**：

+ if-then：`(if)?(then)`
+ if-then-else：`(if)?(then):(else)`
+ default：`(value)?:(defaultValue)`

**特殊语法**：

+ 无操作：`\_`

**所有以上都可以嵌套组合**。

###### 2.6.2.3 属性设置
1、`th:href="@{/product/list}"`

2、`th:attr="class=${active}"`

3、`th:attr="src=@{/images/image.png},title=${logo},alt=#{logo}"`

4、`th:checked="${user.active}"`

###### 2.6.2.4 遍历
> 语法：**th:each=“元素名,迭代状态 : ${集合}”**
>

```java
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
  <head>
    <meta charset="UTF-8" />
    <title>用户列表页</title>
    <link
      href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/css/bootstrap.min.css"
      rel="stylesheet"
      integrity="sha384-T3c6CoIi6uLrA9TneNEoa7RxnatzjcDSCmG1MXxSR1GAsXEV/Dwwykc2MPK8M2HN"
      crossorigin="anonymous"
    />
  </head>

  <body>
    <!-- 导航区使用公共部分进行替换 -->
    <!-- ~{ 模板名 :: 片段名} -->
    <div th:replace="~{common :: myheader}"></div>

    <div class="container">
      <table class="table">
        <thead>
          <tr>
            <th scope="col">序号</th>
            <th scope="col">用户名</th>
            <th scope="col">密码</th>
            <th scope="col">年龄</th>
            <th scope="col">邮箱</th>
            <th scope="col">角色</th>
            <th scope="col">状态信息</th>
          </tr>
        </thead>

        <tbody>
          <tr th:each="user, stats : ${userList}" th:object="${user}">
            <th scope="row" th:text="${user.id}"></th>
            <!-- <td th:text="${user.userName}"></td> -->
            <td th:text="*{userName}"></td>
            <td>[[${user.password}]]</td>
            <td
              th:text="|${user.age}(${user.age >= 18 ? '成年' : '未成年'})|"
            ></td>
            <td
              th:if="${#strings.isEmpty(user.email)}"
              th:text="'联系不上'"
            ></td>
            <td
              th:if="${not #strings.isEmpty(user.email)}"
              th:text="${user.email}"
            ></td>
            <td th:switch="${user.role}">
              <button th:case="'root'" type="button" class="btn btn-primary">
                root 用户
              </button>
              <button th:case="'admin'" type="button" class="btn btn-secondary">
                管理员
              </button>
              <button th:case="'test'" type="button" class="btn btn-success">
                测试员
              </button>
              <button th:case="'user'" type="button" class="btn btn-light">
                用户
              </button>
            </td>
            <td>
              index（索引）：[[${stats.index}]]<br />
              count（计数）：[[${stats.count}]]<br />
              size（大小）：[[${stats.size}]]<br />
              current（当前对象）：[[${stats.current}]]<br />
              even(true)/odd(false)（奇数/偶数）：[[${stats.even}]]<br />
              first（第一个）：[[${stats.first}]]<br />
              last（最后的）：[[${stats.last}]]<br />
            </td>
          </tr>
        </tbody>
      </table>
    </div>

    <script
      src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/js/bootstrap.bundle.min.js"
      integrity="sha384-C6RzsynM9kWDrMNeT87bh95OGNyZPhcTNXj1NW7RuBCsyN/o0jlpcV8Qyq46cDfL"
      crossorigin="anonymous"
    ></script>
  </body>
</html>
```

iterStat 有以下属性：

+ index：当前遍历元素的索引，从 0 开始。
+ count：当前遍历元素的索引，从 1 开始。
+ size：需要遍历元素的总数量。
+ current：当前正在遍历的元素对象。
+ even/odd：是否为偶数/奇数行。
+ first：是否第一个元素。
+ last：是否最后一个元素。

###### 2.6.2.5 判断
**th:if**

```java
<td th:text="|${user.age}(${user.age >= 18 ? '成年' : '未成年'})|"></td>
<td th:if="${#strings.isEmpty(user.email)}" th:text="'联系不上'"></td>
<td th:if="${not #strings.isEmpty(user.email)}" th:text="${user.email}"></td>
```

**th:switch**

```java
<td th:switch="${user.role}">
  <button th:case="'root'" type="button" class="btn btn-primary">
    root 用户
  </button>
  <button th:case="'admin'" type="button" class="btn btn-secondary">
    管理员
  </button>
  <button th:case="'test'" type="button" class="btn btn-success">测试员</button>
  <button th:case="'user'" type="button" class="btn btn-light">用户</button>
</td>
```

###### 2.6.2.6 属性优先级
+ 片段。
+ 遍历。
+ 判断。

```java
<tr th:each="user, stats : ${userList}" th:object="${user}">
  <th scope="row" th:text="${user.id}"></th>
  <!-- <td th:text="${user.userName}"></td> -->
  <td th:text="*{userName}"></td>
</tr>
```

| 顺序 | 特性 | 属性 |
| --- | --- | --- |
| 1 | 片段包含。 | `th:insert`<br/>、`th:replace` |
| 2 | 遍历。 | `th:each` |
| 3 | 判断。 | `th:if`<br/>、`th:unless`<br/>、`th:switch`<br/>、`th:case` |
| 4 | 定义本地变量。 | `th:object`<br/>、`th:with` |
| 5 | 通用方式属性修改。 | `th:attr`<br/>、`th:attrprepend`<br/>、`th:attrappend` |
| 6 | 指定属性修改。 | `th:value`<br/>、`th:href`<br/>、`th:src`<br/>、`...` |
| 7 | 文本值。 | `th:text`<br/>、`th:utext` |
| 8 | 片段指定。 | `th:fragment` |
| 9 | 片段移除。 | `th:remove` |


###### 2.6.2.7 行内写法
`[[...]]` 或 `[(...)]`。

```java
<tr th:each="user, stats : ${userList}" th:object="${user}">
  <th scope="row" th:text="${user.id}"></th>

  <td th:text="*{userName}"></td>
  <td>[[${user.password}]]</td>
</tr>
```

###### 2.6.2.8 变量选择
```java
<tr th:each="user, stats : ${userList}" th:object="${user}">
  <th scope="row" th:text="${user.id}"></th>
  <td th:text="*{userName}"></td>
</tr>
```

等同于

```java
<tr th:each="user, stats : ${userList}">
  <th scope="row" th:text="${user.id}"></th>
  <td th:text="${user.userName}"></td>
</tr>
```

###### 2.6.2.9 模板布局
+ 定义模板：`th:fragment`
+ 引用模板：`~{templatename::selector}`
+ 插入模板：`th:insert`、`th:replace`

```java
<div th:replace="~{common :: myheader}"></div>
```

###### 2.6.2.10 devtools
```java
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-devtools</artifactId>
</dependency>
```

修改页面后，`ctrl+F9` 刷新效果。

Java 代码的修改，如果 `devtools` 热启动了，可能会引起一些 bug，难以排查。

#### 2.7 国际化
国际化的自动配置参照 `MessageSourceAutoConfiguration`。

**实现步骤**：

1、Spring Boot 在类路径根下查找 `messages` 资源绑定文件。文件名为：`messages.properties`。

2、多语言可以定义多个消息文件，命名为 `messages\_区域代码.properties`。例如：

+ ① `<font style="color:#DF2A3F;">messages.properties</font>`：默认。
+ ② `<font style="color:#DF2A3F;">messages_zh_CN.properties</font>`：中文环境。
+ ③ `<font style="color:#DF2A3F;">messages_en_US.properties</font>`：英语环境。

3、在**程序中可以自动注入**`MessageSource` 组件，获取国际化的配置项值。

4、在**页面中可以使用表达式**`#{}` 获取国际化的配置项值。

```java
//适配服务端清染，前后不分离模式
@Controller
public class WelcomeController{
    // 国际化获取消息的组件
    @Autowired
    MessageSource messageSource;

    @GetMapping("/message")
    @ResponseBody
    public String message(HttpServletRequest request){
        Locale locale = request.getLocale();
        // 利用代码的方式获取国际化配置文件中指定的配置项的值
        String login = messageSource.getMessage("login", null, locale);
        return login;
    }
}
```

+ `<font style="color:rgb(0, 0, 0);">messageSource.getMessage()</font>`<font style="color:rgba(0, 0, 0, 0.85) !important;"> 方法的第二个参数是一个 </font>`<font style="color:rgb(0, 0, 0);">Object[]</font>`<font style="color:rgba(0, 0, 0, 0.85) !important;"> 类型，用于传递消息中需要动态替换的参数。</font>
+ <font style="color:rgba(0, 0, 0, 0.85) !important;">当消息不需要参数时（比如你示例中的 "login" 只是一个简单的固定文本），就可以传入 </font>`<font style="color:rgb(0, 0, 0);">null</font>`<font style="color:rgba(0, 0, 0, 0.85) !important;"> 表示没有参数。</font>

<font style="color:rgba(0, 0, 0, 0.85) !important;">举个需要参数的例子：  
</font><font style="color:rgba(0, 0, 0, 0.85) !important;">如果资源文件中有这样的配置：</font>

```java
greeting=Hello, {0}! Today is {1}.
```

<font style="color:rgba(0, 0, 0, 0.85) !important;">这时就需要传递参数数组：</font>

```java
// 传递两个参数
String message = messageSource.getMessage("greeting", new Object[]{"Alice", "Monday"}, locale);
// 结果会是 "Hello, Alice! Today is Monday."
```

<font style="color:rgba(0, 0, 0, 0.85) !important;">所以简单理解：</font>`<font style="color:rgba(0, 0, 0, 0.85) !important;">null</font>`<font style="color:rgba(0, 0, 0, 0.85) !important;"> 在这里表示 “当前消息不需要动态参数”。</font>

#### 2.8 错误处理
##### 2.8.1 默认机制
> **错误处理的自动配置**都在 **ErrorMvcAutoConfiguration** 中，两大核心机制：
>
> + 1、SpringBoot 会**自适应**处理错误，**响应页面**或 **JSON 数据**。
> + 2、**SpringMVC 的错误处理机制**依然保留，**MVC 处理不了**，才会**交给 boot 进行处理**。
>

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1755192536103-a9669b24-fd78-43fb-85b1-047dcb443422.png)

+ 发生错误以后，转发给/error 路径，SpringBoot 在底层写好一个 BasicErrorController 的组件，专门处理这个请求。

```java
// 返回 HTML
@RequestMapping(produces = MediaType.TEXT_HTML_VALUE)
public ModelAndView errorHtml(HttpServletRequest request, HttpServletResponse response){
    HttpStatus status = getStatus(request);
    Map<String, Object> model = Collections
    .unmodifiableMap(getErrorAttributes(request, getErrorAttributeOptions(request, MediaType.TEXT_HTML)));
    response.setStatus(status.value());
    ModelAndView modelAndView = resolveErrorView(request, response, status, model);

    return (modelAndView != null) ? modelAndView : new ModelAndView("error", model);
}

//返回 ResponseEntity，JSON
@RequestMapping
public ResponseEntity<Map<String, Object>> error(HttpServletRequest request){
HttpStatus status = getStatus(request);

if (status == HttpStatus.NO_CONTENT){
    return new ResponseEntity<>(status);
}

Map<String, Object> body = getErrorAttributes(request, getErrorAttributeOptions(request, MediaType.ALL));

return new ResponseEntity<>(body, status);
}
```

+ 错误页面是这么解析到的。

```java
// 1、解析错误的自定义视图地址
ModelAndView modelAndView = resolveErrorView(request, response, status, model);

// 2、如果解析不到错误页面的地址，默认的错误页就是 error
return (modelAndView != null) ? modelAndView : new ModelAndView("error", model);
```

+ 容器中专门有一个错误视图解析器。

```java
@Bean
@ConditionalOnBean(DispatcherServlet.class)
@ConditionalOnMissingBean(ErrorViewResolver.class)
DefaultErrorViewResolver conventionErrorViewResolver(){
    return new DefaultErrorViewResolver(this.applicationContext, this.resources);
}
```

+ SpringBoot 解析自定义错误页的默认规则。

```java
@Override
public ModelAndView resolveErrorView(HttpServletRequest request, HttpStatus status, Map<String, Object> model){
    ModelAndView modelAndView = resolve(String.valueOf(status.value()), model);

    if (modelAndView == null && SERIES_VIEWS.containsKey(status.series())){
        modelAndView = resolve(SERIES_VIEWS.get(status.series()), model);
    }

    return modelAndView;
}

private ModelAndView resolve(String viewName, Map<String, Object> model){
    String errorViewName = "error/" + viewName;
    TemplateAvailabilityProvider provider = this.templateAvailabilityProviders.getProvider(errorViewName,
                                                                                           this.applicationContext);

    if (provider != null){
        return new ModelAndView(errorViewName, model);
    }

    return resolveResource(errorViewName, model);
}

private ModelAndView resolveResource(String viewName, Map<String, Object> model){
    for (String location : this.resources.getStaticLocations()){
        try{
                Resource resource = this.applicationContext.getResource(location);
                resource = resource.createRelative(viewName + ".html");

                if (resource.exists()){
                    return new ModelAndView(new HtmlResourceView(resource), model);
                }
            }
        catch (Exception ex){

            }
    }
    return null;
}
```

+ 容器中有一个默认的名为 error 的 view，提供了默认白页功能。

```java
@Bean(name = "error")
@ConditionalOnMissingBean(name = "error")
public View defaultErrorView(){
    return this.defaultErrorView;
}
```

+ 封装了 JSON 格式的错误信息。

```java
@Bean
@ConditionalOnMissingBean(value = ErrorAttributes.class, search = SearchStrategy.CURRENT)
public DefaultErrorAttributes errorAttributes(){
    return new DefaultErrorAttributes();
}
```

规则：

1、**解析一个错误页**。

+ ① 如果发生了 500、404、503、403 这些错误。
    - (1) 如果有**模板引擎**，默认在 `classpath:/templates/error/精确码.html`。
    - (2) 如果没有模板引擎，在静态资源文件夹下找`精确码.html`。
+ ② 如果匹配不到`精确码.html` 这些精确的错误页，就去找 `5xx.html`，`4xx.html`**模糊匹配**。
    - (1) 如果有模板引擎，默认在 `classpath:/templates/error/5xx.html`。
    - (2) 如果没有模板引擎，在静态资源文件夹下找 `5xx.html`。

2、如果模板引擎路径 `templates` 下有 `error.html` 页面，就直接渲染。

##### 2.8.2 自定义错误响应
###### 2.8.2.1 自定义 json 响应
> 使用 @ControllerAdvice + @ExceptionHandler 进行统一异常处理。
>

###### 2.8.2.2 自定义页面响应
> 根据 boot 的错误页面规则，自定义页面模板。
>

##### 2.8.3 最佳实战
+ **前后分离**。
    - 后台发生的所有错误，`@ControllerAdvice + @ExceptionHandler` 进行统一异常处理。
+ **服务端页面渲染**。
    - `不可预知的一些，HTTP 码表示的服务器或客户端错误`。
        * 给 `classpath:/templates/error/` 下面，放常用精确的错误码页面。`500.html`，`404.html`。
        * 给 `classpath:/templates/error/` 下面，放通用模糊匹配的错误码页面。`5xx.html`，`4xx.html`。
    - **发生业务错误**。
        * **核心业务**，每一种错误，都应该代码控制，**跳转到自己定制的错误页**。
        * **通用业务**，`classpath:/templates/error.html` 页面，**显示错误信息**。

在 HTML 页面、JSON 数据中，可用的 Model 数据如下。

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1755192536264-c4d2c44e-e6ac-474f-a6ca-0dafdf39603f.png)

#### 2.9 嵌入式容器
> **Servlet 容器**：管理、运行 **Servlet 组件**（Servlet、Filter、Listener）的环境，一般指**服务器**。
>

##### 2.9.1 自动配置原理
> + SpringBoot 默认嵌入 Tomcat 作为 Servlet 容器。
> + **自动配置类**是 **ServletWebServerFactoryAutoConfiguration**，**EmbeddedWebServerFactoryCustomizerAutoConfiguration**。
> + 自动配置类开始分析功能，**xxxAutoConfiguration**。
>

```java
@AutoConfiguration
@AutoConfigureOrder(Ordered.HIGHEST_PRECEDENCE)
@ConditionalOnClass(ServletRequest.class)
@ConditionalOnWebApplication(type = Type.SERVLET)
@EnableConfigurationProperties(ServerProperties.class)
@Import({ServletWebServerFactoryAutoConfiguration.BeanPostProcessorsRegistrar.class,
         ServletWebServerFactoryConfiguration.EmbeddedTomcat.class,
         ServletWebServerFactoryConfiguration.EmbeddedJetty.class,
         ServletWebServerFactoryConfiguration.EmbeddedUndertow.class})
public class ServletWebServerFactoryAutoConfiguration{
}
```

1、`ServletWebServerFactoryAutoConfiguration` 自动配置了嵌入式容器场景。

2、绑定了 `ServerProperties` 配置类，所有和服务器有关的配置 `server`。

3、`ServletWebServerFactoryAutoConfiguration` 导入了嵌入式的三大服务器 `Tomcat`、`Jetty`、`Undertow`。

+ ① 导入 `Tomcat`、`Jetty`、`Undertow` 都有条件注解，系统中有这个类才行（也就是导了包）。
+ ② 默认 `Tomcat` 配置生效。给容器中放 TomcatServletWebServerFactory。
+ ③ 都给容器中 `ServletWebServerFactory` 放了一个 **Web 服务器工厂**（**造 Web 服务器的**）。
+ ④ **Web 服务器工厂都有一个功能**，`getWebServer`获取 Web 服务器。
+ ⑤ TomcatServletWebServerFactory 创建了 tomcat。

4、ServletWebServerFactory 什么时候会创建 webServer 出来。

5、`ServletWebServerApplicationContext` ioc 容器，启动的时候会调用创建 Web 服务器。

6、Spring **容器刷新**（**启动**）的时候，会预留一个时机，刷新子容器，`onRefresh()`。

7、refresh() 容器刷新，十二个步骤的刷新，子容器会调用 `onRefresh()`。

```java
@Override
protected void onRefresh(){
    super.onRefresh();
    try{
        createWebServer();
    }
    catch (Throwable ex){
            throw new ApplicationContextException("Unable to start web server", ex);
        }
}
```

> Web 场景的 Spring 容器启动，在 onRefresh 的时候，会调用创建 Web 服务器的方法。
>
> Web 服务器的创建是通过 WebServerFactory 搞定的。容器中又会根据导了什么包条件注解，启动相关的 服务器配置，默认 **EmbeddedTomcat** 会给容器中放一个 **TomcatServletWebServerFactory**，导致项目启动，自动创建出 Tomcat。
>

##### 2.9.2 自定义
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1755192536317-063f6b06-9ff2-4504-931d-cef1fc039b94.png)

> 切换服务器。
>

```java
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-web</artifactId>
  <exclusions>
    <!-- 排除 Tomcat 依赖项 -->
    <exclusion>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-tomcat</artifactId>
    </exclusion>
  </exclusions>
</dependency>

<!-- 改为使用 Jetty -->
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-jetty</artifactId>
</dependency>
```

##### 2.9.3 最佳实践
**用法**：

+ 修改 `server` 下的相关配置就可以修改**服务器参数**。
+ 通过给容器中放一个 `ServletWebServerFactory`，来禁用掉 SpringBoot 默认放的服务器工厂，实现自定义嵌入**任意服务器**。

```java
@Configuration
public class CustomServerConfig {

    /**
     * 自定义ServletWebServerFactory，会覆盖Spring Boot默认的服务器工厂
     * 此处以Tomcat为例，如需其他服务器可替换为JettyServletWebServerFactory或UndertowServletWebServerFactory
     */
    @Bean
    public TomcatServletWebServerFactory customTomcatServerFactory() {
        // 创建Tomcat服务器工厂
        TomcatServletWebServerFactory factory = new TomcatServletWebServerFactory() {
            @Override
            public WebServer getWebServer(ServletContextInitializer... initializers) {
                // 可以重写WebServer创建逻辑，添加自定义初始化操作
                WebServer webServer = super.getWebServer(initializers);
                System.out.println("自定义Tomcat服务器启动完成，端口: " + getPort());
                return webServer;
            }
        };

        // 基础配置
        factory.setPort(8888); // 自定义端口
        factory.setContextPath("/custom"); // 上下文路径
        factory.setBaseDirectory(new File("./custom-tomcat")); // 自定义工作目录

        // 配置连接参数
        factory.addAdditionalTomcatConnectors(createAdditionalConnector()); // 添加额外连接器

        // 配置编码
        factory.setUriEncoding(StandardCharsets.UTF_8);

        // 配置错误页面
        factory.addErrorPages(new org.springframework.boot.web.server.ErrorPage(
            org.springframework.http.HttpStatus.NOT_FOUND, "/404.html")
        );

        // 配置Tomcat特定参数
        factory.addContextCustomizers(context -> {
            context.setReloadable(false); // 生产环境禁用热部署
            context.setSessionTimeout(60); // 会话超时时间(秒)
            context.setUseHttpOnly(true); // 启用HttpOnly保护
        });

        return factory;
    }

    /**
     * 创建额外的HTTP连接器（例如同时支持8080和8888端口）
     */
    private org.apache.catalina.connector.Connector createAdditionalConnector() {
        org.apache.catalina.connector.Connector connector = 
            new org.apache.catalina.connector.Connector("org.apache.coyote.http11.Http11NioProtocol");
        
        // 配置额外端口
        connector.setPort(8080);
        connector.setScheme("http");
        connector.setSecure(false);
        
        return connector;
    }
}
```

#### 2.10 全面接管 SpringMVC
> + SpringBoot 默认配置好了 SpringMVC 的所有常用特性。
> + 如果需要全面接管 SpringMVC 的所有配置并**禁用默认配置**，仅需要编写一个 **WebMvcConfigurer** 配置类，并标注 **@EnableWebMvc** 即可。
> + 全手动模式。
>     - **@EnableWebMvc** : 禁用默认配置。
>     - **WebMvcConfigurer** 组件：定义 MVC 的底层行为。
>

##### 2.10.1 WebMvcAutoConfiguration 到底自动配置了哪些规则
> SpringMVC 自动配置场景配置了如下所有**默认行为**。
>

1、`WebMvcAutoConfiguration` Web 场景的自动配置类。

+ ① 支持 RESTful 的 filter：HiddenHttpMethodFilter。
+ ② 支持非 POST 请求，请求体携带数据：FormContentFilter。
+ ③ 导入 `EnableWebMvcConfiguration`：
    - (1) `RequestMappingHandlerAdapter`。
    - (2) `WelcomePageHandlerMapping`：**欢迎页功能**支持（模板引擎目录、静态资源目录放 index.html），项目访问 / 就默认展示这个页面。
    - (3) `RequestMappingHandlerMapping`：找每个请求由谁处理的映射关系。
    - (4) `ExceptionHandlerExceptionResolver`：默认的异常解析器。
    - (5) `LocaleResolver`：国际化解析器。
    - (6) `ThemeResolver`：主题解析器。
    - (7) `FlashMapManager`：临时数据共享。
    - (8) `FormattingConversionService`：数据格式化、类型转化。
    - (9) `Validator`：数据校验 `JSR303` 提供的数据校验功能。
    - (10) `WebBindingInitializer`：请求参数的封装与绑定。
    - (11) `ContentNegotiationManager`：内容协商管理器。
+ ④ `WebMvcAutoConfigurationAdapter` 配置生效，它是一个 `WebMvcConfigurer`，定义 mvc 底层组件。
    - (1) 定义好 `WebMvcConfigurer`**底层组件默认功能**，**所有功能详见列表**。
    - (2) 视图解析器：`InternalResourceViewResolver`。
    - (3) 视图解析器：`BeanNameViewResolver`，**视图名**（**controller 方法的返回值字符串**）就是组件名。
    - (4) 内容协商解析器：`ContentNegotiatingViewResolver`。
    - (5) 请求上下文过滤器：`RequestContextFilter`，任意位置直接获取当前请求。
    - (6) 静态资源链规则。
    - (7) `ProblemDetailsExceptionHandler`：错误详情。
        * [1] SpringMVC 内部场景异常被它捕获。
+ ⑤ 定义了 MVC 默认的底层行为: `WebMvcConfigurer`。

##### 2.10.2 @EnableWebMvc 禁用默认行为
1、`@EnableWebMvc` 给容器中导入 `DelegatingWebMvcConfiguration` 组件，  
是 `WebMvcConfigurationSupport`。

2、`WebMvcAutoConfiguration` 有一个核心的条件注解，`@ConditionalOnMissingBean(WebMvcConfigurationSupport.class)`，容器中没有 `WebMvcConfigurationSupport`，`WebMvcAutoConfiguration` 才生效。

3、 @EnableWebMvc 导入 `WebMvcConfigurationSupport` 导致 `WebMvcAutoConfiguration` 失效，导致禁用了默认行为。

> @EnableWebMVC 禁用了 Mvc 的自动配置。
>
> WebMvcConfigurer 定义 SpringMVC 底层组件的功能类。
>

##### 2.10.3 WebMvcConfigurer 功能
> 定义扩展 SpringMVC 底层功能。
>

| 提供方法 | 核心参数 | 功能 | 默认 |
| --- | --- | --- | --- |
| addFormatters | FormatterRegistry | **格式化器**：支持属性上 `@NumberFormat`<br/> 和`@DatetimeFormat`<br/> 的数据类型转换。 | GenericConversionService |
| getValidator | 无 | **数据校验**：校验 Controller 上使用 `@Valid`<br/> 标注的参数合法性，需要导入 `starter-validator`<br/>。 | 无 |
| `addInterceptors` | InterceptorRegistry | **拦截器**：拦截收到的所有请求。 | 无 |
| `configureContentNegotiation` | ContentNegotiationConfigurer | **内容协商**：支持多种数据格式返回，需要配合支持这种类型的 `HttpMessageConverter`<br/>。 | 支持 json。 |
| `configureMessageConverters` | `List<HttpMessageConverter<?>>` | **消息转换器**：标注 `@ResponseBody`<br/> 的返回值会利用 `MessageConverter`<br/> 直接写出去。 | 8 个，支持 `byte`<br/>, `string`<br/>, `multipart`<br/>, `resource`<br/>, `json`<br/>。 |
| addViewControllers | ViewControllerRegistry | **视图映射**：直接将请求路径与物理视图映射，用于无 Java 业务逻辑的直接视图页渲染。 | `<mvc:view-controller>` |
| configureViewResolvers | ViewResolverRegistry | **视图解析器**：逻辑视图转为物理视图。 | iewResolverComposite |
| addResourceHandlers | ResourceHandlerRegistry | **静态资源处理**：静态资源路径映射、缓存控制。 | ResourceHandlerRegistry |
| configureDefaultServletHandling | DefaultServletHandlerConfigurer | **默认 Servlet**：可以覆盖 Tomcat 的 `DefaultServlet`<br/>，让 `DispatcherServlet`<br/> 拦截 `/`<br/>。 | 无 |
| configurePathMatch | PathMatchConfigurer | **路径匹配**：自定义 URL 路径匹配，可以自动为所有路径加上指定前缀，比如 `/api`<br/>。 | 无 |
| `configureAsyncSupport` | AsyncSupportConfigurer | **异步支持**： | `TaskExecutionAutoConfiguration` |
| addCorsMappings | CorsRegistry | **跨域**： | 无 |
| addArgumentResolvers | `List<HandlerMethodArgumentResolver>` | **参数解析器**： | mvc 默认提供。 |
| addReturnValueHandlers | `List<HandlerMethodReturnValueHandler>` | 返回值解析器： | mvc 默认提供。 |
| configureHandlerExceptionResolvers | List | **异常处理器**： | 默认 3 个，ExceptionHandlerExceptionResolver、ResponseStatusExceptionResolver、DefaultHandlerExceptionResolver。 |
| getMessageCodesResolver | 无 | **消息码解析器**：国际化使用 | 无 |


#### 2.11 最佳实践
> SpringBoot 已经默认配置好了 **Web 开发场景**常用功能，直接使用即可。
>

##### 2.11.1 三种方式
| 方式 | 用法 | 效果 |
| --- | --- | --- |
| **全自动** | 直接编写控制器逻辑。 | 全部使用**自动配置默认效果**。 |
| **半自动** | 注解 `@Configuration`<br/> + 配置 `WebMvcConfigurer`<br/> + 配置 `WebMvcRegistrations`<br/>，**不要标注** 注解 `@EnableWebMvc`<br/>。 | **保留自动配置效果**，**手动设置部分功能**，定义 MVC 底层组件。 |
| **全手动** | 注解 `@Configuration`<br/> + 配置 `WebMvcConfigurer`<br/>，**标注**注解 `@EnableWebMvc`<br/>。 | **禁用自动配置效果**，**全手动设置**。 |


总结：

**给容器中写一个配置类**`@Configuration`**实现**`WebMvcConfigurer`**但是不要标注**`@EnableWebMvc`**注解**，**实现半自动的效果**。

##### 2.11.2 两种模式
1、`前后分离模式`：`@RestController` 响应 JSON 数据。

2、`前后不分离模式`：`@Controller` + Thymeleaf 模板引擎。

#### 2.12 Web 新特性
##### 2.12.1 Problemdetails
> RFC 7807: https://www.rfc-editor.org/rfc/rfc7807
>
> **错误信息**返回新格式。
>

原理：

```java
@Configuration(proxyBeanMethods = false)
// 配置一个属性 spring.mvc.problemdetails.enabled=true
@ConditionalOnProperty(prefix = "spring.mvc.problemdetails", name = "enabled", havingValue = "true")
static class ProblemDetailsErrorHandlingConfiguration{
    @Bean
    @ConditionalOnMissingBean(ResponseEntityExceptionHandler.class)
    ProblemDetailsExceptionHandler problemDetailsExceptionHandler(){
        return new ProblemDetailsExceptionHandler();
    }
}
```

1、`ProblemDetailsExceptionHandler` 是一个 `@ControllerAdvice` 集中处理系统异常。

2、处理以下异常。如果系统出现以下异常，会被 SpringBoot 支持以 `RFC 7807` 规范方式返回错误数据。

```java
@ExceptionHandler({
    // 请求方式不支持
    HttpRequestMethodNotSupportedException.class,
    // 媒体类型不支持
    HttpMediaTypeNotSupportedException.class,
    // 不支持的媒体类型（客户端可接受的）
    HttpMediaTypeNotAcceptableException.class,
    // 路径参数缺失
    MissingPathVariableException.class,
    // 请求参数缺失
    MissingServletRequestParameterException.class,
    // 请求部分缺失
    MissingServletRequestPartException.class,
    // 请求绑定异常
    ServletRequestBindingException.class,
    // 方法参数验证失败
    MethodArgumentNotValidException.class,
    // 未找到处理器
    NoHandlerFoundException.class,
    // 异步请求超时
    AsyncRequestTimeoutException.class,
    // 错误响应异常
    ErrorResponseException.class,
    // 转换不支持
    ConversionNotSupportedException.class,
    // 类型不匹配
    TypeMismatchException.class,
    // HTTP消息不可读
    HttpMessageNotReadableException.class,
    // HTTP消息不可写
    HttpMessageNotWritableException.class,
    // 绑定异常
    BindException.class
})
```

效果：

默认响应错误的 json。状态码 405。

```java
{
  "timestamp": "2023-09-18T11:13:05.515+00:00",
  "status": 405,
  "error": "Method Not Allowed",
  "trace": "org.springframework.web.HttpRequestMethodNotSupportedException: Request method 'POST' is not supported\r\n\tat org.springframework.web.servlet.mvc.method.RequestMappingInfoHandlerMapping.handleNoMatch(RequestMappingInfoHandlerMapping.java:265)\r\n\tat org.springframework.web.servlet.handler.AbstractHandlerMethodMapping.lookupHandlerMethod(AbstractHandlerMethodMapping.java:441)\r\n\tat org.springframework.web.servlet.handler.AbstractHandlerMethodMapping.getHandlerInternal(AbstractHandlerMethodMapping.java:382)\r\n\tat org.springframework.web.servlet.mvc.method.RequestMappingInfoHandlerMapping.getHandlerInternal(RequestMappingInfoHandlerMapping.java:126)\r\n\tat org.springframework.web.servlet.mvc.method.RequestMappingInfoHandlerMapping.getHandlerInternal(RequestMappingInfoHandlerMapping.java:68)\r\n\tat org.springframework.web.servlet.handler.AbstractHandlerMapping.getHandler(AbstractHandlerMapping.java:505)\r\n\tat org.springframework.web.servlet.DispatcherServlet.getHandler(DispatcherServlet.java:1275)\r\n\tat org.springframework.web.servlet.DispatcherServlet.doDispatch(DispatcherServlet.java:1057)\r\n\tat org.springframework.web.servlet.DispatcherServlet.doService(DispatcherServlet.java:974)\r\n\tat org.springframework.web.servlet.FrameworkServlet.processRequest(FrameworkServlet.java:1011)\r\n\tat org.springframework.web.servlet.FrameworkServlet.doPost(FrameworkServlet.java:914)\r\n\tat jakarta.servlet.http.HttpServlet.service(HttpServlet.java:563)\r\n\tat org.springframework.web.servlet.FrameworkServlet.service(FrameworkServlet.java:885)\r\n\tat jakarta.servlet.http.HttpServlet.service(HttpServlet.java:631)\r\n\tat org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:205)\r\n\tat org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:149)\r\n\tat org.apache.tomcat.websocket.server.WsFilter.doFilter(WsFilter.java:53)\r\n\tat org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:174)\r\n\tat org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:149)\r\n\tat org.springframework.web.filter.RequestContextFilter.doFilterInternal(RequestContextFilter.java:100)\r\n\tat org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:116)\r\n\tat org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:174)\r\n\tat org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:149)\r\n\tat org.springframework.web.filter.FormContentFilter.doFilterInternal(FormContentFilter.java:93)\r\n\tat org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:116)\r\n\tat org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:174)\r\n\tat org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:149)\r\n\tat org.springframework.web.filter.CharacterEncodingFilter.doFilterInternal(CharacterEncodingFilter.java:201)\r\n\tat org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:116)\r\n\tat org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:174)\r\n\tat org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:149)\r\n\tat org.apache.catalina.core.StandardWrapperValve.invoke(StandardWrapperValve.java:166)\r\n\tat org.apache.catalina.core.StandardContextValve.invoke(StandardContextValve.java:90)\r\n\tat org.apache.catalina.authenticator.AuthenticatorBase.invoke(AuthenticatorBase.java:493)\r\n\tat org.apache.catalina.core.StandardHostValve.invoke(StandardHostValve.java:115)\r\n\tat org.apache.catalina.valves.ErrorReportValve.invoke(ErrorReportValve.java:93)\r\n\tat org.apache.catalina.core.StandardEngineValve.invoke(StandardEngineValve.java:74)\r\n\tat org.apache.catalina.connector.CoyoteAdapter.service(CoyoteAdapter.java:341)\r\n\tat org.apache.coyote.http11.Http11Processor.service(Http11Processor.java:390)\r\n\tat org.apache.coyote.AbstractProcessorLight.process(AbstractProcessorLight.java:63)\r\n\tat org.apache.coyote.AbstractProtocol$ConnectionHandler.process(AbstractProtocol.java:894)\r\n\tat org.apache.tomcat.util.net.NioEndpoint$SocketProcessor.doRun(NioEndpoint.java:1741)\r\n\tat org.apache.tomcat.util.net.SocketProcessorBase.run(SocketProcessorBase.java:52)\r\n\tat org.apache.tomcat.util.threads.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1191)\r\n\tat org.apache.tomcat.util.threads.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:659)\r\n\tat org.apache.tomcat.util.threads.TaskThread$WrappingRunnable.run(TaskThread.java:61)\r\n\tat java.base/java.lang.Thread.run(Thread.java:833)\r\n",
  "message": "Method 'POST' is not supported.",
  "path": "/list"
}
```

开启 ProblemDetails 返回，使用新的 MediaType  
`Content-Type: application/problem+json` + 额外扩展返回。

##### 2.12.2 函数式 Web
> **SpringMVC 5.2** 以后允许使用**函数式**的方式，**定义 Web 的请求处理流程**。
>
> 函数式接口。
>
> Web 请求处理的方式：
>
> 1、**@Controller + @RequestMapping**：**耦合式**（**路由、业务**耦合）。
>
> 2、**函数式 Web**：分离式（路由、业务分离）。
>

###### 2.12.2.1 场景
> 场景：User RESTful - CRUD。
>

+ GET /user/1 获取 1 号用户。
+ GET /users 获取所有用户。
+ POST /user **请求体**携带 JSON，新增一个用户。
+ PUT /user/1 **请求体**携带 JSON，修改 1 号用户。
+ DELETE /user/1 **删除** 1 号用户。

###### 2.12.2.2 核心类
+ **RouterFunction**
+ **RequestPredicate**
+ **ServerRequest**
+ **ServerResponse**

###### 2.12.2.3 示例
```java
/**
 * @author MYXH
 * @date 2023/9/18
 * @description
 * 场景：User RESTful - CRUD
 * GET /user/1 获取 1 号用户
 * GET /users 获取所有用户
 * POST /user 请求体携带 JSON，新增一个用户
 * PUT /user/1 请求体携带 JSON，修改 1 号用户
 * DELETE /user/1 删除 1 号用户
 */
@Configuration
public class WebFunctionConfig
{
    /**
     * 函数式 Web
     * 1、给容器中放一个 Bean: 类型是 RouterFunction<ServerResponse>，集中所有路由信息
     * 2、每个业务准备一个自己的 Handler
     * <br>
     * 核心四大对象
     * 1、RouterFunction：定义路由信息，发什么请求，谁来处理
     * 2、RequestPredicate：定义请求，请求谓语，请求方式（GET、POST），请求参数
     * 3、ServerRequest：封装请求完整数据
     * 4、ServerResponse：封装响应完整数据
     *
     * @param userBizHandler 用户业务处理程序（userBizHandler 会被自动注入进来）
     * @return routerFunction 路由器功能
     */
    @Bean
    public RouterFunction<ServerResponse> userRouter(UserBizHandler userBizHandler)
    {
        // 开始定义路由信息
        RouterFunction<ServerResponse> routerFunction = RouterFunctions.route()
                .GET("/user/{id}", RequestPredicates.accept(MediaType.ALL), userBizHandler::getUser)
                .GET("/users", userBizHandler::getUsers)
                .POST("/user", RequestPredicates.accept(MediaType.APPLICATION_JSON), userBizHandler::saveUser)
                .PUT("/user{id}", RequestPredicates.accept(MediaType.APPLICATION_JSON), userBizHandler::updateUser)
                .DELETE("/user{id}", userBizHandler::deleteUser)
                .build();

        return routerFunction;
    }
}
```

```java
//专门处理 User 有关的业务
@Slf4j
@Service
public class UserBizHandler
{
    @Autowired
    private User user;

    /**
     * 查询指定 id 的用户
     *
     * @param request 请求
     * @return response 响应
     * @throws Exception 异常
     */
    public ServerResponse getUser(ServerRequest request) throws Exception
    {
        // 业务处理
        String id = request.pathVariable("id");

        user.setId(1L);
        user.setUserName("MYXH");
        user.setPassword("520.ILY!");
        user.setAge(21);
        user.setEmail("1735350920@qq.com");

        log.info("查询 {} 号用户信息成功", id);

        // 构造响应
        ServerResponse response = ServerResponse
        .ok()
        // 凡是 body 中的对象，就是以前 @ResponseBody 原理，利用 HttpMessageConverter 写出为 json
        .body(user);

        return response;
    }

    /**
     * 获取所有用户
     *
     * @param request 请求
     * @return response 响应
     * @throws Exception 异常
     */
    public ServerResponse getUsers(ServerRequest request) throws Exception
    {
        // 业务处理
        List<User> userList = Arrays.asList(
            new User(1L, "MYXH","520.ILY!",21,"1735350920@qq.com","root"),
            new User(2L, "root","000000",21,"root@qq.com","root"),
            new User(3L, "admin","123456",21,"admin@qq.com","admin"),
            new User(4L, "test","test",18,"test@qq.com","test"),
            new User(5L, "张三","123456",18,"","user")
        );

        log.info("查询所有用户信息成功");

        // 构造响应
        ServerResponse response = ServerResponse
        .ok()
        .body(userList);

        return response;
    }

    /**
     * 保存用户
     *
     * @param request 请求
     * @return response 响应
     * @throws Exception 异常
     */
    public ServerResponse saveUser(ServerRequest request) throws Exception
    {
        // 业务处理
        // 提取请求体
        User body = request.body(User.class);

        log.info("保存用户信息成功，用户信息：{}", body);

        // 构造响应
        ServerResponse response = ServerResponse
        .ok()
        .build();

        return response;
    }

    /**
     * 更新指定 id 的用户
     *
     * @param request 请求
     * @return response 响应
     * @throws Exception 异常
     */
    public ServerResponse updateUser(ServerRequest request) throws Exception
    {
        // 业务处理
        // 提取请求体
        User body = request.body(User.class);

        log.info("更新用户信息成功，用户信息：{}", body);

        // 构造响应
        ServerResponse response = ServerResponse
        .ok()
        .build();

        return response;
    }

    /**
     * 删除指定 id 的用户
     *
     * @param request 请求
     * @return response 响应
     * @throws Exception 异常
     */
    public ServerResponse deleteUser(ServerRequest request) throws Exception
    {
        // 业务处理
        String id = request.pathVariable("id");

        log.info("删除 {} 号用户信息成功", id);

        // 构造响应
        ServerResponse response = ServerResponse
        .ok()
                .build();

        return response;
    }
}
```

### 第 3 章 SpringBoot3-数据访问
**整合 SSM 场景**

> SpringBoot 整合 **Spring**、**SpringMVC**、**MyBatis** 进行**数据访问场景**开发。
>

#### 3.1 创建 SSM 整合项目
```java
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>3.1.4</version>
    <relativePath/> 
  </parent>
  <groupId>com.myxh.springboot</groupId>
  <artifactId>boot3-05-ssm</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <name>boot3-05-ssm</name>
  <description>boot3-05-ssm</description>
  <properties>
    <java.version>17</java.version>
  </properties>
  <dependencies>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
      <groupId>org.mybatis.spring.boot</groupId>
      <artifactId>mybatis-spring-boot-starter</artifactId>
      <version>3.0.2</version>
    </dependency>

    <dependency>
      <groupId>com.mysql</groupId>
      <artifactId>mysql-connector-j</artifactId>
      <scope>runtime</scope>
    </dependency>
    <dependency>
      <groupId>org.projectlombok</groupId>
      <artifactId>lombok</artifactId>
      <optional>true</optional>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-test</artifactId>
      <scope>test</scope>
    </dependency>
    <dependency>
      <groupId>org.mybatis.spring.boot</groupId>
      <artifactId>mybatis-spring-boot-starter-test</artifactId>
      <version>3.0.2</version>
      <scope>test</scope>
    </dependency>
  </dependencies>

  <build>
    <plugins>
      <plugin>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-maven-plugin</artifactId>
        <configuration>
          <excludes>
            <exclude>
              <groupId>org.projectlombok</groupId>
              <artifactId>lombok</artifactId>
            </exclude>
          </excludes>
        </configuration>
      </plugin>
    </plugins>
  </build>

</project>
```

#### 3.2 配置数据源
```java
# 1、先配置数据源信息
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
spring.datasource.type=com.zaxxer.hikari.HikariDataSource
spring.datasource.url=jdbc:mysql://localhost:3306/spring_boot
spring.datasource.username=MYXH
spring.datasource.password=520.ILY!
```

安装 Free MyBatis Tool 或 MyBatisX 插件，生成 Mapper 接口的 xml 文件即可。

#### 3.3 配置 MyBatis
```java
# 2、配置整合 MyBatis
mybatis.mapper-locations=classpath:/mapper/*.xml

# 打开驼峰命名规则
mybatis.configuration.map-underscore-to-camel-case=true
```

#### 3.4 CRUD 编写
+ 编写 Bean。
+ 编写 Mapper。
+ 使用 `Free MyBatis Tool` 插件，快速生成 MapperXML。
+ 测试 CRUD。

#### 3.5 自动配置原理
**SSM 整合总结**：

> 1、**导入****mybatis-spring-boot-starter**。
>
> 2、配置**数据源**信息。
>
> 3、配置 MyBatis 的 **Mapper 接口扫描**与 **xml 映射文件扫描**。
>
> 4、编写 Bean，Mapper，生成 xml，编写 SQL 进行 CRUD。**事务等操作依然和 Spring 中用法一样**。
>
> 5、效果：
>
> + ① 所有 SQL 写在 xml 中。
> + ② 所有 **MyBatis 配置**写在 **application.properties** 下面。
>

+ `jdbc 场景的自动配置`：
    - `mybatis-spring-boot-starter` 导入 `spring-boot-starter-jdbc`，jdbc 是操作数据库的场景。
    - `jdbc` 场景的几个自动配置。
        * org.springframework.boot.autoconfigure.jdbc.**DataSourceAutoConfiguration**
            + **数据源的自动配置**。
            + 所有和数据源有关的配置都绑定在 `DataSourceProperties`。
            + 默认使用 `HikariDataSource`。
        * org.springframework.boot.autoconfigure.jdbc.**JdbcTemplateAutoConfiguration**
            + 给容器中放了 `JdbcTemplate` 操作数据库。
        * org.springframework.boot.autoconfigure.jdbc.**JndiDataSourceAutoConfiguration**
        * org.springframework.boot.autoconfigure.jdbc.**XADataSourceAutoConfiguration**
            + **基于 XA 二阶提交协议的分布式事务数据源**。
        * org.springframework.boot.autoconfigure.jdbc.**DataSourceTransactionManagerAutoConfiguration**
            + **支持事务**。
    - **具有的底层能力**：**数据源**、`JdbcTemplate`、**事务**。
+ `MyBatisAutoConfiguration`：配置了 MyBatis 的整合流程。
    - `mybatis-spring-boot-starter` 导入 `mybatis-spring-boot-autoconfigure（mybatis 的自动配置包）`。
    - 默认加载两个自动配置类：
        * org.mybatis.spring.boot.autoconfigure.MybatisLanguageDriverAutoConfiguration
        * org.mybatis.spring.boot.autoconfigure.**MybatisAutoConfiguration**
            + **必须在数据源配置好之后才配置**。
            + 给容器中 `SqlSessionFactory` 组件，创建和数据库的一次会话。
            + 给容器中 `SqlSessionTemplate` 组件，操作数据库。
    - **MyBatis 的所有配置绑定在**`MybatisProperties`。
    - 每个 **Mapper 接口**的**代理对象**是怎么创建放到容器中，详见 **@MapperScan** 原理：
        * 利用 `@Import(MapperScannerRegistrar.class)` 批量给容器中注册组件，解析指定的包路径里面的每一个类，为每一个 Mapper 接口类，创建 Bean 定义信息，注册到容器中。

> 如何分析哪个场景导入以后，开启了哪些自动配置类。
>
> 寻找：**classpath:/META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports** 文件中配置的所有值，就是要开启的自动配置类，但是每个类可能有条件注解，基于条件注解判断哪个自动配置类生效了。
>

#### 3.6 快速定位生效的配置
```java
# 开启调试模式，详细打印开启了哪些自动配置
# Positive（生效的自动配置），Negative（不生效的自动配置）
debug=true
```

#### 3.7 扩展：整合其他数据源
##### 3.7.1 Druid 数据源
> 暂不支持 **SpringBoot3**
>
> + 导入 **druid-starter**。
> + 写配置。
> + 分析自动配置了哪些东西，怎么用。
>

Druid 官网：https://github.com/alibaba/druid

```java
# 数据源基本配置
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
spring.datasource.type=com.alibaba.druid.pool.DruidDataSource
spring.datasource.url=jdbc:mysql://localhost:3306/spring_boot
spring.datasource.username=MYXH
spring.datasource.password=520.ILY!

# 配置 StatFilter 监控
spring.datasource.druid.filter.stat.enabled=true
spring.datasource.druid.filter.stat.db-type=mysql
spring.datasource.druid.filter.stat.log-slow-sql=true
spring.datasource.druid.filter.stat.slow-sql-millis=2000

# 配置 WallFilter 防火墙
spring.datasource.druid.filter.wall.enabled=true
spring.datasource.druid.filter.wall.db-type=mysql
spring.datasource.druid.filter.wall.config.delete-allow=false
spring.datasource.druid.filter.wall.config.drop-table-allow=false

# 配置监控页，内置监控页面的首页是 /druid/index.html
spring.datasource.druid.stat-view-servlet.enabled=true
spring.datasource.druid.stat-view-servlet.login-username=admin
spring.datasource.druid.stat-view-servlet.login-password=admin
spring.datasource.druid.stat-view-servlet.allow=\*

# 其他 Filter 配置不再演示

# 目前为以下 Filter 提供了配置支持，请参考文档或者根据 IDE 提示（spring.datasource.druid.filter.\*）进行配置。
# StatFilter
# WallFilter
# ConfigFilter
# EncodingConvertFilter
# Slf4jLogFilter
# Log4jFilter
# Log4j2Filter
# CommonsLogFilter
```

#### 3.8 附录：示例数据库
```java
# 创建数据库 spring_boot
CREATE DATABASE IF NOT EXISTS spring_boot;

# 选择数据库 spring_boot
USE spring_boot;

# 创建用户表 t_user

CREATE TABLE `t_user`
(
  `id`         BIGINT(20)   NOT NULL AUTO_INCREMENT COMMENT '编号',
  `login_name` VARCHAR(200) NULL DEFAULT NULL COMMENT '用户名称' COLLATE 'utf8_general_ci',
  `nick_name`  VARCHAR(200) NULL DEFAULT NULL COMMENT '用户昵称' COLLATE 'utf8_general_ci',
  `password`   VARCHAR(200) NULL DEFAULT NULL COMMENT '用户密码' COLLATE 'utf8_general_ci',
  PRIMARY KEY (`id`)
);

# 添加用户表 t_user 的数据
INSERT INTO `t_user`(`id`, `login_name`, `nick_name`, `password`)
VALUES (1, 'MYXH', '末影小黑xh', '520.ILY!'),
(2, 'root', 'root 用户', '000000'),
(3, 'admin', '管理员', '123456'),
(4, 'test', '测试员', 'test');
```

### 第 4 章 SpringBoot3-基础特性
#### 4.1 SpringApplication
##### 4.1.1 自定义 banner
1、类路径添加 `banner.txt` 或设置 `spring.banner.location` 就可以定制 banner。

2、推荐网站：链接: [Spring Boot banner 在线生成工具](https://www.bootschool.net/ascii)，制作下载英文 banner.txt，修改替换 banner.txt 文字实现自定义，个性化启动。

##### 4.1.2 自定义 SpringApplication
```java
/**
 * 主程序类
*/
@SpringBootApplication
public class Boot306FeaturesApplication{
    public static void main(String[] args){
        // 1、SpringApplication：Boot 应用的核心 API 入口
        // SpringApplication.run(Boot306FeaturesApplication.class, args);

        // 1.1 自定义 SpringApplication 的底层设置
        SpringApplication springApplication = new SpringApplication(Boot306FeaturesApplication.class);

        // 1.2 程序化调整 SpringApplication 的参数，配置文件优先级高于程序化调整的优先级
        springApplication.setBannerMode(Banner.Mode.OFF);

        // 1.3 SpringApplication 运行起来
        springApplication.run(args);
    }
}
```

##### 4.1.3 FluentBuilder API
```java
@SpringBootApplication
public class Boot306FeaturesApplication
{
    public static void main(String[] args)
    {
        // 1、SpringApplication：Boot 应用的核心 API 入口
        // SpringApplication.run(Boot306FeaturesApplication.class, args);

        // 1.1 自定义 SpringApplication 的底层设置
        // SpringApplication springApplication = new SpringApplication(Boot306FeaturesApplication.class);

        // 1.2 程序化调整 SpringApplication 的参数，配置文件优先级高于程序化调整的优先级
        // springApplication.setBannerMode(Banner.Mode.OFF);

        // 1.3 SpringApplication 运行起来
        // springApplication.run(args);

        // 2、Builder 方式构建 SpringApplication：通过 FluentAPI 进行设置
        ConfigurableApplicationContext applicationContext = new SpringApplicationBuilder()
                .main(Boot306FeaturesApplication.class)
                .sources(Boot306FeaturesApplication.class)
                .bannerMode(Banner.Mode.OFF)
                .run(args);
    }
}
```

#### 4.2 Profiles
> 环境隔离能力，快速切换开发、测试、生产环境。
>
> 步骤：
>
> 1、**标识环境**：指定哪些组件、配置在哪个环境生效。
>
> 2、**切换环境**：这个环境对应的所有组件和配置就应该生效。
>

##### 4.2.1 使用
###### 4.2.1.1 指定环境
+ Spring Profiles 提供一种**隔离配置**的方式，使其仅在**特定环境**生效。
+ 任何 `@Component`，`@Configuration` 或 `@ConfigurationProperties` 可以使用 `@Profile` 标记，来指定何时被加载。（**容器中的组件**都可以被 `@Profile` 标记）

###### 4.2.1.2 环境激活
1、配置激活指定环境，配置文件。

```java
# 激活指定的一个或多个环境
spring.profiles.active=dev, test
```

2、也可以使用命令行激活。`--spring.profiles.active=dev, test`。

3、还可以配置**默认环境**，不标注 @Profile 的组件永远都存在。

+ ① 以前默认环境叫 default。
+ ② `spring.profiles.default=dev`。

4、推荐使用激活方式激活指定环境。

###### 4.2.1.3 环境包含
注意：

1、`spring.profiles.active` 和 `spring.profiles.default` 只能用到**无 profile 的文件**中，如果在 `application-dev.properties/yaml` 中编写就是无效的。

2、也可以额外添加生效文件，而不是激活替换。比如：

```java
# 包含指定环境，不管激活哪个环境，这个环境都有，总是要生效的环境
spring.profiles.include=dev, test
```

最佳实战：

+ **生效的环境 = 激活的环境/默认环境 + 包含的环境**。
+ 项目里面这么用。
    - 基础的配置 `mybatis`、`log`：写到**包含环境中**。
    - 需要动态切换变化的 `db`、`redis`：写到**激活的环境中**。

##### 4.2.2 Profile 分组
创建 `prod` 组，指定包含 `db` 和 `mq` 配置。

```java
# Profile 分组
spring.profiles.group.prod[0]=db
spring.profiles.group.prod[1]=mq
```

使用 `--spring.profiles.active=prod`，就会激活 `prod`，`db`，`mq` 配置文件。

##### 4.2.3 Profile 配置文件
+ `application-{profile}.properties` 可以作为**指定环境的配置文件**。
+ 激活这个环境，**配置**就会生效。最终生效的所有**配置**是。
    - `application.properties`：主配置文件，任意时候都生效。
    - `application-{profile}.properties`：指定环境配置文件，激活指定环境生效。

profile 优先级 > application 优先级。

#### 4.3 外部化配置
> **场景**：线上应用如何**快速修改配置**，并**应用最新配置**？
>
> SpringBoot 使用**配置优先级 + 外部配置**简化配置更新、简化运维。
>
> 只需要给 **jar** 应用所在的文件夹放一个 **application.properties** 最新配置文件，重启项目就能自动应用最新配置。
>

##### 4.3.1 配置优先级
Spring Boot 允许将**配置外部化**，以便可以在不同的环境中使用相同的应用程序代码。

可以使用各种**外部配置源**，包括 `Java Properties 文件`、`YAML 文件`、`环境变量`和`命令行参数`。

`@Value` 可以获取值，也可以用 `@ConfigurationProperties` 将所有属性绑定到 `Java Object` 中。

**以下是 SpringBoot 属性源加载顺序**，**后面的会覆盖前面的值**，**由低到高，高优先级配置覆盖低优先级**。

1、**默认属性**（通过 `SpringApplication.setDefaultProperties` 指定的）。

2、`@PropertySource` 指定加载的配置（需要写在`@Configuration` 类上才可生效）。

3、**配置文件**（**application.properties/yml 等**）。

4、`RandomValuePropertySource` 支持的 `random.*` 配置（如：@Value(“${random.int}”)）。

5、OS 环境变量。

6、Java 系统属性（`System.getProperties()`）。

7、JNDI 属性（来自 `java:comp/env`）。

`8、ServletContext` 初始化参数。

`9、ServletConfig` 初始化参数。

10、`SPRING_APPLICATION_JSON` 属性（内置在环境变量或系统属性中的 JSON）。

11、**命令行参数**。

12、测试属性。（`@SpringBootTest` 进行测试时指定的属性）。

13、测试类 `@TestPropertySource` 注解。

14、Devtools 设置的全局属性。（`$HOME/.config/spring-boot`）。

> 结论：配置可以写到很多位置，常见的优先级顺序：
>
> + **命令行** > **配置文件** > **springapplication 配置**
>

**配置文件优先级**如下：（**后面覆盖前面**）

1、**jar 包内**的 `application.properties/yml`。

2、**jar 包内**的 `application-{profile}.properties/yml`。

3、**jar 包外**的 `application.properties/yml`。

4、**jar 包外**的 `application-{profile}.properties/yml`。

**建议**：**用一种格式的配置文件**。**如果**`.properties`**和**`.yml`**同时存在，则**`.properties`**优先**。

> 结论：**包外 > 包内**，同级情况：**profile 配置 > application 配置**。
>

**所有参数均可由命令行传入**，**使用**`--参数项=参数值`，**将会被添加到环境变量中**，**并优先于**`配置文件`。

**比如**`java -jar app.jar --name="Spring"`，**可以使用**`@Value("${name}")`**获取**。

演示场景：

+ 包内：application.properties `server.port=8080`。
+ 包内：application-dev.properties `server.port=8081`
+ 包外：application.properties `server.port=8180`
+ 包外：application-dev.properties `server.port=8181`

启动端口？：`命令行` > `8181` > `8180` > `8081` > `8080`。

##### 4.3.2 外部配置
SpringBoot 应用启动时会自动寻找 `application.properties` 和 `application.yaml` 位置，进行加载。顺序如下：（**后面覆盖前面**）

1、类路径：内部。

+ ① 类根路径.
+ ② 类下 `/config` 包。

2、当前路径：项目所在的位置。

+ ① 当前路径。
+ ② 当前下 `/config` 子目录。
+ ③ `/config` 目录的直接子目录。

最终效果：优先级由高到低，前面覆盖后面。

+ 命令行 > 包外 config 直接子目录 > 包外 config 目录 > 包外根目录 > 包内目录。
+ 同级比较：
    - profile 配置 > 默认配置。
    - properties 配置 > yaml 配置。

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1755192536739-2bfde630-69d2-485f-a283-5895520e9ade.png)

规律：最外层的最优先。

+ 命令行 > 所有。
+ 包外 > 包内。
+ config 目录 > 根目录。
+ profile > application。

配置不同就都生效（互补），配置相同高优先级覆盖低优先级。

##### 4.3.3 导入配置
使用 `spring.config.import` 可以导入额外配置。

```java
# 导入指定的配置
spring.config.import=classpath:/a.properties

# 导入配置优先级低于配置文件的优先级
a=c
```

无论以上写法的先后顺序，`a.properties` 的值总是优先于直接在文件中编写的 a。

##### 4.3.4 属性占位符
配置文件中可以使用 `${name:default}` 形式取出之前配置过的值。

```java
server.port=8080

my.server.port=我的服务端口是：${server.port}，我的用户名是：${my.username:黄某某}
```

#### 4.4 单元测试 JUnit5
##### 4.4.1 整合
SpringBoot 提供一系列测试工具集及注解方便进行测试。

`spring-boot-test` 提供核心测试能力，`spring-boot-test-autoconfigure` 提供测试的一些自动配置。

只需要导入 `spring-boot-starter-test` 即可整合测试。

```java
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-test</artifactId>
  <scope>test</scope>
</dependency>
```

`spring-boot-starter-test` 默认提供了以下库供测试使用。

+ [JUnit 5](https://junit.org/junit5/)
+ [Spring Test](https://docs.spring.io/spring-framework/docs/6.0.4/reference/html/testing.html#integration-testing)
+ [AssertJ](https://assertj.github.io/doc/)
+ [Hamcrest](https://github.com/hamcrest/JavaHamcrest)
+ [Mockito](https://site.mockito.org/)
+ [JSONassert](https://github.com/skyscreamer/JSONassert)
+ [JsonPath](https://github.com/json-path/JsonPath)

##### 4.4.2 测试
###### 4.4.2.1 组件测试
直接 `@Autowired` 容器中的组件进行测试。

###### 4.4.2.2 注解
JUnit5 的注解与 JUnit4 的注解有所变化。

https://junit.org/junit5/docs/current/user-guide/#writing-tests-annotations

+ **@Test**：表示方法是测试方法，但是与 JUnit4 的 @Test 不同，他的职责非常单一不能声明任何属性，拓展的测试将会由 Jupiter 提供额外测试。
+ **@ParameterizedTest**：表示方法是参数化测试，下方会有详细介绍。
+ **@RepeatedTest**：表示方法可重复执行，下方会有详细介绍。
+ **@DisplayName**：为测试类或者测试方法设置展示名称。
+ **@BeforeEach**：表示在每个单元测试之前执行。
+ **@AfterEach**：表示在每个单元测试之后执行。
+ **@BeforeAll**：表示在所有单元测试之前执行。
+ **@AfterAll**：表示在所有单元测试之后执行。
+ **@Tag**：表示单元测试类别，类似于 JUnit4 中的 @Categories。
+ **@Disabled**：表示测试类或测试方法不执行，类似于 JUnit4 中的 @Ignore。
+ **@Timeout**：表示测试方法运行如果超过了指定时间将会返回错误。
+ **@ExtendWith**：为测试类或测试方法提供扩展类引用。

```java
// 具备测试 SpringBoot 应用容器中所有组件的功能，测试类必须在主程序所在的包及其子包
@SpringBootTest
class Boot306FeaturesApplicationTests{
    // 自动注入任意组件即可测试
    @Autowired
    HelloService helloService;

    @DisplayName("测试 sum() 方法 ")
    @Test
    void contextLoads(){
        Integer sum = helloService.sum(1, 2);
        Assertions.assertEquals(3, sum);
    }

    @DisplayName("测试1")
    @Test
    public void test1(){
        System.out.println("test1");
    }

    // 所有测试方法运行之前先运行这个，只打印一次
    @BeforeAll
    static void initAll(){
        System.out.println("hello");
    }

    // 每个测试方法运行之前先运行这个，每个方法运行打印一次
    @BeforeEach
    void init(){
        System.out.println("world");
    }
}
```

###### 4.4.2.3 断言
| 方法 | 说明 |
| --- | --- |
| assertEquals | 判断两个对象或两个原始类型是否相等。 |
| assertNotEquals | 判断两个对象或两个原始类型是否不相等。 |
| assertSame | 判断两个对象引用是否指向同一个对象。 |
| assertNotSame | 判断两个对象引用是否指向不同的对象。 |
| assertTrue | 判断给定的布尔值是否为 true。 |
| assertFalse | 判断给定的布尔值是否为 false。 |
| assertNull | 判断给定的对象引用是否为 null。 |
| assertNotNull | 判断给定的对象引用是否不为 null。 |
| **assertArrayEquals** | **数组断言**。 |
| **assertAll** | **组合断言**。 |
| **assertThrows** | **异常断言**。 |
| **assertTimeout** | **超时断言**。 |
| **fail** | **快速失败**。 |


###### 4.4.2.4 嵌套测试
> JUnit 5 可以通过 Java 中的内部类和 @Nested 注解实现嵌套测试，从而可以更好的把相关的测试方法组织在一起。在内部类中可以使用 @BeforeEach 和 @AfterEach 注解，而且嵌套的层次没有限制。
>

```java
@SpringBootTest
@DisplayName("一个堆栈")
public class HelloTest{
    Stack<Object> stack;

    @Test
    @DisplayName("使用 new Stack() 实例化堆栈")
    void isInstantiatedWithNew(){
        new Stack<>();
    }

    @Nested
    @DisplayName("当新建时")
    class WhenNew{
        @BeforeEach
        void createNewStack(){
            stack = new Stack<>();
        }

        @Test
        @DisplayName("为空")
        void isEmpty(){
            assertTrue(stack.isEmpty());
        }

        @Test
        @DisplayName("在弹出时抛出 EmptyStackException 异常")
        void throwsExceptionWhenPopped(){
            assertThrows(EmptyStackException.class, stack::pop);
        }

        @Test
        @DisplayName("在查看时抛出 EmptyStackException 异常")
        void throwsExceptionWhenPeeked(){
            assertThrows(EmptyStackException.class, stack::peek);
        }

        @Nested
        @DisplayName("在推入元素后")
        class AfterPushing{

            String anElement = "一个元素";

            @BeforeEach
            void pushAnElement(){
                stack.push(anElement);
            }

            @Test
            @DisplayName("它不再为空")
            void isNotEmpty(){
                assertFalse(stack.isEmpty());
            }

            @Test
            @DisplayName("当弹出元素并且为空时返回该元素")
            void returnElementWhenPopped(){
                assertEquals(anElement, stack.pop());
                assertTrue(stack.isEmpty());
            }

            @Test
            @DisplayName("当查看元素时返回该元素，但仍然保持非空")
            void returnElementWhenPeeked(){
                assertEquals(anElement, stack.peek());
                assertFalse(stack.isEmpty());
            }
        }
    }
}
```

###### 4.4.2.5 参数化测试
参数化测试是 JUnit5 很重要的一个新特性，它使得用不同的参数多次运行测试成为了可能，也为单元测试带来许多便利。

利用 @**ValueSource** 等注解，指定入参，将可以使用不同的参数进行多次单元测试，而不需要每新增一个参数就新增一个单元测试，省去了很多冗余代码。

@**ValueSource**：为参数化测试指定入参来源，支持八大基础类以及 String 类型，Class 类型。

@NullSource：表示为参数化测试提供一个 null 的入参。

@EnumSource：表示为参数化测试提供一个枚举入参。

@CsvFileSource：表示读取指定 CSV 文件内容作为参数化测试入参。

@MethodSource：表示读取指定方法的返回值作为参数化测试入参（注意方法返回需要是一个流）。

```java
// 具备测试 SpringBoot 应用容器中所有组件的功能，测试类必须在主程序所在的包及其子包
@SpringBootTest
class Boot306FeaturesApplicationTests{
    // 自动注入任意组件即可测试
    @Autowired
    HelloService helloService;

    @DisplayName("测试 sum() 方法 ")
    @Test
    void contextLoads(){
        Integer sum = helloService.sum(1, 2);
        Assertions.assertEquals(3, sum);
    }

    @DisplayName("测试1")
    @Test
    public void test1(){
        System.out.println("test1");
    }

    // 参数化测试
    @ParameterizedTest
    @ValueSource(strings = {"one", "two", "three"})
    @DisplayName("参数化测试1")
    public void parameterizedTest1(String string){
        System.out.println(string);
        Assertions.assertTrue(StringUtils.isNotBlank(string));
    }

    @ParameterizedTest
    // 指定方法名，返回值就是测试用的参数
    @MethodSource("method")
    @DisplayName("方法来源参数")
    public void testWithExplicitLocalMethodSource(String name){
        System.out.println(name);
        Assertions.assertNotNull(name);
    }

    static Stream<String> method(){
        // 返回 Stream 则可
        return Stream.of("apple", "banana");
    }

    // 所有测试方法运行之前先运行这个，只打印一次
    @BeforeAll
    static void initAll(){
        System.out.println("hello");
    }

    // 每个测试方法运行之前先运行这个，每个方法运行打印一次
    @BeforeEach
    void init(){
        System.out.println("world");
    }

```

### 第 5 章 SpringBoot3-核心原理
#### 5.1 事件和监听器
##### 5.1.1 生命周期监听
场景：监听**应用**的**生命周期**。

###### 5.1.1.1 监听器 SpringApplicationRunListener
1、自定义 `SpringApplicationRunListener` 来**监听事件**。

+ ① 编写 `SpringApplicationRunListener`**实现类**。

```java
/**
 * 1、引导：利用 BootstrapContext 引导整个项目启动
 *         starting：               应用开始，SpringApplication 的 run 方法一调用，只要有了 BootstrapContext 就执行
 *         environmentPrepared：    环境准备好（把启动参数等绑定到环境变量中），但是 ioc 还没有创建（调一次）
 * 2、启动：
 *        contextPrepared：         ioc 容器创建并准备好，但是 sources（主配置类）没加载，并关闭引导上下文，组件都没创建（调一次）
 *        contextLoaded：           ioc 容器加载，主配置类加载进去了，但是 ioc 容器还没刷新（bean 没创建）
 *        ----- 截止现在，ioc 容器里面还没造 bean -----
 *        started：                 ioc 容器刷新了（所有 bean 造好了），但是 runner 没调用
 *        ready：                   ioc 容器刷新了（所有 bean 造好了），所有 runner 调用完了
 * 3、运行：
 *       以上步骤都正确执行，代表容器 running。
 */
public class MyAppListener implements SpringApplicationRunListener{
    @Override
    public void starting(ConfigurableBootstrapContext bootstrapContext){
        System.out.println("--------------- starting() 正在启动 ---------------");
    }

    @Override
    public void environmentPrepared(ConfigurableBootstrapContext bootstrapContext, ConfigurableEnvironment environment){
        System.out.println("--------------- environmentPrepared() 环境准备完成 ---------------");
    }

    @Override
    public void contextPrepared(ConfigurableApplicationContext context){
        System.out.println("--------------- contextPrepared() ioc 容器准备完成 ---------------");
    }

    @Override
    public void contextLoaded(ConfigurableApplicationContext context){
        System.out.println("--------------- contextLoaded() ioc 容器加载完成 ---------------");
    }

    @Override
    public void started(ConfigurableApplicationContext context, Duration timeTaken){
        System.out.println("--------------- started() 启动完成 ---------------");
    }

    @Override
    public void ready(ConfigurableApplicationContext context, Duration timeTaken){
        System.out.println("--------------- ready() 准备就绪 ---------------");
    }

    @Override
    public void failed(ConfigurableApplicationContext context, Throwable exception){
        System.out.println("--------------- failed() 应用启动失败 ---------------");
    }
}
```

<font style="color:rgb(0, 0, 0);">代码解析：</font>

`<font style="color:rgba(0, 0, 0, 0.85) !important;">SpringApplicationRunListener</font>`<font style="color:rgba(0, 0, 0, 0.85) !important;"> 是 Spring Boot 特有的监听器接口，其方法对应应用启动的不同阶段</font>

```java
@Component // 注册为Spring组件
public class MyListener implements ApplicationListener<ApplicationEvent>{
    @Override
    public void onApplicationEvent(ApplicationEvent event){
        System.out.println("-------- 事件到达 --------");
        System.out.println("event = " + event);
        System.out.println("------------------------");
    }
}
```

<font style="color:rgb(0, 0, 0);">代码解析：</font>

1. **<font style="color:rgb(0, 0, 0) !important;">实现接口</font>**<font style="color:rgba(0, 0, 0, 0.85) !important;">：</font>`<font style="color:rgba(0, 0, 0, 0.85) !important;">MyListener</font>`<font style="color:rgba(0, 0, 0, 0.85) !important;"> </font><font style="color:rgba(0, 0, 0, 0.85) !important;">实现了</font><font style="color:rgba(0, 0, 0, 0.85) !important;"> </font>`<font style="color:rgba(0, 0, 0, 0.85) !important;">ApplicationListener<ApplicationEvent></font>`<font style="color:rgba(0, 0, 0, 0.85) !important;"> </font><font style="color:rgba(0, 0, 0, 0.85) !important;">接口，泛型指定为</font><font style="color:rgba(0, 0, 0, 0.85) !important;"> </font>`<font style="color:rgba(0, 0, 0, 0.85) !important;">ApplicationEvent</font>`<font style="color:rgba(0, 0, 0, 0.85) !important;"> </font><font style="color:rgba(0, 0, 0, 0.85) !important;">表示监听所有类型的应用事件（Spring 中所有事件的父类）。</font>
2. **<font style="color:rgb(0, 0, 0) !important;">重写方法</font>**<font style="color:rgba(0, 0, 0, 0.85) !important;">：</font>`<font style="color:rgba(0, 0, 0, 0.85) !important;">onApplicationEvent</font>`<font style="color:rgba(0, 0, 0, 0.85) !important;"> </font><font style="color:rgba(0, 0, 0, 0.85) !important;">方法是事件处理的核心，当事件发布时会被自动调用：</font>
    - <font style="color:rgba(0, 0, 0, 0.85) !important;">参数</font><font style="color:rgba(0, 0, 0, 0.85) !important;"> </font>`<font style="color:rgb(0, 0, 0);">event</font>`<font style="color:rgba(0, 0, 0, 0.85) !important;"> </font><font style="color:rgba(0, 0, 0, 0.85) !important;">是具体的事件对象（可能是上下文刷新事件、启动事件等）</font>
    - <font style="color:rgba(0, 0, 0, 0.85) !important;">代码中打印了事件到达的标记和事件详情</font>
+ ② 在 `META-INF/spring.factories` 中配置 `org.springframework.boot.SpringApplicationRunListener=自己的 Listener`，还可以指定一个**有参构造器**，接受两个参数`(SpringApplication application, String[] args)`。

```java
org.springframework.boot.SpringApplicationRunListener=\
com.myxh.springboot.core.listener.MyAppListener

org.springframework.context.ApplicationListener=\
com.myxh.springboot.core.listener.MyListener
```

+ ③ springboot 在 `spring-boot.jar` 中配置了默认的 Listener，如下。

```java
# Run Listeners
org.springframework.boot.SpringApplicationRunListener=\
org.springframework.boot.context.event.EventPublishingRunListener
```

###### 5.1.1.2 生命周期全流程
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1755192536585-496d36ac-dda2-45aa-a08f-43fea1686f0e.png)

##### 5.1.2 事件触发时机
###### 5.1.2.1 各种回调监听器
+ `BootstrapRegistryInitializer`：**感知特定阶段**，感知**引导初始化**。
    - `META-INF/spring.factories`。
    - **创建引导上下文**`bootstrapContext`**的时候触发**。
    - `application.addBootstrapRegistryInitializer();`
    - **场景**：**进行密钥校对授权**。
+ ApplicationContextInitializer：**感知特定阶段**，感知 ioc 容器初始化。
    - `META-INF/spring.factories`。
    - `application.addInitializers();`
+ **ApplicationListener**：**感知全阶段，基于事件机制，感知事件，一旦到了哪个阶段可以做别的事。**
    - `@Bean` 或 `@EventListener`：**事件驱动**。
    - `SpringApplication.addListeners(...)` 或 `SpringApplicationBuilder.listeners(...)`。
    - `META-INF/spring.factories`。
+ **SpringApplicationRunListener**：**感知全阶段生命周期 + 各种阶段都能自定义操作，功能更完善。**
    - `META-INF/spring.factories`。
+ **ApplicationRunner**：**感知特定阶段，感知应用就绪 Ready，卡死应用，就不会就绪。**
    - `@Bean`。
+ **CommandLineRunner**：**感知特定阶段，感知应用就绪 Ready，卡死应用，就不会就绪。**
    - `@Bean`。

最佳实战：

+ 如果想要在项目启动前操作：`BootstrapRegistryInitializer` 和 `ApplicationContextInitializer`。
+ 如果想要在项目启动完成后操作：**ApplicationRunner** 和 **CommandLineRunner**。
+ **如果要干涉生命周期做事**：**SpringApplicationRunListener**。
+ **如果想要用事件机制**：**ApplicationListener**。

###### 5.1.2.2 完整触发流程
**9 大事件**触发顺序和时机：

1、`ApplicationStartingEvent`：应用启动但未做任何事情, 除过注册 `listeners` 和 `initializers`。

2、`ApplicationEnvironmentPreparedEvent`：Environment 准备好，但 context 未创建。

3、`ApplicationContextInitializedEvent`：ApplicationContext 准备好，ApplicationContextInitializers 调用，但是任何 bean 未加载。

4、`ApplicationPreparedEvent`：容器刷新之前，bean 定义信息加载。

5、`ApplicationStartedEvent`：容器刷新完成， runner 未调用。

> 以下就开始插入了探针机制。
>

6、`AvailabilityChangeEvent`：`LivenessState.CORRECT` 应用存活，**存活探针**。

7、`ApplicationReadyEvent`：任何 runner 被调用。

8、`AvailabilityChangeEvent`：`ReadinessState.ACCEPTING_TRAFFIC`**就绪探针**，可以接请求。

9、`ApplicationFailedEvent`：启动出错。

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1755192536770-4389fb0e-8b2b-4455-b20e-669f02a2bcfe.png)

应用事件发送顺序如下：

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1755192536909-e4e4d477-eae2-489d-a276-6fe4c0188c6d.png)

感知应用是否**存活**了：可能植物状态，虽然活着但是不能处理请求。

应用是否**就绪**了：能响应请求，说明确实活的比较好。

###### 5.1.2.3 SpringBoot 事件驱动开发
> 应用启动过程生命周期事件感知（9 大事件）、应用运行中事件感知（无数种）。
>

+ **事件发布**：`ApplicationEventPublisherAware` 或`注入 ApplicationEventMulticaster`。
+ **事件监听**：`组件 + @EventListener`。

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1755192536861-24fa2303-dbfb-4c03-8323-4aaa3cc99336.png)

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1755192537119-96b9affc-b08b-425f-b31c-2f1472edd296.png)

> 事件发布者。
>

```java
//事件是广播出去的，所有监听这个事件的监听器都可以收到
@Service
public class EventPublisher implements ApplicationEventPublisherAware{
    // 底层发送事件用的组件，SpringBoot 会通过 ApplicationEventPublisherAware 接口自动注入
    ApplicationEventPublisher applicationEventPublisher;

    /**
     * 所有事件都可以发(类型为ApplicationEvent)
     * event 事件
     */
    public void sendEvent(ApplicationEvent event){
        // 调用底层 API 发送事件
        applicationEventPublisher.publishEvent(event);
    }
    /**
     * 会被自动调用，把真正发事件的底层组组件给注入进来
     * applicationEventPublisher 此对象要使用的事件发布者
     */
    @Override
    public void setApplicationEventPublisher(ApplicationEventPublisher 
                                             applicationEventPublisher){
        this.applicationEventPublisher = applicationEventPublisher;
    }
}
```

```java
//登录成功事件，所有事件都推荐继承 ApplicationEvent
public class LoginSuccessEvent extends ApplicationEvent{
    /**
     * 登录成功事件
     * source 事件来源，代表是谁登录成了
     */
    public LoginSuccessEvent(UserEntity source){
        super(source);
    }
}
```

> 事件订阅者。
>

```java
@Service
public class SystemService{
    @Order(1)
    @EventListener
    public void onEvent(LoginSuccessEvent event){
        System.out.println("-------- SystemService 感知到事件 --------");
        System.out.println("event = " + event);
        System.out.println("------------------------");
        UserEntity source = (UserEntity) event.getSource();
        recordLog(source.getUsername());
    }

    public void recordLog(String username){
        System.out.println(username + "登录信息已被记录");
    }
}
```

```java
@Order(2)
@Service
public class AccountService implements ApplicationListener<LoginSuccessEvent>{
    public void addAccountScore(String username){
        System.out.println(username + " 加了 1 分");
    }

    @Override
    public void onApplicationEvent(LoginSuccessEvent event){
        System.out.println("-------- AccountService  收到事件 --------");
        UserEntity source = (UserEntity) event.getSource();
        addAccountScore(source.getUsername());
    }
}
```

```java
@Service
public class CouponService{
    public CouponService(){
        System.out.println("构造器调用");
    }

    @Async
    @Order(3)
    @EventListener
    public void onEvent(LoginSuccessEvent loginSuccessEvent){
        System.out.println("-------- CouponService 感知到事件 --------");
        System.out.println("loginSuccessEvent = " + loginSuccessEvent);
        System.out.println("------------------------");
        UserEntity source = (UserEntity) loginSuccessEvent.getSource();
        sendCoupon(source.getUsername());
    }

    public void sendCoupon(String username){
        System.out.println(username + " 随机得到了一张优惠券");
    }
}
```

#### 5.2 自动配置原理
##### 5.2.1 入门理解
> 应用关注的三大核心：场景、配置、组件。
>

###### 5.2.1.1 自动配置流程
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1755192537542-4e31a381-4d6b-4b63-bd36-1ca5b5af9456.png)

1、导入 `starter`。

2、依赖导入 `autoconfigure`。

3、寻找类路径下 `META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports` 文件。

4、启动，加载所有**自动配置类**`xxxAutoConfiguration`。

+ ① 给容器中配置**功能组件**。
+ ② `组件参数`绑定到`属性类`中，`xxxProperties`。
+ ③ `属性类`和`配置文件`前缀项绑定。
+ ④ `@Contional 派生的条件注解进`行判断**是否组件生效**。

5、效果：

+ ① 修改配置文件，修改底层参数。
+ ② 所有场景自动配置好直接使用。
+ ③ 可以注入 SpringBoot 配置好的组件随时使用。

###### 5.2.1.2 SPI 机制
> + **Java 中的 SPI（Service Provider Interface）是一种软件设计模式，用于在应用程序中动态地发现和加载组件**。**SPI 的思想**是，定义一个接口或抽象类，然后通过在 classpath 中定义实现该接口的类来实现对组件的动态发现和加载。
> + **SPI** 的主要目的是解决在应用程序中使用可插拔组件的问题。例如，一个应用程序可能需要使用不同的日志框架或数据库连接池，但是这些组件的选择可能取决于运行时的条件。通过使用 **SPI**，应用程序可以在运行时发现并加载适当的组件，而无需在代码中硬编码这些组件的实现类。
> + 在 Java 中，**SPI** 的实现方式是通过在 **META-INF/services** 目录下创建一个以服务接口全限定名为名字的文件，文件中包含实现该服务接口的类的全限定名。当应用程序启动时，Java 的 **SPI** 机制会自动扫描 classpath 中的这些文件，并根据文件中指定的类名来加载实现类。
> + 通过使用 **SPI**，应用程序可以实现更灵活、可扩展的架构，同时也可以避免硬编码依赖关系和增加代码的可维护性。
>

在 SpringBoot 中，`META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports`。

> 写一段 Java 的 SPI 机制代码。
>

下面是一个简单的 Java SPI（Service Provider Interface）机制的示例代码，代码中包含了详细的注释说明：

```java
// 定义服务接口
public interface MyService{
    void execute();
}
```

```java
// 定义服务提供者 1
public class MyService1 implements MyService{
    @Override
    public void execute(){
        System.out.println("MyService1 正在执行。");
    }
}
```

```java
// 定义服务提供者 2
public class MyService2 implements MyService{
    @Override
    public void execute(){
        System.out.println("MyService2 正在执行。");
    }
}
```

```java
// 定义服务加载器
public class ServiceLoader{
    // 加载并执行服务
    public void loadAndExecuteServices(){
        // 使用Java SPI机制加载所有的服务提供者
        java.util.ServiceLoader<MyService> serviceLoader = java.util.ServiceLoader.load(MyService.class);

        // 遍历所有的服务提供者，并执行服务
        for (MyService service : serviceLoader){
            service.execute();
        }
    }
}
```

```java
// 测试代码
public class Main{
    public static void main(String[] args){
        ServiceLoader loader = new ServiceLoader();
        loader.loadAndExecuteServices();
    }
}
```

上面的代码展示了如何使用 Java SPI 机制来加载并执行服务提供者。其中，`MyService` 是服务接口，`MyService1` 和 `MyService2` 是两个具体的服务提供者实现。`ServiceLoader` 是服务加载器，它通过 `java.util.ServiceLoader` 来加载所有的服务提供者，并执行服务的 `execute()` 方法。最后，在 `Main` 类的 `main()` 方法中调用 `ServiceLoader` 的 `loadAndExecuteServices()` 方法即可加载并执行服务提供者。

注意：在实际使用 Java SPI 机制时，需要在 `src/main/resources/META-INF/services` 目录下创建一个以服务接口全限定名命名的文件，文件内容为具体的服务提供者实现类的全限定名。在本示例中，需要创建 `src/main/resources/META-INF/services/com.example.MyService` 文件，并在其中分别写入 `com.example.MyService1` 和 `com.example.MyService2`。这样，在调用 `java.util.ServiceLoader.load()` 方法时，就能正确加载到所有的服务提供者实现类。

###### 5.2.1.3 功能开关
+ 自动配置：全部都配置好，什么都不用管，自动批量导入。
    - 项目一启动，SPI 文件中指定的所有都加载。
+ `@EnableXxx`：手动控制哪些功能的开启，手动导入。
    - 开启 xxx 功能。
    - 都是利用 @Import 把此功能要用的组件导入进去。

##### 5.2.2 进阶理解
###### 5.2.2.1 @SpringBootApplication
**@****<font style="color:#DF2A3F;">SpringBootConfiguration</font>**。

就是 @Configuration，容器中的组件，配置类，Spring IOC 启动就会加载创建这个类对象。

**@****<font style="color:#DF2A3F;">EnableAutoConfiguration</font>****：开启自动配置**。

开启自动配置。

+ **@AutoConfigurationPackage：扫描主程序包，加载自己的组件**。
    - 利用 `@Import(AutoConfigurationPackages.Registrar.class)` 想要给容器中导入组件。
    - 把**主程序所在的包**的所有组件导入进来。
    - **为什么 SpringBoot 默认只扫描主程序所在的包及其子包。**
+ **@Import(AutoConfigurationImportSelector.class)：加载所有自动配置类，加载 starter 导入的组件**。
    - 扫描 SPI 文件：**META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports**。

```java
List<String> configurations = ImportCandidates.load(AutoConfiguration.class, getBeanClassLoader())
.getCandidates();
```

**<font style="color:#DF2A3F;">@ComponentScan</font>**<font style="color:#DF2A3F;"></font>

> 组件扫描：排除一些组件（哪些不要）。
>
> 排除前面已经扫描进来的**配置类**、和**自动配置类**。
>

```java
@ComponentScan(excludeFilters = {@Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
                                 @Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class)})
```

###### 5.2.2.2 完整启动加载流程
> 生命周期启动加载流程。
>

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1755192537629-37978466-13db-483d-8132-70cdb38c416a.png)

#### 5.3 自定义 starter
> 场景：**抽取聊天机器人场景，它可以打招呼**。
>
> 效果：任何项目导入此 **starter** 都具有打招呼功能，并且**问候语**中的**人名**需要可以在**配置文件**中修改。
>

+ 1、创建`自定义 starter` 项目，引入 `spring-boot-starter` 基础依赖。
+ 2、编写模块功能，引入模块所有需要的依赖。
+ 3、编写 `xxxAutoConfiguration` 自动配置类，帮其他项目导入这个模块需要的所有组件。
+ 4、编写配置文件 `META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports` 指定启动需要加载的自动配置。
+ 5、其他项目引入即可使用。

##### 5.3.1 业务代码
> 自定义配置有提示，导入以下依赖重启项目，再写配置文件就有提示。
>

```java
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-configuration-processor</artifactId>
  <optional>true</optional>
</dependency>
```

```java
@ConfigurationProperties(prefix = "robot")
@Component
@Data
// 此属性类和配置文件指定前缀绑定
public class RobotProperties{
    private String name;
    private Integer age;
    private String email;
}
```

```java
robot.name=MYXH
robot.age=21
robot.email=1735350920@qq.com
```

##### 5.3.2 基本抽取
+ 创建 starter 项目，把公共代码需要的所有依赖导入。
+ 把公共代码复制进来。
+ 自己写一个 `RobotAutoConfiguration`，给容器中导入这个场景需要的所有组件。
    - 为什么这些组件默认不会扫描进去？
    - **starter 所在的包和引入它的项目的主程序所在的包不是父子层级**。

```java
// 给容器中导入 Robot 功能要用的所有组件
@Import({RobotProperties.class, RobotController.class, RobotService.class})
@Configuration
public class RobotAutoConfiguration{
    /*
    // 把组件导入到容器中就可以
    @Bean
    public RobotController robotController(){
        return new RobotController();
    }
    */
}
```

+ 别人引用这个 `starter`，直接导入这个 `RobotAutoConfiguration`，就能把这个场景的组件导入进来。
+ 功能生效。
+ 测试编写配置文件。

##### 5.3.3 使用@EnableXxx 机制
```java
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.TYPE})
@Documented
@Import(RobotAutoConfiguration.class)
public @interface EnableRobot{
}
```

别人引入 `starter` 需要使用 `@EnableRobot` 开启功能。

##### 5.3.4 完全自动配置
+ 依赖 SpringBoot 的 SPI 机制。
+ 在需要被导入的 stater 项目中添加如下 文件`META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports`**文件中编写好自动配置类的全类名即可**。
+ **项目启动，自动加载自动配置类**。

---

## 二、Spring Boot 3-场景整合
### 1.环境准备
####  0. 云服务器  
+ 阿⾥云、腾讯云、华为云 服务器开通； 按量付费，省钱省⼼ 
+ 安装以下组件
    -  docker 
    -  redis
    -  kafka
    -  prometheus
    -  grafana  
+ https://github.com/kingToolbox/WindTerm/releases/download/2.5.0/WindTerm_2.5.0_Windows_Portable_x86_64.zip 下载windterm

**<font style="color:#DF2A3F;"> 重要：开通云服务器以后，请⼀定在安全组设置规则，放⾏端⼝ </font>**

**<font style="color:#DF2A3F;"> 重要：开通云服务器以后，请⼀定在安全组设置规则，放⾏端⼝ </font>**

**<font style="color:#DF2A3F;"> 重要：开通云服务器以后，请⼀定在安全组设置规则，放⾏端⼝  </font>**

####  1. Docker安装  
```java
sudo yum install -y yum-utils
sudo yum-config-manager \--add-repo \
https://download.docker.com/linux/centos/docker-ce.repo
sudo yum install docker-ce docker-ce-cli containerd.io docker-buildx-plugi
n docker-compose-plugin

sudo systemctl enable docker --now
 #测试⼯作
docker ps
 #  批量安装所有软件
docker compose  
```

再下载相应的软件

### <font style="color:rgb(79, 79, 79);">1 NoSQL</font>
**<font style="color:rgb(26, 67, 156);">Redis整合</font>**

#### <font style="color:rgb(79, 79, 79);">1.1 场景整合</font>
**<font style="color:rgb(77, 77, 77);">依赖导入</font>**

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

**<font style="color:rgb(77, 77, 77);">配置</font>**

```java
spring.data.redis.host=192.168.200.100
spring.data.redis.password=Lfy123!@!
```

**<font style="color:rgb(77, 77, 77);">测试</font>**

```java
@Autowired
StringRedisTemplate redisTemplate;

@Test
void redisTest(){
    redisTemplate.opsForValue().set("a","1234");
    Assertions.assertEquals("1234",redisTemplate.opsForValue().get("a"));
}
```

---

<font style="color:rgb(77, 77, 77);">RedisTestController </font>

```java
@Autowired
StringRedisTemplate stringRedisTemplate;

//为了后来系统的兼容性，应该所有对象都是以json的方式进行保存
@Autowired //如果给redis中保存数据会使用默认的序列化机制，导致redis中保存的对象不可视
RedisTemplate<Object, Object>  redisTemplate;

@GetMapping("/count")
public String count(){

    Long hello = stringRedisTemplate.opsForValue().increment("hello");

    //常见数据类型  k: v value可以有很多类型
    //string： 普通字符串 ： redisTemplate.opsForValue()
    //list:    列表：       redisTemplate.opsForList()
    //set:     集合:       redisTemplate.opsForSet()
    //zset:    有序集合:    redisTemplate.opsForZSet()
    //hash：   map结构：    redisTemplate.opsForHash()

    return "访问了【"+hello+"】次";
}
```

<font style="color:rgb(77, 77, 77);">Boot309RedisApplicationTests </font>

```java
package com.atguigu.boot3.redis;

import org.junit.jupiter.api.Assertions;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.data.redis.core.StringRedisTemplate;
import org.springframework.data.redis.core.ZSetOperations;

import java.util.Map;
import java.util.Set;
import java.util.UUID;

@SpringBootTest
class Boot309RedisApplicationTests {


    @Autowired  //key.value 都是字符串
    StringRedisTemplate redisTemplate;
    /**
     * string： 普通字符串 ： redisTemplate.opsForValue()
     */
    @Test
    void contextLoads() {

        redisTemplate.opsForValue().set("haha", UUID.randomUUID().toString());

        String haha = redisTemplate.opsForValue().get("haha");
        System.out.println(haha);
    }

    /**
     * list:    列表：       redisTemplate.opsForList()
     */
    @Test
    void testList(){
        String listName = "listtest";
        redisTemplate.opsForList().leftPush(listName,"1");
        redisTemplate.opsForList().leftPush(listName,"2");
        redisTemplate.opsForList().leftPush(listName,"3");

        String pop = redisTemplate.opsForList().leftPop(listName);
        Assertions.assertEquals("3",pop);

    }

    /**
     * set:     集合:       redisTemplate.opsForSet()
     */
    @Test
    void testSet(){

        String setName = "settest";
        //1、给集合中添加元素
        redisTemplate.opsForSet().add(setName,"1","2","3","3");

        Boolean aBoolean = redisTemplate.opsForSet().isMember(setName, "2");
        Assertions.assertTrue(aBoolean);

        Boolean aBoolean1 = redisTemplate.opsForSet().isMember(setName, "5");
        Assertions.assertFalse(aBoolean1);
    }


    /**
     * zset:    有序集合:    redisTemplate.opsForZSet()
     */
    @Test
    void testzset(){
        String setName = "zsettest";
        redisTemplate.opsForZSet().add(setName,"雷丰阳",90.00);
        redisTemplate.opsForZSet().add(setName,"张三",99.00);
        redisTemplate.opsForZSet().add(setName,"李四",9.00);
        redisTemplate.opsForZSet().add(setName,"王五",97.10);

        ZSetOperations.TypedTuple<String> popMax = redisTemplate.opsForZSet().popMax(setName);
        String value = popMax.getValue();
        Double score = popMax.getScore();
        System.out.println(value + "==>" + score);
    }

    /**
     * hash：   map结构：    redisTemplate.opsForHash()
     */
    @Test
    void testhash(){
        String mapName = "amap";
        redisTemplate.opsForHash().put(mapName,"name","张三");
        redisTemplate.opsForHash().put(mapName,"age","18");


        System.out.println(redisTemplate.opsForHash().get(mapName, "name"));


    }
}
```

#### <font style="color:rgb(79, 79, 79);">1.2 自动配置原理</font>
<font style="color:rgb(213, 97, 97);background-color:rgb(240, 244, 248);">1 META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports</font><font style="color:rgb(48, 54, 63);">中导入了</font><font style="color:rgb(213, 97, 97);background-color:rgb(240, 244, 248);">RedisAutoConfiguration</font><font style="color:rgb(48, 54, 63);">、</font><font style="color:rgb(213, 97, 97);background-color:rgb(240, 244, 248);">RedisReactiveAutoConfiguration</font><font style="color:rgb(48, 54, 63);">和</font><font style="color:rgb(213, 97, 97);background-color:rgb(240, 244, 248);">RedisRepositoriesAutoConfiguration。</font><font style="color:rgb(48, 54, 63);">所有属性绑定在</font><font style="color:rgb(213, 97, 97);background-color:rgb(240, 244, 248);">RedisProperties</font><font style="color:rgb(48, 54, 63);">中</font>

<font style="color:rgb(213, 97, 97);background-color:rgb(240, 244, 248);">2 RedisReactiveAutoConfiguration</font><font style="color:rgb(48, 54, 63);">属于响应式编程，不用管。</font><font style="color:rgb(213, 97, 97);background-color:rgb(240, 244, 248);">RedisRepositoriesAutoConfiguration</font><font style="color:rgb(48, 54, 63);">属于 JPA 操作，也不用管</font>

<font style="color:rgb(213, 97, 97);background-color:rgb(240, 244, 248);">3 RedisAutoConfiguration</font><font style="color:rgb(77, 77, 77);"> </font><font style="color:rgb(48, 54, 63);">配置了以下组件</font>

<font style="color:rgb(213, 97, 97);">a）</font><font style="color:rgb(213, 97, 97);background-color:rgb(240, 244, 248);">LettuceConnectionConfiguration</font><font style="color:rgb(48, 54, 63);">： 给容器中注入了连接工厂</font><font style="color:rgb(213, 97, 97);background-color:rgb(240, 244, 248);">LettuceConnectionFactory</font><font style="color:rgb(48, 54, 63);">，和操作 redis 的客户端</font><font style="color:rgb(213, 97, 97);background-color:rgb(240, 244, 248);">DefaultClientResources</font><font style="color:rgb(48, 54, 63);">。</font>

<font style="color:rgb(213, 97, 97);">b）</font><font style="color:rgb(213, 97, 97);background-color:rgb(240, 244, 248);">RedisTemplate<Object, Object></font><font style="color:rgb(48, 54, 63);">： 可给 redis 中存储任意对象，会使用 jdk 默认序列化方式。</font>

<font style="color:rgb(213, 97, 97);">c）</font><font style="color:rgb(213, 97, 97);background-color:rgb(240, 244, 248);">StringRedisTemplate</font><font style="color:rgb(48, 54, 63);">： 给 redis 中存储字符串，如果要存对象，需要开发人员自己进行序列化。key-value都是字符串进行操作··</font>

#### <font style="color:rgb(79, 79, 79);">1.3 定制化</font>
#### <font style="color:rgb(79, 79, 79);">1.3.1 序列化机制</font>
<font style="color:rgb(77, 77, 77);">AppRedisConfiguration</font>

```java
@Configuration
public class AppRedisConfiguration {


    /**
     * 允许Object类型的key-value，都可以被转为json进行存储。
     * @param redisConnectionFactory 自动配置好了连接工厂
     * @return
     */
    @Bean
    public RedisTemplate<Object, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory) {
        RedisTemplate<Object, Object> template = new RedisTemplate<>();
        template.setConnectionFactory(redisConnectionFactory);
        //把对象转为json字符串的序列化工具
        template.setDefaultSerializer(new GenericJackson2JsonRedisSerializer());
        return template;
    }
}
```

<font style="color:rgb(77, 77, 77);">Person </font>

```java
package com.atguigu.boot3.redis.entity;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

import java.io.Serializable;
import java.util.Date;

/**
 * @author lfy
 * @Description
 * @create 2023-04-28 16:05
 */
@AllArgsConstructor
@NoArgsConstructor
@Data
public class Person implements Serializable {

    private Long id;
    private String name;
    private Integer age;
    private Date birthDay;
}
```

<font style="color:rgb(77, 77, 77);">RedisTestController </font>

```java
package com.atguigu.boot3.redis.controller;

import com.atguigu.boot3.redis.entity.Person;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.core.StringRedisTemplate;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.Date;

/**
 * @author lfy
 * @Description
 * @create 2023-04-28 15:43
 */
@RestController
public class RedisTestController {

    @Autowired
    StringRedisTemplate stringRedisTemplate;

    //为了后来系统的兼容性，应该所有对象都是以json的方式进行保存
    @Autowired //如果给redis中保存数据会使用默认的序列化机制，导致redis中保存的对象不可视
    RedisTemplate<Object, Object>  redisTemplate;

    @GetMapping("/person/save")
    public String savePerson(){
        Person person = new Person(1L,"张三",18,new Date());

        //1、序列化： 对象转为字符串方式
        redisTemplate.opsForValue().set("person",person);

        return "ok";
    }

    @GetMapping("/person/get")
    public Person getPerson(){
        Person person = (Person) redisTemplate.opsForValue().get("person");

        return person;
    }
}
```

#### <font style="color:rgb(79, 79, 79);">1.3.2 redis客户端</font>
<font style="color:rgb(79, 79, 79);">RedisTemplate、StringRedisTemplate： 操作redis的工具类</font>

<font style="color:rgb(79, 79, 79);">● 以上2个工具类要从redis的连接工厂获取连接才能操作redis。redis的连接工厂Spring有两种：Lettuce和Jedis。RedisTemplate从底层客户端获取连接，进行操作</font>

<font style="color:rgb(79, 79, 79);">● Redis客户端</font>

<font style="color:rgb(79, 79, 79);">○ Lettuce： 默认</font>

<font style="color:rgb(79, 79, 79);">○ Jedis：可以使用以下切换</font>

<font style="color:rgb(77, 77, 77);">从默认Lettuce切换到Jedis，首先排除默认导入的lettuce包，并导入Jedis包 </font>

```java
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-data-redis</artifactId>
  <exclusions>
    <exclusion>
      <groupId>io.lettuce</groupId>
      <artifactId>lettuce-core</artifactId>
    </exclusion>
  </exclusions>
</dependency>

<!--切换 jedis 作为操作redis的底层客户端-->
<dependency>
  <groupId>redis.clients</groupId>
  <artifactId>jedis</artifactId>
</dependency>
```

#### <font style="color:rgb(79, 79, 79);">1.3.3 配置参考</font>
```java
spring.data.redis.host=8.130.74.183
spring.data.redis.port=6379
#spring.data.redis.client-type=lettuce

#设置lettuce的底层参数
#spring.data.redis.lettuce.pool.enabled=true
#spring.data.redis.lettuce.pool.max-active=8

spring.data.redis.client-type=jedis
spring.data.redis.jedis.pool.enabled=true
spring.data.redis.jedis.pool.max-active=8
```

### <font style="color:rgb(79, 79, 79);">2 接口文档</font>
**<font style="color:rgb(26, 67, 156);">OpenAPI 3 与 Swagger</font>**

> <font style="color:rgb(79, 79, 79);">Swagger 可以快速生成</font>**<font style="color:rgb(79, 79, 79);">实时接口</font>**<font style="color:rgb(79, 79, 79);">文档，方便前后开发人员进行协调沟通。遵循</font><font style="color:rgb(79, 79, 79);"> </font>**<font style="color:rgb(79, 79, 79);">OpenAPI</font>**<font style="color:rgb(79, 79, 79);"> </font><font style="color:rgb(79, 79, 79);">规范。</font>
>
> <font style="color:rgb(79, 79, 79);">文档：</font>[<font style="color:rgb(78, 161, 219);">https://springdoc.org/v2/</font>](https://springdoc.org/v2/)
>

#### <font style="color:rgb(79, 79, 79);">2.1 OpenAPI 3 架构</font>
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1755876567442-2114fd8d-484a-4ec6-bdcf-5968bdf35080.png)

#### <font style="color:rgb(79, 79, 79);">2.2 整合</font>
**<font style="color:rgb(77, 77, 77);">导入场景</font>**

```java
<dependency>
    <groupId>org.springdoc</groupId>
    <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
    <version>2.1.0</version>
</dependency>
```

<font style="color:rgb(77, 77, 77);"> 重启后，swagger默认访问地址 /swagger-ui/index.html</font>

**<font style="color:rgb(77, 77, 77);">配置</font>**

```java
# /api-docs endpoint custom path 默认 /v3/api-docs
springdoc.api-docs.path=/api-docs

# swagger 相关配置在  springdoc.swagger-ui
# swagger-ui custom path
springdoc.swagger-ui.path=/swagger-ui.html

springdoc.show-actuator=true
```

#### <font style="color:rgb(79, 79, 79);">2.3 使用</font>
##### <font style="color:rgb(79, 79, 79);">2.3.1 常用注解</font>
| <font style="color:rgb(79, 79, 79);">注解</font> | <font style="color:rgb(79, 79, 79);">标注位置</font> | <font style="color:rgb(79, 79, 79);">作用</font> |
| :--- | :--- | :--- |
| <font style="color:rgb(79, 79, 79);">@Tag</font> | <font style="color:rgb(79, 79, 79);">controller 类</font> | <font style="color:rgb(79, 79, 79);">标识 controller 作用</font> |
| <font style="color:rgb(79, 79, 79);">@Parameter</font> | <font style="color:rgb(79, 79, 79);">参数</font> | <font style="color:rgb(79, 79, 79);">标识参数作用</font> |
| <font style="color:rgb(79, 79, 79);">@Parameters</font> | <font style="color:rgb(79, 79, 79);">参数</font> | <font style="color:rgb(79, 79, 79);">参数多重说明</font> |
| <font style="color:rgb(79, 79, 79);">@Schema</font> | <font style="color:rgb(79, 79, 79);">model 层的 JavaBean</font> | <font style="color:rgb(79, 79, 79);">描述模型作用及每个属性</font> |
| <font style="color:rgb(79, 79, 79);">@Operation</font> | <font style="color:rgb(79, 79, 79);">方法</font> | <font style="color:rgb(79, 79, 79);">描述方法作用</font> |
| <font style="color:rgb(79, 79, 79);">@ApiResponse</font> | <font style="color:rgb(79, 79, 79);">方法</font> | <font style="color:rgb(79, 79, 79);">描述响应状态码等</font> |


<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1755876716763-a037909d-e5fc-4b1f-915f-96bd9e910366.png)

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1755876716719-21ccbe02-90fd-482a-958f-3a825b3bdfbe.png)

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1755876725467-0c6fd474-2bc9-44b5-8493-1a6689252094.png)

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1755876725488-776c92af-4ff0-4bb6-82e6-4d0beeba38d0.png)

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1755876725606-aa53fa34-656b-468f-9bb2-0818c5b806c1.png)

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1755876725664-e0c5d7d9-a85c-4df8-9d67-fac8c0eb25de.png)<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1755876725661-8ee73e87-6d20-427c-b8cd-ae27493bce75.png)<font style="color:rgb(77, 77, 77);"> </font>

```java
/**
 * 部门信息实体类
 */
@Schema(title = "部门信息") // 类级别的Schema注解，描述这个类是部门信息
@Data // Lombok注解，自动生成getter、setter、toString等方法
public class Dept {

    @Schema(title = "部门id") // 字段级别的Schema注解，描述该字段为部门id
    private Long id;
    
    @Schema(title = "部门名字") // 字段级别的Schema注解，描述该字段为部门名字
    private String deptName;
}
```

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1755876757424-04f0ab80-7059-4a4c-af05-4113c4cd59b2.png)

##### <font style="color:rgb(79, 79, 79);">2.3.2 Docket配置</font>
<font style="color:rgb(77, 77, 77);">如果有多个Docket，配置如下</font>

```java
/**
 * 配置公共API分组
 * 该分组包含所有以/public开头的接口
 * @return GroupedOpenApi实例
 */
@Bean
public GroupedOpenApi publicApi() {
    return GroupedOpenApi.builder()
            .group("springshop-public")  // 设置分组名称为springshop-public
            .pathsToMatch("/public/**")  // 匹配所有以/public开头的路径
            .build();
}

/**
 * 配置管理员API分组
 * 该分组包含以/admin开头的接口，并且需要方法上有@Admin注解
 * @return GroupedOpenApi实例
 */
@Bean
public GroupedOpenApi adminApi() {
    return GroupedOpenA 
            // 设置分组名称为springshop-adminpi.builder()
            .group("springshop-admin")  
            // 匹配所有以/admin开头的路径
            .pathsToMatch("/admin/**")  
            // 添加方法过滤器，只包含带有@Admin注解的方法  
            .addMethodFilter(method -> method.isAnnotationPresent(Admin.class))  
            .build();
}
```

<font style="color:rgb(77, 77, 77);">如果只有一个Docket，可以配置如下 </font>

```java
# 配置SpringDoc要扫描的包路径，多个包名用逗号分隔
springdoc.packagesToScan=package1, package2

# 配置SpringDoc要匹配的接口路径，多个路径用逗号分隔
springdoc.pathsToMatch=/v1, /api/balance/**
```

<font style="color:rgb(77, 77, 77);">分组方式的Open API</font>

```java
@Configuration
public class ApiUiConfig {

    /**
     * 分组设置
     * @return
     */
    @Bean
    public GroupedOpenApi empApi() {
        return GroupedOpenApi.builder()
        .group("员工管理")
        .pathsToMatch("/emp/**","/emps")
        .build();
    }
    @Bean
    public GroupedOpenApi deptApi() {
        return GroupedOpenApi.builder()
        .group("部门管理")
        .pathsToMatch("/dept/**","/depts")
        .build();
    }
}
```

##### <font style="color:rgb(79, 79, 79);">2.3.3 OpenAPI配置</font>
```java
/**
 * 配置OpenAPI基本信息
 * 创建SpringShop应用的API文档配置
 * @return OpenAPI实例，包含API的基本信息
 */
@Bean
public OpenAPI springShopOpenAPI() {
    return new OpenAPI()
            // 设置API基本信息
            .info(new Info().title("SpringShop API")  // API标题
                    .description("Spring shop sample application")  // API描述
                    .version("v0.0.1")  // API版本号
                    .license(new License().name("Apache 2.0").url("http://springdoc.org")))  // 许可证信息
            // 设置外部文档链接
            .externalDocs(new ExternalDocumentation()
                    .description("SpringShop Wiki Documentation")  // 外部文档描述
                    .url("https://springshop.wiki.github.org/docs"));  // 外部文档URL
}
```

```java
@Configuration
public class ApiUiConfig {

    /**
     * 分组设置
     * @return
     */
    @Bean
    public OpenAPI docsOpenAPI() {
        return new OpenAPI()
                .info(new Info()
                        .title("SpringBoot3-CRUD API")
                        .description("专门测试接口文件")
                        .version("v0.0.1")
                        .license(new License().name("Apache 2.0").url("http://springdoc.org")))
                .externalDocs(new ExternalDocumentation()
                        .description("哈哈 Wiki Documentation")
                        .url("https://springshop.wiki.github.org/docs"));
    }
}
```

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1755876892088-9ef653fa-783f-4053-b1c7-44f1a1b8c595.png)

<font style="color:rgb(77, 77, 77);">注：Knife4j ，对swagger的增强，生成的文档是另外一个界面</font>

#### <font style="color:rgb(79, 79, 79);">2.4 Springfox迁移</font>
##### <font style="color:rgb(79, 79, 79);">2.4.1 注解变化</font>
| <font style="color:rgb(79, 79, 79);">原注解</font> | <font style="color:rgb(79, 79, 79);">现注解</font> | <font style="color:rgb(79, 79, 79);">作用</font> |
| :--- | :--- | :--- |
| <font style="color:rgb(79, 79, 79);">@Api</font> | <font style="color:rgb(79, 79, 79);">@Tag</font> | <font style="color:rgb(79, 79, 79);">描述Controller</font> |
| <font style="color:rgb(79, 79, 79);">@ApiIgnore</font> | <font style="color:rgb(79, 79, 79);">@Parameter(hidden = true)   </font><font style="color:rgb(79, 79, 79);">@Operation(hidden = true)   </font><font style="color:rgb(79, 79, 79);">@Hidden</font> | <font style="color:rgb(79, 79, 79);">描述忽略操作</font> |
| <font style="color:rgb(79, 79, 79);">@ApiImplicitParam</font> | <font style="color:rgb(79, 79, 79);">@Parameter</font> | <font style="color:rgb(79, 79, 79);">描述参数</font> |
| <font style="color:rgb(79, 79, 79);">@ApiImplicitParams</font> | <font style="color:rgb(79, 79, 79);">@Parameters</font> | <font style="color:rgb(79, 79, 79);">描述参数</font> |
| <font style="color:rgb(79, 79, 79);">@ApiModel</font> | <font style="color:rgb(79, 79, 79);">@Schema</font> | <font style="color:rgb(79, 79, 79);">描述对象</font> |
| <font style="color:rgb(79, 79, 79);">@ApiModelProperty(hidden = true)</font> | <font style="color:rgb(79, 79, 79);">@Schema(accessMode = READ_ONLY)</font> | <font style="color:rgb(79, 79, 79);">描述对象属性</font> |
| <font style="color:rgb(79, 79, 79);">@ApiModelProperty</font> | <font style="color:rgb(79, 79, 79);">@Schema</font> | <font style="color:rgb(79, 79, 79);">描述对象属性</font> |
| <font style="color:rgb(79, 79, 79);">@ApiOperation(value = "foo", notes = "bar")</font> | <font style="color:rgb(79, 79, 79);">@Operation(summary = "foo", description = "bar")</font> | <font style="color:rgb(79, 79, 79);">描述方法</font> |
| <font style="color:rgb(79, 79, 79);">@ApiParam</font> | <font style="color:rgb(79, 79, 79);">@Parameter</font> | <font style="color:rgb(79, 79, 79);">描述参数</font> |
| <font style="color:rgb(79, 79, 79);">@ApiResponse(code = 404, message = "foo")</font> | <font style="color:rgb(79, 79, 79);">@ApiResponse(responseCode = "404", description = "foo")</font> | <font style="color:rgb(79, 79, 79);">描述响应</font> |


##### <font style="color:rgb(79, 79, 79);">2.4.2 Docket配置</font>
###### <font style="color:rgb(79, 79, 79);">2.4.2.1 以前写法</font>
```java
@Bean
  public Docket publicApi() {
      return new Docket(DocumentationType.SWAGGER_2)
              .select()
              .apis(RequestHandlerSelectors.basePackage("org.github.springshop.web.public"))
              .paths(PathSelectors.regex("/public.*"))
              .build()
              .groupName("springshop-public")
              .apiInfo(apiInfo());
  }

  @Bean
  public Docket adminApi() {
      return new Docket(DocumentationType.SWAGGER_2)
              .select()
              .apis(RequestHandlerSelectors.basePackage("org.github.springshop.web.admin"))
              .paths(PathSelectors.regex("/admin.*"))
              .apis(RequestHandlerSelectors.withMethodAnnotation(Admin.class))
              .build()
              .groupName("springshop-admin")
              .apiInfo(apiInfo());
  }
```

###### <font style="color:rgb(79, 79, 79);">2.4.2.2 新写法</font>
```java
/**
 * 配置公共API分组
 * 该分组包含所有以/public开头的接口，无需特殊权限即可访问
 * @return GroupedOpenApi实例
 */
@Bean
public GroupedOpenApi publicApi() {
    return GroupedOpenApi.builder()
            .group("springshop-public")  // 设置分组名称为springshop-public
            .pathsToMatch("/public/**")  // 匹配所有以/public开头的路径
            .build();
}

/**
 * 配置管理员API分组
 * 该分组包含以/admin开头的接口，并且需要方法上有@Admin注解（管理员权限）
 * @return GroupedOpenApi实例
 */
@Bean
public GroupedOpenApi adminApi() {
    return GroupedOpenApi.builder()
            .group("springshop-admin")  // 设置分组名称为springshop-admin
            .pathsToMatch("/admin/**")  // 匹配所有以/admin开头的路径
            .addOpenApiMethodFilter(method -> method.isAnnotationPresent(Admin.class))  // 添加方法过滤器，只包含带有@Admin注解的方法
            .build();
}
```

###### <font style="color:rgb(79, 79, 79);">2.4.2.3 添加OpenAPI组件</font>
```java
@Bean
  public OpenAPI springShopOpenAPI() {
      return new OpenAPI()
              .info(new Info().title("SpringShop API")
              .description("Spring shop sample application")
              .version("v0.0.1")
              .license(new License().name("Apache 2.0").url("http://springdoc.org")))
              .externalDocs(new ExternalDocumentation()
              .description("SpringShop Wiki Documentation")
              .url("https://springshop.wiki.github.org/docs"));
  }
```

### <font style="color:rgb(79, 79, 79);">3 远程调用</font>
**<font style="color:rgb(77, 77, 77);">RPC（Remote Procedure Call）：远程过程调用</font>**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1755877013628-48f314d9-5c1b-4ceb-8fa7-156870c343f5.png)

**<font style="color:rgb(77, 77, 77);">本地过程调用</font>**<font style="color:rgb(77, 77, 77);">：a()；b()； a() { b()；}： 不同方法都在</font>**<font style="color:rgb(77, 77, 77);">同一个JVM进程中运行</font>**

**<font style="color:rgb(77, 77, 77);">远程过程调用</font>**<font style="color:rgb(77, 77, 77);">：</font>

+ <font style="color:rgb(51, 51, 51);">服务提供者：</font>
+ <font style="color:rgb(51, 51, 51);">服务消费者：</font>
+ <font style="color:rgb(51, 51, 51);">通过连接对方服务器进行请求\响应交互，来实现调用效果</font>

<font style="color:rgb(77, 77, 77);">API/SDK的区别是什么？</font>

<font style="color:rgb(77, 77, 77);">api：接口（Application Programming Interface）</font>

+ <font style="color:rgb(51, 51, 51);">远程提供功能；</font>

<font style="color:rgb(77, 77, 77);">sdk：工具包（Software Development Kit）</font>

+ <font style="color:rgb(51, 51, 51);">导入jar包，直接调用功能即可</font>

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1755877866542-af5d16cc-8720-41e1-8733-22311679ae9a.png)

#### <font style="color:rgb(79, 79, 79);">3.1 WebClient</font>
<font style="color:rgb(77, 77, 77);">非阻塞、响应式HTTP客户端</font>

##### <font style="color:rgb(79, 79, 79);">3.1.1 创建与配置</font>
<font style="color:rgb(77, 77, 77);">发请求：</font>

+ <font style="color:rgb(51, 51, 51);">请求方式： GET\POST\DELETE\xxxx</font>
+ <font style="color:rgb(51, 51, 51);">请求路径： /xxx</font>
+ <font style="color:rgb(51, 51, 51);">请求参数：aa=bb&cc=dd&xxx</font>
+ <font style="color:rgb(51, 51, 51);">请求头： aa=bb,cc=ddd</font>
+ <font style="color:rgb(51, 51, 51);">请求体：</font>

<font style="color:rgb(48, 54, 63);">创建 </font><font style="color:rgb(213, 97, 97);background-color:rgb(240, 244, 248);">WebClient</font><font style="color:rgb(77, 77, 77);"> </font><font style="color:rgb(48, 54, 63);">非常简单:</font>

+ <font style="color:rgb(213, 97, 97);background-color:rgb(240, 244, 248);">WebClient.create()</font>
+ <font style="color:rgb(213, 97, 97);background-color:rgb(240, 244, 248);">WebClient.create(String baseUrl)</font>

<font style="color:rgb(48, 54, 63);">还可以使用 </font><font style="color:rgb(213, 97, 97);background-color:rgb(240, 244, 248);">WebClient.builder()</font><font style="color:rgb(77, 77, 77);"> </font><font style="color:rgb(48, 54, 63);">配置更多参数项:</font>

+ <font style="color:rgb(213, 97, 97);background-color:rgb(240, 244, 248);">uriBuilderFactory</font><font style="color:rgb(48, 54, 63);">:</font><font style="color:rgb(48, 54, 63);"> </font><font style="color:rgb(48, 54, 63);">自定义</font><font style="color:rgb(213, 97, 97);background-color:rgb(240, 244, 248);">UriBuilderFactory</font><font style="color:rgb(51, 51, 51);"> </font><font style="color:rgb(48, 54, 63);">，定义 baseurl.</font>
+ <font style="color:rgb(213, 97, 97);background-color:rgb(240, 244, 248);">defaultUriVariables</font><font style="color:rgb(48, 54, 63);">:</font><font style="color:rgb(48, 54, 63);"> </font><font style="color:rgb(48, 54, 63);">默认 uri 变量.</font>
+ <font style="color:rgb(213, 97, 97);background-color:rgb(240, 244, 248);">defaultHeader</font><font style="color:rgb(48, 54, 63);">:</font><font style="color:rgb(48, 54, 63);"> </font><font style="color:rgb(48, 54, 63);">每个请求默认头.</font>
+ <font style="color:rgb(213, 97, 97);background-color:rgb(240, 244, 248);">defaultCookie</font><font style="color:rgb(48, 54, 63);">:</font><font style="color:rgb(48, 54, 63);"> </font><font style="color:rgb(48, 54, 63);">每个请求默认 cookie.</font>
+ <font style="color:rgb(213, 97, 97);background-color:rgb(240, 244, 248);">defaultRequest</font><font style="color:rgb(48, 54, 63);">:</font><font style="color:rgb(48, 54, 63);"> </font><font style="color:rgb(213, 97, 97);background-color:rgb(240, 244, 248);">Consumer</font><font style="color:rgb(51, 51, 51);"> </font><font style="color:rgb(48, 54, 63);">自定义每个请求.</font>
+ <font style="color:rgb(213, 97, 97);background-color:rgb(240, 244, 248);">filter</font><font style="color:rgb(48, 54, 63);">:</font><font style="color:rgb(48, 54, 63);"> </font><font style="color:rgb(48, 54, 63);">过滤 client 发送的每个请求</font>
+ <font style="color:rgb(213, 97, 97);background-color:rgb(240, 244, 248);">exchangeStrategies</font><font style="color:rgb(48, 54, 63);">: HTTP</font><font style="color:rgb(48, 54, 63);"> </font><font style="color:rgb(48, 54, 63);">消息 reader/writer 自定义.</font>
+ <font style="color:rgb(213, 97, 97);background-color:rgb(240, 244, 248);">clientConnector</font><font style="color:rgb(48, 54, 63);">: HTTP client</font><font style="color:rgb(48, 54, 63);"> </font><font style="color:rgb(48, 54, 63);">库设置.</font>

```java
//获取响应完整信息
WebClient client = WebClient.create("https://example.org");
```

##### <font style="color:rgb(79, 79, 79);">3.1.2 获取响应</font>
<font style="color:rgb(213, 97, 97);background-color:rgb(240, 244, 248);">retrieve()</font><font style="color:rgb(48, 54, 63);">方法用来声明如何提取响应数据。比如</font>

```java
//获取响应完整信息
WebClient client = WebClient.create("https://example.org");

Mono<ResponseEntity<Person>> result = client.get()
        .uri("/persons/{id}", id).accept(MediaType.APPLICATION_JSON)
        .retrieve()
        .toEntity(Person.class);

//只获取body
WebClient client = WebClient.create("https://example.org");

Mono<Person> result = client.get()
        .uri("/persons/{id}", id).accept(MediaType.APPLICATION_JSON)
        .retrieve()
        .bodyToMono(Person.class);

//stream数据
Flux<Quote> result = client.get()
        .uri("/quotes").accept(MediaType.TEXT_EVENT_STREAM)
        .retrieve()
        .bodyToFlux(Quote.class);

//定义错误处理
Mono<Person> result = client.get()
        .uri("/persons/{id}", id).accept(MediaType.APPLICATION_JSON)
        .retrieve()
        .onStatus(HttpStatus::is4xxClientError, response -> ...)
        .onStatus(HttpStatus::is5xxServerError, response -> ...)
        .bodyToMono(Person.class);
```

##### <font style="color:rgb(79, 79, 79);">3.1.3 定义请求体</font>
```java
//1、响应式-单个数据
Mono<Person> personMono = ... ;

Mono<Void> result = client.post()
        .uri("/persons/{id}", id)
        .contentType(MediaType.APPLICATION_JSON)
        .body(personMono, Person.class)
        .retrieve()
        .bodyToMono(Void.class);

//2、响应式-多个数据
Flux<Person> personFlux = ... ;

Mono<Void> result = client.post()
        .uri("/persons/{id}", id)
        .contentType(MediaType.APPLICATION_STREAM_JSON)
        .body(personFlux, Person.class)
        .retrieve()
        .bodyToMono(Void.class);

//3、普通对象
Person person = ... ;

Mono<Void> result = client.post()
        .uri("/persons/{id}", id)
        .contentType(MediaType.APPLICATION_JSON)
        .bodyValue(person)
        .retrieve()
        .bodyToMono(Void.class);
```

---

<font style="color:rgb(77, 77, 77);">WeatherController</font>

```java
/**
 * 天气查询控制器
 * 提供天气查询相关的REST接口
 */
@RestController  // 标识为REST控制器，返回值自动转换为JSON响应
public class WeatherController {

    @Autowired  // 自动注入天气服务
    WeatherService weatherService;

    /**
     * 根据城市名称查询天气信息
     * @param city 城市名称，通过请求参数传递
     * @return 包含天气信息的Mono异步响应
     */
    @GetMapping("/weather")  // 处理GET请求，路径为/weather
    public Mono<String> weather(@RequestParam("city") String city){
        // 调用天气服务查询指定城市的天气信息
        Mono<String> weather = weatherService.weather(city);

        // 返回异步天气信息
        return weather;
    }
}
```

<font style="color:rgb(77, 77, 77);">WeatherService</font>

```java
/**
 * 天气服务类
 * 负责处理天气相关的业务逻辑，包括调用第三方天气接口
 * @author lfy
 * @Description 天气服务组件
 * @create 2023-05-07 12:16
 */
@Service
public class WeatherService {

    @Autowired  // 自动注入天气接口组件
    WeatherInterface weatherInterface;

    /**
     * 使用WebClient调用第三方天气API获取天气信息
     * @param city 城市名称
     * @return 包含天气信息的Mono异步响应
     */
    private static Mono<String> getByWebClient(String city) {
        // 1、创建WebClient实例用于发送HTTP请求
        WebClient client = WebClient.create();

        // 2、准备请求参数
        Map<String,String> params = new HashMap<>();
        params.put("area", city);
        
        // 3、定义并发送HTTP GET请求
        Mono<String> mono = client.get()
                .uri("https://ali-weather.showapi.com/area-to-weather-date?area={area}", params)  // 设置请求URL和参数
                .accept(MediaType.APPLICATION_JSON)  // 设置接受响应类型为JSON
                .header("Authorization", "APPCODE 93b7e19861a24c519a7548b17dc16d75")  // 设置认证请求头
                .retrieve()  // 发送请求并检索响应
                .bodyToMono(String.class);  // 将响应体转换为String类型的Mono
        
        return mono;  // 返回异步结果
    }
}
```

#### <font style="color:rgb(79, 79, 79);">3.2 HTTP Interface</font>
<font style="color:rgb(48, 54, 63);">Spring 允许我们通过定义接口的方式，给任意位置发送 http 请求，实现远程调用，可以用来简化 HTTP 远程访问。需要</font><font style="color:rgb(213, 97, 97);background-color:rgb(240, 244, 248);">webflux</font><font style="color:rgb(48, 54, 63);">场景才可</font>

##### <font style="color:rgb(79, 79, 79);">3.2.1 导入依赖</font>
```java
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-webflux</artifactId>
</dependency>
```

##### <font style="color:rgb(79, 79, 79);">3.2.2 定义接口</font>
```java
/**
 * Bing搜索服务接口
 * 使用Spring的HttpInterface声明式HTTP客户端
 * 定义与Bing搜索API的交互接口
 */
public interface BingService {

    /**
     * 执行Bing搜索
     * @param keyword 搜索关键词
     * @return 搜索结果的字符串形式
     */
    @GetExchange(url = "/search")  // 声明GET请求，指定请求路径为/search
    String search(@RequestParam("keyword") String keyword);  // 使用@RequestParam注解将keyword参数作为查询参数q传递
}
```

##### <font style="color:rgb(79, 79, 79);">3.2.3 创建代理&测试</font>
```java
@SpringBootTest
class Boot05TaskApplicationTests {

    @Test
    void contextLoads() throws InterruptedException {
        // 1、创建WebClient客户端实例，配置基础URL和编解码器
        WebClient client = WebClient.builder()
                .baseUrl("https://cn.bing.com")  // 设置基础URL为必应搜索
                .codecs(clientCodecConfigurer -> {
                    clientCodecConfigurer
                            .defaultCodecs()
                            .maxInMemorySize(256*1024*1024);  // 设置最大内存缓冲区为256MB，防止大数据量响应超出限制
                })
                .build();
        
        // 2、创建HttpServiceProxyFactory工厂，用于生成代理对象
        HttpServiceProxyFactory factory = HttpServiceProxyFactory
                .builder(WebClientAdapter.forClient(client)).build();  // 使用WebClient适配器构建工厂
        
        // 3、通过工厂创建BingService接口的代理对象
        BingService bingService = factory.createClient(BingService.class);

        // 4、测试调用搜索接口
        Mono<String> search = bingService.search("尚硅谷");  // 异步调用搜索方法
        System.out.println("==========");  // 打印分隔符
        
        // 订阅异步响应，当结果返回时打印搜索结果
        search.subscribe(str -> System.out.println(str));

        // 主线程休眠100秒，等待异步操作完成（测试环境使用）
        Thread.sleep(100000);
    }
}
```

---

<font style="color:rgb(77, 77, 77);">com/atguigu/boot3/rpc/service/WeatherInterface.java</font>

```java
/**
 * 天气接口服务
 * 使用Spring的HttpInterface声明式HTTP客户端
 * 定义与第三方天气API的交互接口
 */
public interface WeatherInterface {

    /**
     * 根据城市名称获取天气信息
     * @param city 城市名称
     * @return 包含天气信息的Mono异步响应
     */
    @GetExchange(url = "https://ali-weather.showapi.com/area-to-weather-date", accept = "application/json")
    Mono<String> getWeather(@RequestParam("area") String city);
}
```

<font style="color:rgb(77, 77, 77);">com/atguigu/boot3/rpc/service/ExpressApi.java</font>

```java
/**
 * 快递查询接口服务
 * 使用Spring的HttpInterface声明式HTTP客户端
 * 定义与第三方快递查询API的交互接口
 */
public interface ExpressApi {
    
    /**
     * 根据快递单号查询快递信息
     * @param number 快递单号
     * @return 包含快递信息的Mono异步响应
     */
    @GetExchange(url = "https://express3.market.alicloudapi.com/express3", accept = "application/json")
    Mono<String> getExpress(@RequestParam("number") String number);
}
```

<font style="color:rgb(77, 77, 77);">application.properties</font>

```java
aliyun.appcode=93b7e19861a24c519a7548b17dc16d75
```

<font style="color:rgb(77, 77, 77);">com/atguigu/boot3/rpc/config/WeatherConfiguration.java</font>

```java
@Configuration //最好起名为 AliyunApiConfiguration
public class WeatherConfiguration {

    @Bean
    HttpServiceProxyFactory httpServiceProxyFactory(@Value("${aliyun.appcode}") String appCode){
        //1、创建客户端
        WebClient client = WebClient.builder()
                .defaultHeader("Authorization","APPCODE "+appCode)
                .codecs(clientCodecConfigurer -> {
                    clientCodecConfigurer
                            .defaultCodecs()
                            .maxInMemorySize(256*1024*1024);
                    //响应数据量太大有可能会超出BufferSize，所以这里设置的大一点
                })
                .build();
        //2、创建工厂
        HttpServiceProxyFactory factory = HttpServiceProxyFactory
                .builder(WebClientAdapter.forClient(client)).build();
        return factory;
    }

    @Bean
    WeatherInterface weatherInterface(HttpServiceProxyFactory httpServiceProxyFactory){
        //3、获取代理对象
        WeatherInterface weatherInterface = httpServiceProxyFactory.createClient(WeatherInterface.class);
        return weatherInterface;
    }

    @Bean
    ExpressApi expressApi(HttpServiceProxyFactory httpServiceProxyFactory){
        //3、获取代理对象
        ExpressApi client = httpServiceProxyFactory.createClient(ExpressApi.class);
        return client;
    }
}
```

<font style="color:rgb(77, 77, 77);">com/atguigu/boot3/rpc/service/WeatherService.java</font>

```java
/**
 * 天气服务实现类
 * 负责调用天气接口获取天气信息
 */
@Service
public class WeatherService {

    @Autowired  // 自动注入天气接口代理对象
    WeatherInterface weatherInterface;

    /**
     * 根据城市名称获取天气信息
     * @param city 城市名称
     * @return 包含天气信息的Mono异步响应
     */
    public Mono<String> weather(String city){
        // 使用声明式HTTP客户端远程调用阿里云天气API
        Mono<String> weather = weatherInterface.getWeather(city);

        return weather;
    }

    /**
     * 使用WebClient方式调用天气API（备用方法）
     * @param city 城市名称
     * @return 包含天气信息的Mono异步响应
     */
    private static Mono<String> getByWebClient(String city) {
        // 1、创建WebClient实例
        WebClient client = WebClient.create();

        // 2、准备请求参数
        Map<String,String> params = new HashMap<>();
        params.put("area", city);
        
        // 3、构建并发送HTTP GET请求
        Mono<String> mono = client.get()
                .uri("https://ali-weather.showapi.com/area-to-weather-date?area={area}", params)  // 设置请求URL和参数
                .accept(MediaType.APPLICATION_JSON)  // 设置接受响应类型为JSON
                .header("Authorization", "APPCODE 93b7e19861a24c519a7548b17dc16d75")  // 设置认证请求头
                .retrieve()  // 发送请求并检索响应
                .bodyToMono(String.class);  // 将响应体转换为String类型的Mono
        
        return mono;
    }
}
```

<font style="color:rgb(77, 77, 77);">com/atguigu/boot3/rpc/controller/WeatherController.java</font>

```java
/**
 * 天气和快递查询控制器
 * 提供REST接口用于查询天气和快递信息
 */
@RestController  // 标识为REST控制器，返回值自动转换为JSON响应
public class WeatherController {

    @Autowired  // 自动注入天气服务
    WeatherService weatherService;

    @Autowired  // 自动注入快递查询接口
    ExpressApi expressApi;

    /**
     * 根据城市名称查询天气信息
     * @param city 城市名称，通过请求参数传递
     * @return 包含天气信息的Mono异步响应
     */
    @GetMapping("/weather")  // 处理GET请求，路径为/weather
    public Mono<String> weather(@RequestParam("city") String city){
        // 调用天气服务查询指定城市的天气信息
        Mono<String> weather = weatherService.weather(city);

        return weather;
    }

    /**
     * 根据快递单号查询物流信息
     * @param number 快递单号，通过请求参数传递
     * @return 包含物流信息的Mono异步响应
     */
    @GetMapping("/express")  // 处理GET请求，路径为/express
    public Mono<String> express(@RequestParam("number") String number){
        // 调用快递查询接口获取物流信息
        return expressApi.getExpress(number);
    }
}
```

#### <font style="color:rgb(79, 79, 79);">3.3 </font><font style="color:rgba(0, 0, 0, 0.85);background-color:rgba(0, 0, 0, 0.04);">RESTTemplate</font>
<font style="color:rgba(0, 0, 0, 0.85);background-color:rgb(249, 250, 251);">使用</font>`<font style="color:rgba(0, 0, 0, 0.85);">RESTTemplate</font>`<font style="color:rgba(0, 0, 0, 0.85);background-color:rgb(249, 250, 251);">进行远程调用是 Spring Boot 中实现 HTTP 通信的常用方式，它简化了发起 HTTP 请求和处理响应的过程。下面通过一个完整示例，详细说明如何使用</font>`<font style="color:rgba(0, 0, 0, 0.85);">RESTTemplate</font>`<font style="color:rgba(0, 0, 0, 0.85);background-color:rgb(249, 250, 251);">实现远程调用。</font>

##### <font style="color:rgb(0, 0, 0);background-color:rgb(249, 250, 251);">3.3.1 准备工作</font>
<font style="color:rgba(0, 0, 0, 0.85) !important;background-color:rgb(249, 250, 251);">假设我们需要调用一个远程用户服务的 API，该服务提供以下接口：</font>

+ `<font style="color:rgb(0, 0, 0);background-color:rgb(249, 250, 251);">GET /api/users/{id}</font>`<font style="color:rgba(0, 0, 0, 0.85) !important;background-color:rgb(249, 250, 251);">：根据 ID 查询用户</font>
+ `<font style="color:rgb(0, 0, 0);background-color:rgb(249, 250, 251);">POST /api/users</font>`<font style="color:rgba(0, 0, 0, 0.85) !important;background-color:rgb(249, 250, 251);">：创建新用户</font>
+ `<font style="color:rgb(0, 0, 0);background-color:rgb(249, 250, 251);">PUT /api/users/{id}</font>`<font style="color:rgba(0, 0, 0, 0.85) !important;background-color:rgb(249, 250, 251);">：更新用户信息</font>
+ `<font style="color:rgb(0, 0, 0);background-color:rgb(249, 250, 251);">DELETE /api/users/{id}</font>`<font style="color:rgba(0, 0, 0, 0.85) !important;background-color:rgb(249, 250, 251);">：删除用户</font>

##### <font style="color:rgb(0, 0, 0);background-color:rgb(249, 250, 251);">3.3.2 完整实现步骤</font>
###### <font style="color:rgb(0, 0, 0);background-color:rgb(249, 250, 251);">3.3.2.1 定义实体类（与远程服务数据结构对应）</font>
<font style="color:rgba(0, 0, 0, 0.85) !important;background-color:rgb(249, 250, 251);">首先创建一个</font>`<font style="color:rgba(0, 0, 0, 0.85) !important;background-color:rgb(249, 250, 251);">User</font>`<font style="color:rgba(0, 0, 0, 0.85) !important;background-color:rgb(249, 250, 251);">类，用于数据的序列化和反序列化：</font>

```java
public class User {
    private Long id;
    private String username;
    private String email;
    
    // 构造方法
    public User() {}
    public User(String username, String email) {
        this.username = username;
        this.email = email;
    }
    
    // Getter和Setter
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }
    public String getUsername() { return username; }
    public void setUsername(String username) { this.username = username; }
    public String getEmail() { return email; }
    public void setEmail(String email) { this.email = email; }
}
```

###### <font style="color:rgb(0, 0, 0);background-color:rgb(249, 250, 251);">3.3.2.2 配置</font>`<font style="color:rgb(0, 0, 0);background-color:rgb(249, 250, 251);">RESTTemplate</font>`
<font style="color:rgba(0, 0, 0, 0.85) !important;background-color:rgb(249, 250, 251);">在 Spring 配置类中注册</font>`<font style="color:rgba(0, 0, 0, 0.85) !important;background-color:rgb(249, 250, 251);">RESTTemplate</font>`<font style="color:rgba(0, 0, 0, 0.85) !important;background-color:rgb(249, 250, 251);">的 Bean，以便在项目中注入使用：</font>

```java
@Configuration
public class RestTemplateConfig {

    @Bean
    public RestTemplate restTemplate() {
        // 创建请求工厂，用于配置超时时间
        SimpleClientHttpRequestFactory requestFactory = new SimpleClientHttpRequestFactory();
        requestFactory.setConnectTimeout(5000);  // 连接超时时间：5秒
        requestFactory.setReadTimeout(3000);     // 数据读取超时时间：3秒
        
        // 初始化RestTemplate并设置请求工厂
        RestTemplate restTemplate = new RestTemplate(requestFactory);
        
        return restTemplate;
    }
}

```

###### <font style="color:rgb(0, 0, 0);background-color:rgb(249, 250, 251);">3.3.2.3. 编写远程服务类实现远程调用</font>
<font style="color:rgba(0, 0, 0, 0.85) !important;background-color:rgb(249, 250, 251);">创建一个服务类，注入</font>`<font style="color:rgba(0, 0, 0, 0.85) !important;background-color:rgb(249, 250, 251);">RESTTemplate</font>`<font style="color:rgba(0, 0, 0, 0.85) !important;background-color:rgb(249, 250, 251);">并实现对远程 API 的调用：</font>

```java
@Service
public class UserRemoteService {

    // 远程服务基础URL（实际开发中可配置在application.properties中）
    private static final String REMOTE_SERVICE_URL = "http://localhost:8081/api/users";

    @Autowired
    private RestTemplate restTemplate;

    /**
     * 远程调用：根据ID查询用户
     */
    public User getUserById(Long id) {
        try {
            // 方式1：使用getForObject直接返回实体对象
            return restTemplate.getForObject(
                REMOTE_SERVICE_URL + "/{id}",  // URL模板，{id}为占位符
                User.class,                    // 响应数据类型
                id                             // 替换占位符的参数
            );
        } catch (RestClientException e) {
            throw new RuntimeException("查询用户失败：" + e.getMessage());
        }
    }

    /**
     * 远程调用：创建用户（带完整响应信息处理）
     */
    public User createUser(User user) {
        try {
            // 方式2：使用postForEntity获取完整响应（包含状态码、响应头等）
            ResponseEntity<User> response = restTemplate.postForEntity(
                REMOTE_SERVICE_URL,  // 请求URL
                user,                // 请求体（自动序列化为JSON）
                User.class           // 响应数据类型
            );
            
            // 检查响应状态码（200-299为成功）
            if (response.getStatusCode().is2xxSuccessful()) {
                return response.getBody();  // 返回响应体
            } else {
                throw new RuntimeException("创建用户失败，状态码：" + response.getStatusCodeValue());
            }
        } catch (RestClientException e) {
            throw new RuntimeException("创建用户失败：" + e.getMessage());
        }
    }

    /**
     * 远程调用：更新用户（带自定义请求头）
     */
    public void updateUser(Long id, User user, String token) {
        try {
            // 1. 创建请求头（例如添加认证Token）
            HttpHeaders headers = new HttpHeaders();
            headers.setContentType(MediaType.APPLICATION_JSON);  // 声明请求体为JSON
            headers.set("Authorization", "Bearer " + token);     // 添加认证信息

            // 2. 封装请求体和头信息
            HttpEntity<User> requestEntity = new HttpEntity<>(user, headers);

            // 3. 发送PUT请求（使用exchange通用方法）
            restTemplate.exchange(
                REMOTE_SERVICE_URL + "/{id}",  // URL模板
                HttpMethod.PUT,                // 请求方法
                requestEntity,                 // 包含头和体的请求实体
                Void.class,                    // 响应类型（无返回值）
                id                             // URL参数
            );
        } catch (RestClientException e) {
            throw new RuntimeException("更新用户失败：" + e.getMessage());
        }
    }

    /**
     * 远程调用：删除用户
     */
    public void deleteUser(Long id) {
        try {
            restTemplate.delete(
                REMOTE_SERVICE_URL + "/{id}",  // URL模板
                id                             // 参数
            );
        } catch (RestClientException e) {
            throw new RuntimeException("删除用户失败：" + e.getMessage());
        }
    }

    /**
     * 远程调用：分页查询用户（带查询参数）
     */
    public User[] getUsersByPage(int page, int size) {
        try {
            // 构建带查询参数的URL（page=1&size=10）
            String url = UriComponentsBuilder
                .fromHttpUrl(REMOTE_SERVICE_URL)
                .queryParam("page", page)
                .queryParam("size", size)
                .toUriString();

            // 发送GET请求，返回用户数组
            return restTemplate.getForObject(url, User[].class);
        } catch (RestClientException e) {
            throw new RuntimeException("分页查询用户失败：" + e.getMessage());
        }
    }
}

```

##### <font style="color:rgb(0, 0, 0);background-color:rgb(249, 250, 251);">3.3.3 核心 API 说明</font>
<font style="color:rgba(0, 0, 0, 0.85) !important;background-color:rgb(249, 250, 251);">上述示例中使用了</font>`<font style="color:rgba(0, 0, 0, 0.85) !important;background-color:rgb(249, 250, 251);">RESTTemplate</font>`<font style="color:rgba(0, 0, 0, 0.85) !important;background-color:rgb(249, 250, 251);">的几个核心方法，各自适用场景如下：</font>

| **<font style="color:rgb(0, 0, 0) !important;background-color:rgb(249, 250, 251);">方法</font>** | **<font style="color:rgb(0, 0, 0) !important;background-color:rgb(249, 250, 251);">用途</font>** | **<font style="color:rgb(0, 0, 0) !important;background-color:rgb(249, 250, 251);">特点</font>** |
| :--- | :--- | :--- |
| `<font style="color:rgb(0, 0, 0);background-color:rgb(249, 250, 251);">getForObject()</font>` | <font style="color:rgba(0, 0, 0, 0.85) !important;background-color:rgb(249, 250, 251);">发送 GET 请求，获取响应体</font> | <font style="color:rgba(0, 0, 0, 0.85) !important;background-color:rgb(249, 250, 251);">直接返回转换后的对象，简洁易用</font> |
| `<font style="color:rgb(0, 0, 0);background-color:rgb(249, 250, 251);">getForEntity()</font>` | <font style="color:rgba(0, 0, 0, 0.85) !important;background-color:rgb(249, 250, 251);">发送 GET 请求，获取完整响应</font> | <font style="color:rgba(0, 0, 0, 0.85) !important;background-color:rgb(249, 250, 251);">包含状态码、响应头、响应体，适合需要处理响应元数据的场景</font> |
| `<font style="color:rgb(0, 0, 0);background-color:rgb(249, 250, 251);">postForObject()</font>`<br/><font style="color:rgba(0, 0, 0, 0.85) !important;background-color:rgb(249, 250, 251);">/</font>`<font style="color:rgb(0, 0, 0);background-color:rgb(249, 250, 251);">postForEntity()</font>` | <font style="color:rgba(0, 0, 0, 0.85) !important;background-color:rgb(249, 250, 251);">发送 POST 请求</font> | <font style="color:rgba(0, 0, 0, 0.85) !important;background-color:rgb(249, 250, 251);">自动将请求体序列化为 JSON，适合创建资源</font> |
| `<font style="color:rgb(0, 0, 0);background-color:rgb(249, 250, 251);">put()</font>` | <font style="color:rgba(0, 0, 0, 0.85) !important;background-color:rgb(249, 250, 251);">发送 PUT 请求</font> | <font style="color:rgba(0, 0, 0, 0.85) !important;background-color:rgb(249, 250, 251);">用于更新资源，无返回值</font> |
| `<font style="color:rgb(0, 0, 0);background-color:rgb(249, 250, 251);">delete()</font>` | <font style="color:rgba(0, 0, 0, 0.85) !important;background-color:rgb(249, 250, 251);">发送 DELETE 请求</font> | <font style="color:rgba(0, 0, 0, 0.85) !important;background-color:rgb(249, 250, 251);">用于删除资源，无返回值</font> |
| `<font style="color:rgb(0, 0, 0);background-color:rgb(249, 250, 251);">exchange()</font>` | <font style="color:rgba(0, 0, 0, 0.85) !important;background-color:rgb(249, 250, 251);">通用请求方法</font> | <font style="color:rgba(0, 0, 0, 0.85) !important;background-color:rgb(249, 250, 251);">支持所有 HTTP 方法，可自定义请求头、请求体，灵活性最高</font> |


##### <font style="color:rgb(0, 0, 0);background-color:rgb(249, 250, 251);">3.3.4 异常处理</font>
`<font style="color:rgba(0, 0, 0, 0.85) !important;background-color:rgb(249, 250, 251);">RESTTemplate</font>`<font style="color:rgba(0, 0, 0, 0.85) !important;background-color:rgb(249, 250, 251);">在请求失败时会抛出</font>`<font style="color:rgba(0, 0, 0, 0.85) !important;background-color:rgb(249, 250, 251);">RestClientException</font>`<font style="color:rgba(0, 0, 0, 0.85) !important;background-color:rgb(249, 250, 251);">的子类，常见异常及处理方式：</font>

+ `<font style="color:rgb(0, 0, 0);background-color:rgb(249, 250, 251);">HttpClientErrorException</font>`<font style="color:rgba(0, 0, 0, 0.85) !important;background-color:rgb(249, 250, 251);">：客户端错误（4xx 状态码，如 404、401）</font>
+ `<font style="color:rgb(0, 0, 0);background-color:rgb(249, 250, 251);">HttpServerErrorException</font>`<font style="color:rgba(0, 0, 0, 0.85) !important;background-color:rgb(249, 250, 251);">：服务器错误（5xx 状态码，如 500）</font>
+ `<font style="color:rgb(0, 0, 0);background-color:rgb(249, 250, 251);">ResourceAccessException</font>`<font style="color:rgba(0, 0, 0, 0.85) !important;background-color:rgb(249, 250, 251);">：网络问题（如连接超时、无法访问）</font>

<font style="color:rgba(0, 0, 0, 0.85) !important;background-color:rgb(249, 250, 251);">在示例中，我们通过</font>`<font style="color:rgba(0, 0, 0, 0.85) !important;background-color:rgb(249, 250, 251);">try-catch</font>`<font style="color:rgba(0, 0, 0, 0.85) !important;background-color:rgb(249, 250, 251);">捕获异常并转换为业务异常，便于上层处理。</font>

##### <font style="color:rgb(0, 0, 0);background-color:rgb(249, 250, 251);">3.3.5 使用场景总结</font>
`<font style="color:rgba(0, 0, 0, 0.85) !important;background-color:rgb(249, 250, 251);">RESTTemplate</font>`<font style="color:rgba(0, 0, 0, 0.85) !important;background-color:rgb(249, 250, 251);">适合以下场景：</font>

+ <font style="color:rgba(0, 0, 0, 0.85) !important;background-color:rgb(249, 250, 251);">简单的同步 HTTP 调用（非高并发场景）</font>
+ <font style="color:rgba(0, 0, 0, 0.85) !important;background-color:rgb(249, 250, 251);">需要直接控制 HTTP 请求细节（如自定义头、超时时间）</font>
+ <font style="color:rgba(0, 0, 0, 0.85) !important;background-color:rgb(249, 250, 251);">调用第三方 API 或内部非微服务架构的服务</font>

### <font style="color:rgb(79, 79, 79);">4 消息服务</font>
[<font style="color:rgb(78, 161, 219);">Apache Kafka</font>](https://kafka.apache.org/documentation/)

#### <font style="color:rgb(79, 79, 79);">4.1 消息队列-场景</font>
##### <font style="color:rgb(79, 79, 79);">4.1.1 异步</font>
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1755894600788-fd49d4d3-f618-48cd-8f6d-d8d0cd2839f5.png)

##### <font style="color:rgb(79, 79, 79);">4.1.2 解耦</font>
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1755894600816-38297f13-5d46-4056-b1b2-2ee210c1aa4c.png)

##### <font style="color:rgb(79, 79, 79);">4.1.3 削峰</font>
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1755894600815-ece966e9-f5ff-41b1-809d-ca336ee2f980.png)

##### <font style="color:rgb(79, 79, 79);">4.1.4 缓冲</font>
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1755894600779-f878291b-9fd7-4592-8fc3-c4ba04431fb0.png)

#### <font style="color:rgb(79, 79, 79);">4.2 消息队列-Kafka</font>
##### <font style="color:rgb(79, 79, 79);">4.2.1 消息模式</font>
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1755894600861-a291a72a-b7bd-43da-9ba7-7dc88124df52.png)

##### <font style="color:rgb(79, 79, 79);">4.2.2 Kafka工作原理</font>
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1755894601205-e05cede1-c18f-45e1-8c37-de25b958f22e.png)

```java
  KafkaAutoConfiguration提供如下功能
  1、KafkaProperties：kafka的所有配置; 以 spring.kafka开始
     - bootstrapServers: kafka集群的所有服务器地址
     - properties: 参数设置
     - consumer: 消费者
     - producer: 生产者
     ...
  2、@EnableKafka: 开启Kafka的注解驱动功能
  3、KafkaTemplate: 收发消息
  4、KafkaAdmin： 维护主题等...
  5、@EnableKafka +  @KafkaListener 接受消息
     1）、消费者来接受消息，需要有group-id
     2）、收消息使用 @KafkaListener + ConsumerRecord
     3）、spring.kafka 开始的所有配置
  6、核心概念
     分区：  分散存储，1T的数据分散到N个节点
     副本：  备份机制，每个小分区的数据都有备份
     主题： topics； 消息是发送给某个主题
```

##### <font style="color:rgb(79, 79, 79);">4.2.3 SpringBoot整合</font>
<font style="color:rgb(77, 77, 77);">参照：</font>[<font style="color:rgb(78, 161, 219);">Overview :: Spring Kafka</font>](https://docs.spring.io/spring-kafka/reference/)

```java
<!--整合kafka-->
<dependency>
    <groupId>org.springframework.kafka</groupId>
    <artifactId>spring-kafka</artifactId>
</dependency>
```

<font style="color:rgb(77, 77, 77);">配置 </font>

```java
spring.kafka.bootstrap-servers=172.20.128.1:9092
```

<font style="color:rgb(77, 77, 77);">修改C:\Windows\System32\drivers\etc\hosts文件，配置8.130.32.70 kafka </font>

##### <font style="color:rgb(79, 79, 79);">4.2.4 消息发送</font>
```java
// 导入Spring Boot测试注解，用于标注这是一个Spring Boot测试类
@SpringBootTest
class Boot07KafkaApplicationTests {

    // 自动注入KafkaTemplate，用于操作Kafka消息队列
    @Autowired
    KafkaTemplate kafkaTemplate;
    
    // 测试方法，用于测试Kafka消息发送性能
    // throws ExecutionException, InterruptedException：声明可能抛出的异常，处理异步操作可能的异常
    @Test
    void contextLoads() throws ExecutionException, InterruptedException {
        // 创建StopWatch对象，用于计时
        StopWatch watch = new StopWatch();
        // 开始计时
        watch.start();
        
        // 创建一个CompletableFuture数组，用于存储10000个异步发送消息的结果
        CompletableFuture[] futures = new CompletableFuture[10000];
        
        // 循环发送10000条消息到Kafka
        for (int i = 0; i < 10000; i++) {
            // 异步发送消息到名为"order"的主题，指定key为"order.create.i"，消息内容为"订单创建了：i"
            // kafkaTemplate.send()方法返回CompletableFuture对象，表示异步操作
            CompletableFuture send = kafkaTemplate.send("order", "order.create."+i, "订单创建了："+i);
            // 将异步操作结果存入数组
            futures[i] = send;
        }
        
        // 等待所有异步消息发送完成
        CompletableFuture.allOf(futures).join();
        
        // 停止计时
        watch.stop();
        
        // 输出总耗时（毫秒），用于评估发送10000条消息的性能
        System.out.println("总耗时："+watch.getTotalTimeMillis());
    }

}
```

```java
// 标注该类为Spring组件，会被Spring容器自动扫描并管理
@Component
public class MyBean {

    // 声明KafkaTemplate变量，用于与Kafka交互，泛型指定键和值的类型为String
    private final KafkaTemplate<String, String> kafkaTemplate;

    // 构造函数，通过构造器注入KafkaTemplate实例
    // 这种注入方式有利于依赖管理和单元测试
    public MyBean(KafkaTemplate<String, String> kafkaTemplate) {
        this.kafkaTemplate = kafkaTemplate;
    }

    // 自定义业务方法，用于发送消息到Kafka
    public void someMethod() {
        // 使用kafkaTemplate发送消息到指定主题"someTopic"，消息内容为"Hello"
        // 这里发送操作是异步的，不会阻塞当前线程
        this.kafkaTemplate.send("someTopic", "Hello");
    }

}
```

---

<font style="color:rgb(77, 77, 77);">com/atguigu/boot3/message/entity/Person.java</font>

```java
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

@AllArgsConstructor
@NoArgsConstructor
@Data
public class Person {
    private Long id;
    private String name;
    private String email;
}
```

<font style="color:rgb(77, 77, 77);">com/atguigu/boot3/message/Boot312MessageApplicationTests.java</font>

```java
// Spring Boot测试注解，标识这是一个Spring Boot应用的测试类
// 会自动加载Spring上下文，方便进行依赖注入和集成测试
@SpringBootTest
class Boot312MessageApplicationTests {

    // 自动注入KafkaTemplate对象，用于与Kafka进行交互发送消息
    @Autowired
    KafkaTemplate kafkaTemplate;

    /**
     * 测试Kafka批量发送消息的性能
     * 发送10000条消息到指定主题，并统计总耗时
     */
    @Test
    void contextLoads() {
        // 创建StopWatch对象，用于精确计时
        StopWatch stopWatch = new StopWatch();

        // 创建CompletableFuture数组，用于存储10000个异步发送消息的结果
        CompletableFuture[] futures = new CompletableFuture[10000];

        // 开始计时
        stopWatch.start();
        
        // 循环发送10000条消息
        for (int i = 0; i < 10000; i++) {
            // 异步发送消息到"newshaha"主题
            // 参数分别为：主题名、消息键、消息内容
            // 返回CompletableFuture对象，代表异步操作的结果
            CompletableFuture future = kafkaTemplate.send("newshaha", "haha-"+i, "哈哈哈-"+i);
            // 将异步操作结果存入数组
            futures[i] = future;
        }

        // 等待所有异步消息发送操作完成
        // join()方法会阻塞当前线程，直到所有异步操作完成
        CompletableFuture.allOf(futures).join();

        // 停止计时
        stopWatch.stop();

        // 获取总耗时（毫秒）并打印
        long millis = stopWatch.getTotalTimeMillis();
        System.out.println("10000消息发送完成：ms时间："+millis);
    }

    /**
     * 测试发送对象类型的消息到Kafka
     * 发送一个Person对象到指定主题
     */
    @Test
    void send(){
        // 发送Person对象到"newshaha"主题，消息键为"person"
        // 注意：需要确保Kafka已配置合适的序列化器来处理Person对象
        CompletableFuture future = kafkaTemplate.send("newshaha", "person", new Person(1L, "张三", "hjaha@qq.com"));

        // 等待消息发送完成
        future.join();
        // 打印发送成功的提示信息
        System.out.println("消息发送成功...");
    }

}
```

<font style="color:rgb(77, 77, 77);">application.properties </font>

```java
# Kafka broker的地址列表，用于建立与Kafka集群的连接
# 格式为"host:port"，多个地址用逗号分隔
spring.kafka.bootstrap-servers=8.130.32.70:9092

# 配置Kafka生产者的键（key）的序列化器
# StringSerializer用于将字符串类型的键序列化为字节数组
spring.kafka.producer.key-serializer=org.apache.kafka.common.serialization.StringSerializer

# 配置Kafka生产者的值（value）的序列化器
# JsonSerializer是Spring Kafka提供的序列化器，能将Java对象序列化为JSON格式的字节数组
# 用于发送对象（如之前定义的Person类）可以直接作为消息发送，无需手动转换为字符串
spring.kafka.producer.value-serializer=org.springframework.kafka.support.serializer.JsonSerializer
```

#### <font style="color:rgb(79, 79, 79);">4.2.5 消息监听</font>
```java
// 标注为Spring组件，使其能被Spring容器扫描并管理
@Component
public class OrderMsgListener {

    /**
     * 监听"order"主题的消息
     * 属于"order-service"消费者组
     * 特点：只会消费监听器启动后发送到该主题的新消息，不会消费历史消息
     * @param record 接收到的Kafka消息记录，包含消息的键、值、分区、偏移量等信息
     */
    @KafkaListener(topics = "order", groupId = "order-service")
    public void listen(ConsumerRecord record) {
        // 打印接收到的消息详情
        System.out.println("收到消息：" + record); // 可以监听到发给kafka的新消息，以前的拿不到
    }

    /**
     * 监听"order"主题的0号分区消息
     * 属于"order-service-2"消费者组
     * 特点：从0号分区的偏移量0开始消费，会消费该分区的所有历史消息和新消息
     * @param record 接收到的Kafka消息记录
     */
    @KafkaListener(
        groupId = "order-service-2",  // 消费者组ID，不同组可以独立消费消息
        topicPartitions = {  // 精确指定要监听的主题分区及偏移量
            @TopicPartition(
                topic = "order",  // 要监听的主题名称
                partitionOffsets = {  // 指定分区和初始偏移量
                    @PartitionOffset(
                        partition = "0",  // 要监听的分区编号
                        initialOffset = "0"  // 从该分区的偏移量0开始消费
                    )
                }
            )
        })
    public void listenAll(ConsumerRecord record) {
        // 打印接收到的0号分区消息详情
        System.out.println("收到partion-0消息：" + record);
    }
}
```

---

```java
// 标注为Spring组件，让Spring容器自动扫描并管理该类
@Component
public class MyHahaTopicListener {

    /**
     * 监听"newshaha"主题的消息，属于"haha"消费者组
     * 默认消费行为：从主题的最新偏移量开始消费，只能接收到监听器启动后的新消息
     * @param record 封装了Kafka消息的对象，包含消息的键、值、分区、偏移量等信息
     */
    @KafkaListener(topics="newshaha",groupId="haha")
    public void haha(ConsumerRecord record){
        // 获取消息的键和值
        Object key = record.key();
        Object value = record.value();
        // 打印接收到的消息内容
        System.out.println("收到消息：key【"+key+"】 value【"+value+"】");
    }

    /**
     * 监听"newshaha"主题的消息，属于"hehe"消费者组
     * 自定义消费行为：从0号分区的偏移量0开始消费，能获取该分区的所有历史消息和新消息
     * @param record 封装了Kafka消息的对象
     */
    @KafkaListener(
        groupId = "hehe",  // 消费者组ID，与"haha"组相互独立
        topicPartitions = {  // 精确指定监听的分区及初始偏移量
            @TopicPartition(
                topic = "newshaha",  // 要监听的主题名称
                partitionOffsets = {  // 分区偏移量配置
                    @PartitionOffset(
                        partition = "0",  // 指定监听0号分区
                        initialOffset = "0"  // 从偏移量0开始消费（即分区的第一条消息）
                    )
                }
            )
        }
    )
    public void hehe(ConsumerRecord record){
        // 获取消息的键和值
        Object key = record.key();
        Object value = record.value();
        // 打印接收到的消息内容（使用特殊标记区分不同监听器的输出）
        System.out.println("======收到消息：key【"+key+"】 value【"+value+"】");
    }
}
```

#### <font style="color:rgb(79, 79, 79);">4.2.6 参数配置</font>
<font style="color:rgb(77, 77, 77);">消费者</font>

```properties
# 配置Kafka消费者的值（value）的反序列化器
# JsonDeserializer是Spring Kafka提供的反序列化器，能将JSON格式的字节数组反序列化为Java对象
spring.kafka.consumer.value-deserializer=org.springframework.kafka.support.serializer.JsonDeserializer

# 指定JSON反序列化时的默认目标类型
# 当消息中没有包含类型信息时，将使用该配置指定的类（com.example.Invoice）进行反序列化
spring.kafka.consumer.properties[spring.json.value.default.type]=com.example.Invoice

# 配置JSON反序列化器信任的包列表
# 出于安全考虑，JsonDeserializer默认只信任特定包下的类进行反序列化
# 这里指定了允许反序列化的包：com.example.main和com.example.another
spring.kafka.consumer.properties[spring.json.trusted.packages]=com.example.main,com.example.another
```

<font style="color:rgb(77, 77, 77);">生产者 </font>

```properties
# 配置Kafka生产者的消息值（value）序列化器
# JsonSerializer是Spring Kafka提供的序列化器，核心功能是将Java对象自动序列化为JSON格式的字节数组
# 配合消费者端的JsonDeserializer，实现Java对象在Kafka中的"对象→JSON→对象"传输
# 无需手动编写JSON转换代码（如Jackson的ObjectMapper），简化对象消息发送逻辑
spring.kafka.producer.value-serializer=org.springframework.kafka.support.serializer.JsonSerializer

# 控制JsonSerializer在序列化时是否向消息头（headers）中添加类型信息
# 配置值为false：不添加类型头，消息中仅包含纯JSON数据，不携带Java对象的全类名（如com.atguigu.boot3.message.entity.Person）
# 注意事项：
# 1. 若关闭此配置，消费者端必须通过配置"spring.kafka.consumer.properties[spring.json.value.default.type]"指定默认反序列化类型
# 2. 若开启（值为true，默认可能为true），序列化器会自动在消息头添加"__TypeId__"字段存储类名，消费者可直接根据头信息反序列化，无需指定默认类型
# 3. 关闭此配置可减少消息额外开销，但需保证消费者端明确知道反序列化目标类型
spring.kafka.producer.properties[spring.json.add.type.headers]=false
```

#### <font style="color:rgb(79, 79, 79);">4.2.7 自动配置原理</font>
<font style="color:rgb(48, 54, 63);">kafka 自动配置在</font><font style="color:rgb(213, 97, 97);background-color:rgb(240, 244, 248);">KafkaAutoConfiguration</font>

1. <font style="color:rgb(48, 54, 63);">容器中放了</font><font style="color:rgb(48, 54, 63);"> </font><font style="color:rgb(213, 97, 97);background-color:rgb(240, 244, 248);">KafkaTemplate</font><font style="color:rgb(51, 51, 51);"> </font><font style="color:rgb(48, 54, 63);">可以进行消息收发</font>
2. <font style="color:rgb(48, 54, 63);">容器中放了</font><font style="color:rgb(213, 97, 97);background-color:rgb(240, 244, 248);">KafkaAdmin</font><font style="color:rgb(51, 51, 51);"> </font><font style="color:rgb(48, 54, 63);">可以进行 Kafka 的管理，比如创建 topic 等</font>
3. <font style="color:rgb(48, 54, 63);">kafka 的配置在</font><font style="color:rgb(213, 97, 97);background-color:rgb(240, 244, 248);">KafkaProperties</font><font style="color:rgb(48, 54, 63);">中</font><font style="color:rgb(48, 54, 63);"></font>
4. <font style="color:rgb(213, 97, 97);background-color:rgb(240, 244, 248);">@EnableKafka</font><font style="color:rgb(48, 54, 63);">可以开启基于注解的模式</font>

### <font style="color:rgb(79, 79, 79);">5 Web安全</font>
+ <font style="color:rgb(51, 51, 51);">Apache Shiro</font>
+ <font style="color:rgb(51, 51, 51);">Spring Security</font>
+ <font style="color:rgb(51, 51, 51);">自研：Filter</font>

#### <font style="color:rgb(79, 79, 79);">5.1 安全架构</font>
##### <font style="color:rgb(79, 79, 79);">5.1.1 认证：Authentication</font>
<font style="color:rgb(77, 77, 77);">who are you?</font>

<font style="color:rgb(77, 77, 77);">登录系统，用户系统</font>

##### <font style="color:rgb(79, 79, 79);">5.1.2 授权：Authorization</font>
<font style="color:rgb(77, 77, 77);">what are you allowed to do？</font>

<font style="color:rgb(77, 77, 77);">权限管理，用户授权</font>

##### <font style="color:rgb(79, 79, 79);">5.1.3 攻击防护</font>
+ <font style="color:rgb(77, 77, 77);">XSS（Cross-site scripting）跨站脚本攻击</font>
+ <font style="color:rgb(77, 77, 77);">CSRF（Cross-site request forgery）跨站请求伪造</font>
+ <font style="color:rgb(77, 77, 77);">CORS（Cross-Origin Resource Sharing）跨域资源共享</font>
+ <font style="color:rgb(77, 77, 77);">SQL注入</font>

#### <font style="color:rgb(79, 79, 79);">5.1.4 扩展. 权限模型</font>
##### <font style="color:rgb(79, 79, 79);">5.1.4.1 RBAC(Role Based Access Controll)</font>
> <font style="color:rgb(79, 79, 79);">用户（t_user）</font>
>
> + <font style="color:rgb(79, 79, 79);">id,username,password，xxx</font>
> + <font style="color:rgb(79, 79, 79);">1,zhangsan</font>
> + <font style="color:rgb(79, 79, 79);">2,lisi</font>
>
> <font style="color:rgb(79, 79, 79);">用户_角色（t_user_role）【N对N关系需要中间表】</font>
>
> + <font style="color:rgb(79, 79, 79);">zhangsan, admin</font>
> + <font style="color:rgb(79, 79, 79);">zhangsan,common_user</font>
> + **<font style="color:rgb(79, 79, 79);">lisi,</font>**<font style="color:rgb(79, 79, 79);"> </font>**<font style="color:rgb(79, 79, 79);">hr</font>**
> + **<font style="color:rgb(79, 79, 79);">lisi, common_user</font>**
>
> <font style="color:rgb(79, 79, 79);">角色（t_role）</font>
>
> + <font style="color:rgb(79, 79, 79);">id,role_name</font>
> + <font style="color:rgb(79, 79, 79);">admin</font>
> + <font style="color:rgb(79, 79, 79);">hr</font>
> + <font style="color:rgb(79, 79, 79);">common_user</font>
>
> <font style="color:rgb(79, 79, 79);">角色_权限(t_role_perm)  【N对N关系需要中间表】</font>
>
> + <font style="color:rgb(79, 79, 79);">admin, 文件r</font>
> + <font style="color:rgb(79, 79, 79);">admin, 文件w</font>
> + <font style="color:rgb(79, 79, 79);">admin, 文件执行</font>
> + <font style="color:rgb(79, 79, 79);">admin, 订单query，create,xxx</font>
> + **<font style="color:rgb(79, 79, 79);">hr, 文件r</font>**
>
> <font style="color:rgb(79, 79, 79);">权限（t_permission）</font>
>
> + <font style="color:rgb(79, 79, 79);">id,perm_id</font>
> + <font style="color:rgb(79, 79, 79);">文件 r,w,x</font>
> + <font style="color:rgb(79, 79, 79);">订单 query,create,xxx</font>
>

##### <font style="color:rgb(79, 79, 79);">5.1.4.2 ACL(Access Controll List)</font>
> <font style="color:rgb(79, 79, 79);">直接用户和权限挂钩</font>
>
> + <font style="color:rgb(79, 79, 79);">用户（t_user）</font>
> + <font style="color:rgb(79, 79, 79);">zhangsan</font>
> + <font style="color:rgb(79, 79, 79);">lisi</font>
>
> <font style="color:rgb(79, 79, 79);">用户_权限(t_user_perm)</font>
>
> + <font style="color:rgb(79, 79, 79);">zhangsan,文件 r</font>
> + <font style="color:rgb(79, 79, 79);">zhangsan,文件 x</font>
> + <font style="color:rgb(79, 79, 79);">zhangsan,订单 query</font>
>
> <font style="color:rgb(79, 79, 79);">权限（t_permission）</font>
>
> + <font style="color:rgb(79, 79, 79);">id,perm_id</font>
> + <font style="color:rgb(79, 79, 79);">文件 r,w,x</font>
> + <font style="color:rgb(79, 79, 79);">订单 query,create,xxx</font>
>

```java
@Secured("文件 r")
public void readFile(){
    //读文件
}
```

#### <font style="color:rgb(79, 79, 79);">5.2 Spring Security原理</font>
##### <font style="color:rgb(79, 79, 79);">5.2.1 过滤器链架构</font>
<font style="color:rgb(79, 79, 79);background-color:rgb(238, 240, 244);">Spring Security利用 FilterChainProxy 封装一系列拦截器链，实现各种安全拦截功能</font>

<font style="color:rgb(79, 79, 79);background-color:rgb(238, 240, 244);">Servlet三大组件：Servlet、Filter、Listener</font>

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1755895817754-315a4712-edb2-4f0f-9825-da8b6d0dde92.png)

##### <font style="color:rgb(79, 79, 79);">5.2.2 FilterChainProxy</font>
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1755895817852-bd3fccf1-db91-41ca-8250-c39edc5f3653.png)

##### <font style="color:rgb(79, 79, 79);">5.2.3 SecurityFilterChain</font>
##### <!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1755895817825-a55ad6f1-bbdd-4068-911f-fea1a65914d7.png)
#### <font style="color:rgb(79, 79, 79);">5.3 使用</font>
##### <font style="color:rgb(79, 79, 79);">5.3.1 HttpSecurity</font>
```java
// 标注此类为配置类，Spring会自动扫描并应用其中的配置
@Configuration
// 设置配置类的加载顺序，比默认的基本认证配置优先级高（-10表示提前10个优先级）
// 确保此配置会覆盖默认配置中冲突的部分
@Order(SecurityProperties.BASIC_AUTH_ORDER - 10)
// 继承WebSecurityConfigurerAdapter，用于自定义Spring Security配置
public class ApplicationConfigurerAdapter extends WebSecurityConfigurerAdapter {
  
  /**
   * 重写configure方法，自定义HTTP请求的安全配置
   * @param http HttpSecurity对象，用于配置HTTP安全相关的规则
   * @throws Exception 可能抛出的异常
   */
  @Override
  protected void configure(HttpSecurity http) throws Exception {
    // 配置仅对路径匹配"/match1/**"的请求生效
    http.antMatcher("/match1/**")
      // 开启请求授权配置
      .authorizeRequests()
        // 配置"/match1/user"路径，仅允许拥有"USER"角色的用户访问
        // 注意：这里的hasRole("USER")会自动加上"ROLE_"前缀，实际匹配的是"ROLE_USER"角色
        .antMatchers("/match1/user").hasRole("USER")
        // 配置"/match1/spam"路径，仅允许拥有"SPAM"角色的用户访问
        // 实际匹配的是"ROLE_SPAM"角色
        .antMatchers("/match1/spam").hasRole("SPAM")
        // 对于其他所有匹配"/match1/**"的请求，要求用户必须已认证（登录状态）
        .anyRequest().isAuthenticated();
    
    // 注意：此配置中没有显式配置登录页面、退出登录等功能
    // 会使用Spring Security的默认配置（如默认登录页面、HTTP Basic认证等）
  }
}
```

##### <font style="color:rgb(79, 79, 79);">5.3.2 MethodSecurity</font>
```java
// 标注为Spring Boot应用的主类，包含组件扫描、自动配置等功能
@SpringBootApplication
// 启用全局方法安全注解，securedEnabled = true表示开启@Secured注解支持
// 用于控制类或方法的访问权限，实现方法级别的安全控制
@EnableGlobalMethodSecurity(securedEnabled = true)
public class SampleSecureApplication {
    // Spring Boot应用的入口点，main方法会启动Spring容器
    public static void main(String[] args) {
        SpringApplication.run(SampleSecureApplication.class, args);
    }
}

// 标注为Spring服务类，会被Spring容器管理
@Service
public class MyService {

    /**
     * 受保护的方法，仅允许拥有ROLE_USER角色的用户访问
     * @Secured注解指定访问该方法所需的角色
     * 注意：这里必须使用完整的角色名称（包含ROLE_前缀），与hasRole()不同
     * @return 安全提示信息
     */
    @Secured("ROLE_USER")
    public String secure() {
        return "Hello Security";
    }

}
```

<font style="color:rgb(77, 77, 77);">核心</font>

+ **<font style="color:rgb(51, 51, 51);">WebSecurityConfigurerAdapter</font>**
+ <font style="color:rgb(51, 51, 51);">@</font>**<font style="color:rgb(51, 51, 51);">EnableGlobalMethodSecurity</font>**<font style="color:rgb(51, 51, 51);">： 开启全局方法安全配置</font>
    - <font style="color:rgb(51, 51, 51);">@Secured</font>
    - <font style="color:rgb(25, 30, 30);">@PreAuthorize</font>
    - <font style="color:rgb(25, 30, 30);">@PostAuthorize</font>
+ **<font style="color:rgb(51, 51, 51);">UserDetailService： 去数据库查询用户详细信息的service（用户基本信息、用户角色、用户权限）</font>**

#### <font style="color:rgb(79, 79, 79);">5.4 实战</font>
##### <font style="color:rgb(79, 79, 79);">5.4.1 引入依赖</font>
```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
<!--引入security场景-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>3.0.0</version>
</dependency>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
    <scope>runtime</scope>
    <optional>true</optional>
</dependency>
<dependency>
    <groupId>com.mysql</groupId>
    <artifactId>mysql-connector-j</artifactId>
    <scope>runtime</scope>
</dependency>
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <optional>true</optional>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.thymeleaf.extras</groupId>
    <artifactId>thymeleaf-extras-springsecurity6</artifactId>
    <!-- Temporary explicit version to fix Thymeleaf bug -->
    <version>3.1.1.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-test</artifactId>
    <scope>test</scope>
</dependency>
```

##### <font style="color:rgb(79, 79, 79);">5.4.2 页面</font>
###### <font style="color:rgb(79, 79, 79);">5.4.2.1 首页</font>
```java
<p>Click <a th:href="@{/hello}">here</a> to see a greeting.</p>
```

###### <font style="color:rgb(79, 79, 79);">5.4.2.2 Hello页</font>
```html
<h1>Hello</h1>
```

###### <font style="color:rgb(79, 79, 79);">5.4.2.3 登录页</font>
```html
<!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml" xmlns:th="https://www.thymeleaf.org">
  <head>
    <title>Spring Security Example</title>
  </head>
  <body>
    <div th:if="${param.error}">Invalid username and password.</div>
    <div th:if="${param.logout}">You have been logged out.</div>
    <form th:action="@{/login}" method="post">
      <div>
        <label> User Name : <input type="text" name="username" /> </label>
      </div>
      <div>
        <label> Password: <input type="password" name="password" /> </label>
      </div>
      <div><input type="submit" value="登录" /></div>
    </form>
  </body>
</html>
```

##### <font style="color:rgb(79, 79, 79);">5.4.3 配置类</font>
###### <font style="color:rgb(79, 79, 79);">5.4.3.1 视图控制</font>
```java
@Configuration
public class MvcConfig implements WebMvcConfigurer {

    public void addViewControllers(ViewControllerRegistry registry) {
        registry.addViewController("/home").setViewName("index");
        registry.addViewController("/").setViewName("index");
        registry.addViewController("/hello").setViewName("hello");
        registry.addViewController("/login").setViewName("login");
    }
}
```

##### <font style="color:rgb(79, 79, 79);">5.4.3.2 Security配置</font>
```java
// 标注为配置类，Spring会自动扫描并应用该配置
@Configuration
// 启用Web安全功能，是Spring Security用于Web应用的核心注解
@EnableWebSecurity
public class WebSecurityConfig {

    /**
     * 配置安全过滤器链，定义HTTP请求的安全规则
     * SecurityFilterChain是Spring Security 5.7+推荐的配置方式，替代了WebSecurityConfigurerAdapter
     * 
     * @param http HttpSecurity对象，用于配置HTTP安全相关的所有内容
     * @return 构建好的SecurityFilterChain对象
     * @throws Exception 配置过程中可能抛出的异常
     */
    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
            // 配置HTTP请求的授权规则
            .authorizeHttpRequests((requests) -> requests
                // 允许所有人访问根路径("/")和"/home"路径，无需认证
                .requestMatchers("/", "/home").permitAll()
                // 其他所有请求都需要认证（登录）后才能访问
                .anyRequest().authenticated()
            )
            // 配置表单登录
            .formLogin((form) -> form
                // 指定自定义的登录页面路径为"/login"
                .loginPage("/login")
                // 允许所有人访问登录页面，否则未登录用户无法进入登录页面
                .permitAll()
            )
            // 配置注销功能
            .logout((logout) -> logout
                // 允许所有人访问注销功能
                .permitAll());

        // 构建并返回配置好的SecurityFilterChain
        return http.build();
    }

    /**
     * 配置用户详情服务，用于提供用户信息（用户名、密码、角色等）
     * 这里使用内存中的用户存储，适合演示和简单场景
     * 
     * @return UserDetailsService对象，提供用户信息查询服务
     */
    @Bean
    public UserDetailsService userDetailsService() {
        // 创建一个用户实例
        UserDetails user =
            // 使用默认密码编码器（仅用于演示，生产环境不推荐）
            User.withDefaultPasswordEncoder()
                // 用户名设置为"admin"
                .username("admin")
                // 密码设置为"admin"
                .password("admin")
                // 为用户分配"USER"角色
                .roles("USER")
                // 构建用户对象
                .build();

        // 返回基于内存的用户详情管理器，管理上面创建的用户
        return new InMemoryUserDetailsManager(user);
    }
}
```

##### <font style="color:rgb(79, 79, 79);">5.4.4 改造Hello页</font>
```html
<!DOCTYPE html>
<html
  xmlns="http://www.w3.org/1999/xhtml"
  xmlns:th="https://www.thymeleaf.org"
  xmlns:sec="https://www.thymeleaf.org/thymeleaf-extras-springsecurity6"
>
  <head>
    <title>Hello World!</title>
  </head>
  <body>
    <h1 th:inline="text">
      Hello <span th:remove="tag" sec:authentication="name">thymeleaf</span>!
    </h1>
    <form th:action="@{/logout}" method="post">
      <input type="submit" value="Sign Out" />
    </form>
  </body>
</html>
```

---

<font style="color:rgb(77, 77, 77);">com/atguigu/boot3/security/Boot313SecurityApplication.java</font>

```java
/**
 * Security场景的自动配置类：
 * SecurityAutoConfiguration、SpringBootWebSecurityConfiguration、SecurityFilterAutoConfiguration、
 * 1、security的所有配置在 SecurityProperties： 以spring.security开头
 * 2、默认SecurityFilterChain组件：
 *   - 所有请求都需要认证（登录）
 *   - 开启表单登录: spring security提供一个默认登录页，未经登录的所有请求都需要登录
 *   - httpbasic方式登录
 * 3、@EnableWebSecurity 生效
 *   - WebSecurityConfiguration生效：web安全配置
 *   - HttpSecurityConfiguration生效：http安全规则
 *   - @EnableGlobalAuthentication生效：全局认证生效
 *     - AuthenticationConfiguration：认证配置
 */
@SpringBootApplication
public class Boot313SecurityApplication {

    public static void main(String[] args) {
        SpringApplication.run(Boot313SecurityApplication.class, args);
    }

}
```

<font style="color:rgb(77, 77, 77);">application.properties</font>

```properties
spring.security.user.name=zhangsan
spring.security.user.password=123456
spring.security.user.roles=admin,common,hr
```

<font style="color:rgb(77, 77, 77);">com/atguigu/boot3/security/config/AppSecurityConfiguration.java</font>

```java
/**
 * 1、自定义请求授权规则：http.authorizeHttpRequests
 * 2、自定义登录规则：http.formLogin
 * 3、自定义用户信息查询规则：UserDetailsService
 * 4、开启方法级别的精确权限控制：@EnableMethodSecurity + @PreAuthorize("hasAuthority('world_exec')")
 */
@EnableMethodSecurity
@Configuration
public class AppSecurityConfiguration {

    @Bean
    SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        //请求授权
        http.authorizeHttpRequests(registry -> {
            registry.requestMatchers("/").permitAll() //1、首页所有人都允许
                    .anyRequest().authenticated(); //2、剩下的任意请求都需要 认证（登录）
        });

        //表单登录
        //3、表单登录功能：开启默认表单登录功能；Spring Security提供默认登录页
        http.formLogin(formLogin -> {
            formLogin.loginPage("/login").permitAll(); //自定义登录页位置，并且所有人都能访问
        });

        return http.build();
    }

    @Bean //查询用户详情；
    UserDetailsService userDetailsService(PasswordEncoder passwordEncoder){
        UserDetails zhangsan = User.withUsername("zhangsan")
                .password(passwordEncoder.encode("123456")) //使用密码加密器加密密码进行存储
                .roles("admin", "hr")
                .authorities("file_read", "file_write")
                .build();

        UserDetails lisi = User.withUsername("lisi")
                .password(passwordEncoder.encode("123456"))
                .roles("hr")
                .authorities("file_read")
                .build();

        UserDetails wangwu = User.withUsername("wangwu")
                .password(passwordEncoder.encode("123456"))
                .roles("admin")
                .authorities("file_write","world_exec")
                .build();

        //默认内存中保存所有用户信息
        InMemoryUserDetailsManager manager = new InMemoryUserDetailsManager(zhangsan,lisi,wangwu);
        return manager;
    }


    @Bean //密码加密器
    PasswordEncoder passwordEncoder(){
        return new BCryptPasswordEncoder();
    }
}
```

<font style="color:rgb(77, 77, 77);">com/atguigu/boot3/security/controller/HelloController.java</font>

```java
package com.atguigu.boot3.security.controller;

import org.springframework.security.access.prepost.PreAuthorize;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HelloController {

    @GetMapping("/hello")
    public String hello(){
        return "Hello!Spring Security";
    }

    @PreAuthorize("hasAuthority('world_exec')")
    @GetMapping("/world")
    public String world(){
        return "Hello World!!!";
    }
}
```

<font style="color:rgb(77, 77, 77);">com/atguigu/boot3/security/controller/LoginController.java</font>

```java
package com.atguigu.boot3.security.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;

@Controller
public class LoginController {

    @GetMapping("/login")
    public String loginPage(){

        return "login";
    }
}
```

<font style="color:rgb(77, 77, 77);">templates/index.html</font>

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
Welcome To  尚硅谷 <br/>
<a th:href="@{/hello}">hello</a> <br/>
<a th:href="@{/world}">world</a> <br/>
</body>
</html>
```

<font style="color:rgb(77, 77, 77);">templates/login.html</font>

```html
<!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml" xmlns:th="https://www.thymeleaf.org">
<head>
  <title>Spring Security Example</title>
</head>
<body>
<div th:if="${param.error}">Invalid username and password.</div>
<div th:if="${param.logout}">You have been logged out.</div>
<form th:action="@{/login}" method="post">
  <div>
    <label> User Name : <input type="text" name="username" /> </label>
  </div>
  <div>
    <label> Password: <input type="password" name="password" /> </label>
  </div>
  <div><input type="submit" value="登录" /></div>
</form>
</body>
</html>
```

### <font style="color:rgb(79, 79, 79);">6 可观测性</font>
> <font style="color:rgb(79, 79, 79);background-color:rgb(238, 240, 244);">可观测性 Observability</font>
>

<font style="color:rgb(77, 77, 77);">对线上应用进行观测、监控、预警...</font>

+ <font style="color:rgb(51, 51, 51);">健康状况【组件状态、存活状态】Health</font>
+ <font style="color:rgb(51, 51, 51);">运行</font>**<font style="color:rgb(51, 51, 51);">指标</font>**<font style="color:rgb(51, 51, 51);">【cpu、内存、垃圾回收、吞吐量、响应成功率...】</font>**<font style="color:rgb(51, 51, 51);">Metrics</font>**
+ <font style="color:rgb(51, 51, 51);">链路追踪</font>
+ <font style="color:rgb(51, 51, 51);">...</font>

#### <font style="color:rgb(79, 79, 79);">6.1 SpringBoot Actuator</font>
##### <font style="color:rgb(79, 79, 79);">6.1.1 实战</font>
###### <font style="color:rgb(79, 79, 79);">6.1.1.1 场景引入</font>
```plain
<!--可观测性场景启动器，线上指标监控、运行状态监控-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

###### <font style="color:rgb(79, 79, 79);">6.1.1.2 暴露指标</font>
```java
management:
  endpoints:
    enabled-by-default: true #暴露所有端点信息
    web:
      exposure:
        include: '*'  #以web方式暴露
```

###### <font style="color:rgb(79, 79, 79);">6.1.1.3 访问数据</font>
+ <font style="color:rgb(51, 51, 51);">访问</font><font style="color:rgb(51, 51, 51);"> </font>[<font style="color:rgb(78, 161, 219);">http://localhost:8080/actuator</font>](http://localhost:8080/actuator/)<font style="color:rgb(51, 51, 51);">；展示出所有可以用的监控端点</font>
+ [<font style="color:rgb(78, 161, 219);">http://localhost:8080/actuator/beans</font>](http://localhost:8080/actuator/beans)
+ [<font style="color:rgb(78, 161, 219);">http://localhost:8080/actuator/configprops</font>](http://localhost:8080/actuator/configprops)
+ [<font style="color:rgb(78, 161, 219);">http://localhost:8080/actuator/metrics</font>](http://localhost:8080/actuator/metrics)
+ [<font style="color:rgb(78, 161, 219);">http://localhost:8080/actuator/metrics/jvm.gc.pause</font>](http://localhost:8080/actuator/metrics/jvm.gc.pause)
+ [<font style="color:rgb(78, 161, 219);">http://localhost:8080/actuator/</font>](http://localhost:8080/actuator/metrics)<font style="color:rgb(51, 51, 51);">endpointName/detailPath</font>

##### <font style="color:rgb(79, 79, 79);">6.1.2 Endpoint</font>
###### <font style="color:rgb(79, 79, 79);">6.1.2.1 常用端点</font>
| <font style="color:rgb(79, 79, 79);">ID</font> | <font style="color:rgb(79, 79, 79);">描述</font> |
| :--- | :--- |
| `<font style="color:rgb(79, 79, 79);">auditevents</font>` | <font style="color:rgb(79, 79, 79);">暴露当前应用程序的审核事件信息。需要一个</font>`<font style="color:rgb(79, 79, 79);">AuditEventRepository组件</font>`<font style="color:rgb(79, 79, 79);">。</font> |
| `<font style="color:rgb(79, 79, 79);">beans</font>` | <font style="color:rgb(79, 79, 79);">显示应用程序中所有Spring Bean的完整列表。</font> |
| `<font style="color:rgb(79, 79, 79);">caches</font>` | <font style="color:rgb(79, 79, 79);">暴露可用的缓存。</font> |
| `<font style="color:rgb(79, 79, 79);">conditions</font>` | <font style="color:rgb(79, 79, 79);">显示自动配置的所有条件信息，包括匹配或不匹配的原因。</font> |
| `<font style="color:rgb(79, 79, 79);">configprops</font>` | <font style="color:rgb(79, 79, 79);">显示所有</font>`<font style="color:rgb(79, 79, 79);">@ConfigurationProperties</font>`<font style="color:rgb(79, 79, 79);">。</font> |
| `<font style="color:rgb(79, 79, 79);">env</font>` | <font style="color:rgb(79, 79, 79);">暴露Spring的属性</font>`<font style="color:rgb(79, 79, 79);">ConfigurableEnvironment</font>` |
| `<font style="color:rgb(79, 79, 79);">flyway</font>` | <font style="color:rgb(79, 79, 79);">显示已应用的所有Flyway数据库迁移。   </font><font style="color:rgb(79, 79, 79);">需要一个或多个</font>`<font style="color:rgb(79, 79, 79);">Flyway</font>`<font style="color:rgb(79, 79, 79);">组件。</font> |
| `<font style="color:rgb(79, 79, 79);">health</font>` | <font style="color:rgb(79, 79, 79);">显示应用程序运行状况信息。</font> |
| `<font style="color:rgb(79, 79, 79);">httptrace</font>` | <font style="color:rgb(79, 79, 79);">显示HTTP跟踪信息（默认情况下，最近100个HTTP请求-响应）。需要一个</font>`<font style="color:rgb(79, 79, 79);">HttpTraceRepository</font>`<font style="color:rgb(79, 79, 79);">组件。</font> |
| `<font style="color:rgb(79, 79, 79);">info</font>` | <font style="color:rgb(79, 79, 79);">显示应用程序信息。</font> |
| `<font style="color:rgb(79, 79, 79);">integrationgraph</font>` | <font style="color:rgb(79, 79, 79);">显示Spring </font>`<font style="color:rgb(79, 79, 79);">integrationgraph</font>`<font style="color:rgb(79, 79, 79);"> </font><font style="color:rgb(79, 79, 79);">。需要依赖</font>`<font style="color:rgb(79, 79, 79);">spring-integration-core</font>`<font style="color:rgb(79, 79, 79);">。</font> |
| `<font style="color:rgb(79, 79, 79);">loggers</font>` | <font style="color:rgb(79, 79, 79);">显示和修改应用程序中日志的配置。</font> |
| `<font style="color:rgb(79, 79, 79);">liquibase</font>` | <font style="color:rgb(79, 79, 79);">显示已应用的所有Liquibase数据库迁移。需要一个或多个</font>`<font style="color:rgb(79, 79, 79);">Liquibase</font>`<font style="color:rgb(79, 79, 79);">组件。</font> |
| `<font style="color:rgb(79, 79, 79);">metrics</font>` | <font style="color:rgb(79, 79, 79);">显示当前应用程序的“指标”信息。</font> |
| `<font style="color:rgb(79, 79, 79);">mappings</font>` | <font style="color:rgb(79, 79, 79);">显示所有</font>`<font style="color:rgb(79, 79, 79);">@RequestMapping</font>`<font style="color:rgb(79, 79, 79);">路径列表。</font> |
| `<font style="color:rgb(79, 79, 79);">scheduledtasks</font>` | <font style="color:rgb(79, 79, 79);">显示应用程序中的计划任务。</font> |
| `<font style="color:rgb(79, 79, 79);">sessions</font>` | <font style="color:rgb(79, 79, 79);">允许从Spring Session支持的会话存储中检索和删除用户会话。需要使用Spring Session的基于Servlet的Web应用程序。</font> |
| `<font style="color:rgb(79, 79, 79);">shutdown</font>` | <font style="color:rgb(79, 79, 79);">使应用程序正常关闭。默认禁用。</font> |
| `<font style="color:rgb(79, 79, 79);">startup</font>` | <font style="color:rgb(79, 79, 79);">显示由</font>`<font style="color:rgb(79, 79, 79);">ApplicationStartup</font>`<font style="color:rgb(79, 79, 79);">收集的启动步骤数据。需要使用</font>`<font style="color:rgb(79, 79, 79);">SpringApplication</font>`<font style="color:rgb(79, 79, 79);">进行配置</font>`<font style="color:rgb(79, 79, 79);">BufferingApplicationStartup</font>`<font style="color:rgb(79, 79, 79);">。</font> |
| `<font style="color:rgb(79, 79, 79);">threaddump</font>` | <font style="color:rgb(79, 79, 79);">执行线程转储。</font> |
| `<font style="color:rgb(79, 79, 79);">heapdump</font>` | <font style="color:rgb(79, 79, 79);">返回</font>`<font style="color:rgb(79, 79, 79);">hprof</font>`<font style="color:rgb(79, 79, 79);">堆转储文件。</font> |
| `<font style="color:rgb(79, 79, 79);">jolokia</font>` | <font style="color:rgb(79, 79, 79);">通过HTTP暴露JMX bean（需要引入Jolokia，不适用于WebFlux）。需要引入依赖</font>`<font style="color:rgb(79, 79, 79);">jolokia-core</font>`<font style="color:rgb(79, 79, 79);">。</font> |
| `<font style="color:rgb(79, 79, 79);">logfile</font>` | <font style="color:rgb(79, 79, 79);">返回日志文件的内容（如果已设置</font>`<font style="color:rgb(79, 79, 79);">logging.file.name</font>`<font style="color:rgb(79, 79, 79);">或</font>`<font style="color:rgb(79, 79, 79);">logging.file.path</font>`<font style="color:rgb(79, 79, 79);">属性）。支持使用HTTP</font>`<font style="color:rgb(79, 79, 79);">Range</font>`<font style="color:rgb(79, 79, 79);">标头来检索部分日志文件的内容。</font> |
| `<font style="color:rgb(79, 79, 79);">prometheus</font>` | <font style="color:rgb(79, 79, 79);">以Prometheus服务器可以抓取的格式公开指标。需要依赖</font>`<font style="color:rgb(79, 79, 79);">micrometer-registry-prometheus</font>`<font style="color:rgb(79, 79, 79);">。</font> |


`<font style="color:rgb(13, 0, 22);">threaddump</font>`<font style="color:rgb(13, 0, 22);">、</font>`<font style="color:rgb(13, 0, 22);">heapdump</font>`<font style="color:rgb(13, 0, 22);">、</font>`<font style="color:rgb(13, 0, 22);">metrics</font>`

###### <font style="color:rgb(79, 79, 79);">6.1.2.2 定制端点</font>
+ <font style="color:rgb(51, 51, 51);">健康监控：返回存活、死亡</font>
+ <font style="color:rgb(51, 51, 51);">指标监控：次数、率</font>

**<font style="color:rgb(26, 67, 156);">1. HealthEndpoint</font>**

```java
import org.springframework.boot.actuate.health.Health;
import org.springframework.boot.actuate.health.HealthIndicator;
import org.springframework.stereotype.Component;

@Component
public class MyHealthIndicator implements HealthIndicator {

    @Override
    public Health health() {
        int errorCode = check(); // perform some specific health check
        if (errorCode != 0) {
            return Health.down().withDetail("Error Code", errorCode).build();
        }
        return Health.up().build();
    }

}

构建Health
Health build = Health.down()
                .withDetail("msg", "error service")
                .withDetail("code", "500")
                .withException(new RuntimeException())
                .build();
```

```java
management:
    health:
      enabled: true
      show-details: always #总是显示详细信息。可显示每个模块的状态信息
```

```java
@Component
public class MyComHealthIndicator extends AbstractHealthIndicator {

    /**
     * 真实的检查方法
     * @param builder
     * @throws Exception
     */
    @Override
    protected void doHealthCheck(Health.Builder builder) throws Exception {
        //mongodb。  获取连接进行测试
        Map<String,Object> map = new HashMap<>();
        // 检查完成
        if(1 == 2){
//            builder.up(); //健康
            builder.status(Status.UP);
            map.put("count",1);
            map.put("ms",100);
        }else {
//            builder.down();
            builder.status(Status.OUT_OF_SERVICE);
            map.put("err","连接超时");
            map.put("ms",3000);
        }


        builder.withDetail("code",100)
                .withDetails(map);

    }
}
```

---

<font style="color:rgb(77, 77, 77);">application.properties</font>

```java
server.port=9999
#通过web方式暴露所有监控端点
management.endpoints.web.exposure.include=*
management.endpoint.health.enabled=true
management.endpoint.health.show-details=always
```

<font style="color:rgb(77, 77, 77);">com.atguigu.boot3.actuator.component.MyHahaComponent</font>

```java
package com.atguigu.boot3.actuator.component;

import io.micrometer.core.instrument.Counter;
import io.micrometer.core.instrument.MeterRegistry;
import org.springframework.stereotype.Component;

@Component
public class MyHahaComponent {

    public  int check(){
        //业务代码判断这个组件是否该是存活状态
        return 1;
    }

}
```

<font style="color:rgb(77, 77, 77);">com.atguigu.boot3.actuator.health.MyHahaHealthIndicator</font>

```java
package com.atguigu.boot3.actuator.health;

import com.atguigu.boot3.actuator.component.MyHahaComponent;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.actuate.health.AbstractHealthIndicator;
import org.springframework.boot.actuate.health.Health;
import org.springframework.boot.actuate.health.HealthIndicator;
import org.springframework.stereotype.Component;

/**
 *
 * 第一种：实现 HealthIndicator 接口来定制组件的健康状态对象（Health） 返回
 * 第二种：继承 AbstractHealthIndicator抽象类，重写 doHealthCheck
 */
@Component
public class MyHahaHealthIndicator extends AbstractHealthIndicator {

    @Autowired
    MyHahaComponent myHahaComponent;
    /**
     * 健康检查
     * @param builder
     * @throws Exception
     */
    @Override
    protected void doHealthCheck(Health.Builder builder) throws Exception {
        //自定义检查方法

        int check = myHahaComponent.check();
        if(check == 1){
            //存活
            builder.up()
                    .withDetail("code","1000")
                    .withDetail("msg","活的很健康")
                    .withDetail("data","我的名字叫haha")
                    .build();
        }else {
            //下线
            builder.down()
                    .withDetail("code","1001")
                    .withDetail("msg","死的很健康")
                    .withDetail("data","我的名字叫haha完蛋")
                    .build();
        }

    }
}
```

**<font style="color:rgb(26, 67, 156);">2. MetricsEndpoint</font>**

```java
class MyService{
    Counter counter;
    public MyService(MeterRegistry meterRegistry){
         counter = meterRegistry.counter("myservice.method.running.counter");
    }

    public void hello() {
        counter.increment();
    }
}
```

---

<font style="color:rgb(77, 77, 77);">com.atguigu.boot3.actuator.controller.HelloController</font>

```java
package com.atguigu.boot3.actuator.controller;

import com.atguigu.boot3.actuator.component.MyHahaComponent;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HelloController {

    @Autowired
    MyHahaComponent myHahaComponent;

    @GetMapping("/hello")
    public String hello(){
        //业务调用
        myHahaComponent.hello();
        return "哈哈哈";
    }
}
```

<font style="color:rgb(77, 77, 77);">com.atguigu.boot3.actuator.component.MyHahaComponent</font>

```java
package com.atguigu.boot3.actuator.component;

import io.micrometer.core.instrument.Counter;
import io.micrometer.core.instrument.MeterRegistry;
import org.springframework.stereotype.Component;

@Component
public class MyHahaComponent {
    Counter counter = null;

    /**
     * 注入 meterRegistry 来保存和统计所有指标
     * @param meterRegistry
     */
    public MyHahaComponent(MeterRegistry meterRegistry){
        //得到一个名叫 myhaha.hello 的计数器
        counter = meterRegistry.counter("myhaha.hello");
    }
    public  int check(){
        //业务代码判断这个组件是否该是存活状态
        return 1;
    }

    public void hello(){
        System.out.println("hello");
        counter.increment();
    }
}
```

#### <font style="color:rgb(79, 79, 79);">6.2 监控案例落地</font>
<font style="color:rgb(77, 77, 77);">基于 Prometheus + Grafana</font>

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1755981423525-5afbd646-1f8b-4226-a5ab-cf175af38780.png)

##### <font style="color:rgb(79, 79, 79);">6.2.1 安装 Prometheus + Grafana</font>
```java
#安装prometheus:时序数据库
docker run -p 9090:9090 -d \
-v pc:/etc/prometheus \
prom/prometheus

#安装grafana；默认账号密码 admin:admin
docker run -d --name=grafana -p 3000:3000 grafana/grafana
```

##### <font style="color:rgb(79, 79, 79);">6.2.2 导入依赖</font>
```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
<dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-registry-prometheus</artifactId>
    <version>1.10.6</version>
</dependency>
```

```java
management:
  endpoints:
    web:
      exposure: #暴露所有监控的端点
        include: '*'
```

<font style="color:rgb(77, 77, 77);">访问： </font>[<font style="color:rgb(78, 161, 219);">http://localhost:8001/actuator/prometheus</font>](http://localhost:8001/actuator/prometheus)<font style="color:rgb(77, 77, 77);"> 验证，返回 prometheus 格式的所有指标</font>

<font style="color:rgb(79, 79, 79);background-color:rgb(238, 240, 244);">部署Java应用</font>

```java
#安装上传工具
yum install lrzsz

#安装openjdk
# 下载openjdk
wget https://download.oracle.com/java/17/latest/jdk-17_linux-x64_bin.tar.gz

mkdir -p /opt/java
tar -xzf jdk-17_linux-x64_bin.tar.gz -C /opt/java/
sudo vi /etc/profile
#加入以下内容
export JAVA_HOME=/opt/java/jdk-17.0.7
export PATH=$PATH:$JAVA_HOME/bin

#环境变量生效
source /etc/profile


# 后台启动java应用
nohup java -jar boot3-14-actuator-0.0.1-SNAPSHOT.jar > output.log 2>&1 &
```

<font style="color:rgb(77, 77, 77);">确认可以访问到： </font>[<font style="color:rgb(78, 161, 219);">http://8.130.32.70:9999/actuator/prometheus</font>](http://8.130.32.70:9999/actuator/prometheus)<font style="color:rgb(77, 77, 77);"> </font>

##### <font style="color:rgb(79, 79, 79);">6.2.3 配置 Prometheus 拉取数据</font>
```java
## 修改 prometheus.yml 配置文件
scrape_configs:
  - job_name: 'spring-boot-actuator-exporter'
    metrics_path: '/actuator/prometheus' #指定抓取的路径
    static_configs:
      - targets: ['192.168.200.1:8001']
        labels:
          nodename: 'app-demo'
```

##### <font style="color:rgb(79, 79, 79);">6.2.4 配置 Grafana 监控面板</font>
+ <font style="color:rgb(51, 51, 51);">添加数据源（Prometheus）</font>
+ <font style="color:rgb(51, 51, 51);">添加面板。可去 dashboard 市场找一个自己喜欢的面板，也可以自己开发面板;</font>[<font style="color:rgb(78, 161, 219);">Dashboards | Grafana Labs</font>](https://grafana.com/grafana/dashboards/?plcmt=footer)

##### <font style="color:rgb(79, 79, 79);">6.2.5 效果</font>
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1755981423497-5d8ca24a-e023-407d-8bae-2fb5eca421c6.png)

---

```java
package com.atguigu.boot3.actuator;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;


/**
 * 整合Prometheus+Grafana 完成线上应用指标监控系统
 * 1、改造SpringBoot应用，产生Prometheus需要的格式数据
 *   - 导入 micrometer-registry-prometheus
 * 2、部署java应用。在同一个机器内，访问 http://172.25.170.71:9999/actuator/prometheus 就能得到指标数据
 *    在外部访问：http://8.130.32.70:9999/actuator/prometheus
 * 3、修改prometheus配置文件，让他拉取某个应用的指标数据
 * 4、去grafana添加一个prometheus数据源，配置好prometheus地址
 *
 */
@SpringBootApplication
public class Boot314ActuatorApplication {

    public static void main(String[] args) {
        SpringApplication.run(Boot314ActuatorApplication.class, args);
    }

}
```



### <font style="color:rgb(79, 79, 79);">7 AOT</font>
#### <font style="color:rgb(79, 79, 79);">7.1 AOT与JIT</font>
**<font style="color:rgb(96, 27, 222);">AOT</font>**<font style="color:rgb(77, 77, 77);">：Ahead-of-Time（提前</font>**<font style="color:rgb(77, 77, 77);">编译</font>**<font style="color:rgb(77, 77, 77);">）：</font>**<font style="color:rgb(77, 77, 77);">程序执行前</font>**<font style="color:rgb(77, 77, 77);">，全部被编译成</font>**<font style="color:rgb(77, 77, 77);">机器码</font>**

**<font style="color:rgb(96, 27, 222);">JIT</font>**<font style="color:rgb(77, 77, 77);">：Just in Time（即时</font>**<font style="color:rgb(77, 77, 77);">编译</font>**<font style="color:rgb(77, 77, 77);">）: 程序边</font>**<font style="color:rgb(77, 77, 77);">编译</font>**<font style="color:rgb(77, 77, 77);">，边运行；</font>

**<font style="color:rgb(77, 77, 77);">编译：</font>**

+ **<font style="color:rgb(51, 51, 51);">源代码（.c、.cpp、.go、.java。。。） ===编译=== 机器码</font>**

**<font style="color:rgb(77, 77, 77);">语言：</font>**

+ **<font style="color:rgb(96, 27, 222);">编译</font>****<font style="color:rgb(51, 51, 51);">型语言：编译器</font>**
+ **<font style="color:rgb(51, 51, 51);">解释型语言：解释器</font>**

##### <font style="color:rgb(79, 79, 79);">7.1.1 Complier 与 Interpreter</font>
<font style="color:rgb(77, 77, 77);">Java：</font>**<font style="color:rgb(77, 77, 77);">半编译半解释</font>**

[<font style="color:rgb(78, 161, 219);">https://anycodes.cn/editor</font>](https://anycodes.cn/editor)<font style="color:rgb(77, 77, 77);"> （在线编码）</font>

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1755981647857-f0e029e8-4052-4fcd-a9f7-9581ff2a5399.png)

| <font style="color:rgb(79, 79, 79);">对比项</font> | **<font style="color:rgb(79, 79, 79);">编译器</font>** | **<font style="color:rgb(79, 79, 79);">解释器</font>** |
| --- | --- | --- |
| **<font style="color:rgb(79, 79, 79);">机器执行速度</font>** | **<font style="color:rgb(79, 79, 79);">快</font>**<font style="color:rgb(79, 79, 79);">，因为源代码只需被转换一次</font> | **<font style="color:rgb(79, 79, 79);">慢</font>**<font style="color:rgb(79, 79, 79);">，因为每行代码都需要被解释执行</font> |
| **<font style="color:rgb(79, 79, 79);">开发效率</font>** | **<font style="color:rgb(79, 79, 79);">慢</font>**<font style="color:rgb(79, 79, 79);">，因为需要耗费大量时间编译</font> | **<font style="color:rgb(79, 79, 79);">快</font>**<font style="color:rgb(79, 79, 79);">，无需花费时间生成目标代码，更快的开发和测试</font> |
| **<font style="color:rgb(0, 0, 0);">调试</font>** | **<font style="color:rgb(0, 0, 0);">难以调试</font>**<font style="color:rgb(0, 0, 0);">编译器生成的目标代码</font> | **<font style="color:rgb(0, 0, 0);">容易调试</font>**<font style="color:rgb(0, 0, 0);">源代码，因为解释器一行一行地执行</font> |
| **<font style="color:rgb(79, 79, 79);">可移植性（跨平台）</font>** | <font style="color:rgb(79, 79, 79);">不同平台需要重新编译目标平台代码</font> | <font style="color:rgb(79, 79, 79);">同一份源码可以跨平台执行，因为每个平台会开发对应的解释器</font> |
| **<font style="color:rgb(0, 0, 0);">学习难度</font>** | <font style="color:rgb(0, 0, 0);">相对较高，需要了解源代码、编译器以及目标机器的知识</font> | <font style="color:rgb(0, 0, 0);">相对较低，无需了解机器的细节</font> |
| **<font style="color:rgb(79, 79, 79);">错误检查</font>** | <font style="color:rgb(79, 79, 79);">编译器可以在编译代码时检查错误</font> | <font style="color:rgb(79, 79, 79);">解释器只能在执行代码时检查错误</font> |
| **<font style="color:rgb(79, 79, 79);">运行时增强</font>** | <font style="color:rgb(79, 79, 79);">无</font> | <font style="color:rgb(79, 79, 79);">可以</font>**<font style="color:rgb(79, 79, 79);">动态增强</font>** |


##### <font style="color:rgb(79, 79, 79);">7.1.2 AOT 与 JIT 对比</font>
|  | <font style="color:rgb(79, 79, 79);">JIT</font> | <font style="color:rgb(79, 79, 79);">AOT</font> |
| --- | --- | --- |
| <font style="color:rgb(79, 79, 79);">优点</font> | <font style="color:rgb(79, 79, 79);">1.具备</font>**<font style="color:rgb(79, 79, 79);">实时调整</font>**<font style="color:rgb(79, 79, 79);">能力   </font><font style="color:rgb(79, 79, 79);">2.生成</font>**<font style="color:rgb(79, 79, 79);">最优机器指令</font>**<font style="color:rgb(79, 79, 79);">   </font><font style="color:rgb(79, 79, 79);">3.根据代码运行情况</font>**<font style="color:rgb(79, 79, 79);">优化内存占用</font>** | <font style="color:rgb(79, 79, 79);">1.速度快，优化了运行时编译时间和内存消耗   </font><font style="color:rgb(79, 79, 79);">2.程序初期就能达最高性能   </font><font style="color:rgb(79, 79, 79);">3.加快程序启动速度</font> |
| <font style="color:rgb(79, 79, 79);">缺点</font> | <font style="color:rgb(79, 79, 79);">1.运行期边编译</font>**<font style="color:rgb(79, 79, 79);">速度慢</font>**<font style="color:rgb(79, 79, 79);">   </font><font style="color:rgb(79, 79, 79);">2.初始编译不能达到</font>**<font style="color:rgb(79, 79, 79);">最高性能</font>** | <font style="color:rgb(79, 79, 79);">1.程序第一次编译占用时间长   </font><font style="color:rgb(79, 79, 79);">2.牺牲</font>**<font style="color:rgb(79, 79, 79);">高级语言</font>**<font style="color:rgb(79, 79, 79);">一些特性</font> |


<font style="color:rgb(77, 77, 77);">在 OpenJDK 的官方 Wiki 上，介绍了HotSpot 虚拟机一个相对比较全面的、</font>**<font style="color:rgb(77, 77, 77);">即时编译器（JIT）</font>**<font style="color:rgb(77, 77, 77);">中采用的</font>[<font style="color:rgb(78, 161, 219);">优化技术列表</font>](https://xie.infoq.cn/link?target=https%3A%2F%2Fwiki.openjdk.java.net%2Fdisplay%2FHotSpot%2FPerformanceTacticIndex)<font style="color:rgb(77, 77, 77);">。 </font>

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1755981647856-1355b523-6a99-477a-ad1f-9333f54c3770.png)

<font style="color:rgb(77, 77, 77);">可使用：-XX:+PrintCompilation 打印JIT编译信息 </font>

##### <font style="color:rgb(79, 79, 79);">7.1.3 JVM架构</font>
<font style="color:rgb(77, 77, 77);">.java === .class === 机器码</font>

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1755981647844-8077a4a9-aee9-4baa-b829-7173e6bc6974.png)

**<font style="color:rgb(77, 77, 77);">JVM</font>**<font style="color:rgb(77, 77, 77);">: 既有</font>**<font style="color:rgb(77, 77, 77);">解释器</font>**<font style="color:rgb(77, 77, 77);">，又有</font>**<font style="color:rgb(77, 77, 77);">编辑器（JIT：即时编译）</font>**<font style="color:rgb(77, 77, 77);">；</font>

##### <font style="color:rgb(79, 79, 79);">7.1.4 Java的执行过程</font>
<font style="color:rgb(79, 79, 79);background-color:rgb(238, 240, 244);">建议阅读：</font>

+ <font style="color:rgb(51, 51, 51);background-color:rgb(238, 240, 244);">美团技术：</font>[<font style="color:rgb(78, 161, 219);background-color:rgb(238, 240, 244);">基本功 | Java即时编译器原理解析及实践 - 美团技术团队</font>](https://tech.meituan.com/2020/10/22/java-jit-practice-in-meituan.html)
+ <font style="color:rgb(51, 51, 51);background-color:rgb(238, 240, 244);">openjdk官网：</font>[<font style="color:rgb(78, 161, 219);background-color:rgb(238, 240, 244);">Compiler - Compiler - OpenJDK Wiki</font>](https://wiki.openjdk.org/display/HotSpot/Compiler)

###### <font style="color:rgb(79, 79, 79);">7.1.4.1 流程概要</font>
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1755981647868-5115c7cd-d8cd-422f-bbca-152ad8178425.png)

<font style="color:rgb(77, 77, 77);">解释执行：</font>

<font style="color:rgb(77, 77, 77);">编译执行：</font>

###### <font style="color:rgb(79, 79, 79);">7.1.4.2 详细流程</font>
<font style="color:rgb(77, 77, 77);">热点代码：调用次数非常多的代码</font>

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1755981647911-14ff5864-53db-404e-a0b8-345879e3ad51.png)

##### <font style="color:rgb(79, 79, 79);">7.1.5 JVM编译器</font>
<font style="color:rgb(77, 77, 77);">JVM中集成了两种编译器，Client Compiler 和 Server Compiler；</font>

+ <font style="color:rgb(51, 51, 51);">Client Compiler注重启动速度和局部的优化</font>
+ <font style="color:rgb(51, 51, 51);">Server Compiler更加关注全局优化，性能更好，但由于会进行更多的全局分析，所以启动速度会慢。</font>

<font style="color:rgb(77, 77, 77);">Client Compiler：</font>

+ <font style="color:rgb(51, 51, 51);">HotSpot VM带有一个Client Compiler</font><font style="color:rgb(51, 51, 51);"> </font>**<font style="color:rgb(51, 51, 51);">C1编译器</font>**
+ <font style="color:rgb(51, 51, 51);">这种编译器</font>**<font style="color:rgb(51, 51, 51);">启动速度快</font>**<font style="color:rgb(51, 51, 51);">，但是性能比较Server Compiler来说会差一些。</font>
+ <font style="color:rgb(51, 51, 51);">编译后的机器码执行效率没有C2的高</font>

<font style="color:rgb(77, 77, 77);">Server Compiler：</font>

+ <font style="color:rgb(51, 51, 51);">Hotspot虚拟机中使用的Server Compiler有两种：</font>**<font style="color:rgb(51, 51, 51);">C2</font>**<font style="color:rgb(51, 51, 51);"> </font><font style="color:rgb(51, 51, 51);">和</font><font style="color:rgb(51, 51, 51);"> </font>**<font style="color:rgb(51, 51, 51);">Graal</font>**<font style="color:rgb(51, 51, 51);">。</font>
+ <font style="color:rgb(51, 51, 51);">在Hotspot VM中，默认的Server Compiler是</font>**<font style="color:rgb(51, 51, 51);">C2编译器。</font>**

##### <font style="color:rgb(79, 79, 79);">7.1.6 分层编译</font>
<font style="color:rgb(77, 77, 77);">Java 7开始引入了分层编译(</font>**<font style="color:rgb(77, 77, 77);">Tiered Compiler</font>**<font style="color:rgb(77, 77, 77);">)的概念，它结合了</font>**<font style="color:rgb(77, 77, 77);">C1</font>**<font style="color:rgb(77, 77, 77);">和</font>**<font style="color:rgb(77, 77, 77);">C2</font>**<font style="color:rgb(77, 77, 77);">的优势，追求启动速度和峰值性能的一个平衡。分层编译将JVM的执行状态分为了五个层次。</font>**<font style="color:rgb(77, 77, 77);">五个层级</font>**<font style="color:rgb(77, 77, 77);">分别是：</font>

+ <font style="color:rgb(51, 51, 51);">解释执行。</font>
+ <font style="color:rgb(51, 51, 51);">执行不带profiling的C1代码。</font>
+ <font style="color:rgb(51, 51, 51);">执行仅带方法调用次数以及循环回边执行次数profiling的C1代码。</font>
+ <font style="color:rgb(51, 51, 51);">执行带所有profiling的C1代码。</font>
+ <font style="color:rgb(51, 51, 51);">执行C2代码。</font>

**<font style="color:rgb(77, 77, 77);">profiling就是收集能够反映程序执行状态的数据</font>**<font style="color:rgb(77, 77, 77);">。其中最基本的统计数据就是方法的调用次数，以及循环回边的执行次数。</font>

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1755981648296-a06ebb1d-c981-4c0e-baa9-ca3f1aa65787.png)

+ <font style="color:rgb(77, 77, 77);">图中第①条路径，代表编译的一般情况，</font>**<font style="color:rgb(77, 77, 77);">热点方法</font>**<font style="color:rgb(77, 77, 77);">从解释执行到被3层的C1编译，最后被4层的C2编译。</font>
+ <font style="color:rgb(77, 77, 77);">如果</font>**<font style="color:rgb(77, 77, 77);">方法比较小</font>**<font style="color:rgb(77, 77, 77);">（比如Java服务中常见的</font>**<font style="color:rgb(77, 77, 77);">getter/setter</font>**<font style="color:rgb(77, 77, 77);">方法），3层的profiling没有收集到有价值的数据，JVM就会断定该方法对于C1代码和C2代码的执行效率相同，就会执行图中第②条路径。在这种情况下，JVM会在3层编译之后，放弃进入C2编译，</font>**<font style="color:rgb(77, 77, 77);">直接选择用1层的C1编译运行</font>**<font style="color:rgb(77, 77, 77);">。</font>
+ <font style="color:rgb(77, 77, 77);">在</font>**<font style="color:rgb(77, 77, 77);">C1忙碌</font>**<font style="color:rgb(77, 77, 77);">的情况下，执行图中第③条路径，在解释执行过程中对程序进行</font>**<font style="color:rgb(77, 77, 77);">profiling</font>**<font style="color:rgb(77, 77, 77);"> ，根据信息直接由第4层的</font>**<font style="color:rgb(77, 77, 77);">C2编译</font>**<font style="color:rgb(77, 77, 77);">。</font>
+ <font style="color:rgb(77, 77, 77);">前文提到C1中的执行效率是</font>**<font style="color:rgb(77, 77, 77);">1层>2层>3层</font>**<font style="color:rgb(77, 77, 77);">，</font>**<font style="color:rgb(77, 77, 77);">第3层</font>**<font style="color:rgb(77, 77, 77);">一般要比</font>**<font style="color:rgb(77, 77, 77);">第2层</font>**<font style="color:rgb(77, 77, 77);">慢35%以上，所以在</font>**<font style="color:rgb(77, 77, 77);">C2忙碌</font>**<font style="color:rgb(77, 77, 77);">的情况下，执行图中第④条路径。这时方法会被2层的C1编译，然后再被3层的C1编译，以减少方法在</font>**<font style="color:rgb(77, 77, 77);">3层</font>**<font style="color:rgb(77, 77, 77);">的执行时间。</font>
+ <font style="color:rgb(77, 77, 77);">如果</font>**<font style="color:rgb(77, 77, 77);">编译器</font>**<font style="color:rgb(77, 77, 77);">做了一些比较</font>**<font style="color:rgb(77, 77, 77);">激进的优化</font>**<font style="color:rgb(77, 77, 77);">，比如分支预测，在实际运行时</font>**<font style="color:rgb(77, 77, 77);">发现预测出错</font>**<font style="color:rgb(77, 77, 77);">，这时就会进行</font>**<font style="color:rgb(77, 77, 77);">反优化</font>**<font style="color:rgb(77, 77, 77);">，重新进入</font>**<font style="color:rgb(77, 77, 77);">解释执行</font>**<font style="color:rgb(77, 77, 77);">，图中第⑤条执行路径代表的就是</font>**<font style="color:rgb(77, 77, 77);">反优化</font>**<font style="color:rgb(77, 77, 77);">。</font>

<font style="color:rgb(77, 77, 77);">总的来说，C1的编译速度更快，C2的编译质量更高，分层编译的不同编译路径，也就是JVM根据当前服务的运行情况来寻找当前服务的最佳平衡点的一个过程。从JDK 8开始，JVM默认开启分层编译。</font>

**<font style="color:rgb(77, 77, 77);">云原生</font>**<font style="color:rgb(77, 77, 77);">：Cloud Native； Java小改版；</font>

<font style="color:rgb(77, 77, 77);">最好的效果：</font>

<font style="color:rgb(77, 77, 77);">存在的问题：</font>

+ <font style="color:rgb(51, 51, 51);">java应用如果用jar，解释执行，热点代码才编译成机器码；初始启动速度慢，初始处理请求数量少。</font>
+ <font style="color:rgb(51, 51, 51);">大型云平台，要求每一种应用都必须秒级启动。每个应用都要求效率高。</font>

<font style="color:rgb(77, 77, 77);">希望的效果：</font>

+ <font style="color:rgb(51, 51, 51);">java应用也能提前被编译成</font>**<font style="color:rgb(51, 51, 51);">机器码</font>**<font style="color:rgb(51, 51, 51);">，随时</font>**<font style="color:rgb(51, 51, 51);">急速启动</font>**<font style="color:rgb(51, 51, 51);">，一启动就急速运行，最高性能</font>

<font style="color:rgb(77, 77, 77);">编译成机器码的好处：</font>

<font style="color:rgb(77, 77, 77);">Java应用如果打成一个jar包，部署到另外的服务器还需要安装Java环境；如果编译成</font>**<font style="color:rgb(77, 77, 77);">机器码</font>**<font style="color:rgb(77, 77, 77);">的，则可以在这个平台 Windows X64 </font>**<font style="color:rgb(77, 77, 77);">直接运行</font>**<font style="color:rgb(77, 77, 77);">。（0 1这种机器码，不需要什么环境，电脑通电就行）</font>

**<font style="color:rgb(77, 77, 77);">原生</font>**<font style="color:rgb(77, 77, 77);">镜像：</font>**<font style="color:rgb(77, 77, 77);">native</font>**<font style="color:rgb(77, 77, 77);">-image（机器码、本地镜像、直接的可执行程序）</font>

+ <font style="color:rgb(51, 51, 51);">把应用打包成能适配本机平台 的可执行文件（机器码、本地镜像）</font>

#### <font style="color:rgb(79, 79, 79);">7.2 GraalVM</font>
[<font style="color:rgb(78, 161, 219);">GraalVM</font>](https://www.graalvm.org/)

**<font style="color:rgb(79, 79, 79);">GraalVM</font>**<font style="color:rgb(79, 79, 79);">是一个高性能的</font>**<font style="color:rgb(79, 79, 79);">JDK</font>**<font style="color:rgb(79, 79, 79);">，旨在</font>**<font style="color:rgb(79, 79, 79);">加速</font>**<font style="color:rgb(79, 79, 79);">用Java和其他JVM语言编写的</font>**<font style="color:rgb(79, 79, 79);">应用程序</font>**<font style="color:rgb(79, 79, 79);">的</font>**<font style="color:rgb(79, 79, 79);">执行</font>**<font style="color:rgb(79, 79, 79);">，同时还提供JavaScript、Python和许多其他流行语言的运行时。</font>

**<font style="color:rgb(79, 79, 79);">GraalVM</font>**<font style="color:rgb(79, 79, 79);">提供了</font>**<font style="color:rgb(79, 79, 79);">两种</font>**<font style="color:rgb(79, 79, 79);">运行</font>**<font style="color:rgb(79, 79, 79);">Java应用程序</font>**<font style="color:rgb(79, 79, 79);">的方式：</font>

+ <font style="color:rgb(51, 51, 51);">1. 在HotSpot JVM上使用</font>**<font style="color:rgb(51, 51, 51);">Graal即时（JIT）编译器</font>**
+ <font style="color:rgb(51, 51, 51);">2. 作为</font>**<font style="color:rgb(51, 51, 51);">预先编译（AOT）</font>**<font style="color:rgb(51, 51, 51);">的本机</font>**<font style="color:rgb(51, 51, 51);">可执行文件</font>**<font style="color:rgb(51, 51, 51);">运行（</font>**<font style="color:rgb(51, 51, 51);">本地镜像</font>**<font style="color:rgb(51, 51, 51);">）。</font>

<font style="color:rgb(79, 79, 79);">GraalVM的多语言能力使得在单个应用程序中混合多种编程语言成为可能，同时消除了外部语言调用的成本。</font>

##### <font style="color:rgb(79, 79, 79);">7.2.1 架构</font>
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1755981648310-41aab20a-de52-4496-b60f-2656dd8084fa.png)

##### <font style="color:rgb(79, 79, 79);">7.2.2 安装</font>
<font style="color:rgb(77, 77, 77);">跨平台提供原生镜像原理：</font>

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1755981648457-cc67d6fb-8fcd-4989-821d-9c555c852ee3.png)

###### <font style="color:rgb(79, 79, 79);">7.2.2.1 VisualStudio</font>
[<font style="color:rgb(78, 161, 219);">免费的开发人员软件和服务 - Visual Studio</font>](https://visualstudio.microsoft.com/zh-hans/free-developer-offers/)

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1755981648558-7cc87b40-bdae-4c2e-8535-845e522617c1.png)

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1755981648565-fc75e2dc-970f-4df0-bc06-c20c0907cf5c.png)

<font style="color:rgb(77, 77, 77);">别选中文</font>

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1755981649095-dae897ba-34bd-4034-99a8-b99028413d76.png)

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1755981649119-93e536b0-3349-45b7-a942-2b16e78ccebd.png)

<font style="color:rgb(77, 77, 77);">记住你安装的地址；</font>

###### <font style="color:rgb(79, 79, 79);">7.2.2.2 GraalVM</font>
**<font style="color:rgb(26, 67, 156);">1 安装</font>**

<font style="color:rgb(77, 77, 77);">下载 GraalVM + native-image</font>

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1755981649249-0bbf556c-eded-4fd8-b676-aa463c399486.png)

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1755981649694-2e2a2d7f-4df0-4a72-b326-780b97d40df5.png)

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1755981649686-5ef378a4-1701-44bd-84ab-b4f40c47374a.png)

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1755981649984-f7b2dc51-0b74-4c11-95b1-af4cf996d133.png)

**<font style="color:rgb(26, 67, 156);">2 配置</font>**

<font style="color:rgb(77, 77, 77);">修改 JAVA_HOME 与 Path，指向新bin路径</font>

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1755981649983-0a9ef11b-89f7-4c75-820e-1d9def089733.png)

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1755981650409-434b43a6-72f3-4f28-b2d3-1acd13fb72f6.png)

<font style="color:rgb(77, 77, 77);">验证JDK环境为GraalVM提供的即可：</font>

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1755981650697-d329646b-0ebb-4ff1-82f3-c16b9297471d.png)

**<font style="color:rgb(26, 67, 156);">3 依赖</font>**

<font style="color:rgb(77, 77, 77);">安装 native-image 依赖：</font>

<font style="color:rgb(77, 77, 77);">网络环境好：参考：</font>[<font style="color:rgb(78, 161, 219);">Native Image</font>](https://www.graalvm.org/latest/reference-manual/native-image/#install-native-image)

```java
gu install native-image
```

<font style="color:rgb(77, 77, 77);">网络不好，使用我们下载的离线jar; native-image-xxx.jar 文件</font>

```java
gu install --file native-image-installable-svm-java17-windows-amd64-22.3.2.jar
```

**<font style="color:rgb(26, 67, 156);">4 验证</font>**

```java
native-image
```

##### <font style="color:rgb(79, 79, 79);">7.2.3 测试</font>
###### <font style="color:rgb(79, 79, 79);">7.2.3.1 创建项目</font>
<font style="color:rgb(77, 77, 77);">创建普通java项目。编写HelloWorld类；</font>

+ <font style="color:rgb(51, 51, 51);">使用mvn clean package进行打包</font>
+ <font style="color:rgb(51, 51, 51);">确认jar包是否可以执行java -jar xxx.jar</font>
+ <font style="color:rgb(51, 51, 51);">可能需要给 MANIFEST.MF添加 Main-Class: 你的主类</font>

###### <font style="color:rgb(79, 79, 79);">7.2.3.2 编译镜像</font>
<font style="color:rgb(77, 77, 77);">编译为原生镜像（native-image）：使用native-tools终端</font>

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1755981650705-ee548244-8d95-43a2-ab13-dace50e56c4e.png)

```java
#第一种：从入口开始，编译整个jar
native-image -cp boot3-15-aot-common-1.0-SNAPSHOT.jar com.atguigu.MainApplication -o Haha

#第二种：编译.class字节码文件，编译某个类【必须有main入口方法，否则无法编译】
native-image -cp .\classes com.atguigu.MainApplication -o Haha
```

###### <font style="color:rgb(79, 79, 79);">7.2.3.3 Linux平台测试</font>
<font style="color:rgb(77, 77, 77);">1 安装gcc等环境</font>

```java
yum install lrzsz
sudo yum install gcc glibc-devel zlib-devel
```

<font style="color:rgb(77, 77, 77);">2 下载安装配置Linux下的GraalVM、native-image</font>

+ <font style="color:rgb(51, 51, 51);">下载：</font>[<font style="color:rgb(78, 161, 219);">https://www.graalvm.org/downloads/</font>](https://www.graalvm.org/downloads/)
+ <font style="color:rgb(51, 51, 51);">安装：GraalVM、native-image</font>
+ <font style="color:rgb(51, 51, 51);">配置：JAVA环境变量为GraalVM</font>

```java
tar -zxvf graalvm-ce-java17-linux-amd64-22.3.2.tar.gz -C /opt/java/

sudo vim /etc/profile
#修改以下内容
export JAVA_HOME=/opt/java/graalvm-ce-java17-22.3.2
export PATH=$PATH:$JAVA_HOME/bin

source /etc/profile
```

<font style="color:rgb(77, 77, 77);">3 安装native-image</font>

```java
gu install --file native-image-installable-svm-java17-linux-amd64-22.3.2.jar
```

<font style="color:rgb(77, 77, 77);">4 使用native-image编译jar为原生程序</font>

```java
#第一种：从入口开始，编译整个jar
native-image -cp boot3-15-aot-common-1.0-SNAPSHOT.jar com.atguigu.MainApplication -o Demo
```

---

```java
package com.atguigu;


/**
 * 打包成本地镜像：
 *
 * 1、打成jar包:  注意修改 jar包内的 MANIFEST.MF 文件，指定Main-Class的全类名
 *        - java -jar xxx.jar 就可以执行。
 *        - 切换机器，安装java环境。默认解释执行，启动速度慢，运行速度慢
 * 2、打成本地镜像（可执行文件）：
 *        - native-image -cp  你的jar包/路径  你的主类  -o 输出的文件名
 *        - native-image -cp boot3-15-aot-common-1.0-SNAPSHOT.jar com.atguigu.MainApplication -o Demo
 *
 * 并不是所有的Java代码都能支持本地打包；
 * SpringBoot保证Spring应用的所有程序都能在AOT的时候提前告知graalvm怎么处理？
 *
 *   - 动态能力损失：反射的代码：（动态获取构造器，反射创建对象，反射调用一些方法）；
 *     解决方案：额外处理（SpringBoot 提供了一些注解）：提前告知 graalvm 反射会用到哪些方法、构造器
 *   - 配置文件损失：
 *     解决方案：额外处理（配置中心）：提前告知 graalvm 配置文件怎么处理
 *   - 【好消息：新版GraalVM可以自动进行预处理，不用我们手动进行补偿性的额外处理。】
 *   二进制里面不能包含的，不能动态的都得提前处理；
 *
 *   不是所有框架都适配了 AOT特性；Spring全系列栈适配OK
 *
 *  application.properties
 *  a(){
 *      //ssjsj  bcde();
 *      //提前处理
 *  }
 */
public class MainApplication {
    public static void main(String[] args) {
        System.out.println("Hello world!");
    }
}
```

#### <font style="color:rgb(79, 79, 79);">7.3 SpringBoot整合</font>
##### <font style="color:rgb(79, 79, 79);">7.3.1 依赖导入</font>
```java
<build>
        <plugins>
            <plugin>
                <groupId>org.graalvm.buildtools</groupId>
                <artifactId>native-maven-plugin</artifactId>
            </plugin>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
```

##### <font style="color:rgb(79, 79, 79);">7.3.2 生成native-image</font>
<font style="color:rgb(77, 77, 77);">1、运行aot提前处理命令：mvn springboot:process-aot</font>

<font style="color:rgb(77, 77, 77);">2、运行native打包：mvn -Pnative native:build</font>

```java
# 推荐加上 -Pnative
mvn -Pnative native:build -f pom.xml
```

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1755981651281-ed361767-7b65-4db6-abb4-e3a9cec960e8.png)

##### <font style="color:rgb(79, 79, 79);">7.3.3 常见问题</font>
<font style="color:rgb(77, 77, 77);">可能提示如下各种错误，无法构建原生镜像，需要配置环境变量；</font>

+ <font style="color:rgb(51, 51, 51);">出现cl.exe找不到错误</font>
+ <font style="color:rgb(51, 51, 51);">出现乱码</font>
+ <font style="color:rgb(51, 51, 51);">提示no include path set</font>
+ <font style="color:rgb(51, 51, 51);">提示fatal error LNK1104: cannot open file 'LIBCMT.lib'</font>
+ <font style="color:rgb(51, 51, 51);">提示 LINK : fatal error LNK1104: cannot open file 'kernel32.lib'</font>
+ <font style="color:rgb(51, 51, 51);">提示各种其他找不到</font>

**<font style="color:rgb(223, 42, 63);">需要修改三个环境变量</font>**<font style="color:rgb(77, 77, 77);">：Path、INCLUDE、lib</font>

<font style="color:rgb(77, 77, 77);">1、 Path：添加如下值</font>

+ <font style="color:rgb(51, 51, 51);">C:\Program Files\Microsoft Visual Studio\2022\Community\VC\Tools\MSVC\14.33.31629\bin\Hostx64\x64</font>

<font style="color:rgb(77, 77, 77);">2、新建INCLUDE环境变量：值为</font>

```java
C:\Program Files\Microsoft Visual Studio\2022\Community\VC\Tools\MSVC\14.33.31629\include;C:\Program Files (x86)\Windows Kits\10\Include\10.0.19041.0\shared;C:\Program Files (x86)\Windows Kits\10\Include\10.0.19041.0\ucrt;C:\Program Files (x86)\Windows Kits\10\Include\10.0.19041.0\um;C:\Program Files (x86)\Windows Kits\10\Include\10.0.19041.0\winrt
```

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1755981651393-a673ecb1-37ba-4da6-9d5b-09a366734d0d.png)

<font style="color:rgb(77, 77, 77);">3、新建lib环境变量：值为</font>

```plain
C:\Program Files\Microsoft Visual Studio\2022\Community\VC\Tools\MSVC\14.33.31629\lib\x64;C:\Program Files (x86)\Windows Kits\10\Lib\10.0.19041.0\um\x64;C:\Program Files (x86)\Windows Kits\10\Lib\10.0.19041.0\ucrt\x64
```

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1755981651603-e67e3440-a27d-48a9-b9aa-154193a0a30e.png)

