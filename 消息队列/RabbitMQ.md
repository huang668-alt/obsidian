## MQ的相关概念
### 什么是MQ
MQ(message queue),从字面意思上看，本质是个队列，FIFO先入先出，只不过队列中存放的内容是message而已，还是一种跨进程的通信机制，用于上下游传递消息。在互联网架构中，MQ是一种非常常见的上下游“逻辑解耦+物理解耦”的消息通信服务。使用了MQ之后，消息发送上游只需要依赖MQ,不用依赖其他服务。

### 为什么要用MQ
#### 流量消峰
举个例子，如果订单系统最多能处理一万次订单，这个处理能力应付正常时段的下单时绰绰有余，正常时段我们下单一秒后就能返回结果。但是在高峰期，如果有两万次下单操作系统是处理不了的，只能限制订单超过一万后不允许用户下单。使用消息队列做缓冲，我们可以取消这个限制，把一秒内下的订单分散成一段时间来处理，这时有些用户可能在下单十几秒后才能收到下单成功的操作，但是比不能下单的体验要好。

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151855755-ad7480b7-bfdc-4878-837a-be8bfc40624c.png)

#### 应用解耦
以电商应用为例，应用中有订单系统、库存系统、物流系统、支付系统。用户创建订单后，如果耦合调用库存系统、物流系统、支付系统，任何一个子系统出了故障，都会造成下单操作异常。当转变成基于消息队列的方式后，系统间调用的问题会减少很多，比如物流系统因为发生故障，需要几分钟来修复。在这几分钟的时间里，物流系统要处理的内存被缓存在消息队列中，用户的下单操作可以正常完成。当物流系统恢复后，继续处理订单信息即可，中单用户感受不到物流系统的故障，提升系统的可用性。

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151855513-d2642ec5-bb16-49fe-839b-ea662b0c178b.png)

#### 异步处理
有些服务间调用是异步的，例如A调用B,B需要花费很长时间执行，但是A需要知道B什么时候可以执行完，以前一般有两种方式，A过一段时间去调用B的查询api查询。或者A提供一个callback api,B执行完之后调用api通知A服务。这两种方式都不是很优雅，使用消息总线，可以很方便解决这个问题，A调用B服务后，只需要监听B处理完成的消息，当B处理完成后，会发送一条消息给MQ,MQ会将此消息转发给A服务。这样A服务既不用循环调用B的查询api,也不用提供callback api。同样B服务也不用做这些操作。A服务还能及时的得到异步处理成功的消息。

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151855490-2b2ccb9a-5153-4b59-b39c-9d3360eb5791.png)

### MQ的分类
#### 消息队列底层实现的两大主流方式
+ 由于消息队列执行的是跨应用的信息传递，所以制定**底层通信标准**非常必要
+ 目前主流的消息队列通信协议标准包括： 
    - AMQP(Advanced Message Queuing Protocol):**通用**协议，IBM公司研发
    - JMS(Java Message Service):**专门**为**Java**语言服务，SUN公司研发，一组由Java接口组成的Java标准

#### AMQP与JMS对比
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151855972-dbb7fd89-f650-4147-89ef-6646e15cb2ce.png)

#### 各主流MQ产品对比
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151855859-4fc97ad7-d7b1-438e-a0d6-b18cedc0b25f.png)

**1、ActiveMQ**

优点：单机吞吐量万级，时效性ms级，可用性高，基于主从架构实现高可用性，消息可靠性较低的概率丢失数据  
缺点：官方社区现在对ActiveMQ5.x维护越来越少，高吞吐量场景较少使用。

**2、Kafka**

大数据的杀手锏，谈到大数据领域内的消息传输，则绕不开Kafka,这款**为大数据而生**的消息中间件，以其**百万级TPS**的吞吐量名声大噪，迅速成为大数据领域的宠儿，在数据采集、传输、存诸的过程中发挥着举足轻重的作用。目前已经被LinkedIn，Uber，Twitter，Netflix等大公司所采纳。

优点：性能卓越，单机写入TPS约在百万条/秒，最大的优点，就是**吞吐量高**。时效性ms级可用性非常高，kafka是分布式的，一个数据多个副本，少数机器宕机，不会丢失数据，不会导致不可用，消费者采用Pull方式获取消息，消息有序，通过控制能够保证所有消息被消费且仅被消费一次;有优秀的第三方Kafka Web管理界面Kafka-Manager;在日志领域比较成熟，被多家公司和多个开源项目使用；功能支持：功能较为简单，主要支持简单的MQ功能，在大数据领域的实时计算以及**日志采集**被大规模使用

缺点：Kafka单机超过64个队列/分区，Load会发生明显的飙高现象，队列越多，load越高，发送消息响应时间变长，使用短轮询方式，实时性取决于轮询间隔时间，消费失败不支持重试；支持消息顺序，但是一台代理宕机后，就会产生消息乱序，**社区更新较慢**；

**3、RocketMQ**

RocketMQ出自阿里巴巴的开源产品，用java语言实现，在设计时参考了Kafka,并做出了自己的一些改进。被阿里巴巴广泛应用在订单，交易，充值，流计算，消息推送，日志流式处理，binglog分发等场景。

优点：单**机吞吐量十万级**，可用性非常高，分布式架构，**消息可以做到0丢失**，MQ功能较为完善，还是分布式的，扩展性好，**支持10亿级别的消息堆积**，不会因为堆积导致性能下降，源码是jva我们可以自己阅读源码，定制自己公司的MQ

缺点：**支持的客户端语言不多**，目前是java及c++,其中c++不成熟；社区活跃度一般，没有在MQ核心中去实现JMS等接口，有些系统要迁移需要修改大量代码

**4、RabbitMQ**

2007年发布，是一个在AMQP(高级消息队列协议)基础上完成的，可复用的企业消息系统，是**当前最主流的消息中间件之一**。

优点：由于erlang语言的**高并发特性**，性能较好；**吞吐量到万级**，MQ功能比较完备，健壮、稳定、易用、跨平台、**支持多种语言**如：Python、Ruby、.NET、Java、JS、C、PHP、ActionScript、XMPP、STOMP等，支持A]AX文档齐全；开源提供的管理界面非常棒，用起来很好用，**社区活跃度高**；更新频率相当高

缺点：商业版需要收费，学习成本较高

### MQ的选择
**1、Kafka**  
Kafka主要特点是基于Pul的模式来处理消息消费，追求高吞吐量，一开始的目的就是用于日志收集和传输，适合产生**大量数据**的互联网服务的数据收集业务。**大型公司**建议可以选用，如果有**日志采集**功能，肯定是首选kafka了。

**2、RocketMQ**  
天生为**金融互联网**领域而生，对于可靠性要求很高的场景，尤其是电商里面的订单扣款，以及业务削峰，在大量交易涌入时，后端可能无法及时处理的情况。RoketMQ在稳定性上可能更值得信赖，这些业务场景在阿里双11已经经历了多次考验，如果你的业务有上述并发场景，建议可以选择RocketMQ。

**3、RabbitMQ**  
结合erlang语言本身的并发优势，性能好**时效性微秒级**，**社区活跃度也比较高**，管理界面用起来十分  
方便，如果你的**数据量没有那么大**，中小型公司优先选择功能比较完备的RabbitMQ。

## RabbitMQ介绍
### RabbitMQ的概念
RabbitMQ是一个消息中间件：它接受并转发消息。你可以把它当做一个快递站点，当你要发送一个包裹时，你把你的包裹放到快递站，快递员最终会把你的快递送到收件人那里，按照这种逻辑RabbitMQ是一个快递站，一个快递员帮你传递快件。RabbitMQ与快递站的主要区别在于，它不处理快件而是接收，存储和转发消息数据。

### 四大核心概念
**生产者**  
产生数据发送消息的程序是生产者

**交换机**  
交换机是RabbitMQ非常重要的一个部件，一方面它接收来自生产者的消息，另一方面它将消息推送到队列中。交换机必须确切知道如何处理它接收到的消息，是将这些消息推送到特定队列还是推送到多个队列，亦或者是把消息丢弃，这个得有交换机类型决定。

**队列**

队列是RabbitMQ内部使用的一种数据结构，尽管消息流经RabbitMQ和应用程序，但它们只能存储在队列中。队列仅受主机的内存和滋盘限制的约束，本质上是一个大的消息缓冲区。许多生产者可以将消息发送到一个队列，许多消费者可以尝试从一个队列接收数据。这就是我们使用队列的方式。

**消费者**  
消费与接收具有相似的含义。消费者大多时候是一个等待接收消息的程序。请注意生产者，消费者和消息中间件很多时候并不在同一机器上。同一个应用程序既可以是生产者又是可以是消费者。

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151856006-ce9fc5c1-d5d2-494d-92a5-b9d3ab8e4693.png)

### RabbitMQ核心部分
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151856031-8ad9c817-0190-4791-a9c9-c04eb7f248b0.png)

### 各个名词介绍
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151856163-a30dd407-cc3e-486d-9cc5-a74787bbb175.png)

**Broker**：接收和分发消息的应用， RabbitMQ Server 就是 Message Broker

**Virtual host**：出于多租户和安全因素设计的，把 AMQP 的基本组件划分到一个虚拟的分组中，类似于网络中的 namespace 概念。当多个不同的用户使用同一个 RabbitMQ server 提供的服务时，可以划分出多个 vhost，每个用户在自己的 vhost 创建 exchange／ queue 等

**Connection**： publisher／ consumer 和 broker 之间的 TCP 连接

**Channel**：如果每一次访问 RabbitMQ 都建立一个 Connection，在消息量大的时候建立 TCP Connection 的开销将是巨大的，效率也较低。 Channel 是在 connection 内部建立的逻辑连接，如果应用程序支持多线程，通常每个 thread 创建单独的 channel 进行通讯， AMQP method 包含了 channel id 帮助客户端和 message broker 识别 channel，所以 channel 之间是完全隔离的。 Channel 作为轻量级的**Connection 极大减少了操作系统建立 TCP connection 的开销**

**Exchange**： message 到达 broker 的第一站，根据分发规则，匹配查询表中的 routing key，分发消息到 queue 中去。常用的类型有： direct (point-to-point), topic (publish-subscribe) and fanout (multicast)

**Queue**： 消息最终被送到这里等待 consumer 取走

**Binding**： exchange 和 queue 之间的虚拟连接， binding 中可以包含 routing key， Binding 信息被保存到 exchange 中的查询表中，用于 message 的分发依据

## 安装RabbitMQ
### 手动安装
**1、官网地址**

https://www.rabbitmq.com/download.html

2、**文件上传上传到**`**/usr/local/software**`** 目录下**(如果没有 software 需要自己创建)

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151856237-65d5c81d-0957-443e-b665-f4507660f28b.png)

**3、安装文件(分别按照以下顺序安装)**

```powershell
rpm -ivh erlang-21.3-1.el7.x86_64.rpm
yum install socat -y
rpm -ivh rabbitmq-server-3.8.8-1.el7.noarch.rpm
```

**4、常用命令(按照以下顺序执行)**

添加开机启动 RabbitMQ 服务

```powershell
chkconfig rabbitmq-server on
```

启动服务

```powershell
/sbin/service rabbitmq-server start
```

查看服务状态

```powershell
/sbin/service rabbitmq-server status
```

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151856493-f0d95d54-25df-4881-821f-03cd099cfd36.png)

停止服务(选择执行)

```powershell
/sbin/service rabbitmq-server stop
```

开启 web 管理插件

```powershell
rabbitmq-plugins enable rabbitmq_management
```

用默认账号密码(guest)访问地址 `http://47.115.185.244:15672`出现权限问题

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151856376-a767ad73-f123-4c5b-b65e-08e37e8e1ec9.png)

**5、添加一个新的用户**

创建账号

```plain
rabbitmqctl add_user admin 123
```

设置用户角色

```plain
rabbitmqctl set_user_tags admin administrator
```

设置用户权限

```powershell
# set_permissions [-p <vhostpath>] <user> <conf> <write> <read> 
rabbitmqctl set_permissions -p "/" admin ".*" ".*" ".*"4
```

用户 user_admin 具有/vhost1 这个 virtual host 中所有资源的配置、写、读权限

当前用户和角色

```plain
rabbitmqctl list_users
```

**6、再次利用 admin 用户登录**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151856424-72d68be8-efdd-42c1-a883-26928676b289.png)

**7、重置命令**

关闭应用的命令为:

```plain
rabbitmqctl stop_app
```

清除的命令为

```plain
rabbitmqctl reset
```

重新启动命令为

```plain
rabbitmqctl start_app
```

### Docker安装
**1、安装**

```powershell
# 拉取镜像
docker pull rabbitmq:3.13-management

# -d 参数：后台运行 Docker 容器
# --name 参数：设置容器名称
# -p 参数：映射端口号,格式为 "宿主机端口号:容器内部端口号" 5672供客户端程序访问,15672供后台应用管理界面访问
# -v 参数：卷映射目录
# -e 参数：设置容器内的环境变量,这里我们设置了登录RabbitMQ管理后台的默认用户和密码
docker run -d \
--name rabbitmq \
-p 5672:5672 \
-p 15672:15672 \
-v rabbitmq-plugin:/plugins \
-e RABBITMQ_DEFAULT_USER=guest \
-e RABBITMQ_DEFAULT_PASS=123456 \
rabbitmq:3.13-management
```

**2、验证**

访问后台管理界面： `http://<ip>:15672`

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151856559-7c1d31f5-d8de-4bf0-b3cb-9fc250e9823c.png)

登录后界面如图:  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151856761-a91a9963-8f2b-4196-abdd-254e396cdac0.png)

## Hello World
我们将用java编写两个程序。发送单个消息的生产者和接收消息并打印出来的消费者。  
下图中，”P”是我们的生产者，”C”是我们的消费者。中间的框是一个队列-RabbitMQ 代表使用者保留的消息缓冲区

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151856777-0db0a07b-8827-474d-b35c-761ecd005496.png)

### 导入依赖
```xml
<dependency>
  <groupId>com.rabbitmq</groupId>
  <artifactId>amqp-client</artifactId>
  <version>5.20.0</version>
</dependency>
```

### 消息发送端(生产者)
```java
package com.shiguang.rabbitmq.simple;

import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;

import java.io.IOException;
import java.util.concurrent.TimeoutException;

/**
 * Created By Shiguang On 2024/10/11 10:05
 */
public class Producer {
    public static void main(String[] args) throws IOException, TimeoutException {
        // 创建连接工厂
        ConnectionFactory connectionFactory = new ConnectionFactory();

        // 设置主机地址
        connectionFactory.setHost("192.168.10.66");

        // 设置端口号: 默认为5672
        connectionFactory.setPort(5672);

        // 虚拟主机名称: 默认为/
        connectionFactory.setVirtualHost("/");

        // 设置连接用户名: 默认为guest
        connectionFactory.setUsername("guest");

        // 设置连接密码: 默认为guest
        connectionFactory.setPassword("123456");

        // 创建连接
        Connection connection = connectionFactory.newConnection();

        // 创建通道
        Channel channel = connection.createChannel();

        // 声明队列
        // queue 名称
        // durable 是否持久化
        // exclusive 是否独占本次连接,若为true,则队列仅在本次连接可见,连接关闭后,队列自动删除
        // autoDelete 是否自动删除,若为true,则当最后一个消费者断开连接后,队列会被删除
        // arguments 其他参数
        channel.queueDeclare("simple_queue", false, false, false, null);

        // 发布消息
        String message = "hello rabbitmq";

        // exchange 交换机名称
        // routingKey 路由键,用于将消息路由到指定的队列,如果没有指定,消息将发送到默认的交换机,默认的交换机名称为空字符串
        // props 消息属性,用于设置消息的属性,如消息的优先级、过期时间等
        // body 消息体,即要发送的消息内容
        channel.basicPublish("", "simple_queue", null, message.getBytes());

        System.out.println("消息已发送:" + message + "");

        // 关闭资源
        channel.close();
        connection.close();

    }
}
```

执行后如下所示：

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151856799-eade7d2f-739d-4789-9854-4f3c7931176d.png)

可以在后台管理界面查看状态

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151856874-90aa3ee9-5fb3-40bc-96a4-248f975527e9.png)

查看消息队列

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151856915-408caf2e-3627-45da-acad-0b5d3be62faa.png)

### 消息接收端(消费者)
```java
package com.shiguang.rabbitmq.simple;

import com.rabbitmq.client.*;

import java.io.IOException;

/**
 * Created By Shiguang On 2024/10/11 11:17
 */
public class Consumer {
    public static void main(String[] args) throws Exception {
        // 1、创建一个ConnectionFactory，并设置主机名、端口号、虚拟主机、用户名和密码。
        ConnectionFactory factory = new ConnectionFactory();
        // 2、设置连接参数。
        factory.setHost("192.168.10.66");
        factory.setPort(5672);
        factory.setVirtualHost("/");
        factory.setUsername("guest");
        factory.setPassword("123456");

        // 3、通过工厂创建连接。
        Connection connection = factory.newConnection();

        // 4、通过连接创建通道。
        Channel channel = connection.createChannel();

        // 5、创建队列，并指定队列名称、是否持久化、是否独占、是否自动删除、其他参数。
        // 生产者已经创建了队列，这里不需要再创建
        //        channel.queueDeclare("simple.queue", true, false, false, null);

        // 6、消费消息
        DefaultConsumer consumer = new DefaultConsumer(channel) {

            // consumerTag 消费者标签，用来标识消费者，在监听消息时使用。
            // envelope 消息的元数据，包括交换机、路由键、投递模式等。
            // properties 消息的属性，如消息的优先级、过期时间等。
            // body 消息的正文，即要发送的数据。
            @Override
            public void handleDelivery(String consumerTag,
                                       Envelope envelope,
                                       AMQP.BasicProperties properties,
                                       byte[] body) throws IOException {
                System.out.println("consumerTag: " + consumerTag);
                System.out.println("Exchange: " + envelope.getExchange());
                System.out.println("RoutingKey: " + envelope.getRoutingKey());
                System.out.println("properties: " + properties);
                System.out.println("body: " + new String(body));
            }
        };

        // 7、注册消费者，指定队列名称、是否自动应答、消费者。
        channel.basicConsume("simple_queue", true, consumer);

        // 8、关闭资源
        channel.close();
        connection.close();

    }
}
```

执行结果如下：

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151857201-a834935a-3615-460b-80f6-a384e67d65a9.png)

再次查看状态

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151857238-b40be965-e53a-41a5-aac1-d0f3aa9b331a.png)

再次查看消息队列

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151857329-57f3d86a-63a7-4aad-908e-f179bb7ea484.png)

## RabbitMQ工作模式
### 工作模式概述
RabbitMQ有7种用法：

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151857375-0b95700e-4e32-40e9-b33d-5f84e1f79c6b.png)

以下是 RabbitMQ 的一些常见用法：

1. 消息队列：

RabbitMQ 最基本的用法是作为消息队列。生产者将消息发送到 RabbitMQ 服务器，消费者从队列中获取消息并进行处理。这种模式可以实现应用程序的解耦和异步通信。

2. 发布/订阅模式：

RabbitMQ 支持发布/订阅模式，允许生产者将消息发布到一个或多个交换机（Exchange），消费者订阅感兴趣的队列。当有新消息到达时，RabbitMQ 会将消息路由到所有订阅了相应队列的消费者。

3. 路由模式：

在路由模式中，生产者将消息发送到交换机，并指定一个路由键（Routing Key）。RabbitMQ 根据路由键将消息路由到绑定了相应路由键的队列。这种模式可以实现更精细的消息路由。

4. 主题模式：

主题模式是路由模式的扩展，允许使用通配符来匹配路由键。例如，可以使用“*”通配符匹配一个单词，使用“#”通配符匹配任意数量的单词。这种模式可以实现更灵活的消息路由。

5. RPC（远程过程调用）：

RabbitMQ 可以用于实现 RPC 机制，允许客户端调用远程服务器上的方法。客户端将请求消息发送到 RabbitMQ，服务器处理请求并将响应消息发送回客户端。

### 工作队列模式
工作队列(又称任务队列)的主要思想是避免立即执行资源密集型任务，而不得不等待它完成。相反我们安排任务在之后执行。我们把任务封装为消息并将其发送到队列。在后台运行的工作进程将弹出任务并最终执行作业。 当有多个工作线程时，这些工作线程将一起处理这些任务。

本质上我们刚刚写的HelloWorld程序就是这种模式，只是简化到了最简单的情况：

+ 生产者只有一个
+ 发送一个消息
+ 消费者也只有一个，消息也只能被这个消费者消费

所以HelloWorld也称为简单模式。

现在我们还原一下常规情况：

+ 生产者发送多个消息
+ 由多个消费者来竞争
+ 谁抢到算谁的

结论：

多个消费者监听同一个队列，则各消费者之间对同一个消息是竞争的关系。Work Queues工作模式适用于任务较重或任务较多的情况，多消费者分摊任务，可以提高消息处理的效率。

#### 生产者代码
##### 封装工具类
```java
package com.shiguang.rabbitmq.util;

import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;

/**
 * Created By Shiguang On 2024/10/11 12:25
 */
public class ConnectionUtil {
    public static final String HOST_ADDRESS = "192.168.10.66";
    public static final int PORT = 5672;
    public static final String VIRTUAL_HOST = "/";
    public static final String USERNAME = "guest";
    public static final String PASSWORD = "123456";

    /**
     * 获取与 RabbitMQ 服务器的连接
     *
     * @return 与 RabbitMQ 服务器的连接对象
     * @throws Exception 如果在创建连接时发生错误
     */
    public static Connection getConnection() throws Exception {
        // 创建一个新的连接工厂对象
        ConnectionFactory factory = new ConnectionFactory();
        // 设置 RabbitMQ 服务器的主机地址
        factory.setHost(HOST_ADDRESS);
        // 设置 RabbitMQ 服务器的端口号
        factory.setPort(PORT);
        // 设置 RabbitMQ 服务器的虚拟主机
        factory.setVirtualHost(VIRTUAL_HOST);
        // 设置连接 RabbitMQ 服务器的用户名
        factory.setUsername(USERNAME);
        // 设置连接 RabbitMQ 服务器的密码
        factory.setPassword(PASSWORD);
        // 返回新创建的连接对象
        return factory.newConnection();
    }

    public static void main(String[] args) throws Exception {
        Connection connection = getConnection();
        if (connection != null) {
            System.out.println("连接成功!!");
            System.out.println("connection = " + connection + "");
        } else {
            System.out.println("连接失败!!");
        }
    }
}
```

