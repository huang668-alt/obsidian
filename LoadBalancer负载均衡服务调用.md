在微服务架构中，**LoadBalancer（负载均衡）服务调用**是整个集群生态的“交通枢纽”。

前面我们聊 OpenFeign 的时候反复提到过它，今天我们就把它的面纱彻底揭开，深入最底层，看看 **Spring Cloud LoadBalancer (SCLB)** 到底是如何在幕后默默操控流量、分发请求的。

> 💡 **时代背景交代：** 过去 Spring Cloud 默认使用 Netflix 公司的 **Ribbon** 组件做负载均衡。但 Ribbon 早已停止维护并被官方全面移出生态。现在的绝对主角是 Spring 官方亲儿子 —— **Spring Cloud LoadBalancer**。

## 一、 核心概念：客户端负载均衡 vs 服务端负载均衡

要彻底搞懂 LoadBalancer 的调用流程，必须先理清它和 Nginx 的本质区别。

### 1. 服务端负载均衡（如 Nginx / 硬件 F5）

- **怎么玩：** 客户端只知道 Nginx 的唯一 IP。客户端把请求无脑发给 Nginx，Nginx 再在服务器内部通过算法把请求分发给背后的微服务。
    
- **特点：** 客户端**不知道**背后到底有多少台微服务机器，也不关心它们的死活。
    

### 2. 客户端负载均衡（Spring Cloud LoadBalancer）

- **怎么玩：** 负载均衡器直接住在你的**消费者微服务（如 consumer-service）内部**。
    
- **特点：** 消费者服务自己去注册中心（Eureka/Nacos）把所有的服务实例列表（比如 order-service 有 3 台机器的 IP）全部拉取到本地缓存。当要发起调用时，**客户端自己在本地通过算法挑出一台 IP**，直接对准目标机器开火。
    

## 二、 核心魔法：`@LoadBalanced` 到底对 RestTemplate 做了什么？

你在写基础代码时，只需要给 `RestTemplate` 加一个 `@LoadBalanced` 注解，它就突然具备了识别微服务别名（如 `http://provider-service/hello`）的能力。这背后其实是一个标准的**拦截器机制**。

整个服务调用的全套动作，可以用 **“五步走”** 来拆解：

### 第一步：拦截请求

当你执行 `restTemplate.getForObject("http://provider-service/hello")` 时，Spring 发现这个 RestTemplate 被打上了 `@LoadBalanced` 标记。于是，底层的 `LoadBalancerInterceptor`（负载均衡拦截器）会立刻出手，一把死死拦住这个请求。

### 第二步：提取服务名

拦截器把原本的虚拟 URL 揪出来，拆解出里面的服务别名：`provider-service`。

### 第三步：拉取并筛选实例（核心）

拦截器把服务名交给 `LoadBalancerClient`（负载均衡客户端）。它会去本地缓存的注册中心列表里查找 `provider-service`。

- 假设查出来有 3 台机器：`192.168.1.10:8081`、`192.168.1.11:8081`、`192.168.1.12:8081`。
    

### 第四步：算法选妃

负载均衡器根据你配置的策略（默认是 **RoundRobin 轮询**），从这 3 台健康的机器里挑出一台。假设这次轮到了 `192.168.1.11:8081`。

### 第五步：URL 偷天换日

拦截器把原本的虚拟地址 `http://provider-service/hello` 彻底擦除，**重写替换**为真实的物理地址 `http://192.168.1.11:8081/hello`。最后放行，交由底层的 HTTP 工具真正发起网络请求。

## 三、 Spring Cloud LoadBalancer 的两大核心内置策略

SCLB 官方目前默认内置了两种核心的负载均衡策略类：

1. **`RoundRobinLoadBalancer`（轮询策略 - 默认）：**
    
    - **原理：** 内部使用了一个原子计数器 `AtomicInteger`。每次来请求，计数器 $+1$，然后对服务实例总数**取模**。
        
    - **效果：** 极其公平。第 1 次调 A，第 2 次调 B，第 3 次调 C，第 4 次又回到 A。
        
