# SpringCloud 实战项目一

## 1 多模块

- product-server: 所有业务逻辑
- product-client: 对外暴露的接口
- product-common: 公用的对象

![项目依赖关系](https://cdn.jsdelivr.net/gh/Clarencezero/poi/project relation.png)

## 2 Eureka

Netflix 开源。

### 2.1 原理

![Eureka](https://cdn.jsdelivr.net/gh/Clarencezero/poi/eureka.png)

上图是来自 eureka 的官方架构图，这是基于集群配置的 eureka； 

- 处于不同节点的 eureka 通过 Replicate 进行数据同步 
- Application Service 为服务提供者 
- Application Client 为服务消费者 
- Make Remote Call 完成一次服务调用

服务启动后向 Eureka 注册，Eureka Server 会将注册信息向其他 Eureka Server 进行同步，当服务消费者要调用服务提供者，则向服务注册中心获取服务提供者地址，然后会将服务提供者地址缓存在本地，下次再调用时，则直接从本地缓存中取，完成一次调用。

当服务注册中心 Eureka Server 检测到服务提供者因为宕机、网络原因不可用时，则在服务注册中心将服务置为`DOWN`状态，并把当前服务提供者状态向订阅者发布，订阅过的服务消费者更新本地缓存。

服务提供者在启动后，周期性（默认 30 秒）向 Eureka Server 发送心跳，以证明当前服务是可用状态。Eureka Server 在一定的时间（默认 90 秒）未收到客户端的心跳，则认为服务宕机，注销该实例。

### 2.2 Eureka 自我保护机制

在默认配置中, Eureka Server 在默认 90S 内没有得到客户心跳,则注销该实例。但是因为网络通信会面临着各种各样的问题, 如服务状态正常,但是网络分区故障时,Eureka Server 会注销该实例, 这很危险, 为了解决这个问题, Eureka 会有自我保护机制,

```properties
eureka.server.enable-self-preservation=true
```

原理: 当 Eureka Server 节点在短时间内丢失过多的客户端时 (可能发送了网络故障), 那么这个节点将进入自我保护模式,不再注销任何微服务,当网络故障恢复后,该节点会自动退出自我保护模式。

Eureka 保证的是 AP,而 Zookeer 保证的 CP。Zookeeper 当某个节点不可用时, 它会在内部举行选举, 但是这个选举过程太长了,会导致注册服务瘫痪。

- A: 可用性
- C: 一致性
- P: 分区容错性

### 2.3 Eureka 如何保证自身高可用

Eureka 各个节点都是平等的,几个节点挂掉了不会影响正常的工作,剩余的节点依然可以提供注册和查询服务。只要 Eureka 客户端在向某个 Eureka Server 注册时发现连接失败,则会自动切换到其它节点, 只要有一台 Eureka 还在, 就能保证注册服务高可用, 但是可能查询不到最新的信息。

![Eureka高可用](https://cdn.jsdelivr.net/gh/Clarencezero/poi/Eureka high.png)

```yml
# Eureka 服务端的高可用需要把自身注册到另外的一台服务器上去,如当前端口 8761,另一台为 8762
spring:
  profiles: peer2
  application:
    name: eureka-b
eureka:
  instance:
    hostname: peer2
  client:
    serviceUrl:
      defaultZone: http://peer1:8761/eureka/
    register-with-eureka: false
```



## 3 应用间通信 Ribbon

客户端负载均衡器: Ribbon

- RestTemplate

  ```java
  @Component
  public class RestTemplateConfig {
      @Bean
      @LoadBalanced
      public RestTemplate getRestTemplate() {
          return new RestTemplate();
      }
  }	
  
  
  // 通过负载均衡查找 Eureka 里面的服务, 然后返回 IP 地址
  ResultVO resultVO = new ResultVO();
  
  // 第一种方式直接通过 RestTemplate 调用绝对 URL
  
  // 第二种方式通过利用 loadBalancerClient 通过应用名称获取 URL,然后再使用 RestTemplate
  // RestTemplate restTemplate = new RestTemplate();
  // ServiceInstance instance = loadBalancerClient.choose(PRODUCT_HOST_NAME);
  // String url = String.format("http://%s:%s", instance.getHost(), instance.getPort() + "/product/list");
  // String forObject = restTemplate.getForObject(url, String.class);
  
  
  // 第三种方式
  String forObject = restTemplate.getForObject("http://" + PRODUCT_HOST_NAME + "/product/list", String.class);
  ```

  

- Feign

  ```java
  // 1. 添加依赖
   <dependency>
   	<groupId>org.springframework.cloud</groupId>
   	<artifactId>spring-cloud-starter-feign</artifactId>
   	<version>1.4.7.RELEASE</version>
   </dependency>
   
   // 2.Enable
   @EnableFeignClients
   
   // 3.定义好需要调用的接口
   /**
   * name: 指定访问应用的接口
   */
  @FeignClient(name = "product")
  public interface ProductClient {
      @GetMapping("/product/list")
      String listProduct();
  }
  ```
  
  - 声明式 REST 客户端 (伪 RPC)
  
- 采用了基于接口的注解
  
- Zuul

核心:

- 服务发现
- 服务选择规则
- 服务监听

组件:

- ServerList: 获取所有可用的服务列表
- ServerListFilter: 过滤部分地址
- IRule: 选择最终目标返回给客户端

```yaml
# 修改负载均衡方式
PRODUCT:
  ribbon:
  	NFLoadBalanceRuleClassName: com.netflix.loadbalancer.RandomRule
```



## 4. 统一配置中心 ConfigServer

### 4.1 新建一个项目

#### 4.1.1 添加注解和配置

```yml
@EnableDiscoveryClient
@EnableConfigServer

spring:
  application:
    name: config
  cloud:
    config:
      server:
        git:
          uri: https://gitee.com/SpringCloud_Sell/config-repo
          username: lly835@163.com
          password: T#27h*E$cg@%}j
          basedir: /Users/admin/code/myProjects/java/imooc/SpringCloud_Sell/config/basedir
          
          
/{name}-{profiles}.yml
/{label}/{name}-{profiles}.yml
```

#### 4.1.2 SpringCloud Bus 自动刷新配置

![SpringCloud Bus示意图](https://cdn.jsdelivr.net/gh/Clarencezero/poi/all config.png)

使用消息队列 (RabbitMQ)

```java
// 1.Config 和使用 BUS 的其他子项目中 OrderPOM.xml 添加组件
spring-cloud-starter-bus-amqp

// 2.在 Config 项目中暴露接口
management.endpoints.web.expose: "*" // 暴露全部

// 3.在诸如 Order 等项目中添加注解
@RestController
@RequestMapping("/env")
@RefreshScope
public class EnvConfig {
	@Value("${env}")
	private String env;
	@GetMapping("/print")
	private String print() {
		return env;
	}
}

// 4.使用 webhooks 设置自动实现,
```



![Rabbit 监控界面 ](https://cdn.jsdelivr.net/gh/Clarencezero/poi/Rabbit.png)



## 5 队列的常见形态 MQ

- 请求/异步响应
- 通知
- 消息

### 5.1 MQ 应用场景

- 异步处理
- 日志处理
- 流量削峰
- 应用解耦

### 5.2 RabbitMQ 基本使用

```yaml
spring:
  rabbitmq:
  	host: localhost
  	port: 5672
  	username: guest
  	password: guest
```



```java
// 接收方
@Component
public class MqReceiver {
	@RabbitListener(queue="myQueue")
	@RabbitListener(queuesToDeclare = @Queue("myQueue")) //自动创建队列
	
	//自动创建 Exchange 和 Queue 绑定
	@RabbitListener(bindings = @QueueBinding = (
		key = ""
		value = @Queue("myQueue"),
		exchange=@Exchange("myExchange")
	))
	public void process(String message) {
		// 处理消息逻辑
	}
}

// 发送方则需要添加 exchange


```

```java
// 发送方,如果接收方没有配置队列的话, 需要提前建立队列
@Autowired
private AmqpTemplate amqpTemplate;
public void send() {
 	// routingKey, message
	amqpTemplate.convertAndSent("myQueue", "now")
}
```



![exchange](https://cdn.jsdelivr.net/gh/Clarencezero/poi/RabbitMQ.png)



## 6 Spring Cloud Stream

![Spring Cloud Stream](https://cdn.jsdelivr.net/gh/Clarencezero/poi/Stream.png)

Spring Cloud Stream 为微服务应用构建**消息驱动**能力的框架，应用程序通过 input，output 来与 Stream 中的 Binder 交互，Stream 中的 Binder 与中间件（Middleware）交互，Binder 是 stream 的抽象概念，是应用与消息中间件的粘合剂，使用 Stream 最大的方便之处莫过于对消息中间件的进一步封装，可以做到代码层面对中间件的无感知，甚至做到动态切换中间件。

- RabbitMQ
- Kafka

```java
spring-cloud-starter-stream-rabbit
// 自动在消息队列创建

// 定义分组,与 Kafka 的组类似
stream:bindings:myMessage:group:order

// 1.
public interface StreamClient {
    @Input("myMessage")
    SubscribableChannel input1();

    @Output("myMessage")
    MessageChannel output1();
    
    
    // 用做消息回复
    @Input("myMessage2")
    SubscribableChannel input2();

    @Output("myMessage2")
    MessageChannel output2();
}

// 2.
/**
 * 接收端
 */
@Component
@EnableBinding(StreamClient.class)
public class StreamReceiver {
    @StreamListener("myMessage")
    @SendTo("myMessage2")
    public void process(Object message) {
        System.out.println(message);
    }
}

/**
 * 接收端 2
 */

    @StreamListener("myMessage2")
    public void process2(Object message) {
        System.out.println(message);
    }


// 3.
@RestController
public class SendMessageController {
	@Autowired
	private StreamClient streamClient;
	
	@GetMapping("sendMessage")
	public void process() {
		streamClient.output().send(MessageBuilder.withPayLoad("message"));
	}
}

// 4.配置
stream:
  bindings:
	myMessage:
	  group: order
	  	content-type: application/json
	  	
//5. 回复

```

作用就是简化对消息的操作



## 7 服务网关和 Zuul

![服务网关 ](https://cdn.jsdelivr.net/gh/Clarencezero/poi/server net.png)

### 7.1 要求: 

- 稳定性、高可用
- 性能、并发性
- 安全性
- 扩展性

### 7.2 常见的网关方案:

- Nginx+Lua
- Kong
- Tyk。Go 语言开发
- Spring cloud Zuul
  - 路由+过滤器=Zuul
  - 核心是一系列过滤器

### 7.3 Zuul

四种过滤器 API:

- 前置 (Pre)
  - 限流
  - 鉴权
  - 参数校验调整
- 后置 (Post)
  - 统计
  - 跨域
- 路由 (Route)
- 错误 (Error)

![Zuul 网关 ](https://cdn.jsdelivr.net/gh/Clarencezero/poi/Zuul guan.png)



![请求生命周期 ](https://cdn.jsdelivr.net/gh/Clarencezero/poi/request life.png)

```yaml
@EnableZuulProxy // 就可以直接通过网关访问后台

// 自定义路径
zuul:
  routes:
  	myProduct:
  	  path: /myProduct/**
  	  serviceId: product
#简洁写法
zuul:
  product: /myProduct/**
ignored-patterns:
  - /**/product/listForOrder
# 权限配置
management:
  security:
  	enabled: false
```



#### 7.3.1 Cookie 和动态路由

关键是*sensitiveHeaders*

#### 7.3.2 Zuul 的高可用

多个 Zuul 节点注册到 Eureka Server

Nginx 和 Zuul 混搭

#### 7.3.3 整个项目架构图

![项目架构图](https://cdn.jsdelivr.net/gh/Clarencezero/poi/system structure.png)

```java
@Component
public class TokenFilter extends ZuulFilter {
	...
	@Override
	public Object run() {
		// 逻辑实现
	}
}
```

### 7.4  限流 (令牌桶)

![令牌桶 ](https://cdn.jsdelivr.net/gh/Clarencezero/poi/token tong.png)

```java
可以使用 Google 开源的 RateLimiter
private static final RateLimiter RATE_LIMITER = RateLimiter.create(100(每秒钟放多少个令牌));

public Object run() {
	if (!RATE_LIMITER.tryAcquire()) {
		// 抛出异常
	}
}
```



// spring-cloud-zuul-ratelimit

### 7.5 权限校验

注:P53 多模块

需求: 

- /order/create: 只能买家访问 (cookie 里有 openid)
- /order/finish: 只能卖家访问 (cookie 里有 token,并且对应的 redis 中的值)
- /product/list: 都可以访问



## 8 服务容错 Hystrix

### 8.1 雪崩效应

### 8.2 Spring Cloud Hystrix

- 服务降级
  - 优先核心服务,非核心服务不可用或弱可用
  - 通过 HystrixCommand 注解指定
  - fallbackMethod(回退函数) 中具体实现降级逻辑
- 服务熔断
- 依赖隔离
- 监控 (Hystrix Dashboard)

### 8.3 使用 Hystrix

```
1. 导入依赖 spring-cloud-starter-hystrix
2. @EnableCircuitBreaker(可以用@SpringCloudApplication 替代)
3. 服务超时。
```

![](https://cdn.jsdelivr.net/gh/Clarencezero/poi/Hystrix-1565792233643.png)

![](https://cdn.jsdelivr.net/gh/Clarencezero/poi/server over.png)

### 8.4 依赖隔离

- 线程池隔离

- Hystrix 自动实现了依赖隔离

- 服务熔断

  ![熔断 ](https://cdn.jsdelivr.net/gh/Clarencezero/poi/rongduan1.png)

  

![熔断 ](https://cdn.jsdelivr.net/gh/Clarencezero/poi/rongduan2.png)

circuit Breaker: 断路器。自我保护,及时切断故障服务	

- 重试机制
  - 长时间服务不可用,则使用断路器,就可以将受保护的服务封装在一个可以监控故障的断路器里面,当故障达到一定的值,断路器就会跳闸,断路器对象返回错误。

![断路器 ](https://cdn.jsdelivr.net/gh/Clarencezero/poi/duanluqi.png)

- circuitBreaker.sleepWindowInMilliseconds。休眠时间窗
- circuitBreaker.requestVolume Threshold
- circuitBreaker.errorThresholdPercentage。

![Hystrix 配置 ](https://cdn.jsdelivr.net/gh/Clarencezero/poi/hystrix config.png)

### 8.5 feign-hystrix 的使用

1. feign 已经包含了 hystrix

```yaml
feign:
  gystrix:
  	enabled: true
```

![Feigh-hystrix](https://cdn.jsdelivr.net/gh/Clarencezero/poi/feign-hystrix.png)

### 8.6 可视化组件

```yaml
spring-cloud-starter-hystrix-dashboard
@EnableHystrixDashboard
management:
  context-path: /
```

### 8.7 服务追踪

- 链路监控 Spring Cloud Sleuth

![链路追踪](https://cdn.jsdelivr.net/gh/Clarencezero/poi/lianlu.png)

- Zipkin 图形化界面 (Docker)

```yaml
1. 添加依赖
spring-cloud-starter-zipkin
2. 配置
zipkin:
  base-url: http://localhost:9411/
spring:
  sleuth:
  	sampler:
  	  percentage: 1
```



- 分布式追踪系统核心步骤
  - 数据采集
  - 数据存储
  - 查询展示

- OpenTracing(CNCF)

  - 统一的标准

- Annotation

  - 事件类型
    - CS(client Send): 客户端发起请求的时间
    - CR(Client Received): 客户端收到处理完请求的时间
    - SS(Server Send): 服务端处理完逻辑的时间
    - SR(Server Received): 服务端收到调用端请求的时间

  - 客户端调用的时间= CR-CS
  - 服务端调用的时间=SR-SS

![Zipkin架构](https://cdn.jsdelivr.net/gh/Clarencezero/poi/Zipkin structure.png)

### 8.8 Zipkin

- traceId
- spanId
- parentId

### 8.9 rancher

Rancher 是一个开源的企业级全栈化窗口部署及管理平台





## 9. 分布式唯一 ID 几种生成方案

### 1. 分布式 ID 特性

- 唯一性：确保生成的 ID 是全网唯一的。
- 有序递增性：确保生成的 ID 是对于某个用户或者业务是按一定的数字有序递增的。
- 高可用性：确保任何时候都能正确的生成 ID。
- 带时间：ID 里面包含时间，一眼扫过去就知道哪天的交易。

### 2. 方案

#### 2.1 UUID

算法的核心思想是结合机器的网卡、当地时间、一个随记数来生成 UUID。

- 优点：本地生成，生成简单，性能好，没有高可用风险
- 缺点：长度过长，存储冗余，且无序不可读，查询效率低

#### 2.2 数据库自增 ID

使用数据库的 id 自增策略，如 MySQL 的 auto_increment。并且可以使用两台数据库分别设置不同步长，生成不重复 ID 的策略来实现高可用。

- 优点：数据库生成的 ID 绝对有序，高可用实现方式简单
- 缺点：需要独立部署数据库实例，成本高，有性能瓶颈

#### 2.3 批量生成 ID

一次按需批量生成多个 ID，每次生成都需要访问数据库，将数据库修改为最大的 ID 值，并在内存中记录当前值及最大值。

- 优点：避免了每次生成 ID 都要访问数据库并带来压力，提高性能
- 缺点：属于本地生成策略，存在单点故障，服务重启造成 ID 不连续

#### 2.4 Redis 生成 ID

Redis 的所有命令操作都是单线程的，本身提供像 **incr** 和 **increby** 这样的自增原子命令，所以能保证生成的 ID 肯定是唯一有序的。

- 优点：不依赖于数据库，灵活方便，且性能优于数据库；数字 ID 天然排序，对分页或者需要排序的结果很有帮助。
- 缺点：如果系统中没有 Redis，还需要引入新的组件，增加系统复杂度；需要编码和配置的工作量比较大。

考虑到单节点的性能瓶颈，可以使用 Redis 集群来获取更高的吞吐量。假如一个集群中有 5 台 Redis。可以初始化每台 Redis 的值分别是 1, 2, 3, 4, 5，然后步长都是 5。各个 Redis 生成的 ID 为：

```
A：1, 6, 11, 16, 21
B：2, 7, 12, 17, 22
C：3, 8, 13, 18, 23
D：4, 9, 14, 19, 24
E：5, 10, 15, 20, 25
```

随便负载到哪个机确定好，未来很难做修改。步长和初始值一定需要事先确定。使用 Redis 集群也可以方式单点故障的问题。

另外，比较适合使用 Redis 来生成每天从 0 开始的流水号。比如订单号 = 日期 + 当日自增长号。可以每天在 Redis 中生成一个 Key ，使用 INCR 进行累加。

#### 2.5. Twitter 的 snowflake(雪花) 算法

Twitter 利用 zookeeper 实现了一个全局 ID 生成的服务 Snowflake：[github.com/twitter/sno…](https://link.juejin.im/?target=https%3A%2F%2Fgithub.com%2Ftwitter%2Fsnowflake)

![Snowflake](https://cdn.jsdelivr.net/gh/Clarencezero/poi/snowflake.jpg)

如上图的所示，Twitter 的 Snowflake 算法由下面几部分组成：

- **1 位符号位：**

由于 long 类型在 java 中带符号的，最高位为符号位，正数为 0，负数为 1，且实际系统中所使用的 ID 一般都是正数，所以最高位为 0。

- **41 位时间戳（毫秒级）：**

需要注意的是此处的 41 位时间戳并非存储当前时间的时间戳，而是存储时间戳的差值（当前时间戳 - 起始时间戳），这里的起始时间戳一般是 ID 生成器开始使用的时间戳，由程序来指定，所以 41 位毫秒时间戳最多可以使用 `(1 << 41) / (1000x60x60x24x365) = 69 年`。

- **10 位数据机器位：**

包括 5 位数据标识位和 5 位机器标识位，这 10 位决定了分布式系统中最多可以部署 `1 << 10 = 1024` s 个节点。超过这个数量，生成的 ID 就有可能会冲突。

- **12 位毫秒内的序列：**

这 12 位计数支持每个节点每毫秒（同一台机器，同一时刻）最多生成 `1 << 12 = 4096 个 ID`

加起来刚好 64 位，为一个 Long 型。

- 优点：高性能，低延迟，按时间有序，一般不会造成 ID 碰撞
- 缺点：需要独立的开发和部署，依赖于机器的时钟

#### 2.6 百度 UidGenerator

UidGenerator 是百度开源的分布式 ID 生成器，基于于 snowflake 算法的实现，看起来感觉还行。不过，国内开源的项目维护性真是担忧。

#### 2.7 美团 Leaf

Leaf 是美团开源的分布式 ID 生成器，能保证全局唯一性、趋势递增、单调递增、信息安全，里面也提到了几种分布式方案的对比，但也需要依赖关系数据库、Zookeeper 等中间件。

















