##### 编写代码（发布消息）
```java
package com.shiguang.rabbitmq.work;

import com.rabbitmq.client.Channel;       // RabbitMQ 的通道接口，用于执行实际的操作
import com.rabbitmq.client.Connection;    // RabbitMQ 的连接接口
import com.shiguang.rabbitmq.util.ConnectionUtil;  // 工具类，用于获取 RabbitMQ 连接

public class Producer {

    // 定义队列名称，工作队列模式下生产者和消费者共用同一个队列
    public static final String QUEUE_NAME = "work_queue";

    public static void main(String[] args) throws Exception {

        // 1. 通过工具类获取与 RabbitMQ 服务器的连接
        // ConnectionUtil.getConnection() 内部封装了 ConnectionFactory 的配置（如 host、port、username、password 等）
        Connection connection = ConnectionUtil.getConnection();

        // 2. 创建一个通信通道（Channel），RabbitMQ 的大部分 API 操作都在 Channel 上进行
        Channel channel = connection.createChannel();

        // 3. 声明（创建）队列
        // 参数说明：
        //   queue       : 队列名称
        //   durable     : true 表示队列持久化（服务器重启后队列仍存在）
        //   exclusive   : false 表示队列非独占，可以被多个消费者共享
        //   autoDelete  : false 表示队列不会在最后一个消费者断开后自动删除
        //   arguments   : null 表示无额外参数
        channel.queueDeclare(QUEUE_NAME, true, false, false, null);

        // 4. 循环发送 10 条消息到队列中
        for (int i = 1; i <= 10; i++) {
            // 消息内容：例如 "1hello rabbitmq"、"2hello rabbitmq" ...
            String body = i + "hello rabbitmq";

            // 5. 发布消息到队列
            // 参数说明：
            //   exchange    : "" 表示使用默认交换机（default exchange），会直接根据 routingKey（这里是队列名）路由
            //   routingKey  : QUEUE_NAME，即目标队列名称
            //   props       : null 表示不设置消息的额外属性（如持久化、优先级等）
            //                 如果需要消息持久化，可使用 MessageProperties.PERSISTENT_TEXT_PLAIN
            //   body        : 消息体的字节数组
            channel.basicPublish("", QUEUE_NAME, null, body.getBytes());

            // 可选：打印日志，便于观察生产者发送情况
            System.out.println(" [x] Sent '" + body + "'");
        }

        // 6. 关闭通道（推荐先关闭通道，再关闭连接）
        channel.close();

        // 7. 关闭连接（代码中未写，但实际项目中建议关闭，以释放资源）
        connection.close();
    }
}
```

##### 发送消息效果
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151857381-df0bfa25-5282-4275-bc7f-e3dd45670a4e.png)

#### 消费者代码
##### 编写代码
Consumer1：

```java
package com.shiguang.rabbitmq.work;

import com.rabbitmq.client.*;
import com.shiguang.rabbitmq.util.ConnectionUtil;

import java.io.IOException;

/**
 * 工作队列模式 - 消费者1
 * 该消费者从工作队列中接收并处理消息
 */
public class Consumer1 {
    // 定义队列名称常量
    static final String QUEUE_NAME = "work_queue";

    public static void main(String[] args) throws Exception {
        // 获取RabbitMQ连接
        Connection connection = ConnectionUtil.getConnection();
        
        // 创建频道
        Channel channel = connection.createChannel();
        
        /**
         * 声明队列
         * 参数说明：
         * queue - 队列名称
         * durable - 是否持久化（true表示重启后队列依然存在）
         * exclusive - 是否排他（是否仅限此连接使用）
         * autoDelete - 是否自动删除（没有消费者时自动删除）
         * arguments - 其他参数
         */
        channel.queueDeclare(QUEUE_NAME, true, false, false, null);
        
        // 创建消费者实例，重写消息处理方法
        DefaultConsumer consumer = new DefaultConsumer(channel) {
            @Override
            public void handleDelivery(String consumerTag,
                                       Envelope envelope,
                                       AMQP.BasicProperties properties,
                                       byte[] body) throws IOException {
                // 处理接收到的消息，将字节数组转换为字符串并打印
                System.out.println("消费者1接收到的消息: " + new String(body));
            }
        };
        
        /**
         * 开始消费消息
         * 参数说明：
         * queue - 要监听的队列名称
         * autoAck - 是否自动确认消息（true表示自动确认）
         * callback - 消费者回调对象
         */
        channel.basicConsume(QUEUE_NAME, true, consumer);
    }
}
```

Consumer2：

```java
package com.shiguang.rabbitmq.work;

import com.rabbitmq.client.*;
import com.shiguang.rabbitmq.util.ConnectionUtil;

import java.io.IOException;

/**
 * 工作队列模式 - 消费者2
 * 该消费者与消费者1共同从同一个工作队列中接收消息，实现负载均衡
 */
public class Consumer2 {
    // 定义队列名称常量，与消费者1使用相同的队列
    static final String QUEUE_NAME = "work_queue";
    
    public static void main(String[] args) throws Exception {
        // 获取RabbitMQ连接
        Connection connection = ConnectionUtil.getConnection();
        
        // 创建频道
        Channel channel = connection.createChannel();
        
        /**
         * 声明队列
         * 参数说明：
         * queue - 队列名称，与消费者1相同，确保连接到同一个队列
         * durable - 是否持久化（true表示重启后队列依然存在）
         * exclusive - 是否排他（是否仅限此连接使用）
         * autoDelete - 是否自动删除（没有消费者时自动删除）
         * arguments - 其他参数
         */
        channel.queueDeclare(QUEUE_NAME, true, false, false, null);
        
        // 创建消费者实例，重写消息处理方法
        DefaultConsumer consumer = new DefaultConsumer(channel) {
            @Override
            public void handleDelivery(String consumerTag,
                                       Envelope envelope,
                                       AMQP.BasicProperties properties,
                                       byte[] body) throws IOException {
                // 处理接收到的消息，将字节数组转换为字符串并打印
                System.out.println("消费者2接收到的消息: " + new String(body));
            }
        };
        
        /**
         * 开始消费消息
         * 参数说明：
         * queue - 要监听的队列名称，与消费者1相同
         * autoAck - 是否自动确认消息（true表示自动确认）
         * callback - 消费者回调对象
         * 
         * 注意：多个消费者监听同一个队列时，RabbitMQ会采用轮询方式分发消息
         * 实现工作队列的负载均衡效果
         */
        channel.basicConsume(QUEUE_NAME, true, consumer);
    }
}
```

##### 运行效果
Consumer1：

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151857492-98726246-0406-42f2-b5ec-394c2853473d.png)

Consumer2:

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151857491-a33719a5-1647-45a0-89d2-c0545fee457c.png)

### 发布订阅模式（Publish/Subscribe）
`Publish`/`Subscribe`模式需要引入新角色：交换机

+ 生产者不是把消息直接发送到队列，而是发送到交换机
+ 交换机接收消息，而如何处理消息取决于交换机的类型
+ 交换机有如下3种常见类型 
    - Fanout: 广播，将消息发送给所有绑定到交换机的队列
    - Direct: 定向，把消息交给符合指定routing key的队列
    - Topic: 通配符，把消息交给符合routing pattern(路由模式)的队列

注意：Exchange(交换机)**只负责转发**消息，**不具备存储**消息的能力，因此如果没有任何队列与Exchange绑定，或者没有符合路由规侧的队列，那么消息会丢失！

组件之间关系：

+ 生产者把消息发送到交换机
+ 队列直接和交换机绑定

工作机制：消息发送到交换机上，就会以**广播**的形式发送给所有已绑定队列

理解概念：

+ Publish:发布，这里就是把消息发送到交换机上
+ Subscribe:订阅，这里只要把队列和交换机绑定，事实上就形成了一种订阅关系

#### 生产者代码
##### 编写代码
```java
package com.shiguang.rabbitmq.fanout;

import com.rabbitmq.client.BuiltinExchangeType;
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.shiguang.rabbitmq.util.ConnectionUtil;

/**
 * Created By Shiguang On 2024/10/11 13:08
 */
public class Producer {
    public static final String EXCHANGE_NAME = "fanout_exchange";
    public static void main(String[] args) throws Exception {
        // 1、获取连接
        Connection connection = ConnectionUtil.getConnection();

        // 2、获取通道
        Channel channel = connection.createChannel();

        // 3、声明交换机
        // exchange 交换机名称
        // type 交换机类型
        // durable 是否持久化
        // autoDelete 是否自动删除
        // arguments 其他参数
        channel.exchangeDeclare(EXCHANGE_NAME, BuiltinExchangeType.FANOUT, true, false, false,null);

        // 4、创建队列
        channel.queueDeclare("fanout_queue1", true, false, false, null);
        channel.queueDeclare("fanout_queue2", true, false, false, null);

        // 5、绑定队列到交换机
        // queue 队列名称
        // exchange 交换机名称
        // routingKey 路由键, 用于指定消息的路由规则
        channel.queueBind("fanout_queue1", EXCHANGE_NAME, "");
        channel.queueBind("fanout_queue2", EXCHANGE_NAME, "");

        // 6、发送消息
        String body = "日志信息: 张三调用了findAll方法 ";
        channel.basicPublish(EXCHANGE_NAME, "", null, body.getBytes());

        // 7、释放资源
        channel.close();
        connection.close();

    }
}
```

##### 执行效果
可以通过后台查看我们刚创建的交换机

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151857695-af30b50e-0133-4498-a7a2-fa99ff45534f.png)

点击 `Name` 栏的交换机名称跳转到详情页，展开`Bindings`查看该交换机绑定的消息队列

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151857711-4e970a80-6fcf-46f8-8771-be3f01e2a74c.png)

可以看到新增两个消息队列并分别发送了一条消息

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151857807-f61bca47-68bd-4cb7-906a-50cd910edfff.png)

点击`Name`栏的消息队列名称可查看详情

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151857879-c84cd473-d42c-41f8-b2e6-9d6c778dc229.png)

通过`Get Messages(s)`按钮可以查看消息详情

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151857820-3ff17d3b-2886-4ddb-8c0d-75a90cfca809.png)

#### 消费者代码
##### 编写代码
**Consumer1：**

```java
package com.shiguang.rabbitmq.fanout;

import com.rabbitmq.client.*;
import com.shiguang.rabbitmq.util.ConnectionUtil;

import java.io.IOException;

/**
 * 扇出交换机模式 - 消费者1
 * 扇出交换机会将消息广播到所有绑定的队列，忽略路由键
 */
public class Consumer1 {
    // 定义队列名称常量 - 消费者1专用的队列
    static final String QUEUE_NAME = "fanout_queue1";

    public static void main(String[] args) throws Exception {
        // 获取RabbitMQ连接
        Connection connection = ConnectionUtil.getConnection();
        
        // 创建频道
        Channel channel = connection.createChannel();
        
        /**
         * 声明队列
         * 参数说明：
         * queue - 队列名称
         * durable - 是否持久化（true表示重启后队列依然存在）
         * exclusive - 是否排他（是否仅限此连接使用）
         * autoDelete - 是否自动删除（没有消费者时自动删除）
         * arguments - 其他参数
         * 
         * 注意：在扇出模式中，通常需要在生产者或单独的绑定程序中
         * 将队列绑定到扇出交换机
         */
        channel.queueDeclare(QUEUE_NAME, true, false, false, null);
        
        // 创建消费者实例，重写消息处理方法
        DefaultConsumer consumer = new DefaultConsumer(channel) {
            @Override
            public void handleDelivery(String consumerTag,
                                       Envelope envelope,
                                       AMQP.BasicProperties properties,
                                       byte[] body) throws IOException {
                // 处理接收到的消息，将字节数组转换为字符串并打印
                System.out.println("消费者1接收到的消息内容: " + new String(body));
                System.out.println("队列1 消费者1 日志打印...");
            }
        };
        
        /**
         * 开始消费消息
         * 参数说明：
         * queue - 要监听的队列名称
         * autoAck - 是否自动确认消息（true表示自动确认）
         * callback - 消费者回调对象
         * 
         * 扇出模式特点：
         * - 同一个消息会被发送到所有绑定到交换机的队列
         * - 路由键在扇出模式中被忽略
         * - 实现消息的广播功能
         * - 每个消费者都会收到完整的消息副本
         */
        channel.basicConsume(QUEUE_NAME, true, consumer);
    }
}
```

**Consumer2：**

```java
package com.shiguang.rabbitmq.fanout;

import com.rabbitmq.client.*;
import com.shiguang.rabbitmq.util.ConnectionUtil;

import java.io.IOException;

/**
 * 扇出交换机模式 - 消费者2
 * 该消费者与消费者1同时监听不同的队列，但都绑定到同一个扇出交换机
 * 实现消息的广播功能，每个消费者都会收到相同的消息副本
 */
public class Consumer2 {
    // 定义队列名称常量 - 消费者2专用的队列
    static final String QUEUE_NAME = "fanout_queue2";
    
    public static void main(String[] args) throws Exception {
        // 获取RabbitMQ连接
        Connection connection = ConnectionUtil.getConnection();
        
        // 创建频道
        Channel channel = connection.createChannel();
        
        /**
         * 声明队列
         * 参数说明：
         * queue - 队列名称，与消费者1不同，表示独立的队列
         * durable - 是否持久化（true表示重启后队列依然存在）
         * exclusive - 是否排他（是否仅限此连接使用）
         * autoDelete - 是否自动删除（没有消费者时自动删除）
         * arguments - 其他参数
         * 
         * 注意：虽然队列名称不同，但通过绑定到同一个扇出交换机，
         * 两个队列都会收到相同的消息
         */
        channel.queueDeclare(QUEUE_NAME, true, false, false, null);
        
        // 创建消费者实例，重写消息处理方法
        DefaultConsumer consumer = new DefaultConsumer(channel) {
            @Override
            public void handleDelivery(String consumerTag,
                                       Envelope envelope,
                                       AMQP.BasicProperties properties,
                                       byte[] body) throws IOException {
                // 处理接收到的消息，将字节数组转换为字符串并打印
                System.out.println("消费者2接收到的消息内容: " + new String(body));
                System.out.println("队列2 消费者2 日志打印...");
                // 这里可以添加消费者2特有的业务逻辑处理
            }
        };
        
        /**
         * 开始消费消息
         * 参数说明：
         * queue - 要监听的队列名称
         * autoAck - 是否自动确认消息（true表示自动确认）
         * callback - 消费者回调对象
         * 
         * 扇出模式工作流程：
         * 1. 生产者发送消息到扇出交换机
         * 2. 扇出交换机会将消息复制多份
         * 3. 每份消息分别发送到所有绑定的队列（fanout_queue1, fanout_queue2等）
         * 4. 每个队列的消费者都会独立处理消息
         */
        channel.basicConsume(QUEUE_NAME, true, consumer);
    }
}
```

##### 执行效果
> 示例代码两个Consumer分别绑定不同的消息队列，为非竞争关系，若绑定相同的消息队列则为竞争关系
>

Consumer1：

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151857984-e42d454e-f2bc-42ad-a217-59e3083dfe70.png)

Consumer2：

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151858070-3c783db7-10a3-4df8-9eb0-820de23a55f8.png)

### 路由模式（Routing）
+ 通过 **路由绑定** 的方式，把交换机和队列关联起来
+ 交换机和队列通过路由键进行绑定
+ 生产者发送消息时不仅要指定交换机，还要指定路由键
+ 交换机接收到消息会发送到路由键绑定的队列
+ 在编码上与Publish/Subscribe发布与订阅模式的区别： 
    - 交换机的类型为：Direct
    - 队列绑定交换机的时候需要指定routing key。

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151858174-85c8e720-05bc-411a-9b45-30bb5d760f47.png)

#### 生产者代码
##### 编写代码
```java
package com.shiguang.rabbitmq.routing;

import com.rabbitmq.client.BuiltinExchangeType;
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.shiguang.rabbitmq.util.ConnectionUtil;

/**
 * 路由模式 - 生产者
 * 使用直连交换机，根据路由键精确匹配将消息路由到不同的队列
 */
public class Producer {
    // 定义直连交换机名称
    public static final String EXCHANGE_NAME = "test_direct";
    
    public static void main(String[] args) throws Exception {
        // 获取RabbitMQ连接
        Connection connection = ConnectionUtil.getConnection();
        
        // 创建频道
        Channel channel = connection.createChannel();
        
        /**
         * 声明直连交换机
         * 参数说明：
         * exchange - 交换机名称
         * type - 交换机类型，DIRECT表示直连交换机
         * durable - 是否持久化（true表示重启后交换机依然存在）
         * autoDelete - 是否自动删除（没有队列绑定时自动删除）
         * internal - 是否内部使用（true表示不能被客户端使用）
         * arguments - 其他参数
         */
        channel.exchangeDeclare(EXCHANGE_NAME, BuiltinExchangeType.DIRECT, true, false, false, null);
        
        // 定义两个队列名称
        String queue1Name = "direct_queue1";
        String queue2Name = "direct_queue2";
        
        /**
         * 声明两个队列
         * 参数说明：
         * queue - 队列名称
         * durable - 是否持久化
         * exclusive - 是否排他
         * autoDelete - 是否自动删除
         * arguments - 其他参数
         */
        channel.queueDeclare(queue1Name, true, false, false, null);
        channel.queueDeclare(queue2Name, true, false, false, null);
        
        /**
         * 将队列绑定到交换机，并指定路由键
         * 绑定关系说明：
         * queue1Name - 只绑定路由键"error"，只接收错误日志
         * queue2Name - 绑定路由键"info"、"error"、"warning"，接收所有级别的日志
         */
        // 队列1只接收error级别的日志
        channel.queueBind(queue1Name, EXCHANGE_NAME, "error");
        
        // 队列2接收info、error、warning三种级别的日志
        channel.queueBind(queue2Name, EXCHANGE_NAME, "info");
        channel.queueBind(queue2Name, EXCHANGE_NAME, "error");
        channel.queueBind(queue2Name, EXCHANGE_NAME, "warning");
        
        /**
         * 发送消息到交换机
         * 参数说明：
         * exchange - 交换机名称
         * routingKey - 路由键"warning"，决定消息路由到哪些队列
         * props - 消息属性（null表示使用默认属性）
         * body - 消息内容
         * 
         * 根据绑定关系，这条消息会路由到：
         * - queue2（因为queue2绑定了"warning"路由键）
         * - queue1不会收到（因为queue1只绑定了"error"路由键）
         */
        String body = "日志信息: 张三调用了delete方法. 执行出错,日志级别warning";
        channel.basicPublish(EXCHANGE_NAME, "warning", null, body.getBytes());
        
        System.out.println("消息发送成功: " + body);
        
        // 关闭资源
        channel.close();
        connection.close();
    }
}
```

##### 运行效果
新创建的交换机如图所示

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151858232-127bfec2-21ea-4b05-876f-3d2448fff567.png)

详情如图所示，可以看到绑定了两个消息队列`direct_queue1` 和`direct_queue2`，`direct_queue1`关联`error`一个路由键，`direct_queue2`关联了`error`、`info`、`warning`三个路由键

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151858223-4d4ffd14-2f22-4c6c-9583-2ca6bacf0ba9.png)

#### 消费者代码
##### 编写代码
**Consumer1：**

```java
package com.shiguang.rabbitmq.routing;

import com.rabbitmq.client.*;
import com.shiguang.rabbitmq.util.ConnectionUtil;

import java.io.IOException;
import java.nio.charset.StandardCharsets;

/**
 * 路由模式（Direct）- 消费者1
 * 只接收 routingKey = "error" 的消息
 */
public class Consumer1 {

    // 交换机名称（必须与生产者一致）
    private static final String EXCHANGE_NAME = "test_direct";
    // 队列名称
    private static final String QUEUE_NAME = "direct_queue1";
    // 路由键（只接收 error）
    private static final String ROUTING_KEY = "error";

    public static void main(String[] args) throws Exception {
        // 1) 获取连接
        Connection connection = ConnectionUtil.getConnection();

        // 2) 创建信道
        Channel channel = connection.createChannel();

        // 3) 声明交换机（direct）
        // durable=true：交换机持久化
        channel.exchangeDeclare(EXCHANGE_NAME, BuiltinExchangeType.DIRECT, true);

        // 4) 声明队列
        // durable=true：队列持久化
        channel.queueDeclare(QUEUE_NAME, true, false, false, null);

        // 5) 绑定队列到交换机，并指定路由键
        // 只有 routingKey 完全匹配 "error" 的消息才会进入该队列
        channel.queueBind(QUEUE_NAME, EXCHANGE_NAME, ROUTING_KEY);

        System.out.println("消费者1已启动：");
        System.out.println("  exchange = " + EXCHANGE_NAME);
        System.out.println("  queue    = " + QUEUE_NAME);
        System.out.println("  bind key = " + ROUTING_KEY);
        System.out.println("等待接收 error 级别的日志消息...");

        // 6) 创建消费者并消费消息
        DeliverCallback deliverCallback = (consumerTag, delivery) -> {
            String msg = new String(delivery.getBody(), StandardCharsets.UTF_8);
            System.out.println("[Consumer1] 收到消息: " + msg);
            System.out.println("[Consumer1] routingKey=" + delivery.getEnvelope().getRoutingKey());
        };

        CancelCallback cancelCallback = consumerTag ->
                System.out.println("[Consumer1] 消费者被取消: " + consumerTag);

        // autoAck=true：自动确认（演示用）
        channel.basicConsume(QUEUE_NAME, true, deliverCallback, cancelCallback);
    }
}
```

**Consumer2：**