2. **`RandomLoadBalancer`（随机策略）：**
    
    - **原理：** 内部利用 `ThreadLocalRandom.current().nextInt(instances.size())` 生成随机数。
        
    - **效果：** 听天由命，随机乱点。在高并发长期运行下，整体流量也是大致均匀的。
        

## 四、 实战：如何修改 LoadBalancer 的负载均衡策略？

如果你不想用默认的轮询，想改成**随机访问**，应该怎么做？SCLB 放弃了 Ribbon 过去臃肿的 YAML 配置，改用**全 Java 注解/配置类**的方式。

### 1. 编写自定义策略配置类

**注意：** 这个类千万不要加上 `@Configuration` 注解，否则它会被 Spring 全局扫描到，导致所有微服务都变成随机策略了。

``` java
import org.springframework.cloud.client.ServiceInstance;
import org.springframework.cloud.loadbalancer.core.RandomLoadBalancer;
import org.springframework.cloud.loadbalancer.core.ReactorLoadBalancer;
import org.springframework.cloud.loadbalancer.core.ServiceInstanceListSupplier;
import org.springframework.cloud.loadbalancer.support.LoadBalancerClientFactory;
import org.springframework.context.annotation.Bean;
import org.springframework.core.env.Environment;

public class MyCustomLoadBalancerConfig {

    // 注入随机负载均衡器
    @Bean
    public ReactorLoadBalancer<ServiceInstance> randomLoadBalancer(
            Environment environment,
            LoadBalancerClientFactory loadBalancerClientFactory) {
        
        String name = loadBalancerClientFactory.getName(environment);
        return new RandomLoadBalancer(
                loadBalancerClientFactory.getLazyProvider(name, ServiceInstanceListSupplier.class), 
                name
        );
    }
}
```

### 2. 在启动类上精准绑定目标服务

使用 `@LoadBalancerClient` 注解，指定只有在调用 `provider-service` 时，才启用上面那个随机配置类：

``` java
@SpringBootApplication
@EnableFeignClients
// 精准打击：只有调 provider-service 时才切换为随机算法
@LoadBalancerClient(name = "provider-service", configuration = MyCustomLoadBalancerConfig.class)
public class ConsumerApplication {
    public static void main(String[] args) {
        SpringApplication.run(ConsumerApplication.class, args);
    }
}
```

## 五、 高级进阶：LoadBalancer 的本地缓存缓存同步机制

客户端负载均衡有一个致命的痛点：**如果某一台服务提供者突然断电挂了，消费者怎么知道？**

如果消费者不知道，LoadBalancer 依然把流量往这台死机上分发，就会导致接口大面积报错。为了解决这个问题，SCLB 引入了**服务实例缓存（Cache）** 和**健康检查机制**。

1. **定时拉取（Pull）：** 消费者内部有一个后台定时任务，默认每隔固定的时间（如 30 秒）去注册中心重新拉取一次最新的健康实例列表，覆盖本地缓存。
    
2. **事件监听（Push）：** 如果使用的是 Nacos 等现代注册中心，当提供者挂掉时，Nacos 服务端会主动向消费者发送一个 UDP 的变动通知，消费者收到后会**秒级刷新**本地的 LoadBalancer 缓存，将死机剔除。
    

## 总结

`LoadBalancer` 本质上是一个“情报贩子”+“策略选择器”。它拿着从注册中心打听到的微服务 IP 列表，根据既定的数学算法（轮询/随机），在每次发起 HTTP 请求前，把虚幻的服务名翻译成实实在在的机器地址。

既然你已经掌握了 LoadBalancer 内部是如何挑选机器并替换 URL 的，你是否有兴趣了解一下：在线上生产环境中，如果某一台机器因为硬件问题响应极其缓慢（但没彻底挂掉），我们如何通过**自定义权重（Weight）负载均衡算法**，让 LoadBalancer 聪明地少给这台慢机器分发流量呢？