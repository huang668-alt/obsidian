
---

### 1. `@Configuration`

**作用**：  
声明一个类是 **Spring 的 Java 配置类**，用于通过 Java 代码的方式注册 Bean。

**详细特点**：
- 属于 **Full 模式** 配置类，Spring 会使用 **CGLIB** 动态代理增强该类。
- 支持在配置类中定义多个 `@Bean` 方法，并且方法之间可以互相调用（保证单例）。
- 允许使用 `@Bean`、`@Import`、`@ComponentScan` 等注解。

**示例**：
```java
@Configuration
public class AppConfig {

    @Bean
    public UserService userService() {
        return new UserService(userDao()); // 方法间调用，返回同一个实例
    }

    @Bean
    public UserDao userDao() {
        return new UserDaoImpl();
    }
}
```

**底层原理**：  
Spring 在解析配置类时，如果发现是 `@Configuration`，会为其生成 CGLIB 子类，拦截 `@Bean` 方法的调用，确保单例行为。

**注意事项**：
- 不要在 `@Configuration` 类上加 `@ComponentScan` 扫描自身（可能导致重复注册）。
- 不能是 final 类（因为需要被代理）。

**适用场景**：大型项目中集中管理 Bean 配置、集成第三方组件。

---

### 2. `@Component`

**作用**：  
最基础的组件注解，告诉 Spring “这个类需要被扫描并注册为 Bean”。

**特点**：
- 属于 **Lite 模式**，不会进行 CGLIB 代理。
- Spring 通过 `ClassPathBeanDefinitionScanner` 扫描带有此注解的类。

**衍生注解**（语义化版本）：
- `@Service`：业务层
- `@Repository`：数据访问层（额外提供异常翻译）
- `@Controller`：MVC 控制器层
- `@RestController` = `@Controller` + `@ResponseBody`

**示例**：
```java
@Component
public class UserDaoImpl implements UserDao { }
```

**注意事项**：
- 必须配合 `@ComponentScan` 使用才能被扫描到。
- 方法间调用不会走 Spring 容器（每次调用都新对象）。

**适用场景**：自己编写的普通业务类、工具类。

---

### 3. `@Bean`

**作用**：  
在配置类的方法上使用，**手动注册一个 Bean** 到 Spring 容器中。

**详细特点**：
- 方法返回值即为 Bean 实例。
- 可以指定 `initMethod`、`destroyMethod`。
- 支持参数注入（方法参数会自动从容器中获取）。

**示例**：
```java
@Bean(name = {"dataSource", "ds"})
@Scope("singleton")
public DataSource dataSource(DataSourceProperties props) {
    // 复杂初始化逻辑
    return DataSourceBuilder.create().build();
}
```

**与 `@Component` 的区别**：
- `@Component` 用于**自己写的类**（自动扫描）
- `@Bean` 用于**第三方类**或**需要复杂初始化**的场景

**适用场景**：注册 RedisTemplate、DataSource、RestTemplate 等第三方 Bean。

---

### 4. `@Autowired`

**作用**：  
自动依赖注入，按 **类型（Type）** 注入。

**注入位置**：
- 构造器（**最推荐**）
- Setter 方法
- 字段（**不推荐**生产环境使用）

**注入规则**：
1. 按类型查找 Bean
2. 如果找到多个，按**参数名/字段名**匹配
3. 仍不唯一 → 抛 `NoUniqueBeanDefinitionException`

**示例**：
```java
@Service
public class UserService {
    private final UserDao userDao;

    @Autowired   // Spring 4.3 后单个构造器可省略
    public UserService(UserDao userDao) {
        this.userDao = userDao;
    }
}
```

**解决歧义**：配合 `@Qualifier("beanName")` 或 `@Primary` 使用。

---

### 5. `@Qualifier`

**作用**：  
与 `@Autowired` 配合使用，**按名称**指定注入哪个 Bean。

**示例**：
```java
@Autowired
@Qualifier("userDaoImplV2")
private UserDao userDao;
```

---

### 6. `@Primary`

**作用**：  
当存在多个同类型 Bean 时，**优先注入**被标记 `@Primary` 的 Bean。

**示例**：
```java
@Primary
@Repository
public class UserDaoImplV1 implements UserDao { }
```

---

### 7. `@PostConstruct`

**作用**：  
Bean **初始化完成后** 自动执行的方法（JSR-250 规范）。

**执行时机**：  
依赖注入完成 → `@PostConstruct` → `InitializingBean.afterPropertiesSet()` → init-method

**示例**：
```java
@PostConstruct
public void init() {
    // 加载缓存、初始化连接池等
    cache.loadAll();
}
```

---

### 8. `@PreDestroy`

**作用**：  
Bean **销毁之前** 自动执行的方法，用于释放资源。

**执行时机**：容器关闭时，`@PreDestroy` → `DisposableBean.destroy()` → destroy-method

**示例**：
```java
@PreDestroy
public void destroy() {
    redisTemplate.shutdown();
}
```

---

### 9. `@Import`

**作用**：  
主动导入其他配置类、普通类、ImportSelector 或 ImportBeanDefinitionRegistrar。

**四种主要用法**：

1. **导入配置类**：
   ```java
   @Import(RedisConfig.class)
   ```

2. **导入普通类**（即使无注解也会注册为 Bean）。

3. **ImportSelector**：动态选择要导入的类。

4. **ImportBeanDefinitionRegistrar**：编程式注册 BeanDefinition（最强大）。

---

### 10. `@Scope`

**作用**：  
设置 Bean 的作用域（singleton、prototype、request、session 等）。

**示例**：
```java
@Scope("prototype")
@Component
public class OrderService { }
```

---

### 11. `@Value`

**作用**：  
注入配置文件中的值（properties / yml）。

**示例**：
```java
@Value("${app.name:默认名称}")
private String appName;

@Value("${app.list[0]}")
private String firstItem;
```

支持 SpEL 表达式。

---

### 12. `@ConfigurationProperties`

**作用**：  
将配置文件中**前缀相同**的配置绑定到实体类上（比 `@Value` 更强大）。

**示例**：
```java
@ConfigurationProperties(prefix = "spring.datasource")
@Component
public class DataSourceProperties {
    private String url;
    private String username;
    // getter/setter
}
```

---

### 13. `@Conditional` 系列（Spring Boot 核心）

- `@ConditionalOnProperty`：当配置文件有某属性时生效
- `@ConditionalOnBean`：当容器中有某 Bean 时生效
- `@ConditionalOnMissingBean`：当容器中**没有**某 Bean 时生效
- `@ConditionalOnClass`：当 classpath 中有某类时生效

---

### 14. 其他重要注解快速补充

- **`@Lazy`**：懒加载 Bean
- **`@DependsOn`**：指定 Bean 初始化顺序
- **`@Order`**：排序
- **`@EnableAsync`**：开启异步支持
- **`@EnableTransactionManagement`**：开启事务注解
- **`@EnableScheduling`**：开启定时任务

---

**总结**：  
- 自己业务类 → 用 `@Component` 系列 + 构造器注入  
- 第三方 Bean → 用 `@Bean`  
- 配置类 → 用 `@Configuration`  
- 复杂导入 → 用 `@Import`

---