```java
package com.shiguang.rabbitmq.routing;

import com.rabbitmq.client.*;
import com.shiguang.rabbitmq.util.ConnectionUtil;

import java.io.IOException;
import java.nio.charset.StandardCharsets;

/**
 * 路由模式（Direct）- 消费者2
 * 监听 direct_queue2 队列
 * 绑定多个 routingKey：info / warning / error
 */
public class Consumer2 {

    // 交换机名称（必须与生产者一致）
    private static final String EXCHANGE_NAME = "direct_logs";

    // 队列名称
    private static final String QUEUE_NAME = "direct_queue2";

    // 绑定的路由键
    private static final String[] ROUTING_KEYS = {"info", "warning", "error"};

    public static void main(String[] args) throws Exception {
        // 1) 获取连接
        Connection connection = ConnectionUtil.getConnection();

        // 2) 创建信道
        Channel channel = connection.createChannel();

        // 3) 声明交换机（direct）
        channel.exchangeDeclare(EXCHANGE_NAME, BuiltinExchangeType.DIRECT, true);

        // 4) 声明队列
        channel.queueDeclare(QUEUE_NAME, true, false, false, null);

        // 5) 队列绑定多个 routingKey
        for (String key : ROUTING_KEYS) {
            channel.queueBind(QUEUE_NAME, EXCHANGE_NAME, key);
        }

        System.out.println("消费者2已启动：");
        System.out.println("  exchange = " + EXCHANGE_NAME);
        System.out.println("  queue    = " + QUEUE_NAME);
        System.out.print("  bind keys= ");
        for (int i = 0; i < ROUTING_KEYS.length; i++) {
            System.out.print(ROUTING_KEYS[i] + (i == ROUTING_KEYS.length - 1 ? "\n" : ", "));
        }
        System.out.println("等待接收所有级别日志 (info/warning/error)...");

        // 6) 消费消息（建议打印实际 routingKey，以 routingKey 为准做分支）
        DeliverCallback deliverCallback = (consumerTag, delivery) -> {
            String routingKey = delivery.getEnvelope().getRoutingKey();
            String message = new String(delivery.getBody(), StandardCharsets.UTF_8);

            System.out.println("[Consumer2] routingKey=" + routingKey + " message=" + message);

            // 根据 routingKey 来分流处理（比 message.contains(...) 更可靠）
            switch (routingKey) {
                case "error":
                    System.out.println(">>> 处理错误日志：进行错误统计和记录");
                    recordErrorLog(message);
                    break;
                case "warning":
                    System.out.println(">>> 处理警告日志：注意潜在问题");
                    recordWarningLog(message);
                    break;
                case "info":
                    System.out.println(">>> 处理信息日志：正常业务记录");
                    recordInfoLog(message);
                    break;
                default:
                    System.out.println(">>> 未知 routingKey，走通用处理");
                    recordGeneralLog(message);
                    break;
            }
        };

        CancelCallback cancelCallback = consumerTag ->
                System.out.println("[Consumer2] 消费者被取消: " + consumerTag);

        // autoAck=true：演示用（生产建议改为 false 手动 ack）
        channel.basicConsume(QUEUE_NAME, true, deliverCallback, cancelCallback);
    }

    // 以下是可以扩展的业务方法
    private static void recordErrorLog(String message) {
        // 错误日志存储逻辑
    }

    private static void recordWarningLog(String message) {
        // 警告日志存储逻辑
    }

    private static void recordInfoLog(String message) {
        // 信息日志存储逻辑
    }

    private static void recordGeneralLog(String message) {
        // 通用日志存储逻辑
    }
}
```

##### 执行效果
由于我们只往`warning`路由键发送消息，而 `direct_queue1`关联`error`一个路由键，`direct_queue2`关联了`error`、`info`、`warning`三个路由键，所以`Consumer1`收不到消息， `Consumer2`可以收到消息

Consumer1：

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151858344-f9f1a415-ac47-46d3-a2e5-7fdd02388681.png)

Consumer2：

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151858401-077c8d7b-609c-46d5-9594-8f973d397bfe.png)

我们可以修改为往`error`路由键发送消息，这样两个消费者就都能接收到消息了

```java
String body = "日志信息: 张三调用了delete方法. 执行出错,日志级别error";
channel.basicPublish(EXCHANGE_NAME, "error", null, body.getBytes());
```

Consumer1：

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151858472-3618e358-79d9-4a09-b718-18612e7fda55.png)

Consumer2：

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151858526-1310b20b-84ee-4e88-b516-2d0cd6696544.png)

### 主题模式（Topics）
+ Topic类型与Direct相比，都是可以根据RoutingKey把消息路由到不同的队列。只不过Topic类型Exchange可以让队列在绑定Routing key的时候使用通配符
+ Routingkey一般都是由一个或多个单词组成，多个单词之间以"`.`"分割，例如：`item.insert`
+ 通配符规则： 
    - #: 匹配零个或多个词
    - *: 匹配一个词、

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151858539-94b07c4f-c25f-48a7-ac69-24b0f0529034.png)

假设有一个主题交换机 `logs`，并且有以下队列和绑定：

+ 队列 `critical_errors` 绑定键为 `*.error`
+ 队列 `user_logs` 绑定键为 `user.*`
+ 队列 `all_logs` 绑定键为 `#`

如果生产者发送一条路由键为 `user.info` 的消息，那么这条消息将被路由到 `user_logs` 和 `all_logs` 队列。

如果生产者发送一条路由键为 `system.error` 的消息，那么这条消息将被路由到 `critical_errors` 和 `all_logs` 队列。

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151858741-bcd05d56-cdb2-42df-a09c-4d25f6c8b041.png)

#### 生产者代码
##### 编写代码
```java
package com.shiguang.rabbitmq.topic;

import com.rabbitmq.client.BuiltinExchangeType;
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.shiguang.rabbitmq.util.ConnectionUtil;

/**
 * 主题交换机模式 - 生产者
 * 使用主题交换机，根据路由键的模式匹配将消息路由到不同的队列
 * 支持通配符匹配，提供更灵活的消息路由能力
 */
public class Producer {
    // 定义主题交换机名称
    public static final String EXCHANGE_NAME = "test_topic";
    
    public static void main(String[] args) throws Exception {
        // 获取RabbitMQ连接
        Connection connection = ConnectionUtil.getConnection();
        
        // 创建频道
        Channel channel = connection.createChannel();
        
        /**
         * 声明主题交换机
         * 参数说明：
         * exchange - 交换机名称
         * type - 交换机类型，TOPIC表示主题交换机
         * durable - 是否持久化（true表示重启后交换机依然存在）
         * autoDelete - 是否自动删除（没有队列绑定时自动删除）
         * internal - 是否内部使用（true表示不能被客户端使用）
         * arguments - 其他参数
         * 
         * 主题交换机特点：
         * - 支持通配符匹配路由键
         * - 提供更灵活的消息路由能力
         * - 适合复杂的消息分发场景
         */
        channel.exchangeDeclare(EXCHANGE_NAME, BuiltinExchangeType.TOPIC, true, false, false, null);
        
        // 定义两个队列名称
        String queue1Name = "topic_queue1";
        String queue2Name = "topic_queue2";
        
        /**
         * 声明两个队列
         * 参数说明：
         * queue - 队列名称
         * durable - 是否持久化
         * exclusive - 是否排他
         * autoDelete - 是否自动删除
         * arguments - 其他参数
         */
        channel.queueDeclare(queue1Name, true, false, false, null);
        channel.queueDeclare(queue2Name, true, false, false, null);
        
        /**
         * 将队列绑定到主题交换机，使用通配符路由键
         * 
         * 通配符说明：
         * * (星号) - 匹配恰好一个单词
         * # (井号) - 匹配零个或多个单词
         * 
         * 绑定关系详解：
         * 
         * 队列1绑定：
         * 1. "#.error" - 匹配任何以.error结尾的路由键
         *    例如: "order.error", "user.error", "system.error"
         * 2. "order.*" - 匹配order后面的任意一个单词
         *    例如: "order.info", "order.error", "order.warning"
         *    
         * 队列2绑定：
         * 1. "*.*" - 匹配恰好有两个单词的路由键
         *    例如: "order.info", "user.error", "system.warning"
         *    不匹配: "order", "user.info.system"
         */
        // 队列1绑定：接收所有错误日志和订单相关的所有日志
        channel.queueBind(queue1Name, EXCHANGE_NAME, "#.error");
        channel.queueBind(queue1Name, EXCHANGE_NAME, "order.*");
        
        // 队列2绑定：接收所有两级路由键的日志
        channel.queueBind(queue2Name, EXCHANGE_NAME, "*.*");
        
        /**
         * 发送消息到主题交换机
         * 参数说明：
         * exchange - 交换机名称
         * routingKey - 路由键"order.info"，包含系统名和日志级别
         * props - 消息属性
         * body - 消息内容
         * 
         * 路由键"order.info"的匹配分析：
         * - 队列1: 匹配 "order.*" (因为order.info符合order.*模式)
         * - 队列2: 匹配 "*.*" (因为order.info正好是两个单词)
         * 
         * 因此这条消息会同时发送到两个队列
         */
        String body = "[所在系统：order][日志级别：info][日志内容: 订单生成,保存成功]";
        channel.basicPublish(EXCHANGE_NAME, "order.info", null, body.getBytes());
        
        System.out.println("消息发送成功: " + body);
        System.out.println("路由键: order.info");
        System.out.println("预计接收队列: topic_queue1, topic_queue2");
        
        // 关闭资源
        channel.close();
        connection.close();
    }
}
```

##### 执行效果
创建的交换机信息如图所示

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151858789-36c7df55-b074-4e38-b87e-3d26ac153a4e.png)

创建的消息队列如图所示：

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151858867-a97dd167-864b-4ac2-99ef-fe4ac7a240f5.png)

#### 消费者代码
##### 编写代码
**Consumer1：**

```java
package com.shiguang.rabbitmq.topic;

import com.rabbitmq.client.*;
import com.shiguang.rabbitmq.util.ConnectionUtil;

import java.io.IOException;

/**
 * 主题交换机模式 - 消费者1
 * 该消费者监听 topic_queue1 队列，专门处理：
 * 1. 所有错误级别的日志 (#.error)
 * 2. 订单系统的所有日志 (order.*)
 * 使用主题交换机的通配符匹配功能，实现灵活的消息路由
 */
public class Consumer1 {
    // 定义队列名称常量 - 消费者1专用的队列
    static final String QUEUE_NAME = "topic_queue1";

    public static void main(String[] args) throws Exception {
        // 获取RabbitMQ连接
        Connection connection = ConnectionUtil.getConnection();
        
        // 创建频道
        Channel channel = connection.createChannel();
        
        /**
         * 声明队列
         * 参数说明：
         * queue - 队列名称，与生产者中定义的队列名称一致
         * durable - 是否持久化（true表示重启后队列依然存在）
         * exclusive - 是否排他（是否仅限此连接使用）
         * autoDelete - 是否自动删除（没有消费者时自动删除）
         * arguments - 其他参数
         */
        channel.queueDeclare(QUEUE_NAME, true, false, false, null);
        
        // 创建消费者实例，重写消息处理方法
        DefaultConsumer consumer = new DefaultConsumer(channel) {
            @Override
            public void handleDelivery(String consumerTag,
                                       Envelope envelope,
                                       AMQP.BasicProperties properties,
                                       byte[] body) throws IOException {
                /**
                 * 处理接收到的消息
                 * 参数说明：
                 * consumerTag - 消费者标签
                 * envelope - 包含交付相关信息（交换机、路由键、交付标签等）
                 * properties - 消息属性
                 * body - 消息内容（字节数组）
                 * 
                 * 此队列接收两种类型的消息：
                 * 1. 所有以.error结尾的错误日志
                 * 2. 所有order开头的订单系统日志
                 */
                String message = new String(body);
                String routingKey = envelope.getRoutingKey();
                
                System.out.println("消费者1接收到的消息内容: " + message);
                System.out.println("路由键: " + routingKey);
                System.out.println("队列1 消费者1 日志打印...");
                
                // 根据路由键进行不同的业务处理
                processMessageByRoutingKey(message, routingKey);
            }
            
            /**
             * 根据路由键进行不同的业务处理
             * @param message 消息内容
             * @param routingKey 路由键
             */
            private void processMessageByRoutingKey(String message, String routingKey) {
                if (routingKey.endsWith(".error")) {
                    // 处理错误日志 - 所有系统的错误日志
                    handleErrorLog(message, routingKey);
                } else if (routingKey.startsWith("order.")) {
                    // 处理订单系统日志 - 订单系统的所有日志
                    handleOrderLog(message, routingKey);
                }
            }
            
            /**
             * 处理错误日志
             * @param message 消息内容
             * @param routingKey 路由键
             */
            private void handleErrorLog(String message, String routingKey) {
                System.out.println(">>> [错误处理] 系统: " + getSystemFromRoutingKey(routingKey) + " - 进行错误报警和紧急处理");
                // 错误日志的具体业务逻辑
                // 1. 发送错误报警通知
                sendErrorAlert(message, routingKey);
                // 2. 记录错误统计
                recordErrorMetrics(routingKey);
                // 3. 触发错误处理流程
                triggerErrorProcess(message);
            }
            
            /**
             * 处理订单系统日志
             * @param message 消息内容
             * @param routingKey 路由键
             */
            private void handleOrderLog(String message, String routingKey) {
                String logLevel = getLogLevelFromRoutingKey(routingKey);
                System.out.println(">>> [订单系统] 级别: " + logLevel + " - 进行订单业务处理");
                
                // 订单日志的具体业务逻辑
                switch (logLevel) {
                    case "info":
                        handleOrderInfoLog(message);
                        break;
                    case "error":
                        handleOrderErrorLog(message);
                        break;
                    case "warning":
                        handleOrderWarningLog(message);
                        break;
                    default:
                        handleOrderGeneralLog(message);
                }
            }
            
            // 以下是可以扩展的业务方法
            private String getSystemFromRoutingKey(String routingKey) {
                // 从路由键中提取系统名称
                if (routingKey.contains(".")) {
                    return routingKey.substring(0, routingKey.lastIndexOf("."));
                }
                return routingKey;
            }
            
            private String getLogLevelFromRoutingKey(String routingKey) {
                // 从路由键中提取日志级别
                if (routingKey.contains(".")) {
                    return routingKey.substring(routingKey.lastIndexOf(".") + 1);
                }
                return "unknown";
            }
            
            private void sendErrorAlert(String message, String routingKey) {
                // 发送错误报警的具体实现
                System.out.println("   发送错误报警: " + routingKey);
            }
            
            private void recordErrorMetrics(String routingKey) {
                // 记录错误统计的具体实现
                System.out.println("   记录错误统计: " + routingKey);
            }
            
            private void triggerErrorProcess(String message) {
                // 触发错误处理流程的具体实现
                System.out.println("   触发错误处理流程");
            }
            
            private void handleOrderInfoLog(String message) {
                // 处理订单信息日志
                System.out.println("   处理订单信息日志 - 正常业务记录");
            }
            
            private void handleOrderErrorLog(String message) {
                // 处理订单错误日志
                System.out.println("   处理订单错误日志 - 紧急业务处理");
            }
            
            private void handleOrderWarningLog(String message) {
                // 处理订单警告日志
                System.out.println("   处理订单警告日志 - 注意潜在问题");
            }
            
            private void handleOrderGeneralLog(String message) {
                // 处理订单通用日志
                System.out.println("   处理订单通用日志");
            }
        };
        
        /**
         * 开始消费消息
         * 参数说明：
         * queue - 要监听的队列名称
         * autoAck - 是否自动确认消息（true表示自动确认）
         * callback - 消费者回调对象
         */
        channel.basicConsume(QUEUE_NAME, true, consumer);
        
        System.out.println("消费者1已启动，正在监听队列: " + QUEUE_NAME);
        System.out.println("等待接收以下类型的消息:");
        System.out.println("1. 所有错误日志 (#.error) - 例如: order.error, user.error, system.error");
        System.out.println("2. 订单系统所有日志 (order.*) - 例如: order.info, order.error, order.warning");
    }
}
```

**Consumer2：**

```java
package com.shiguang.rabbitmq.topic;

import com.rabbitmq.client.*;
import com.shiguang.rabbitmq.util.ConnectionUtil;

import java.io.IOException;

/**
 * 主题交换机模式 - 消费者2
 * 该消费者监听 topic_queue2 队列，专门处理所有两级路由键的日志消息
 * 绑定模式："*.*" - 匹配所有格式为 [系统].[级别] 的日志消息
 * 作为通用日志收集器，负责全面的日志记录和分析
 */
public class Consumer2 {
    // 定义队列名称常量 - 消费者2专用的队列
    static final String QUEUE_NAME = "topic_queue2";

    public static void main(String[] args) throws Exception {
        // 获取RabbitMQ连接
        Connection connection = ConnectionUtil.getConnection();

        // 创建频道
        Channel channel = connection.createChannel();

        /**
         * 声明队列
         * 参数说明：
         * queue - 队列名称，与生产者中定义的队列名称一致
         * durable - 是否持久化（true表示重启后队列依然存在）
         * exclusive - 是否排他（是否仅限此连接使用）
         * autoDelete - 是否自动删除（没有消费者时自动删除）
         * arguments - 其他参数
         */
        channel.queueDeclare(QUEUE_NAME, true, false, false, null);

        // 创建消费者实例，重写消息处理方法
        DefaultConsumer consumer = new DefaultConsumer(channel) {
            @Override
            public void handleDelivery(String consumerTag,
                                       Envelope envelope,
                                       AMQP.BasicProperties properties,
                                       byte[] body) throws IOException {
                /**
                 * 处理接收到的消息
                 * 参数说明：
                 * consumerTag - 消费者标签
                 * envelope - 包含交付相关信息（交换机、路由键、交付标签等）
                 * properties - 消息属性
                 * body - 消息内容（字节数组）
                 * 
                 * 此队列接收所有格式为 [系统].[级别] 的两级路由键消息
                 * 例如：order.info, user.error, system.warning 等
                 */
                String message = new String(body);
                String routingKey = envelope.getRoutingKey();

                System.out.println("消费者2接收到的消息内容: " + message);
                System.out.println("路由键: " + routingKey);
                System.out.println("队列2 消费者2 日志打印...");

                // 根据路由键进行日志分类处理
                processLogMessage(message, routingKey);
            }

            /**
             * 处理日志消息，根据系统类型和日志级别进行分类处理
             * @param message 消息内容
             * @param routingKey 路由键
             */
            private void processLogMessage(String message, String routingKey) {
                // 解析路由键，获取系统名称和日志级别
                String[] parts = routingKey.split("\\.");
                String system = parts.length > 0 ? parts[0] : "unknown";
                String level = parts.length > 1 ? parts[1] : "unknown";

                System.out.println(">>> [通用日志处理] 系统: " + system + ", 级别: " + level);

                // 统一的日志存储
                storeToCentralLog(message, routingKey);

                // 系统级别的统计分析
                recordSystemMetrics(system, level);

                // 根据系统类型进行特殊处理
                handleBySystemType(message, system, level);
                
                // 长期归档处理
                archiveLogForLongTerm(message, routingKey);
            }
            
            /**
             * 存储到中央日志系统
             */
            private void storeToCentralLog(String message, String routingKey) {
                System.out.println("   存储到中央日志系统: " + routingKey);
                // 实际实现可能包括：
                // - 写入Elasticsearch
                // - 存储到数据库
                // - 写入文件系统
            }
            
            /**
             * 记录系统指标统计
             */
            private void recordSystemMetrics(String system, String level) {
                System.out.println("   更新系统指标: " + system + " - " + level);
                // 实际实现可能包括：
                // - 更新Prometheus指标
                // - 记录到统计数据库
                // - 生成实时报表
            }
            
            /**
             * 根据系统类型进行特殊处理
             */
            private void handleBySystemType(String message, String system, String level) {
                switch (system) {
                    case "order":
                        handleOrderSystemLog(message, level);
                        break;
                    case "user":
                        handleUserSystemLog(message, level);
                        break;
                    case "payment":
                        handlePaymentSystemLog(message, level);
                        break;
                    case "inventory":
                        handleInventorySystemLog(message, level);
                        break;
                    default:
                        handleGeneralSystemLog(message, system, level);
                }
            }
            
            /**
             * 订单系统日志处理
             */
            private void handleOrderSystemLog(String message, String level) {
                System.out.println("   [订单系统] 业务指标统计和监控");
                // 订单相关的特殊处理：
                // - 订单量统计
                // - 订单成功率分析
                // - 订单业务流程监控
            }
            
            /**
             * 用户系统日志处理
             */
            private void handleUserSystemLog(String message, String level) {
                System.out.println("   [用户系统] 用户行为分析和统计");
                // 用户相关的特殊处理：
                // - 用户活跃度分析
                // - 用户行为模式分析
                // - 用户增长统计
            }
            
            /**
             * 支付系统日志处理
             */
            private void handlePaymentSystemLog(String message, String level) {
                System.out.println("   [支付系统] 支付成功率和安全监控");
                // 支付相关的特殊处理：
                // - 支付成功率统计
                // - 支付安全监控
                // - 交易流水记录
            }
            
            /**
             * 库存系统日志处理
             */
            private void handleInventorySystemLog(String message, String level) {
                System.out.println("   [库存系统] 库存变化和预警监控");
                // 库存相关的特殊处理：
                // - 库存变化追踪
                // - 库存预警分析
                // - 库存周转率计算
            }
            
            /**
             * 通用系统日志处理
             */
            private void handleGeneralSystemLog(String message, String system, String level) {
                System.out.println("   [通用系统] " + system + " 系统日志记录");
                // 其他系统的通用处理
            }
            
            /**
             * 长期日志归档
             */
            private void archiveLogForLongTerm(String message, String routingKey) {
                System.out.println("   长期日志归档处理");
                // 实际实现可能包括：
                // - 压缩存储到对象存储
                // - 转移到冷数据存储
                // - 生成归档索引
            }
        };
        
        /**
         * 开始消费消息
         * 参数说明：
         * queue - 要监听的队列名称
         * autoAck - 是否自动确认消息（true表示自动确认）
         * callback - 消费者回调对象
         */
        channel.basicConsume(QUEUE_NAME, true, consumer);
        
        System.out.println("消费者2已启动，正在监听队列: " + QUEUE_NAME);
        System.out.println("等待接收所有格式为 [系统].[级别] 的日志消息");
        System.out.println("示例: order.info, user.error, payment.warning, inventory.debug");
        System.out.println("职责: 通用日志收集、统计分析、长期存储");
    }
}
```

##### 执行效果
`topic_queue1`匹配规则满足：所有error级别日志存入数据库,所有order系统的日志存入数据库

`topic_queue2`则匹配所有消息

```java
channel.queueBind(queue1Name, EXCHANGE_NAME, "#.error");
channel.queueBind(queue1Name, EXCHANGE_NAME, "order.*");
channel.queueBind(queue2Name, EXCHANGE_NAME, "*.*");
```

我们先发送`order.info`规则的消息，执行并查看效果

```java
String body = "[所在系统：order][日志级别：info][日志内容: 订单生成,保存成功]";
channel.basicPublish(EXCHANGE_NAME, "order.info", null, body.getBytes());
System.out.println("body发送成功: " + body );
```

由于`topic_queue1`与`topic_queue2`均能匹配`order.info`规则，所以`Consumer1`与`Consumer2`均能接收到消息。

Consumer1：

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151858877-39e89b15-425f-42ea-ae76-40d01c2fc8fa.png)

Consumer2：

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151858963-ed0e0e2d-4b00-4de1-88aa-c096ac1191ca.png)

我们再发送`goods.info`这个规则的消息，清空Consumer日志，重新发送消息

```java
String body = "[所在系统：order][日志级别：info][日志内容: 订单生成,保存成功]";
channel.basicPublish(EXCHANGE_NAME, "order.info", null, body.getBytes());
System.out.println("body发送成功: " + body );

body = "[所在系统：goods][日志级别：info][日志内容: 商品发布成功]";
channel.basicPublish(EXCHANGE_NAME, "goods.info", null, body.getBytes());
System.out.println("body发送成功: " + body );
```

由于`topic_queue1`不能匹配`goods.info`规则，所以`Consumer1`只接收到一条消息，`Consumer2`接收到两条消息。

Consumer1：

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151859138-7e652be8-0418-4633-bb02-f936684ebf90.png)

Consumer2：

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151859144-459d32f0-8171-446e-b7d7-c0fcff90fac5.png)

我们继续追加`goods.error`这个规则的消息

```java
String body = "[所在系统：order][日志级别：info][日志内容: 订单生成,保存成功]";
channel.basicPublish(EXCHANGE_NAME, "order.info", null, body.getBytes());
System.out.println("body发送成功: " + body );

body = "[所在系统：goods][日志级别：info][日志内容: 商品发布成功]";
channel.basicPublish(EXCHANGE_NAME, "goods.info", null, body.getBytes());
System.out.println("body发送成功: " + body );

body = "[所在系统：goods][日志级别：error][日志内容: 商品发布失败]";
channel.basicPublish(EXCHANGE_NAME, "goods.error", null, body.getBytes());
System.out.println("body发送成功: " + body );
```

同理可知`Consumer1`只接收到两条消息，`Consumer2`接收到三条消息。

Consumer1：

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151859249-37114bd2-a2b5-44af-9937-c7508b9680fa.png)

Consumer2：

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151859240-145549ed-db6e-406d-ac9f-cb7c5e0668b6.png)

### 远程过程调用（RPC）
+ 远程过程调用，本质上是同步调用，和我们使用OpenFeign调用远程接口一样
+ 所以这不是典型的消息队列工作方式，我们不展开说明

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151859293-c1bb04c2-bb7c-438a-89ae-bb75efca7115.png)

### 工作模式小结
直接发送到队列：底层使用了默认交换机

经过交换机发送到队列

+ Fanout: 没有Routing key直接绑定队列
+ Direct: 通过Routing key绑定队列，消息发送到绑定的队列上 
    - 一个交换机绑定一个队列：定点发送
    - 一个交换机绑定多个队列：广播发送
+ Topic: 针对Routing key使用通配符

## Spring Boot 整合RabbitMQ
### 基本思路
+ 搭建环境
+ 基础设定：交换机名称、队列名称、绑定关系
+ 发送消息：使用`RabbitTemplate`
+ 接收消息：使用`@RabbitListener`注解

### 消费者操作步骤
#### 创建项目并导入依赖
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.shiguang</groupId>
  <artifactId>module03-springboot-consumer</artifactId>
  <version>1.0-SNAPSHOT</version>

  <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>3.1.5</version>
  </parent>

  <properties>
    <maven.compiler.source>17</maven.compiler.source>
    <maven.compiler.target>17</maven.compiler.target>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
  </properties>

  <dependencies>

    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <dependency>
      <groupId>org.projectlombok</groupId>
      <artifactId>lombok</artifactId>
    </dependency>

    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-amqp</artifactId>
    </dependency>
  </dependencies>

</project>
```

#### 创建配置文件
```yaml
spring:
  rabbitmq:
    host: 192.168.10.66
    port: 5672
    username: guest
    password: 123456
    virtual-host: /
logging:
  level:
    com.shiguang.mq.listener.MyMessageListener: info
```

#### 创建启动类
```java
package com.shiguang.mq;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;


@SpringBootApplication
public class RabbitMQConsumerMainType {
    public static void main(String[] args) {
        SpringApplication.run(RabbitMQConsumerMainType.class, args);
    }
}
```

#### MyMessageListener
```java
package com.shiguang.mq.listener;

import com.rabbitmq.client.Channel;
import lombok.extern.slf4j.Slf4j;
import org.springframework.amqp.core.Message;
import org.springframework.amqp.rabbit.annotation.Exchange;
import org.springframework.amqp.rabbit.annotation.Queue;
import org.springframework.amqp.rabbit.annotation.QueueBinding;
import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.stereotype.Component;

/**
 * Created By Shiguang On 2024/10/11 16:02
 */
@Component
@Slf4j
public class MyMessageListener {
    public static final String EXCHANGE_DIRECT = "exchange.direct.order";
    public static final String ROUTING_KEY = "order";
    public static final String QUEUE_NAME = "queue.order";

    // 写法一: 监听 + 在 RabbitMQ 服务器上创建交换机、队列、绑定关系
//    @RabbitListener(bindings = @QueueBinding(
//            value = @Queue(value = QUEUE_NAME, durable = "true"),
//            exchange = @Exchange(value = EXCHANGE_DIRECT),
//            key = {ROUTING_KEY}
//    ))
//    public void processMessage(String dataString, Message message, Channel channel) {
//        log.info("消费端接收到消息：{}", dataString);
//        System.out.println("消费端接收到消息：" + dataString);
//    }

    // 写法二: 只监听
    @RabbitListener(queues = QUEUE_NAME)
    public void processMessage(String dataString, Message message, Channel channel) {
        log.info("消费端接收到消息：{}", dataString);
        System.out.println("消费端接收到消息：" + dataString);
    }
}
```

#### 测试
启动服务，登录RabbitMQ管理界面查看交换机，消息队列是否创建成功并建立绑定关系

**交换机：**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151859429-dfa1e1a4-b99c-4006-8f51-34774c74e680.png)

**消息队列：**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151859421-883c275f-3d1e-42b0-9acc-290a77c8eeda.png)

**绑定关系：**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151859583-1f5cdbeb-ca30-47bc-8ce3-0316e4a0b7eb.png)

#### 图形化界面操作
**创建交换机：**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151859687-302eedb6-d925-41b9-b034-1c8e7c84f862.png)

**创建消息队列：**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151859841-69f8d689-18bc-4d66-8716-b779ddd2ac12.png)

**建立绑定关系：**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151859788-738afcb0-48f8-42fb-a124-ddeb1c157d6b.png)

添加后如下：

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151859780-416b597b-f2f4-42e1-b7ad-b01e7b6316e0.png)

### 生产者操作步骤
#### 创建项目并导入依赖
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.shiguang</groupId>
  <artifactId>modul04-springboot-producer</artifactId>
  <version>1.0-SNAPSHOT</version>

  <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>3.1.5</version>
  </parent>

  <properties>
    <maven.compiler.source>17</maven.compiler.source>
    <maven.compiler.target>17</maven.compiler.target>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
  </properties>

  <dependencies>

    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-test</artifactId>
    </dependency>

    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-amqp</artifactId>
    </dependency>
  </dependencies>

</project>
```

#### 创建配置文件
```yaml
spring:
  rabbitmq:
    host: 192.168.10.66
    port: 5672
    username: guest
    password: 123456
    virtual-host: /
```

创建启动类

```java
package com.shiguang.mq;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;


@SpringBootApplication
public class RabbitMQProducerMainType {
    public static void main(String[] args) {
        SpringApplication.run(RabbitMQProducerMainType.class, args);
    }
}
```

创建测试类

> 注意测试类包路径应与项目启动类所属包路径一致，否则@Autowired无法自动装配
>

```java
package com.shiguang.mq;

/**
 * Created By Shiguang On 2024/10/11 16:49
 */

import org.junit.jupiter.api.Test;
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;

@SpringBootTest
public class RabbitMQTest {

    public static final String EXCHANGE_DIRECT = "exchange.direct.order";
    public static final String ROUTING_KEY = "order";
    public static final String QUEUE_NAME = "queue.order";
    @Autowired
    private RabbitTemplate rabbitTemplate;

    @Test
    public void test01SendMessage(){
        String message = "Hello Rabbit!!";
        rabbitTemplate.convertAndSend(EXCHANGE_DIRECT,ROUTING_KEY,message);
    }

}
```

#### 测试
执行测试代码，查看后台监控，有一条消息待消费

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151859865-ade48988-67a8-451b-81cf-a13099ef126e.png)

启动消费者服务进行消费

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151860068-8b99d56a-223e-4cbe-a4f8-5d8cb0a628b2.png)

## 消息可靠性投递
### 问题场景及解决方案
#### 问题场景
下单操作的正常流程如下图所示

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151860183-dcae4422-0916-479a-825f-3ef9e7c93620.png)

故障情况1：消息没有发送到消息队列上  
后果：消费者拿不到消息，业务功能缺失，数据错误

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151860221-7fa39fa6-0d7c-423d-a97e-45dc2accacb3.png)

故障情况2：消息成功存入消息队列，但是消息队列服务器宕机了  
原本保存在**内存中的消息**也**丢失**了  
即使服务器重新启动，消息也找不回来了  
后果：消费者拿不到消息，业务功能缺失，数据错误

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151860222-6db73dc4-e0fe-4a7c-8628-308ccab45cf1.png)

故障情况3：消息成功存入消息队列，但是消费端出现问题，例如：宕机、抛异常等等

后果：业务功能缺失，数据错误

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151860239-c51effad-8e51-43c8-8d2a-00ac99e2816a.png)

#### 解决方案
故障情况1：消息没有发送到消息队列

+ 解决思路A：在**生产者端**进行确认，具体操作中我们会分别针对**交换机**和**队列**来确认  
如果没有成功发送到消息队列服务器上，那就可以尝试重新发送
+ 解决思路B：为目标交换机指定**备份交换机**，当目标交换机投递失败时，把消息投递至  
备份交换机

故障情况2：消息队列服务器宕机导致内存中消息丢失

+ 解决思路：**消息持久化**到硬盘上，哪怕服务器重启也不会导致消息丢失

故障情况3：消费端宕机或抛异常导致消息没有成功被消费

+ 消费端消费消息**成功**，给服务器返回**ACK信息**，然后消息队列删除该消息
+ 消费端消费消息**失败**，给服务器端返回**NACK信息**，同时把消息恢复为**待消费**的状态，  
这样就可以再次取回消息，**重试**一次（当然，这就需要消费端接口支持幂等性）

### 故障情况1
#### 生产者端实现
##### 创建项目并导入依赖
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.shiguang</groupId>
  <artifactId>module05-confirm-producer</artifactId>
  <version>1.0-SNAPSHOT</version>

  <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>3.1.5</version>
  </parent>

  <properties>
    <maven.compiler.source>17</maven.compiler.source>
    <maven.compiler.target>17</maven.compiler.target>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
  </properties>

  <dependencies>

    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-test</artifactId>
    </dependency>

    <dependency>
      <groupId>org.projectlombok</groupId>
      <artifactId>lombok</artifactId>
    </dependency>

    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-amqp</artifactId>
    </dependency>
  </dependencies>

</project>
```

##### 主启动类
```java
package com.shiguang.mq;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;


@SpringBootApplication
public class RabbitMQProducerMainType {
    public static void main(String[] args) {
        SpringApplication.run(RabbitMQProducerMainType.class, args);
    }
}
```

##### 配置文件
> 注意：publisher-confirm-type和publisher-returns是两个必须要增加的配置，如果没有则本节功能不生效
>

```yaml
spring:
  rabbitmq:
    host: 192.168.10.66
    port: 5672
    username: guest
    password: 123456
    virtual-host: /
    publisher-confirm-type: CORRELATED #交换机的确认
    publisher-returns: true #队列的确认
logging:
  level:
    com.shiguang.mq.config.MQProducerAckConfig: info
```

##### 配置类
**目标**：首先我们需要声明回调函数来接收RabbitMQ服务器返回的确认信息：

| 方法名 | 方法功能 | 所属接口 | 接口所属类 |
| --- | --- | --- | --- |
| confirm() | 确认消息是否发送到交换机 | ConfirmCallback | RabbitTemplate |
| returnedMessage() | 确认消息是否发送到队列 | ReturnsCallback | RabbitTemplate |


然后，就是对RabbitTemplate的功能进行增强，因为回调函数所在对象必须设置到RabbitTemplate对象中才能生效  
原本RabbitTemplate对象并没有生产者端消息确认的功能，要给它设置对应的组件才可以。  
而设置对应的组件，需要调用RabbitTemplate对象下面两个方法：

| 设置组件调用的方法 | 所需对象类型 |
| --- | --- |
| setConfirmCallback() | ConfirmCallback接口类型 |
| setReturnCallback() | ReturnCallback:接口类型 |


代码如下：

> ① 要点1  
加@Component注解，加入IOC容器（@Configuration已经包含了@Component）  
② 要点2  
配置类自身实现ConfirmCallback、ReturnCallbacki这两个接口，然后通过this指针把配置类的对象设置到RabbitTemplate对象中。  
操作封装到了一个专门的void init()方法中。  
为了保证这个void init()方法在应用启动时被调用，我们使用@PostConstruct注解来修饰这个方法。  
关于@PostConstruct注解大家可以参照以下说明：
>
> @PostConstruct注解是**java中的一个标准注解**，它用于指定在**对象创建之后立即执行**的方法。当使用依赖注入（如Spring框架）或者其他方式创建对象时，@PostConstruct注解可以确保在对象完全初始化之后，执行相应的方法。
>
> 使用@PostConstructi注解的方法必须满足以下条件：
>
> 1. 方法不能有任何参数
> 2. 方法必须是非静态的
> 3. 方法不能返回任何值。
>
> 当容器实例化一个带有@PostConstruct注解的Bean时，它会在**调用构造函数之后**，并在**依赖注入完成之前**调用被@PostConstruct注解标记的方法。这样，我们可以在该方法中进行一些初始化操作，比如读取配置文件、建立数据库连接等。
>

```java
package com.shiguang.mq.config;

import jakarta.annotation.PostConstruct;
import lombok.extern.slf4j.Slf4j;
import org.springframework.amqp.core.ReturnedMessage;
import org.springframework.amqp.rabbit.connection.CorrelationData;
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Configuration;

/**
 * Created By Shiguang On 2024/10/11 19:48
 */

@Configuration
@Slf4j
public class RabbitConfig implements RabbitTemplate.ConfirmCallback, RabbitTemplate.ReturnsCallback {
    @Autowired
    private RabbitTemplate rabbitTemplate;

    @PostConstruct
    public void initRabbitTemplate(){
        rabbitTemplate.setConfirmCallback(this);
        rabbitTemplate.setReturnsCallback(this);
    }

    /**
     * 消息发送到交换机成功或失败时调用这个方法
     *
     * @param correlationData 用于关联消息的唯一标识符
     * @param ack             表示消息是否被成功确认
     * @param cause           如果消息确认失败，这里会包含失败的原因
     */
    @Override
    public void confirm(CorrelationData correlationData, boolean ack, String cause) {
        log.info("confirm() 回调函数打印 CorrelationData: " + correlationData);
        log.info("confirm() 回调函数打印 ack: " + ack);
        log.info("confirm() 回调函数打印 cause: " + cause);
    }

    /**
     * 当消息无法路由到队列时调用这个方法
     *
     * @param returnedMessage 包含无法路由的消息的详细信息
     */
    @Override
    public void returnedMessage(ReturnedMessage returnedMessage) {
        log.info("returnedMessage() 回调函数 消息主体: " + new String(returnedMessage.getMessage().getBody()));
        log.info("returnedMessage() 回调函数 应答码: " + returnedMessage.getReplyCode());
        log.info("returnedMessage() 回调函数 描述: " + returnedMessage.getReplyText());
        log.info("returnedMessage() 回调函数 消息使用的交换器 exchange: " + returnedMessage.getExchange());
        log.info("returnedMessage() 回调函数 消息使用的路由键 routing: " + returnedMessage.getRoutingKey());
    }
}
```

##### 测试类
```java
package com.shiguang.mq;

import org.junit.jupiter.api.Test;
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;

/**
 * Created By Shiguang On 2024/10/11 20:16
 */

@SpringBootTest
public class RabbitMQTest {
    public static final String EXCHANGE_DIRECT = "exchange.direct.order";
    public static final String ROUTING_KEY = "order";

    @Autowired
    private RabbitTemplate rabbitTemplate;

    @Test
    public void test01SendMessage(){
        String message = "Message Confirm Test !!";
        rabbitTemplate.convertAndSend(EXCHANGE_DIRECT,ROUTING_KEY,message);
    }
}
```

##### 测试
正常执行测试代码，查看日志输出，ack为`true`，cause为`null`

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151860529-ac73c4da-d7d0-496a-a778-a9259c5bbc74.png)

调整交换机名称，故意使其发送失败

```java
@Test
public void test01SendMessage() {
    String message = "Message Confirm Test !!";
    //        rabbitTemplate.convertAndSend(EXCHANGE_DIRECT,ROUTING_KEY,message);
    rabbitTemplate.convertAndSend(EXCHANGE_DIRECT + "~", ROUTING_KEY, message);
}
```

重新执行并查看日志输出，ack为`false`，cause有相应错误原因

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151860543-c223db47-3181-4109-9f45-03f6dc8d3126.png)

调整路由键名称，故意使其无法匹配

```java
@Test
public void test01SendMessage() {
    String message = "Message Confirm Test !!";
    //        rabbitTemplate.convertAndSend(EXCHANGE_DIRECT,ROUTING_KEY,message);
    //        rabbitTemplate.convertAndSend(EXCHANGE_DIRECT + "~", ROUTING_KEY, message);
    rabbitTemplate.convertAndSend(EXCHANGE_DIRECT, ROUTING_KEY + "~", message);
}
```

重新执行并查看日志输出，打印了`returnedMessage()`回到函数日志

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151860645-ae141f75-0506-46e6-a0b0-8c7af6e8a3b1.png)

#### 备份交换机实现
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151860685-019eda0c-84bb-45d3-8e5c-3593a7f50290.png)

**1、创建备份交换机**

类型必须为`fanout`，因为消息从目标交换机转至备份交换机时是没有路由键的，只能通过广播的方式查找队列。

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151860628-c727469e-8197-4c1a-b7fc-5a30576f0f4c.png)

**2、创建队列**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151860827-78a0de4f-9f8b-4f61-b888-781765af0e5f.png)

**3、交换机绑定队列**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151860877-3c5826a5-9109-4183-a780-550d219c3b87.png)

**4、执行目标交换机的备份交换机**

由于交换机创建后参数无法修改，所以需要将原来的目标删除重新创建并执行备份交换机

删除原来的目标交换机：

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151860988-c903fb5d-8f5b-49d4-8d0c-4826ec8b2f08.png)

重新创建目标交换机：

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151860978-64a5ba92-74e1-4db7-8c34-5121c2ce1e1b.png)

队列重新绑定交换机：

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151860982-a87dd490-fd6b-4062-9b1a-79bc04f5d83f.png)

**5、重新执行测试**

```java
@Test
public void test01SendMessage() {
    String message = "Message Confirm Test !!";
    //        rabbitTemplate.convertAndSend(EXCHANGE_DIRECT,ROUTING_KEY,message);
    //        rabbitTemplate.convertAndSend(EXCHANGE_DIRECT + "~", ROUTING_KEY, message);
    rabbitTemplate.convertAndSend(EXCHANGE_DIRECT, ROUTING_KEY + "~", message);
```

测试结果：ack为`true`

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151861163-02d93ba5-1908-4185-835f-5f8adaf64e7b.png)

`queue.test.backup`有一条消息待消费  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151861242-5cfed210-f605-4a8f-82b2-b9f1ea539ecc.png)

### 故障情况2
默认情况下，RabbitMQ服务宕机后，消息会丢失吗?

我们手动重启下RabbitMQ服务，然后查看消息消费情况

```plain
docker restart rabbitmq
```

原来有一条消息待消费

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151861307-40871c9c-d181-4f76-a5a4-85391805618e.png)

重启后重新查看，发现带消费消息从0条转变为1条，我们并未重新发送消息，但消息并未丢失

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151861393-30a9605b-20cf-4b6d-9dd7-6bf7c310c5c3.png)

其实默认情况下，RabbitMQ是支持持久化数据的，重启后会将保存到磁盘的数据重新加载到内存中

我们可以查看下`@RabbitListener` 注解的源码，找到`Queue`这个接口

```plain
Queue[] queuesToDeclare() default {};
```

可以看到，`durable()`和 `autoDelete()`虽然默认值都为空，但源码注释中有说明，默认是支持持久化但是并不会自动删除的。

```java
/**
	 * Specifies if this queue should be durable.
	 * By default if queue name is provided it is durable.
	 * @return true if the queue is to be declared as durable.
	 * @see org.springframework.amqp.core.Queue#isDurable()
	 */
String durable() default "";

/**
	 * Specifies if this queue should be auto deleted when not used.
	 * By default if queue name is provided it is not auto-deleted.
	 * @return true if the queue is to be declared as auto-delete.
	 * @see org.springframework.amqp.core.Queue#isAutoDelete()
	 */
String autoDelete() default "";
```

### 故障情况3
#### 消费者端实现
##### 创建项目并导入依赖
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.shiguang</groupId>
  <artifactId>module06-confirm-consumer</artifactId>
  <version>1.0-SNAPSHOT</version>

  <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>3.1.5</version>
  </parent>

  <properties>
    <maven.compiler.source>17</maven.compiler.source>
    <maven.compiler.target>17</maven.compiler.target>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
  </properties>

  <dependencies>

    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <dependency>
      <groupId>org.projectlombok</groupId>
      <artifactId>lombok</artifactId>
    </dependency>

    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-amqp</artifactId>
    </dependency>
  </dependencies>
</project>
```

##### 主启动类
```java
package com.shiguang.mq;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class RabbitMQConsumerMainType {
    public static void main(String[] args) {
        SpringApplication.run(RabbitMQConsumerMainType.class, args);
    }
}
```

##### 配置文件
```yaml
spring:
  rabbitmq:
    host: 192.168.10.66
    port: 5672
    username: guest
    password: 123456
    virtual-host: /
    listener:
      simple:
        acknowledge-mode: manual  # 把消息确认模式改为手动确认
logging:
  level:
    com.shiguang.mq.listener.MyMessageListener: info
```

##### Listener
> channel.basicNack与channel.basicReject的区别
>
> channel.basicReject(long deliveryTag, boolean requeue)
>
> channel.basicReject比channel.basicNack少了个是否批量操作的参数`multiple`，不能控制是否批量操作
>

```java
package com.shiguang.mq.listener;

import com.rabbitmq.client.Channel;
import lombok.extern.slf4j.Slf4j;
import org.springframework.amqp.core.Message;
import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.stereotype.Component;

import java.io.IOException;

@Component
@Slf4j
public class MyMessageListener {
    public static final String QUEUE_NAME = "queue.order";

    @RabbitListener(queues = QUEUE_NAME)
    public void processMessage(String dataString, Message message, Channel channel) throws IOException {
        // 获取当前消息的唯一标识
        long deliveryTag = message.getMessageProperties().getDeliveryTag();
        try {
            // 核心操作
            log.info("消费端接收到消息：{}", dataString);
            // 核心操作成功,返回 ACK 信息
            // deliveryTag: 消息的唯一标识,64 位的长整型,消息往消费端投递时,会分配一个唯一的 deliveryTag 值
            channel.basicAck(deliveryTag, false);
        } catch (Exception e) {
            // 获取当前消息是否是重复投递的,true 说明当前消息已经重试过一次了, false 说明当前消息是第一次投递
            Boolean redelivered = message.getMessageProperties().getRedelivered();

            // 核心操作失败,返回 NACK 信息
            // requeue: 是否重新入队,true 表示重新入队, false 表示丢弃
            if (redelivered){
                // 如果当前消息已经是重复投递的,则说明此前已经重试过一次了,则不再重试过了,直接丢弃
                channel.basicNack(deliveryTag, false, false);
            }else {
                // 如果当前消息不是重复投递的,则说明此前没有重试过一次,则重试过一次,重新入队
                channel.basicNack(deliveryTag, false, true);
            }

            throw new RuntimeException(e);
        }
    }
}
```

##### 消息确认相关方法参数说明
**1、delivery Tag: 交付标签机制**  
消费端把消息处理结果ACK、NACK、Reject等返回给Broker之后，Broker需要对对应的消息执行后续操作，例如删除消息、重新排队或标记为死信等等。那么Broker就必须知道它现在要操作的消息具体是哪一条。而delivery Tag作为消息的唯一标识就很好的满足了这个需求。

提问：如果交换机是Fanout模式，同一个消息广播到了不同队列，delivery Tag会重复吗？

答：不会，deliveryTag在Broker范围内唯一

思考：更新购物车的微服务消费了消息返回ACK确认信息，然后Broker删除了消息，进而导致更新库存  
更新积分的功能拿不到消息一这种情况会发生吗？

**2、multiple: 是否批量处理**

multiple为 `true` 时，采用批量处理

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151861529-4bcc2959-5b59-495c-804d-7fb4913d8637.png)

multiple为`false `时，进行单独处理

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151861607-d0bb9bd3-63fd-4613-bf19-28506ced7df4.png)

由于批量操作可能导致误操作，所以一般将`multiple` 设为`false`

**3、requeue：是否重新入队**

true 表示重新入队, false 表示丢弃

##### 测试
**1、以Debug模式启动Consumer服务**

**2、在图形化界面生成一条消息**

找到`exchange.direct.order`交换机，然后手动发布一条消息

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151861519-d2793bd1-d3a1-4563-a65b-aca98aff89c3.png)

消息发布成功，Debug进入到方法内部

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151861934-d188e744-7621-44bd-b71f-5612df151835.png)

**3、再查看**`**queue.order**`**队列情况**

发现消息已经被消费尚未ACK确认

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151861628-211f3fc0-1c59-4896-802d-44116c560eaf.png)

**4、消费端正常放行，返回ACK进行确认**

再次查看队列情况

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151861883-8dd6e0b0-8809-481b-8958-eb78b00d56a4.png)

接下来我们模拟异常场景，修改代码，手动打印 `1/0`使程序出错，重启服务

```java
log.info("消费端接收到消息：{}", dataString);
System.out.println(1/0);
```

**1、重新发布一条消息**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151861996-b1b8f98d-10de-49a8-aafb-5bd4480008b1.png)

**2、debug逐条执行，观察运行情况**

出现异常被catch捕获，此时 `redelivered `的值为`false`

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151862187-59388885-6f83-40d5-9154-9078bc8e0e39.png)

继续执行，方法进入else ，重新放入队列

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151861987-8f3c91bb-dba3-4bdb-9588-94fe5844778d.png)

放行，此时消息仍是待确认

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151862297-a10ada06-81ec-4391-8510-75fabf8b819c.png)

重新进入Debug，继续逐条执行，这次`redelivered `的值为`true`，不再重试，直接丢弃

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151862334-0d37a3b0-50de-4be6-94ac-3bdc0e890004.png)

放行，此时再查看队列情况

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151862334-ab67c627-bed4-452f-9b92-12d938be81b2.png)

## 消费端限流
消费端限流可以实现削峰减谷的作用，假设消息总量为1万条，如果一次性取出所有消息会导致消费端并发压力过大，我们可以限制**每次最多**从队列取出1000条消息，这样就可以对消费端进行很好的保护。

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151862348-1dfe966b-286f-4389-87cb-98e8dd76f2e0.png)

实现也比较简单，只需添加`prefetch`参数即可

先观察下默认情况下是如何处理的

**1、我们重写一个测试方法，生产端发布100条消息**

```java
@Test
public void test02SendMessage() {
    for (int i = 0; i < 100; i++) {
        String message = "Test Rrefetch!!" + i;
        rabbitTemplate.convertAndSend(EXCHANGE_DIRECT, ROUTING_KEY, message);
    }
}
```

消息发布后查看下队列情况

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151862511-1f38c872-f10f-4e03-a893-ea2d3b051924.png)

2、消费端Listener注释掉原来的方法，新增一个方法进行处理

```java
@RabbitListener(queues = QUEUE_NAME)
public void processMessage(String dataString, Message message, Channel channel) throws IOException, InterruptedException {
    // 核心操作
    log.info("消费端接收到消息：{}", dataString);

    TimeUnit.SECONDS.sleep(1); //延迟 1 秒

    channel.basicAck(message.getMessageProperties().getDeliveryTag(), false);
}
```

**3、运行消费端服务并查看队列情况**

观察发现 `Ready`数量直接从`100`变为`0`，`Unacked`和`Total`随着消息被消费端消费逐渐减少，说明消费时一次性取出队列中的所有消息，然后逐条消费。

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151862638-838dd421-b843-42a3-9b19-cdd2a5b5d5a1.png)

接下来我们限制每次从队列中获取的数量并观察队列运行情况

**1、添加配置，设置每次从队列中获取消息的数量**

```yaml
spring:
  rabbitmq:
    listener:
      simple:
        prefetch: 1 # 每次只消费一个消息
```

**2、重新发布消息以及重启消费端服务并观察队列运行情况**

我们可以看到`Ready`数量每次变化减`5`，这是因为图形化界面每`5`秒刷新一次

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151862696-77ef8233-ab3b-437d-be96-b7a4f589fe40.png)

## 消息超时
给消息设定一个过期时间，超过这个时间没有被取走的消息就会被删除  
我们可以从两个层面来给消息设定过期时间：

+ 队列层面：在队列层面设定消息的过期时间，并不是队列的过期时间。意思是这个队列中的消息全部使用同一个过期时间。
+ 消息本身：给具体的某个消息设定过期时间
+ 如果两个层面都做了设置，那么哪个时间短，哪个生效

### 测试
#### 给队列设置超时时间
**1、创建交换机和队列并建立绑定关系**

交换机：

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151862683-2ee7ad57-ee70-4e29-ae71-3c3c809ff014.png)

队列：

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151862685-c844a408-8aac-485d-b8d0-671daec5d77a.png)

交换机绑定队列：

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151862755-fe92514b-8e6c-421d-a1f0-613a7a165be6.png)

**2、新增测试方法并执行测试**

```java
public static final String EXCHANGE_TIMEOUT = "exchange.test.timeout";
public static final String ROUTING_KEY_TIMEOUT = "routing.key.test.timeout";

@Test
public void test03SendMessage() {
    String message = "Test Timeout!!";
    rabbitTemplate.convertAndSend(EXCHANGE_TIMEOUT, ROUTING_KEY_TIMEOUT, message);
}
```

此时观察队列情况，发现`Total`数量从`0`变为`1`，而我们并未运行消费端进行消费，这是因为我们给队列设置了过期时间，队列内的消息超出过期时间后被丢弃

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151862926-891f6683-209b-4493-9eff-95264fc0e4a0.png)

#### 给消息设置超时时间
**1、删除原来的队列并重新创建，不设置超时时间**

队列：

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151863059-d5b291bf-c7d3-4465-a66b-10ec08b3a5da.png)

重新绑定：

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151863083-025ca542-d66c-4bb8-8d96-a2fddc1e914c.png)

2、新增测试方法，添加后置处理器对象参数

```java
@Test
public void test04SendMessage() {

    // 创建消息后置处理器对象
    MessagePostProcessor processor = message -> {
        // 设置消息的过期时间为 7 秒
        message.getMessageProperties().setExpiration("7000");
        return message;
    };

    String message = "Test Timeout!!";

    rabbitTemplate.convertAndSend(EXCHANGE_TIMEOUT, ROUTING_KEY_TIMEOUT, message,processor);
}
```

**3、设置**`**Ack Mode**`**为**`**Automatic ack**`

这样消息处理失败不会重新加入队列

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151863076-43001150-9832-4635-a304-194435cdd5e5.png)

**4、执行测试方法并观察队列情况**

消息超出超时时间后被清除

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151863081-319086c9-d959-41d0-8a1e-79fe859e3474.png)

## 死信和死信队列
概念：当一个消息无法被消费，它就变成了死信。  
死信产生的原因大致有下面三种：

+ 拒绝：消费者拒接消息，basicNack(/basicReject(),并且不把消息重新放入原目标队列，requeue=false
+ 溢出：队列中消息数量到达限制。比如队列最大只能存储10条消息，且现在已经存储了10条，此时如果再发送一条消息进来，根据先进先出原则，队列中最早的消息会变成死信
+ 超时：消息到达超时时间未被消费

死信的处理方式大致有下面三种：

+ 丢弃：对不重要的消息直接丢弃，不做处理
+ 入库：把死信写入数据库，日后处理
+ 监听：消息变成死信后进入死信队列，我们专门设置消费端监听死信队列，做后续处理（通常采用）

### 测试相关准备
#### 创建死信交换机和死信队列
+ 死信交换机: `exchange.dead.letter.video`
+ 死信队列：`queue.dead.letter.video`
+ 死信路由键：`routing.key.dead.letter.video`

#### 创建正常交换机和正常队列
> 注意：一定要注意正常队列有诸多限定和设置，这样才能让无法处理的消息进入死信交换机
>
> x-dead-letter-exchange: 关联的死信交换机
>
> x-dead-letter-routing-key：关联的死信路由键
>
> x-max-length：队列最大容量长度
>
> x-message-ttl：队列超时时间
>

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151863273-30c30076-03b7-4795-8cfe-1f1653643c60.png)

+ 正常交换机：`exchange.normal.video`
+ 正常队列: `queue.normal.video`
+ 正常路由键：`routing.key.normal.video`

#### java代码中的相关常量声明
```java
public static final String EXCHANGE_NORMAL = "exchange.normal.video";
public static final String EXCHANGE_DEAD_LETTER = "exchange.dead.letter.video";

public static final String ROUTING_KEY_NORMAL = "routing.key.normal.video";
public static final String ROUTING_KEY_DEAD_LETTER = "routing.key.dead.letter.video";

public static final String QUEUE_NORMAL = "queue.normal.video";
public static final String QUEUE_DEAD_LETTER = "queue.dead.letter.video";
```

### 消费端拒收消息
#### 发送消息的代码
> 也可直接在图形化界面操作
>

```java
@Test
public void testSendRejectMessage() {
    rabbitTemplate.convertAndSend(EXCHANGE_NORMAL, ROUTING_KEY_DEAD_LETTER, "测试死信情况1:消息被拒绝");
}
```

#### 接收消息的代码
> 由于监听正常队列的方法一定会拒绝并且不会重新加入队列，那么队列中的消息就会成为死信并加入到死信队列中，死信队列正常返回。
>

① 监听正常队列

```java
@RabbitListener(queues = QUEUE_NORMAL)
public void processNormalMessage(Message message, Channel channel) throws IOException {

    log.info("★[normal] 消息接收到,但我拒绝。");
    channel.basicReject(message.getMessageProperties().getDeliveryTag(), false);
}
```

② 监听死信队列

```java
@RabbitListener(queues = QUEUE_DEAD_LETTER)
public void processDeadMessage(String dataString, Message message, Channel channel) throws IOException {

    log.info("★[dead letter] dataString = " + dataString);
    log.info("★[dead1 etter] 我是死信监听方法,我接收到了死信消息");
    channel.basicAck(message.getMessageProperties().getDeliveryTag(), false);
}
```

#### 执行结果
**1、正常队列发布消息**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151863436-dd2107a3-2aed-4bfa-b649-365416c2a948.png)

**2、重启消费端服务**

后台日志输出情况

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151863480-54b8ac6e-ba5e-401e-af5b-17ecad4419cc.png)

**3、观察队列情况**

正常队列：

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151863498-acb00861-c334-4433-8066-4a27639d575f.png)

死信队列：

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151863499-2f729371-030c-4c41-b8ce-d434347e0696.png)

### 消费数量超过队列容量极限
#### 发送消息的代码
```java
@Test
public void testSendMultiMessage() {
    for (int i = 0; i < 20; i++) {
        rabbitTemplate.
        convertAndSend(
            EXCHANGE_NORMAL,
            ROUTING_KEY_NORMAL,
            "测试死信情况2:数量超过队列最大容量" + i);
    }
}
```

#### 接收消息的代码
#### 执行效果
**1、停止消费端服务，批量发送20条消息**

**2、观察队列情况**

正常队列：

由于我们设置的最容量为`10`，所以我们最多接收`10`条消息，超出设定的超时时间后消息被废弃，数量变为`0`

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151863605-5f780015-25b1-4977-a579-36dcb95cfbc1.png)

死信队列：

由于我们设置的最大容量为`10`，消息成为死信后每`10`条消息为一个批次加入死信队列

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151863854-aea2f151-db3d-4b7d-80d5-eb7e0d6d1a82.png)

此时我们启动消费端服务，观察日志输出情况，可以发现都是`dead`级别的日志，因为此时队列里的所有消息都变为死信了。

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151863932-40e20427-f18a-4ab0-8157-a687618756ab.png)

### 消息超时未消费
#### 发送消息的代码
> 由于我们设置的队列最大容量为10，为了避免由于溢出产生死信的影响，我们发送小于10条的数据
>

```java
@Test
public void testSendDelayMessage() {
    for (int i = 0; i < 8; i++) {
        rabbitTemplate.
        convertAndSend(
            EXCHANGE_NORMAL,
            ROUTING_KEY_NORMAL,
            "测试死信情况3:消息超时未消费" + i);
    }
}
```

#### 执行效果
**1、停止消费端服务，发送消息**

**2、查看队列情况**

正常队列：

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151863901-edb4fd50-e8a7-41e9-91f1-8e7af5b9529c.png)

死信队列：

死信队列从原始的`30`条数量增至`38`条，我们发送的`8`条数据因为超时未消费加入到死信队列中

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151863896-48f99988-3cc0-4b41-ac89-70ba26a04d43.png)

## 延迟队列
### 业务场景
在限定时间内进行支付，否则订单自动取消

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151863925-2901f472-7c43-4ddb-a1b1-7ec560fb9b38.png)

### 实现思路
#### **方案1：设置消息超时时间 + 死信队列**
> 可参考上文介绍，不再演示
>

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151864356-47b54187-cb72-42ab-9b9c-30f51013ab80.png)

#### **方案2：给RabbitMQ安装插件**
##### 插件介绍
官网地址：https:/github.com/rabbitmq/rabbitmq-delayed-message-exchange  
延迟极限：最多两天

##### 安装插件
##### 确定卷映射目录
```plain
docker inspect rabbitmq
```

运行结果：

```json
[
  {
    "Id": "3767efc3f46e05b63dbb6a244f2f5a850a60febe52cc1bdf96e75f5449d7979e",
    "Created": "2024-10-10T14:41:39.651931938Z",
    "Path": "docker-entrypoint.sh",
    "Args": [
      "rabbitmq-server"
    ],
    "State": {
      "Status": "running",
      "Running": true,
      "Paused": false,
      "Restarting": false,
      "OOMKilled": false,
      "Dead": false,
      "Pid": 2671,
      "ExitCode": 0,
      "Error": "",
      "StartedAt": "2024-10-12T01:18:16.845798068Z",
      "FinishedAt": "2024-10-12T01:15:50.852558669Z"
    },
    "Image": "sha256:c7383e9ad93d65dea7219907c8ac08e6f8cdad481f17c78b3864f29b2cd50a7b",
    "ResolvConfPath": "/var/lib/docker/containers/3767efc3f46e05b63dbb6a244f2f5a850a60febe52cc1bdf96e75f5449d7979e/resolv.conf",
    "HostnamePath": "/var/lib/docker/containers/3767efc3f46e05b63dbb6a244f2f5a850a60febe52cc1bdf96e75f5449d7979e/hostname",
    "HostsPath": "/var/lib/docker/containers/3767efc3f46e05b63dbb6a244f2f5a850a60febe52cc1bdf96e75f5449d7979e/hosts",
    "LogPath": "/var/lib/docker/containers/3767efc3f46e05b63dbb6a244f2f5a850a60febe52cc1bdf96e75f5449d7979e/3767efc3f46e05b63dbb6a244f2f5a850a60febe52cc1bdf96e75f5449d7979e-json.log",
    "Name": "/rabbitmq",
    "RestartCount": 0,
    "Driver": "overlay2",
    "Platform": "linux",
    "MountLabel": "",
    "ProcessLabel": "",
    "AppArmorProfile": "",
    "ExecIDs": null,
    "HostConfig": {
      "Binds": [
        "rabbitmq-plugin:/plugins"
      ],
      "ContainerIDFile": "",
      "LogConfig": {
        "Type": "json-file",
        "Config": {}
      },
      "NetworkMode": "bridge",
      "PortBindings": {
        "15672/tcp": [
          {
            "HostIp": "",
            "HostPort": "15672"
          }
        ],
        "5672/tcp": [
          {
            "HostIp": "",
            "HostPort": "5672"
          }
        ]
      },
      "RestartPolicy": {
        "Name": "no",
        "MaximumRetryCount": 0
      },
      "AutoRemove": false,
      "VolumeDriver": "",
      "VolumesFrom": null,
      "ConsoleSize": [
        49,
        108
      ],
      "CapAdd": null,
      "CapDrop": null,
      "CgroupnsMode": "host",
      "Dns": [],
      "DnsOptions": [],
      "DnsSearch": [],
      "ExtraHosts": null,
      "GroupAdd": null,
      "IpcMode": "private",
      "Cgroup": "",
      "Links": null,
      "OomScoreAdj": 0,
      "PidMode": "",
      "Privileged": false,
      "PublishAllPorts": false,
      "ReadonlyRootfs": false,
            "SecurityOpt": null,
            "UTSMode": "",
            "UsernsMode": "",
            "ShmSize": 67108864,
            "Runtime": "runc",
            "Isolation": "",
            "CpuShares": 0,
            "Memory": 0,
            "NanoCpus": 0,
            "CgroupParent": "",
            "BlkioWeight": 0,
            "BlkioWeightDevice": [],
            "BlkioDeviceReadBps": [],
            "BlkioDeviceWriteBps": [],
            "BlkioDeviceReadIOps": [],
            "BlkioDeviceWriteIOps": [],
            "CpuPeriod": 0,
            "CpuQuota": 0,
            "CpuRealtimePeriod": 0,
            "CpuRealtimeRuntime": 0,
            "CpusetCpus": "",
            "CpusetMems": "",
            "Devices": [],
            "DeviceCgroupRules": null,
            "DeviceRequests": null,
            "MemoryReservation": 0,
            "MemorySwap": 0,
            "MemorySwappiness": null,
            "OomKillDisable": false,
            "PidsLimit": null,
            "Ulimits": [],
            "CpuCount": 0,
            "CpuPercent": 0,
            "IOMaximumIOps": 0,
            "IOMaximumBandwidth": 0,
            "MaskedPaths": [
                "/proc/asound",
                "/proc/acpi",
                "/proc/kcore",
                "/proc/keys",
                "/proc/latency_stats",
                "/proc/timer_list",
                "/proc/timer_stats",
                "/proc/sched_debug",
                "/proc/scsi",
                "/sys/firmware",
                "/sys/devices/virtual/powercap"
            ],
            "ReadonlyPaths": [
                "/proc/bus",
                "/proc/fs",
                "/proc/irq",
                "/proc/sys",
                "/proc/sysrq-trigger"
            ]
        },
        "GraphDriver": {
            "Data": {
                "LowerDir": "/var/lib/docker/overlay2/7f9ec2fa1e82857b9f69c15ff993393a2787d8b854cd0b8a56ac6131ec7e6fb2-init/diff:/var/lib/docker/overlay2/cdd788016ee61771c380142548344cbed891addecfd97646c4cf42d9edd3ce8c/diff:/var/lib/docker/overlay2/0b656bd93fa5cdda1adac4843dc83a1d08cf0af5bb45c5a2b73aafed4f90838e/diff:/var/lib/docker/overlay2/6252d4ba56e7b90f4d9e87bf441483853dcefb58e49784cfedfe67e8a48d8d79/diff:/var/lib/docker/overlay2/3383c7042c8fba359d23128aa2c41964e30a96c18e7c3db2f7032dfe17399201/diff:/var/lib/docker/overlay2/78a8fa92f9e0114da9aa6e61acd4977c8a9b954a669bfb2aa90419923573f4da/diff:/var/lib/docker/overlay2/cff69ece62be74cc51d8bbef3742b39f6cc400c7ee3f24058a7a0527e6827d3a/diff:/var/lib/docker/overlay2/8cabb7d5fb5e7367ad9b66f8e17fd900ee3ef0314b2688a2934e780946484861/diff:/var/lib/docker/overlay2/845a32b37870732f9007b1be2e7ab61e6df0bd6292b1fc5198f4306c623b2ab1/diff:/var/lib/docker/overlay2/69d0a01812c1cd2d1f040967b9d0a7a2d79c3ef10413e992762079b9a2ad5b2d/diff:/var/lib/docker/overlay2/e641dae2802f486d2f4b0f8f29b81903470684e403dd74ced36e0146be9a34ea/diff",
                "MergedDir": "/var/lib/docker/overlay2/7f9ec2fa1e82857b9f69c15ff993393a2787d8b854cd0b8a56ac6131ec7e6fb2/merged",
                "UpperDir": "/var/lib/docker/overlay2/7f9ec2fa1e82857b9f69c15ff993393a2787d8b854cd0b8a56ac6131ec7e6fb2/diff",
                "WorkDir": "/var/lib/docker/overlay2/7f9ec2fa1e82857b9f69c15ff993393a2787d8b854cd0b8a56ac6131ec7e6fb2/work"
            },
            "Name": "overlay2"
        },
        "Mounts": [
            {
                "Type": "volume",
                "Name": "rabbitmq-plugin",
                "Source": "/var/lib/docker/volumes/rabbitmq-plugin/_data",
                "Destination": "/plugins",
                "Driver": "local",
                "Mode": "z",
                "RW": true,
                "Propagation": ""
            },
            {
                "Type": "volume",
                "Name": "b7b13350e8b0d3596aff94385354a1b9366dffeb6b38f8e82a519638f22d74a0",
                "Source": "/var/lib/docker/volumes/b7b13350e8b0d3596aff94385354a1b9366dffeb6b38f8e82a519638f22d74a0/_data",
                "Destination": "/var/lib/rabbitmq",
                "Driver": "local",
                "Mode": "",
                "RW": true,
                "Propagation": ""
            }
        ],
        "Config": {
            "Hostname": "3767efc3f46e",
            "Domainname": "",
            "User": "",
            "AttachStdin": false,
            "AttachStdout": false,
            "AttachStderr": false,
            "ExposedPorts": {
                "15671/tcp": {},
                "15672/tcp": {},
                "15691/tcp": {},
                "15692/tcp": {},
                "25672/tcp": {},
                "4369/tcp": {},
                "5671/tcp": {},
                "5672/tcp": {}
            },
            "Tty": false,
            "OpenStdin": false,
            "StdinOnce": false,
            "Env": [
                "RABBITMQ_DEFAULT_USER=guest",
                "RABBITMQ_DEFAULT_PASS=123456",
                "PATH=/opt/rabbitmq/sbin:/opt/erlang/bin:/opt/openssl/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
                "ERLANG_INSTALL_PATH_PREFIX=/opt/erlang",
                "OPENSSL_INSTALL_PATH_PREFIX=/opt/openssl",
                "RABBITMQ_DATA_DIR=/var/lib/rabbitmq",
                "RABBITMQ_VERSION=3.13.7",
                "RABBITMQ_PGP_KEY_ID=0x0A9AF2115F4687BD29803A206B73A36E6026DFCA",
                "RABBITMQ_HOME=/opt/rabbitmq",
                "HOME=/var/lib/rabbitmq",
                "LANG=C.UTF-8",
                "LANGUAGE=C.UTF-8",
                "LC_ALL=C.UTF-8"
            ],
            "Cmd": [
                "rabbitmq-server"
            ],
            "Image": "rabbitmq:3.13-management",
            "Volumes": {
                "/var/lib/rabbitmq": {}
            },
            "WorkingDir": "",
            "Entrypoint": [
                "docker-entrypoint.sh"
            ],
            "OnBuild": null,
            "Labels": {
                "org.opencontainers.image.ref.name": "ubuntu",
                "org.opencontainers.image.version": "22.04"
            }
        },
        "NetworkSettings": {
            "Bridge": "",
            "SandboxID": "8e3bdc85876ee83c4dc6f9e6501e1cdf6a2f6eba255424d3b541ca4043ff6f91",
            "SandboxKey": "/var/run/docker/netns/8e3bdc85876e",
            "Ports": {
                "15671/tcp": null,
                "15672/tcp": [
                    {
                        "HostIp": "0.0.0.0",
                        "HostPort": "15672"
                    },
                    {
                        "HostIp": "::",
                        "HostPort": "15672"
                    }
                ],
                "15691/tcp": null,
                "15692/tcp": null,
                "25672/tcp": null,
                "4369/tcp": null,
                "5671/tcp": null,
                "5672/tcp": [
                    {
                        "HostIp": "0.0.0.0",
                        "HostPort": "5672"
                    },
                    {
                        "HostIp": "::",
                        "HostPort": "5672"
                    }
                ]
            },
            "HairpinMode": false,
            "LinkLocalIPv6Address": "",
            "LinkLocalIPv6PrefixLen": 0,
            "SecondaryIPAddresses": null,
            "SecondaryIPv6Addresses": null,
            "EndpointID": "6fd5e5f59233ec528be7df6e5f500d800b7abb4df049f2576bb92c5b859d3137",
            "Gateway": "172.17.0.1",
            "GlobalIPv6Address": "",
            "GlobalIPv6PrefixLen": 0,
            "IPAddress": "172.17.0.2",
            "IPPrefixLen": 16,
            "IPv6Gateway": "",
            "MacAddress": "02:42:ac:11:00:02",
            "Networks": {
                "bridge": {
                    "IPAMConfig": null,
                    "Links": null,
                    "Aliases": null,
                    "MacAddress": "02:42:ac:11:00:02",
                    "NetworkID": "7cba32bdc71b92580e2873585313c97476d61b466b33335116931c7f3b7dbb8b",
                    "EndpointID": "6fd5e5f59233ec528be7df6e5f500d800b7abb4df049f2576bb92c5b859d3137",
                    "Gateway": "172.17.0.1",
                    "IPAddress": "172.17.0.2",
                    "IPPrefixLen": 16,
                    "IPv6Gateway": "",
                    "GlobalIPv6Address": "",
                    "GlobalIPv6PrefixLen": 0,
                    "DriverOpts": null,
                    "DNSNames": null
                }
            }
        }
    }
]
```

查看`Mounts`中Name为`rabbitmq-plugin`对应的`Source`值

可以看到值为`/var/lib/docker/volumes/rabbitmq-plugin/_data`

```json
"Mounts": [
  {
    "Type": "volume",
    "Name": "rabbitmq-plugin",
    "Source": "/var/lib/docker/volumes/rabbitmq-plugin/_data",
    "Destination": "/plugins",
    "Driver": "local",
    "Mode": "z",
    "RW": true,
    "Propagation": ""
  },
  {
    "Type": "volume",
    "Name": "b7b13350e8b0d3596aff94385354a1b9366dffeb6b38f8e82a519638f22d74a0",
    "Source": "/var/lib/docker/volumes/b7b13350e8b0d3596aff94385354a1b9366dffeb6b38f8e82a519638f22d74a0/_data",
    "Destination": "/var/lib/rabbitmq",
    "Driver": "local",
    "Mode": "",
    "RW": true,
    "Propagation": ""
  }
]
```

##### 下载延迟插件
官方文档说明页地址：https://www.rabbitmq.com/community-plugins

**rabbitmq_delayed_message_exchange**

A plugin that adds delayed-messaging (or scheduled-messaging) to RabbitMQ.

+ [Releases](https://github.com/rabbitmq/rabbitmq-delayed-message-exchange/releases)
+ Author: **Alvaro Videla**
+ GitHub: [rabbitmq/rabbitmq-delayed-message-exchange](https://github.com/rabbitmq/rabbitmq-delayed-message-exchange)

下载插件安装文件：

```plain
cd /var/lib/docker/volumes/rabbitmq-plugin/_data
wget https://github.com/rabbitmq/rabbitmq-delayed-message-exchange/releases/download/v3.13.0/rabbitmq_delayed_message_exchange-3.13.0.ez
```

若连接被拒绝可多次尝试，或手动下载

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151864422-6143e7c4-a305-4c80-8522-4b6aecdd0389.png)

#### 启用插件
```plain
docker exec -it rabbitmq /bin/bash

rabbitmq-plugins enable rabbitmq_delayed_message_exchange

rabbitmq-plugins list

exit

docker restart rabbitmq
```

##### 确认
确认点1：查看当前节点已启用插件的列表：

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151864289-3aab8567-3b5a-4b17-82c9-2690a64e4358.png)

确认点2：如果创建新交换机时在`Type`中可以看到`x-delayed-message`选项，则说明插件安装好了

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151864263-d575cad2-d860-408c-a728-ddb783c89ea3.png)

#### 创建交换机及队列
**创建交换机：**

Type选择`x-delayed-message`，添加`x-delayed-type`来指定交换机类型

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151864525-705ce1e0-d3a5-4965-ae0f-98d1d1125a6b.png)

**创建队列：**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151864606-8678691a-c014-40b3-acb5-4d61351d03be.png)

**队列绑定交换机：**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151864620-00a08193-031c-4f7d-911f-c7cf584d19e2.png)

#### 代码测试
##### 生产者端代码
```java
public static final String EXCHANGE_DELAY = "exchange.test.delay";
public static final String ROUTING_KEY_DELAY = "routing.key.test.delay";

@Test
public void sendDelayMessageByPlugin() {

    MessagePostProcessor processor = message -> {


        message.getMessageProperties().setHeader("x-delay", 10000);
        return message;
    };

    rabbitTemplate.
    convertAndSend(
        EXCHANGE_DELAY,
        ROUTING_KEY_DELAY,
        "Test Delay Message By Plugin" + new SimpleDateFormat("yyyy-MM-dd HH:mm:ss").format(new Date()),
        processor);
}
```

##### 消费者端代码
```java
public static final String QUEUE_DELAY = "queue.test.delay";

@RabbitListener(queues = {QUEUE_DELAY})
public void processDelayMessage(String dataString, Message message, Channel channel) throws IOException {

    log.info("[delay message] [消息本身] " + dataString);
    log.info("[delay message] [当前时间] " + new SimpleDateFormat("yyyy-MM-dd HH:mm:ss").format(new Date()));
    channel.basicAck(message.getMessageProperties().getDeliveryTag(), false);
}
```

启动消费者端服务并发送消息，查看日志输出情况

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151864751-43017627-cc4f-45ec-a420-eaaad865f4df.png)

注意：启用插件后，returnedMessage方法始终会执行

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151864943-4ba8be35-dcac-462b-adb4-ac42ba21d0cc.png)

## 事务消息
> RabbitMQ的事务只是作用到生产者端，而且只起到局部作用
>

RabbitMQ的事务功能非常有限，只是控制是否将缓存中的消息发送到Broker，并不能保证消息的可靠性投递

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151864960-9c066445-a417-4939-b568-02056940991c.png)

### 实操演示
#### 环境准备
##### 创建项目并导入依赖
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.shiguang</groupId>
  <artifactId>module07-tx-producer</artifactId>
  <version>1.0-SNAPSHOT</version>

  <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>3.1.5</version>
  </parent>

  <properties>
    <maven.compiler.source>17</maven.compiler.source>
    <maven.compiler.target>17</maven.compiler.target>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
  </properties>

  <dependencies>

    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-test</artifactId>
    </dependency>

    <dependency>
      <groupId>org.projectlombok</groupId>
      <artifactId>lombok</artifactId>
    </dependency>

    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-amqp</artifactId>
    </dependency>
  </dependencies>

</project>
```

##### 配置文件
```yaml
spring:
  rabbitmq:
    host: 192.168.10.66
    port: 5672
    username: guest
    password: 123456
    virtual-host: /
logging:
  level:
    com.shiguang.mq: info
```

##### 启动类
```java
package com.shiguang.mq;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;


@SpringBootApplication
public class RabbitMQProducerMainType {
    public static void main(String[] args) {
        SpringApplication.run(RabbitMQProducerMainType.class,args);
    }
}
```

##### 配置类
```java
package com.shiguang.mq.config;

import lombok.Data;
import org.springframework.amqp.rabbit.connection.CachingConnectionFactory;
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.amqp.rabbit.transaction.RabbitTransactionManager;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;


@Configuration
@Data
public class RabbitConfig {
    @Bean
    public RabbitTransactionManager transactionManager(CachingConnectionFactory connectionFactory) {
        return new RabbitTransactionManager(connectionFactory);
    }

    @Bean
    public RabbitTemplate rabbitTemplate(CachingConnectionFactory connectionFactory) {
        RabbitTemplate rabbitTemplate = new RabbitTemplate(connectionFactory);
        rabbitTemplate.setChannelTransacted(true);
        return rabbitTemplate;
    }


}
```

##### 测试类
```java
package com.shiguang.mq;

import jakarta.annotation.Resource;
import lombok.extern.slf4j.Slf4j;
import org.junit.jupiter.api.Test;
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.boot.test.context.SpringBootTest;


@SpringBootTest
@Slf4j
public class RabbitMQTest {
    public static final String EXCHANGE_NAME = "exchange.tx.dragon";
    public static final String ROUTING_KEY = "routing.key.tx.dragon";

    @Resource
    private RabbitTemplate rabbitTemplate;

    @Test
    public void testSendMessageTx(){

        rabbitTemplate.convertAndSend(EXCHANGE_NAME, ROUTING_KEY, "hello rabbitmq tx message 1");

        log.info("do bad: "+ 10/0);

        rabbitTemplate.convertAndSend(EXCHANGE_NAME, ROUTING_KEY, "hello rabbitmq tx message 2");
    }
}
```

#### 测试
> 我们分别发送两条消息，两条消息中间手动抛出异常，来观察启用事务前后的区别
>

**1、创建交换机、队列并绑定关系**

交换机名称：exchange.tx.dragon

队列名称：queue.test.tx

路由键：routing.key.tx.dragon

**2、发送消息并观察队列情况**

默认未使用事务的情况：第一条事务发送成功，消息能够正常获取

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151864912-4a164d03-7df2-43a4-88a0-09acfdc6d479.png)

开启事务：

测试类添加`@Transactional`注解，由于JUnit中是默认回滚的，我们想要提交事务，需要添加`@Rollback(value = false)`注解

```java
@Test
@Transactional

public void testSendMessageTx(){

    rabbitTemplate.convertAndSend(EXCHANGE_NAME, ROUTING_KEY, "hello rabbitmq tx message 1");

    log.info("do bad: "+ 10/0);

    rabbitTemplate.convertAndSend(EXCHANGE_NAME, ROUTING_KEY, "hello rabbitmq tx message 2");
}
```

我们保持默认回滚事务，执行测试方法，观察队列情况

由于出现异常，事务被回滚，消息未发送

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151864946-8adcb93f-cd58-4d14-a717-23c04e65330e.png)

## 惰性队列
惰性队列：未设置惰性模式时队列的持久化机制,

<font style="color:rgb(31, 35, 40);">惰性队列（</font>**<font style="color:rgb(31, 35, 40);">Lazy Queue</font>**<font style="color:rgb(31, 35, 40);">）是 RabbitMQ 的一种队列模式：</font>**<font style="color:rgb(31, 35, 40);">尽量把消息存到磁盘上，而不是一直堆在内存里</font>**<font style="color:rgb(31, 35, 40);">，用来应对“消息积压很大、内存扛不住”的场景。</font>

创建队列时，在Durabilityi这里有两个选项可以选择

+ Durable: 持久化队列，消息会持久化到硬盘上
+ Transient: 临时队列，不做持久化操作，broker重启后消息会丢失

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151864997-d4e3111f-4fc5-4bec-89c8-673d7b1059b6.png)

> 思考：Durable队列在存入消息之后，是否是立即保存到硬盘呢？
>

其实并不会立即保存到硬盘，当内存中的队列达到一定容量或者Broker关闭时才会保存到硬盘

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151865240-c4ac80b9-09b5-49bc-ac86-477bbdd1faba.png)

官网上对于惰性队列的介绍

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151865427-f9de5300-9bd5-4a32-b53b-f74f96f524fe.png)

比较下面两个说法是否是相同的意思：

+ 立即移动到硬盘
+ 尽早移动到硬盘

理解：

+ 立即：消息刚进入队列时
+ 尽早：服务器不繁忙时

惰性队列应用场景

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151865440-703f17c8-9805-401e-95a8-d8df9dbc56f1.png)

原文翻译：使用惰性队列的主要原因之一是支持非常长的队列（数百万条消息）  
由于各种原因，排队可能会变得很长：

+ 消费者离线/崩溃/停机进行维护
+ 突然出现消息进入高峰生产者的速度超过了消费者
+ 消费者比正常情况慢

## 优先级队列
机制说明  
默认情况：基于队列先进先出的特性，通常来说，先入队的先投递  
设置优先级之后：优先级高的消息更大几率先投递  
关键参数：`x-max-priority`

RabbitMQ允许我们使用一个正整数给消息设定优先级  
消息的优先级数值取值范围：`1~255`  
RabbitMQ官网建议在`1~5`之间设置消息的优先级（优先级越高，占用CPU、内存等资源越多)

队列在声明时可以指定参数：`x-max-priority`  
默认值：`0` ，此时消息即使设置优先级也无效  
指定一个正整数值：消息的优先级数值不能超过这个值

### 实操演示
**1、创建交换机及队列并绑定**

交换机名称：exchange.test.priority

队列名称：queue.test.priority

> x-max-priority的类型必须是Number
>

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151865332-569a7d18-598a-4e25-b341-bae3cb26d0c7.png)

路由键：routing.key.test.priority

**2、分别发送三条消息，优先级从低到高，后面观察入队情况**

```java
public static final String EXCHANGE_PRIORITY = "exchange.test.priority";
public static final String ROUTING_KEY_PRIORITY = "routing.key.test.priority";
```

发送第一条消息

```java
@Test
public void testSendPriorityMessage() {
    rabbitTemplate.
    convertAndSend(
        EXCHANGE_PRIORITY,
        ROUTING_KEY_PRIORITY,
        "Test Priority Message 1",message -> {
            message.getMessageProperties().setPriority(1);
            return message;
        });
}
```

发送第二条消息

```java
@Test
public void testSendPriorityMessage() {
    rabbitTemplate.
    convertAndSend(
        EXCHANGE_PRIORITY,
        ROUTING_KEY_PRIORITY,
        "Test Priority Message 2",message -> {
            message.getMessageProperties().setPriority(2);
            return message;
        });
}
```

发送第三条消息

```java
@Test
public void testSendPriorityMessage() {
    rabbitTemplate.
    convertAndSend(
        EXCHANGE_PRIORITY,
        ROUTING_KEY_PRIORITY,
        "Test Priority Message 3",message -> {

            message.getMessageProperties().setPriority(3);
            return message;
        });
}
```

**3、启动客户端服务，查看日志输出情况**

```java
public static final String QUEUE_PRIORITY = "queue.test.priority";

@RabbitListener(queues = {QUEUE_PRIORITY})
public void processPriorityMessage(String dataString, Message message, Channel channel) throws IOException {
    log.info("[priority]: " + dataString);
    channel.basicAck(message.getMessageProperties().getDeliveryTag(), false);
}
```

我们可以看到优先级高的先输出

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151865353-94fb6f09-af32-4884-95b1-800ea7941214.png)

## 集群搭建
### 安装RabbitMQ
#### 前置要求
> 课程要求CentOS发行版的版本≥8，CentOS 7.x 其实也可以，后面有详细介绍
>

下载地址：https://mirrors.163.com/centos/

查看当前系统发行版本：

```plain
[root@localhost _data]
   Static hostname: localhost.localdomain
         Icon name: computer-vm
           Chassis: vm
        Machine ID: 1e9464680b694994bb37fa7013bd3ea7
           Boot ID: e0865df1adfa476eb633daed2637bff1
    Virtualization: vmware
  Operating System: CentOS Linux 7 (Core)
       CPE OS Name: cpe:/o:centos:centos:7
            Kernel: Linux 3.10.0-1160.90.1.el7.x86_64
      Architecture: x86-64
```

RabbitMQ安装方式官方指南：

> https://www.rabbitmq.com/docs/install-rpm
>

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151865508-425b38ed-b034-40e5-b758-b2e797b8632c.png)

#### 安装Erlang环境
##### **创建yum库配置文件**
```plain
vim /etc/yum.repos.d/rabbitmq.repo
```

##### **加入配置内容**
> 以下内容来自官网文档：https://www.rabbitmq.com/docs/install-rpm
>

```plain
# In /etc/yum.repos.d/rabbitmq.repo

##
## Zero dependency Erlang RPM
##

[modern-erlang]
name=modern-erlang-el9
# Use a set of mirrors maintained by the RabbitMQ core team.
# The mirrors have significantly higher bandwidth quotas.
baseurl=https://yum1.rabbitmq.com/erlang/el/9/$basearch
        https://yum2.rabbitmq.com/erlang/el/9/$basearch
repo_gpgcheck=1
enabled=1
gpgkey=https://github.com/rabbitmq/signing-keys/releases/download/3.0/cloudsmith.rabbitmq-erlang.E495BB49CC4BBE5B.key
gpgcheck=1
sslverify=1
sslcacert=/etc/pki/tls/certs/ca-bundle.crt
metadata_expire=300
pkg_gpgcheck=1
autorefresh=1
type=rpm-md

[modern-erlang-noarch]
name=modern-erlang-el9-noarch
# Use a set of mirrors maintained by the RabbitMQ core team.
# The mirrors have significantly higher bandwidth quotas.
baseurl=https://yum1.rabbitmq.com/erlang/el/9/noarch
        https://yum2.rabbitmq.com/erlang/el/9/noarch
repo_gpgcheck=1
enabled=1
gpgkey=https://github.com/rabbitmq/signing-keys/releases/download/3.0/cloudsmith.rabbitmq-erlang.E495BB49CC4BBE5B.key
       https://github.com/rabbitmq/signing-keys/releases/download/3.0/rabbitmq-release-signing-key.asc
gpgcheck=1
sslverify=1
sslcacert=/etc/pki/tls/certs/ca-bundle.crt
metadata_expire=300
pkg_gpgcheck=1
autorefresh=1
type=rpm-md

[modern-erlang-source]
name=modern-erlang-el9-source
# Use a set of mirrors maintained by the RabbitMQ core team.
# The mirrors have significantly higher bandwidth quotas.
baseurl=https://yum1.rabbitmq.com/erlang/el/9/SRPMS
        https://yum2.rabbitmq.com/erlang/el/9/SRPMS
repo_gpgcheck=1
enabled=1
gpgkey=https://github.com/rabbitmq/signing-keys/releases/download/3.0/cloudsmith.rabbitmq-erlang.E495BB49CC4BBE5B.key
       https://github.com/rabbitmq/signing-keys/releases/download/3.0/rabbitmq-release-signing-key.asc
gpgcheck=1
sslverify=1
sslcacert=/etc/pki/tls/certs/ca-bundle.crt
metadata_expire=300
pkg_gpgcheck=1
autorefresh=1


##
## RabbitMQ Server
##

[rabbitmq-el9]
name=rabbitmq-el9
baseurl=https://yum2.rabbitmq.com/rabbitmq/el/9/$basearch
        https://yum1.rabbitmq.com/rabbitmq/el/9/$basearch
repo_gpgcheck=1
enabled=1
# Cloudsmith's repository key and RabbitMQ package signing key
gpgkey=https://github.com/rabbitmq/signing-keys/releases/download/3.0/cloudsmith.rabbitmq-server.9F4587F226208342.key
       https://github.com/rabbitmq/signing-keys/releases/download/3.0/rabbitmq-release-signing-key.asc
gpgcheck=1
sslverify=1
sslcacert=/etc/pki/tls/certs/ca-bundle.crt
metadata_expire=300
pkg_gpgcheck=1
autorefresh=1
type=rpm-md

[rabbitmq-el9-noarch]
name=rabbitmq-el9-noarch
baseurl=https://yum2.rabbitmq.com/rabbitmq/el/9/noarch
        https://yum1.rabbitmq.com/rabbitmq/el/9/noarch
repo_gpgcheck=1
enabled=1
# Cloudsmith's repository key and RabbitMQ package signing key
gpgkey=https://github.com/rabbitmq/signing-keys/releases/download/3.0/cloudsmith.rabbitmq-server.9F4587F226208342.key
       https://github.com/rabbitmq/signing-keys/releases/download/3.0/rabbitmq-release-signing-key.asc
gpgcheck=1
sslverify=1
sslcacert=/etc/pki/tls/certs/ca-bundle.crt
metadata_expire=300
pkg_gpgcheck=1
autorefresh=1
type=rpm-md

[rabbitmq-el9-source]
name=rabbitmq-el9-source
baseurl=https://yum2.rabbitmq.com/rabbitmq/el/9/SRPMS
        https://yum1.rabbitmq.com/rabbitmq/el/9/SRPMS
repo_gpgcheck=1
enabled=1
gpgkey=https://github.com/rabbitmq/signing-keys/releases/download/3.0/cloudsmith.rabbitmq-server.9F4587F226208342.key
gpgcheck=0
sslverify=1
sslcacert=/etc/pki/tls/certs/ca-bundle.crt
metadata_expire=300
pkg_gpgcheck=1
autorefresh=1
type=rpm-md
```

##### **更新yum库**
> –nobest 表示所需安装包即使不是最佳选择也接收
>

```plain
yum update -y --nobest
```

若不支持系统`--nobest`参数则可不使用

```plain
yum update -y
```

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151865678-d53c7f2b-f0ee-444f-814e-0a9e6e72b943.png)

##### **正式安装Erlang**
###### CentOS 8
```plain
yum install -y erlang
```

###### **CentOS 7**
**卸载旧版本**

若未安装过，可跳过

> **卸载旧版本的 Erlang**
>
> 1. **查找已安装的 Erlang 包：**
> 2. **卸载旧版本的 Erlang**：
>
> **检查并删除残留文件**
>
> **确保系统中没有其他 Erlang 版本的残留文件或配置。**
>
> 1. **查找并删除所有与 Erlang 相关的目录**：
> 2. **查找并删除所有与 Erlang 相关的文件**：
> 3. **查找并删除所有与 Erlang 相关的符号链接**：
>

```plain
rpm -qa | grep erlang
```

```plain
sudo yum remove erlang-26.2.5.4-1.el7.x86_64
```

```plain
sudo find / -name "erlang" -type d -exec rm -rf {} +
```

```plain
sudo find / -name "*erlang*" -type f -exec rm -f {} +
```

```plain
sudo find /usr/bin /usr/local/bin -name "erl*" -type l -exec rm -f {} +
```

安装时需要注意Erlang与CentOS的版本匹配，详细介绍见官网： https://www.rabbitmq.com/docs/which-erlang

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151865671-e1bdf4fb-c222-405a-8893-a84e3fa20ed5.png)

如课程中RabbitMQ使用的是`v3.13.0`，erlang需要安装的版本需要 >= 26.0

由于`rabbitmq-server` 安装包支持CentOS7的版本较老，如 `v3.9.16`，兼容的erlang最低版本为23.3，最高24.3

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151865807-e4b9ca43-d5db-45af-8a11-51632847db16.png)

**通过RPM安装**

> 可参考文章：[OpenCloudOS 8配置rabbitmq](https://blog.csdn.net/MeltryLL/article/details/141437375)
>

下载地址：https://github.com/rabbitmq/erlang-rpm/releases

我们需要下载与之相兼容的erlang版本如 [erlang-23.3-2.el7.x86_64.rpm](https://github.com/rabbitmq/erlang-rpm/releases/tag/v23.3)， el7 代表 CentOS 7

GitHub仓库地址： https://github.com/rabbitmq/erlang-rpm/releases

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151865860-955c404d-2c61-4a8d-96cd-b7aeb0b233d9.png)

将文件上传到CentOS的某个目录上，如`/opt/rabbitmq`

```plain
sudo rpm -ivh erlang-23.3-2.el7.x86_64.rpm



erl -version





erl
```

**通过yum 安装**

> 可参考文章： [CentOS 7 安装Erlang、RabbitMQ（亲测通过）](https://blog.csdn.net/mumanquan1/article/details/122074059)
>

Erlang安装包下载地址： https://packagecloud.io/rabbitmq/erlang

选择与`rabbitmq-server` 相兼容的版本，如 `erlang-23.3.4.11-1.el7.x86_64.rpm`，el7 代表适用CentOS7

> 若执行第一步出现如下错误
>
> 检查Python版本
>
> 若为3.x，执行如下命令创建软连接，使用python2执行
>

```plain
[root@localhost ~]
Detected operating system as centos/7.
Checking for curl...
Detected curl...
Downloading repository file: https://packagecloud.io/install/repositories/rabbitmq/erlang/config_file.repo?os=centos&dist=7&source=script
done.
Attempting to install pygpgme for your os/dist: centos/7. Only required on older OSes to verify GPG signatures.
Installing yum-utils...
  File "/bin/yum", line 30
    except KeyboardInterrupt, e:
                            ^
SyntaxError: invalid syntax
Generating yum cache for rabbitmq_erlang...
  File "/bin/yum", line 30
    except KeyboardInterrupt, e:
                            ^
SyntaxError: invalid syntax
Generating yum cache for rabbitmq_erlang-source...
  File "/bin/yum", line 30
    except KeyboardInterrupt, e:
                            ^
SyntaxError: invalid syntax

The repository is setup! You can now install packages.
```

```plain
[root@localhost ~]
Python 3.7.0
```

```plain
sudo ln -sf /usr/bin/python2 /usr/bin/python
```

```plain
curl -s https://packagecloud.io/install/repositories/rabbitmq/erlang/script.rpm.sh | sudo bash


sudo yum install -y erlang-23.3.4.11-1.el7.x86_64
```

若下载失败可到官网手动下载安装

下载地址：https://www.erlang.org/downloads，会跳转至GitHub

GitHub: https://github.com/erlang/otp/releases

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151865827-ce96bf66-ec15-44bf-b5cd-17724c80a34e.png)

下载完成后，将文件上传到某个目录，如`/opt/rabbitmq`，通过以下代码完成安装

```plain
yum -y install gcc


tar -zxvf otp_src_23.3.4.11.tar.gz


cd /opt/rabbitmq/otp_src_23.3.4.11/


./configure --prefix=/usr/local/erlang


make install
```

查看是否安装成功以及设置环境变量

```plain
ll /usr/local/erlang/bin


echo 'export PATH=$PATH:/usr/local/erlang/bin' >> /etc/profile


source /etc/profile



erl -version







erl
```

> 安装Erlang最新版会遇到的坑
>

此处发现打印的是版本是 `14.2.5.4`

```plain
erl -version
Erlang (SMP,ASYNC_THREADS) (BEAM) emulator version 14.2.5.4
```

使用 `erl`验证下，发现

```plain
[root@localhost rabbitmq]
Erlang/OTP 26 [erts-14.2.5.4] [source] [64-bit] [smp:4:4] [ds:4:4:10] [async-threads:1]

Eshell V14.2.5.4 (press Ctrl+G to abort, type help(). for help)
1>
```

安装RabbitMQ时提示如下错误

```plain
[root@localhost rabbitmq]
```

#### **安装RabbitMQ**
##### CentOS 8
```plain
rpm --import 'https://github.com/rabbitmq/signing-keys/releases/download/3.0/rabbitmq-release-signing-key.asc'

rpm --import 'https://github.com/rabbitmq/signing-keys/releases/download/3.0/cloudsmith.rabbitmq-erlang.E495BB49CC4BBE5B.key'

rpm --import 'https://github.com/rabbitmq/signing-keys/releases/download/3.0/cloudsmith.rabbitmq-server.9F4587F226208342.key'



wget https://github.com/rabbitmq/rabbitmq-server/releases/download/v3.13.0/rabbitmq-server-3.13.0-1.el8.noarch.rpm


rpm -ivh rabbitmq-server-3.13.0-1.el8.noarch.rpm
```

##### CentOS 7
通过[Release Information | RabbitMQ](https://www.rabbitmq.com/release-information)跳转到github下载界面

https://github.com/rabbitmq/rabbitmq-server/releases

选择与`rabbitmq-server` 相兼容的版本，如 [rabbitmq-server-3.9.16-1.el7.noarch.rpm](https://github.com/rabbitmq/rabbitmq-server/releases/tag/v3.9.16)

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151865959-ddaeb59c-5d01-407b-a7d3-0a984c4ba3bd.png)

上传到CentOS某个目录，如 `/opt/rabbitmq`

```plain
rpm -ivh rabbitmq-server-3.9.16-1.el7.noarch.rpm
```

#### **RabbitMQ基础配置**
> 启动服务前注意停用之前的Docker服务，以免造成端口冲突
>

```plain
rabbitmq-plugins enable rabbitmq_management


systemctl start rabbitmq-server


systemctl enable rabbitmq-server


rabbitmqctl add_user shiguang 123456


rabbitmqctl set_user_tags shiguang administrator
rabbitmqctl set_permissions -p / shiguang ".*" ".*" ".*"


rabbitmqctl enable_feature_flag all


systemctl restart rabbitmq-server
```

#### 收尾工作
> 若不删除该配置，以后用yum安装会受到该配置影响
>

```plain
rm -rf /etc/yum.repos.d/rabbitmq.repo
```

### 克隆 VMWare虚拟机
#### 目标
通过克隆操作，一共准备三台VMWare虚拟机

| 集群节点名称 | 虚拟机IP地址 |
| --- | --- |
| node01 | 192.168.10.66 |
| node02 | 192.168.10.88 |
| node03 | 192.168.10.99 |


#### 克隆虚拟机
需克隆完整连接

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151866084-3befda54-a005-4c60-9c34-8ca20c28b88a.png)

如下：

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151866103-b30b8676-90c5-414d-9e64-670eb1d4ec9e.png)

#### 给新机器设置IP地址
在CentOS 7 中，可以使用`nmcli`命令行工具修改IP地址。以下是具体步骤：

1、查看网络连接信息：

```plain
nmcli con show
```

2、停止指定的网络连接（将 <connection_name>替换为实际的网络连接名称）：

```plain
nmcli con down <connection_name>
```

3、修改IP地址（将 <connection_name>替换为实际的网络连接名称，将 <new_ip_address>替换为新的IP地址，将<subnet_mask>替换为子网掩码，将<gateway>替换为网关）

```plain
nmcli con mod <connection_name> ipv4.addresses <new_ip_address>/<subnet_mask>
nmcli con mod <connection_name> ipv4.gateway <gateway>
nmcli con mod <connection_name> ipv4.method manual
```

4、启动网络连接

```plain
nmcli con up <connection_name>
```

5、验证新的IP地址是否生效：

```plain
ip addr show
```

#### 修改主机名称
主机名称会被RabbitMQ作为集群中的节点名称，后面会用到，所以需要设置一下。  
修改后需重启

```plain
cat /etc/hostname

vim /etc/hostname
```

#### 保险措施
为了在后续操作过程中，万一遇到操作失误，友情建议拍摄快照。

### 集群节点彼此发现
#### node01设置
① 设置IP地址到主机名称的映射

修改文件`/etc/hosts`

```plain
vim /etc/hosts
```

追加如下内容：

```plain
192.168.10.66 node01
192.168.10.88 node02
192.168.10.99 node03
```

② 查看当前RabbitMQ节点的Cookie值并记录

```plain
cat /var/lib/rabbitmq/.erlang.cookie
```

显示如下：

```plain
[root@node01 ~]
KFGJAHXELTVBZJVTEHSG[root@node01 ~]
```

③ 重置节点应用

```plain
rabbitmqctl stop_app
rabbitmqctl reset
rabbitmqctl start_app
```

#### node02设置
① 设置P地址到主机名称的映射

修改文件`/etc/hosts`

```plain
vim /etc/hosts
```

追加如下内容：

```plain
192.168.10.66 node01
192.168.10.88 node02
192.168.10.99 node03
```

② 修改当前RabbitMQ节点的Cookie值  
node02和node03都改成和node01一样：

```plain
vim /var/lib/rabbitmq/.erlang.cookie
```

③ 重置节点应用并加入集群

```plain
rabbitmqctl stop_app
rabbitmqctl reset
rabbitmqctl join_cluster rabbit@node01
rabbitmqctl start_app
```

#### node03设置
① 设置P地址到主机名称的映射

修改文件`/etc/hosts`

```plain
vim /etc/hosts
```

追加如下内容：

```plain
192.168.10.66 node01
192.168.10.88 node02
192.168.10.99 node03
```

② 修改当前RabbitMQ节点的Cookie值  
node02和node03都改成和node01一样：

```plain
vim /var/lib/rabbitmq/.erlang.cookie
```

③ 重置节点应用并加入集群

```plain
rabbitmqctl stop_app
rabbitmqctl reset
rabbitmqctl join_cluster rabbit@node01
rabbitmqctl start_app
```

④ 查看集群状态

```plain
rabbitmqctl cluster_status
```

显示如下：

```plain
[root@node01 ~]
Cluster status of node rabbit@node01 ...
Basics

Cluster name: rabbit@node01

Disk Nodes

rabbit@node01
rabbit@node02
rabbit@node03

Running Nodes

rabbit@node01
rabbit@node02
rabbit@node03

Versions

rabbit@node01: RabbitMQ 3.9.16 on Erlang 23.3
rabbit@node02: RabbitMQ 3.9.16 on Erlang 23.3.4.11
rabbit@node03: RabbitMQ 3.9.16 on Erlang 23.3.4.11

Maintenance status

Node: rabbit@node01, status: not under maintenance
Node: rabbit@node02, status: not under maintenance
Node: rabbit@node03, status: not under maintenance

Alarms

(none)

Network Partitions

(none)

Listeners

Node: rabbit@node01, interface: [::], port: 25672, protocol: clustering, purpose: inter-node and CLI tool communication
Node: rabbit@node01, interface: [::], port: 15672, protocol: http, purpose: HTTP API
Node: rabbit@node01, interface: [::], port: 5672, protocol: amqp, purpose: AMQP 0-9-1 and AMQP 1.0
Node: rabbit@node02, interface: [::], port: 15672, protocol: http, purpose: HTTP API
Node: rabbit@node02, interface: [::], port: 25672, protocol: clustering, purpose: inter-node and CLI tool communication
Node: rabbit@node02, interface: [::], port: 5672, protocol: amqp, purpose: AMQP 0-9-1 and AMQP 1.0
Node: rabbit@node03, interface: [::], port: 15672, protocol: http, purpose: HTTP API
Node: rabbit@node03, interface: [::], port: 25672, protocol: clustering, purpose: inter-node and CLI tool communication
Node: rabbit@node03, interface: [::], port: 5672, protocol: amqp, purpose: AMQP 0-9-1 and AMQP 1.0

Feature flags

Flag: drop_unroutable_metric, state: enabled
Flag: empty_basic_get_metric, state: enabled
Flag: implicit_default_bindings, state: enabled
Flag: maintenance_mode_status, state: enabled
Flag: quorum_queue, state: enabled
Flag: stream_queue, state: enabled
Flag: user_limits, state: enabled
Flag: virtual_host_metadata, state: enabled
```

也可登录管理后台查看

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151866245-dd7c2161-fddd-4922-9cae-425add245a3a.png)

### 负载均衡：Management UI
#### 说明
两个需要暴露的端口：

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151866231-8c70588d-ca51-4711-9cad-df120418238f.png)

目前集群方案：

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151866356-09da0b4c-62be-4462-8ba0-57c96df7993c.png)

管理界面负载均衡：

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151866481-9c2a76c4-e713-4e06-b59b-040af34d7f7d.png)

核心功能负载均衡：

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151866492-6e74dca2-3691-40dd-95a0-2795001a9999.png)

#### 安装HAProxy
```plain
yum install -y haproxy


haproxy -v


systemctl start haproxy


systemctl enable haproxy
```

#### 修改配置文件
> 配置文件位置：/etc/haproxy/haproxy.cfg
>

在配置文件未尾增加如下内容：

```plain
frontend rabbitmq_ui_frontend
bind 192.168.10.66:22222
mode http
default_backend rabbitmq_ui_backend

backend rabbitmq_ui_backend
mode http
balance roundrobin
option httpchk GET /
server rabbitmq_ui1 192.168.10.66:15672 check
server rabbitmq_ui2 192.168.10.88:15672 check
server rabbitmq_ui3 192.168.10.99:15672 check
```

设置SELinux策略，允许HAProxy拥有权限连接任意端口：

> SELinux是Linux系统中的安全模块，它可以限制进程的权限以提高系统的安全性。在某些情况下，SELinux可能会阻止HAProxy绑定指定的端口，这就需要通过设置域(domain)的安全策略来解决此问题。
>
> 通过执行setsebool-P haproxy_connect_any=1命令，您已经为HAProxyi设置了一个布尔值，允许HAProxy连接到任意端口。这样，HAProxy就可以成功绑定指定的socket,并正常工作。
>

```plain
setsebool -P haproxy_connect_any=1
```

重启HAProxy

```plain
systemctl restart haproxy
```

#### 测试效果
访问配置的前台负载均衡地址： http://192.168.10.66:22222

查看是否可以正常打开rabbitmq管理端界面

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151866579-930ee999-21a4-43d6-be7d-777370c30854.png)

### 负载均衡：核心功能
#### 添加配置
> 配置文件位置：/etc/haproxy/haproxy.cfg
>

在配置文件未尾增加如下内容：

```plain
frontend rabbitmq_frontend
bind 192.168.10.66:11111
mode tcp
default_backend rabbitmq_backend

backend rabbitmq_backend
mode tcp
balance roundrobin
server rabbitmq1 192.168.10.66:5672 check
server rabbitmq2 192.168.10.88:5672 check
server rabbitmq3 192.168.10.99:5672 check
```

重启HAProxy

```plain
systemctl restart haproxy
```

#### 测试
##### **创建组件**
+ 交换机：exchange.cluster.test
+ 队列；queue.cluster.test
+ 路由键：routing.key.cluster.test

##### **创建生产者端程序**
**1、POM**

```plain
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.shiguang</groupId>
    <artifactId>module08-cluster-producer</artifactId>
    <version>1.0-SNAPSHOT</version>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.1.5</version>
    </parent>

    <properties>
        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>

    <dependencies>
        
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
        </dependency>
        
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
        </dependency>
        
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-amqp</artifactId>
        </dependency>
    </dependencies>

</project>
```

**2、核心配置文件**

```plain
spring:
  rabbitmq:
    host: 192.168.10.66
    port: 11111
    username: shiguang
    password: 123456
    virtual-host: /
    publisher-confirm-type: CORRELATED 
    publisher-returns: true 
logging:
  level:
    com.shiguang.mq.config.MQProducerAckConfig: info
```

**3、配置类**

```plain
package com.shiguang.mq.config;

import jakarta.annotation.PostConstruct;
import lombok.extern.slf4j.Slf4j;
import org.springframework.amqp.core.ReturnedMessage;
import org.springframework.amqp.rabbit.connection.CorrelationData;
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Configuration;



@Configuration
@Slf4j
public class MQProducerAckConfig implements RabbitTemplate.ConfirmCallback, RabbitTemplate.ReturnsCallback {
    @Autowired
    private RabbitTemplate rabbitTemplate;

    @PostConstruct
    public void init() {
        rabbitTemplate.setConfirmCallback(this);
        rabbitTemplate.setReturnsCallback(this);
    }

    
    @Override
    public void confirm(CorrelationData correlationData, boolean ack, String cause) {
        if (ack) {
            log.info("消息发送到交换机成功！数据: " + correlationData);
        } else {
            log.info("消息发送到交换机失败！ 数据: " + correlationData + " 错误原因: " + cause);
        }

    }

    
    @Override
    public void returnedMessage(ReturnedMessage returnedMessage) {
        log.info("returnedMessage() 回调函数 消息主体: " + new String(returnedMessage.getMessage().getBody()));
        log.info("returnedMessage() 回调函数 应答码: " + returnedMessage.getReplyCode());
        log.info("returnedMessage() 回调函数 描述: " + returnedMessage.getReplyText());
        log.info("returnedMessage() 回调函数 消息使用的交换器 exchange: " + returnedMessage.getExchange());
        log.info("returnedMessage() 回调函数 消息使用的路由键 routing: " + returnedMessage.getRoutingKey());
    }
}
```

**4、启动类**

```plain
package com.shiguang.mq;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;


@SpringBootApplication
public class RabbitMQProducerMainType {
    public static void main(String[] args) {
        SpringApplication.run(RabbitMQProducerMainType.class, args);
    }
}
```

**5、测试类**

```plain
package com.shiguang.mq;

import org.junit.jupiter.api.Test;
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;




@SpringBootTest
public class RabbitMQTest {
    public static final String EXCHANGE_CLUSTER_TEST = "exchange.cluster.test";
    public static final String ROUTING_KEY_CLUSTER_TEST = "routing.key.cluster.test";

    @Autowired
    private RabbitTemplate rabbitTemplate;


    @Test
    public void testSendMessage() {

        String message = "Test Send Message By Cluster !!";

        rabbitTemplate.convertAndSend(EXCHANGE_CLUSTER_TEST, ROUTING_KEY_CLUSTER_TEST, message);
    }


}
```

##### 创建消费者端程序
**1、POM**

```plain
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.chiguang</groupId>
    <artifactId>module09-cluster-consumer</artifactId>
    <version>1.0-SNAPSHOT</version>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.1.5</version>
    </parent>

    <properties>
        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>

    <dependencies>
        
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
        </dependency>
        
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-amqp</artifactId>
        </dependency>
    </dependencies>

</project>
```

**2、核心配置文件**

```plain
spring:
  rabbitmq:
    host: 192.168.10.66
    port: 11111
    username: shiguang
    password: 123456
    virtual-host: /
    listener:
      simple:
        acknowledge-mode: manual 
logging:
  level:
    com.shiguang.mq.config.MQProducerAckConfig: info
```

**3、Listener**

```plain
package com.shiguang.mq.listener;

import com.rabbitmq.client.Channel;
import lombok.extern.slf4j.Slf4j;
import org.springframework.amqp.core.Message;
import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.stereotype.Component;

import java.io.IOException;


@Component
@Slf4j
public class MyMessageListener {
    public static final String QUEUE_CLUSTER = "queue.cluster.test";

    @RabbitListener(queues = {QUEUE_CLUSTER})
    public void processPriorityMessage(String dataString, Message message, Channel channel) throws IOException {
        log.info("[消费者端] 消息内容: " + dataString);
        channel.basicAck(message.getMessageProperties().getDeliveryTag(), false);
    }

}
```

**4、启动类**

```plain
package com.shiguang.mq;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;


@SpringBootApplication
public class RabbitMQConsumerMainType {
    public static void main(String[] args) {
        SpringApplication.run(RabbitMQConsumerMainType.class, args);
    }
}
```

##### 运行结果
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151866566-8d3c6ac9-a1b2-469e-a3b4-b669b0dccb8e.png)

### 镜像队列
> 镜像队列在新版本中已被仲裁队列取代，这里不再介绍
>

### 仲裁队列
RabbitMQ3.8.x版本的主要更新内容，未来有可能取代Classic Queue

创建仲裁队列，可以将队列同步到集群中的每个节点上

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151866710-b741d291-0711-4f74-9d8a-1c8b029362e6.png)

#### 操作步骤
##### 创建仲裁队列
> 需要在集群的基础上创建
>

**1、创建交换机**

和仲裁队列绑定的交换机没有特殊要求，我们还是创建一个direct交换机即可  
交换机名称：exchange.quorum.test

**2、创建仲裁队列**

队列名称：queue.quorum.test

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151866806-1a48e1fe-5ec4-410b-87ea-606c8ef9736f.png)

创建好后如图所示：

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151866891-3f0ff3cb-39a9-4b4c-afff-8bb06b7eddaf.png)

详情信息：

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151866921-16cdc762-a418-4c3c-9c33-266946c01606.png)

3、绑定交换机

路由键：routing.key.quorum.test

##### 测试
###### 常规测试
像使用经典队列一样发送消息、消费消息

**① 生产者端**

```plain
public static final String EXCHANGE_QUORUM_TEST = "exchange.quorum.test";
public static final String ROUTING_KEY_QUORUM_TEST = "routing.key.quorum.test";

@Test
public void testSendMessageToQuorum() {

    String message = "Test Send Message By Quorum!!";

    rabbitTemplate.convertAndSend(EXCHANGE_QUORUM_TEST, ROUTING_KEY_QUORUM_TEST, message);
}
```

日志输出情况：

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151866968-ff64968c-f531-4bc4-92b0-3aebbe5ba0c3.png)

队列情况：

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151867041-34dc6ceb-26a2-4a47-842f-bb8de5b41069.png)

**② 消费者端**

```plain
public static final String QUEUE_QUORUM_TEST = "queue.quorum.test";

@RabbitListener(queues = {QUEUE_QUORUM_TEST})
public void processQuorumMessage(String dataString, Message message, Channel channel) throws IOException {
    log.info("[消费者端] 消息内容: " + dataString);
    channel.basicAck(message.getMessageProperties().getDeliveryTag(), false);
}
```

日志输出情况：

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151867110-5524ccc4-9ee0-4e40-8723-b588b64843ce.png)

队列情况：

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151867211-9f16ee5b-e42e-48d0-a10a-29fde6838dbf.png)

###### 高可用测试
① 停止某个节点的rabbit应用

```plain
rabbitmqctl stop_app
```

此时可以再观察下队列详情，可以发现已自动选举出新的Leader

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151867300-2e6a75d9-0338-4f81-8f25-5ab051e5c689.png)

② 再次发送消息

修改发送消息的内容，以区分之前发送的消息，消费者端能够正常消费

控制台有报错是因为有节点下线，属于正常情况

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151867420-2b3d4f30-5d64-4fcf-bb7c-cc08500e2c8f.png)

### 流式队列
RabbitMQ在 3.9.x 推出的新特性

**工作机制**：

在一个仅追加的日志文件内保存所发送的消息

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151867392-cf969706-f991-4e25-a4d4-fc304a0f7a59.png)

给每个消息都分配个偏移页，即使消息被消费端消费掉，消息依然保存在日志文件中，可重复消费

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151867428-ab6946ee-6ca5-47f2-afe1-03a4bf1d4fc4.png)

**总体评价**

+ 从客户端支持角度来说，生态尚不健全
+ 从使用习惯角度来说，和原有队列用法不完全兼容
+ 从竞品角度来说，**像Kafka，但远远比不上Kafka**
+ 从应用场景角度来说： 
    - 经典队列：适用于系统内部异步通信场景
    - 流式队列：适用于系统间跨平台、大流量、实时计算场景(Kafka主场)
+ 使用建议：Stream队列在目前企业实际应用非常少，真有特定场景需要使用肯定会倾向于使用Kafka,而不是RabbitMQ Stream
+ 未来展望：Classic Queue已经有和Quorum Queue合二为一的趋势,Stream也有加入进来整合成一种队列的趋势，但Stream内部机制决定这很难

#### 使用步骤
##### 启用插件
> 说明：只有启用了Stream插件，才能使用流式队列的完整功能
>

在集群每个节点中依次执行如下操作：

```plain
rabbitmq-plugins enable rabbitmq_stream 


rabbitmqctl stop_app
rabbitmqctl start_app


rabbitmq-plugins list
```

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151867571-70972b62-354f-41dc-b336-4624d948c1c3.png)

##### 负载均衡
> 配置文件位置：/etc/haproxy/haproxy.cfg
>

在配置文件未尾增加如下内容：

```plain
frontend rabbitmq_stream_frontend
bind 192.168.10.66:33333
mode tcp
default_backend rabbitmq_stream_backend

backend rabbitmq_stream_backend
mode tcp
balance roundrobin
server rabbitmq1 192.168.10.66:5552 check
server rabbitmq2 192.168.10.88:5552 check
server rabbitmq3 192.168.10.99:5552 check
```

重启HAProxy

```plain
systemctl restart haproxy
```

##### JAVA代码
Stream专属Java客户端官方网址：https://github.com/rabbitmq/rabbitmq-stream-java-client  
Stream专属Java客户端官方文档网址：https://rabbitmq.github.io/rabbitmq-stream-java-client/stable/htmlsingle/

###### 引入依赖
```plain
<dependencies>
    <dependency>
        <groupId>com.rabbitmq</groupId>
        <artifactId>stream-client</artifactId>
        <version>0.15.0</version>
    </dependency>

    <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>slf4j-api</artifactId>
        <version>1.7.30</version>
    </dependency>

    <dependency>
        <groupId>ch.qos.logback</groupId>
        <artifactId>logback-classic</artifactId>
        <version>1.5.8</version>
    </dependency>
</dependencies>
```

###### 创建Stream
> 不需要创建交换机
>

**① 代码方式创建**

```plain
Environment environment = Environment.builder()
    .host("192.168.10.66")
    .port(33333)
    .username("shiguang")
    .password("123456")
    .build();

environment.streamCreator().stream("stream.shiguang.test").create();
```

**② ManagementUlt创建**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151867564-16b09be8-c4ef-4e70-8ade-6906a166bc36.png)

###### 生产端程序
**① 内部机制说明**  
[1] 官方文档

> Internally，the Environment will query the broker to find out about the topology of the stream and will create or re-use a connection to publish to the leader node of the stream.
>

翻译：

> 在内部，Environment将查问brokerl以了解流的拓扑结构，并将创建或重用连接以发布到流的leader节点。
>

[2] 解析

+ 在Environment中封装的连接信息仅负责连接到 broker
+ Producer在构建对象时会访问broker拉取集群中 Leader 的连接信息
+ 将来实际访问的是集群中的 Leader 节点
+ Leader的连接信息格式是：节点名称:端口号

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151867768-159bf768-0b17-4780-91bd-2ca3dc187d48.png)

[3] 配置

> 文件位置： C:\Windows\System32\drivers\etc
>

为了让本机的应用程序知道Leader节点名称对应的IP地址，我们需要在**本地**配置hosts文件，建立从节点名称到P地址的映射关系

```plain
# rabbitmq 测试
192.168.10.66 node01
192.168.10.88 node02
192.168.10.99 node03
```

**② 示例代码**

```plain
Environment environment = Environment.builder()
    .host("192.168.10.66")
    .port(33333)
    .username("shiguang")
    .password("123456")
    .build();

Producer producer = environment.producerBuilder()
    .stream("stream.shiguang.test")
    .build();

byte[] messagePayload = "hello rabbit stream".getBytes(StandardCharsets.UTF_8);

CountDownLatch countDownLatch = new CountDownLatch(1);

producer.send(
    producer.messageBuilder().addData(messagePayload).build(),
    confirmationStatus -> {
        if (confirmationStatus.isConfirmed()) {
            System.out.println("[生产者端]the message made it to the broker");
        } else {
            System.out.println("[生产者端]the message did not make it to the broker");
        }
        countDownLatch.countDown();
    });
countDownLatch.await();
producer.close();
environment.close();
```

###### 消费端程序
```plain
Environment environment = Environment.builder()
    .host("192.168.10.66")
    .port(33333)
    .username("shiguang")
    .password("123456")
    .build();

environment.consumerBuilder()
    .stream("stream.shiguang.test")
    .name("stream.shiguang.test.consumer")
    .autoTrackingStrategy()
    .builder()
    .messageHandler((offset, message) -> {
        byte[] bodyAsBinary = message.getBodyAsBinary();
        String messageContent = new String(bodyAsBinary);
        System.out.println("[消费者] messagecontent = " + messageContent + " offset = " + offset.offset());
    })
    .build();
```

##### 指定偏移量消费
###### 偏移量
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151867717-54715c13-97ee-4c6c-9e2f-4865faed2ddd.png)

###### 官网文档说明
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151868128-8a8b3d6e-9d5d-4a47-90ab-f8459d1a1983.png)

###### 指定Offset消费
```java
Environment environment = Environment.builder()
    .host("192.168.10.66")
    .port(33333)
    .username("shiguang")
    .password("123456")
    .build();

CountDownLatch countDownLatch = new CountDownLatch(1);

Consumer consumer = environment.consumerBuilder()
    .stream("stream.shiguang.test")
    .offset(OffsetSpecification.first())
    .messageHandler((offset, message) -> {
        byte[] bodyAsBinary = message.getBodyAsBinary();
        String messageContent = new String(bodyAsBinary);
        System.out.println("[消费者端] messagecontent = " + messageContent);
        countDownLatch.countDown();
    })
    .build();

countDownLatch.await();
consumer.close();
```

###### 对比
+ autoTrackingStrategy方式：始终监听Stream中的新消息（狗狗看家，忠于职守）
+ 指定偏移量方式：针对指定偏移量的消息消费之后就停止（狗狗叼飞盘，叼回来就完）

## Federation插件
### 简介
Federation插件的设计目标是使RabbitMQ在不同的Broker节点之间进行消息传送而无须建立集群。

它可以在不同的管理域中的Broker或集群间传递消息，这些管理域可能设置了不同的用户和vhost,也可能运行在不同版本的RabbitMQ和Erang上，Federation基于AMOP 0-9-1协议在不同的Broker之间进行通信。并且设计成能够密忍不稳定的网络连接情况。

### Federation交换机
#### 总体说明
+ 各节点操作：启用联邦插件
+ 下游操作： 
    - 添加上游连接端点
    - 创建控制策略

#### 准备工作
为了执行相关测试，我们使用Dockert创建两个RabbitMQ实例。  
**特别提示**：由于Federation机制的最大特点就是跨集群同步数据，所以这两个Docker容器中的RabbitMQ实例不加入集群！！！是两个**独立的broker实例**。

```shell
docker run -d \
--name rabbitmq-shenzhen \
-p 51000:5672 \
-p 52000:15672 \
-v rabbitmq-plugin:/plugins \
-e RABBITMQ_DEFAULT_USER=guest \
-e RABBITMQ_DEFAULT_PASS=123456 \
rabbitmq:3.13-management


docker run -d \
--name rabbitmq-shanghai \
-p 61000:5672 \
-p 62000:15672 \
-v rabbitmq-plugin:/plugins \
-e RABBITMQ_DEFAULT_USER=guest \
-e RABBITMQ_DEFAULT_PASS=123456 \
rabbitmq:3.13-management
```

#### 启用联邦插件
在上游、下游节点中都需要开启。  
Docker容器中的RabbitMQ已经开启了rabbitmq_federation,还需要开启rabbitmq_federation_management

```plain
docker exec -it <container_name> /bin/bash

rabbitmq-plugins enable rabbitmq_federation

rabbitmq-plugins enable rabbitmq_federation_management
```

rabbitmq_federation_management插件启用后会在Management Ul的Admin选项卡下看到：

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151867967-90f2590b-23a3-40f8-9c1c-270ae9ee7801.png)

#### 添加上游连接端点
在下游节点填写上游节点的连接信息：

```plain
shiguang.upstream

amqp://guest:[redacted]@192.168.10.66:51000
```

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151867945-42c3492a-7d64-4fda-a1d6-0aaff6dcf7c2.png)

#### 创建控制策略
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151867981-ee779b4d-7e4e-4ced-b14c-c57f727f4d28.png)

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151868202-4881233c-d952-42f6-8c67-02e0bc177670.png)

详细配置如下：

```plain
police.federation.exchange


^federated\.


Exchanges


10


federation-upstream = shiguang.upstream
```

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151868251-c3809e5f-14dd-4ca4-81f6-6c123ee1fada.png)

#### 测试
**① 测试计划**  
**特别提示**：

+ 普通交换机和联邦交换机名称要一致
+ 交换机名称要能够和策略正则表达式匹配上
+ 发送消息时，两边使用的路由键也要一致
+ 队列名称不要求一致

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151868326-4688af1f-e3ed-461e-b414-dcb30009698a.png)

**② 创建组件**

| 所在机房 | 交换机名称 | 路由键 | 队列名称 |
| --- | --- | --- | --- |
| 深圳机房（上游） | federated.exchange.demo | routing.key.demo.test | queue.normal.shenzhen |
| 上海机房（下游） | federated.exchange.demo | routing.key.demo.test | queue.normal.shanghai |


创建组件后可以查看一下联邦状态，连接成功的联邦状态如下：

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151868310-3655960c-1f63-4296-b790-7b602e2e51e9.png)

③ 发布消息执行测试

在上游节点向交换机发布消息：

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151868473-0c45318b-3521-4a68-b01d-f77119bf8307.png)

下游两个队列消息总量均变成了1

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151868591-aeb291ea-2639-4446-9cea-5cc667beb50d.png)

### Federation队列
#### 总体说明
Federation队列和Federation交换机的最核心区别就是：

+ Federation Police作用在交换机上，就是Federation交换机
+ Federation Police作用在队列上，就是Federation队列

#### 创建控制策略
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151868595-21bfa54c-38e0-4e5e-a137-40dc05ef0dc2.png)  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151868814-eaf3531f-8d4a-49a6-894f-2efdded6dbc5.png)  
详细配置如下：

```plain
police.federation.queue


^fed\.queue\.


Queues


10


federation-upstream = shiguang.upstream
```

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151868750-80e448c4-75f8-45a0-89dc-ef3c43567bf8.png)

#### 测试
**① 测试计划**  
上游节点和下游节点中队列名称是相同的，只是下游队列中的节点附加了联邦策略而已

| 所在机房 | 交换机名称 | 路由键 | 队列名称 |
| --- | --- | --- | --- |
| 深圳机房（上游） | exchange.normal.shenzhen | routing.key.normal.shenzhen | fed.queue.demo |
| 上海机房（下游） | —— | —— | fed.queue.demo |


**② 创建组件**  
上游节点都是常规操作，此处省略。重点需要关注的是下游节点的联邦队列创建时需要指定相关参数：  
创建组件后可以查看一下联邦状态，连接成功的联邦状态如下：

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151868751-7bcef6d1-b1a3-4174-ac83-045a857d2ed0.png)

**③ 执行测试**  
在上游节点向交换机发布消息：  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151868887-b2cf357e-6073-4f99-a85b-b9d31153ecbb.png)

但此时发现下游节点中联邦队列并没有接收到消息

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151868909-a34b2be2-30bb-4ebb-9caa-d511fadf46f2.png)

这是为什么呢？这里就体现出了联邦队列和联邦交换机工作逻辑的区别。  
对联邦队列来说，如果没有监响联队列的消费端程序，它是不会到上游去拉取消息的！  
如果有消费端监听联邦队列，那么首先消费联邦队列自身的消息；**如果联邦队列为空，这时候才会到上游队列节点中拉取消息。**  
所以现在的测试效果需要消费端程序配合才能看到：

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151869035-df310c58-e3da-4bed-a3b8-5d4f3badb47f.png)

## Shovel插件
> Shovel 是铲子的意思，把消息铲走，从源节点移至目标节点，源节点将收不到消息
>

### 启用Shovel插件
```plain
docker exec -it <container_name> /bin/bash

rabbitmq-plugins enable rabbitmq_shovel

rabbitmq-plugins enable rabbitmq_shovel_management
```

启用后管理界面可以看到如下配置：

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151869055-450e657a-c076-47de-bf33-d7a11bacc314.png)

### 配置Shovel
> 不区分上下游，在哪个节点配置都可以
>

```plain
shiguang.shovel.config

amqp://guest:123456@192.168.10.66:51000

queue.shovel.demo.shenzhen

amqp://guest:123456@192.168.10.66:61000

queue.shovel.demo.shanghai
```

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151869308-3a94faf4-67f3-4e9c-ad62-61c9c19f59ec.png)

### 测试
#### **测试计划**
| 所在机房 | 交换机名称 | 路由键 | 队列名称 |
| --- | --- | --- | --- |
| 深圳机房 | exchange.shovel.test | exchange.shovel.test | queue.shovel.demo.shenzhen |
| 上海机房 | —— | —— | queue.shovel.demo.shanghai |


#### 测试效果
**① 发布消息**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151869333-165134c6-d883-4d38-8ae1-ea89b61a4ce9.png)

**② 源节点**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151869333-a8ac4ed6-ce14-48cd-9c7d-7d48173ba6da.png)

**③ 目标节点**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151869368-410748ad-f677-4cab-a48e-b70d6a7797d6.png)

如果测试效果与视频中演示不一致，可检查配置的账号密码是否正确  
可用 `docker logs <container_name/container_id> 查看日志`

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151869374-60eeca65-2abd-472a-885f-68f82379dff9.png)

可点击 Shovels Name 查看配置详情，例如此处我错误地将用户名写为`gust`，正确应为 `guest`

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151869583-07a0a5d2-9492-4c21-b0b3-e4a3d90610ac.png)

如果账号密码配置错误导致无法连接，实际测试效果将和普通队列相同

源节点：  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151869683-10f9159e-2303-48cd-b053-254bf9433d6a.png)

目标节点：  
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/58546395/1759151869702-b6ded2aa-0b23-470b-ad1d-cb03567cbc26.png)



> 来自: [【尚硅谷】RabbitMQ_rabbitmq尚硅谷-CSDN博客](https://blog.csdn.net/qq_50082325/article/details/144492156)
>





