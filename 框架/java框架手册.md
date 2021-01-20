# Spring家族

## Spring Aop

实现步骤：

1. 导入aop依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```

2. 新建`Action.java`

```java
import java.lang.annotation.*;

@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Action {
    String name();
}
```

3. 新建`Target.java`

```java
import java.lang.annotation.Documented;
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;

@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.ANNOTATION_TYPE)
public @interface Target {
    ElementType[] value();
}
```

4. 新建切面`LogAspect.java`

```java
import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;
import org.aspectj.lang.annotation.Pointcut;
import org.aspectj.lang.reflect.MethodSignature;
import org.springframework.stereotype.Component;

import java.lang.reflect.Method;

@Aspect
@Component
public class LogAspect {

    @Pointcut("@annotation(com.lucfzy.rocketmqdemo.aop.Action)")
    public void annotationPoinCut(){}

    // 在方法运行前运行
    @Before("annotationPoinCut()")
    public void after(JoinPoint joinPoint){
        MethodSignature signature = (MethodSignature) joinPoint.getSignature();
        Method method = signature.getMethod();
        Action action = method.getAnnotation(Action.class);
        System.out.println("注解式拦截 "+action.name());
    }
```

5. 构建测试类，测试方法

```java
	@Action(name = "注解式拦截的add操作")
    @Override
    public ConsumeConcurrentlyStatus consumeMessage(List<MessageExt> list, ConsumeConcurrentlyContext context) {
        if (CollectionUtils.isEmpty(list)){
            return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
        }
```

## 注册中心

### nacos

- 实现动态服务发现
- 服务配置
- 服务元数据
- 流量管理

**当一个微服务下线时的一些现象？**

首先会等待一段时间，这个时候因为微服务要定期发送心跳报文，如果这段时间没有发送心跳报文，那么服务列表中的健康实例数将会减一，并且会触发阈值保护，看看微服务是否可以活起来，并不会立刻剔除微服务，而是令触发阈值保护为true，并且该微服务一栏会变为红色，即警告提示。那么再过一段时间，如果微服务还是没有发送心跳报文，那么实力数目减一。表示该微服务已失去联系。这个和eureka的自我保护机制很像。但是服务目录并不会删除，以防后面服务上线可以直接开启该条目即可。

上述大致过程：实例健康数-1 并且 触发阈值保护(true)---->实力数目-1

**服务列表---改变分组名称**

同一服务名和同一分组名称下可以有多个不同端口的微服务，也就是多个实例，多个健康实例数。其中实例数一般和健康实例数是相等的。如果有微服务上下线的话，那么使得实例数目大于健康实例数。

**nacos的阈值保护和eureka的自我保护机制的异同？？？**

### eureka

> 作用： 微服务间通信框架，保证了微服务间可以进行联动，发送rest请求的基础，替代了restTemplate模板方法。

注意： `默认配置下`，feign最大允许的服务间超时为**992ms**，大约是1s钟，这个配置记得一定要改，因为到生产环境上可能微服务间的调用最大忍耐限度为20s，1s钟这个时间单单查一个数据库的时间都比这个多了。

---

#### 调用方

```java
@EnableEurekaClient // 启用eureka client将自己注册进入eureka
@EnableFeignClients(basePackages = {"com.example.server1.web.client"}) // 扫描带feign注解的包
```

feignclient 使用

```java
@org.springframework.cloud.openfeign.FeignClient(value = "server-provider", fallback = EurekaFeignServiceFailure.class) // 目标是请求server-provider微服务，如果服务间的请求时间超时，那么会触发熔断机制，默认的请求超时时间为992ms，超过这个时间，将会触发熔断，走fallback类-EurekaFeignServiceFailure.class。
// 无需component注解，系统将类注入到容器中，后面通过autowired注解直接注入使用即可，原理就是使用了动态代理的方式。类似于mybatis中mapper。    
public interface FeignClient {
    @RequestMapping(value = "/query2",method = RequestMethod.GET)
    public String queryServer02Info();
}
```

熔断类**EurekaFeignServiceFailure**的实现

```java
@Service
public class EurekaFeignServiceFailure implements FeignClient {
    @Override
    public String queryServer02Info() {
        return "网络繁忙，请稍后再试-_-。PS：服务消费者自己提供的信息";
    }
}
```

> 上面注意如果要使用hytrix的话，一定要启动hytrix组件，具体百度一下吧。。

---

##### 问题

1. 测试如果没有rollback熔断类，那么后果是

![feign调用超时无熔断报错.png](https://image.lucfzy.com/blog/feign%E8%B0%83%E7%94%A8%E8%B6%85%E6%97%B6%E6%97%A0%E7%86%94%E6%96%AD%E6%8A%A5%E9%94%99.png)

2. 如果不启用熔断配置，那么会有什么结果？
   1. 首先pom中设置为false

   ```java
   feign:
     hystrix:
       enabled: true
   ```

   2. 此时如果被调用的接口发生超时，报错如下

   ![hytrix开关未开发生了熔断报错.png](https://image.lucfzy.com/blog/hytrix%E5%BC%80%E5%85%B3%E6%9C%AA%E5%BC%80%E5%8F%91%E7%94%9F%E4%BA%86%E7%86%94%E6%96%AD%E6%8A%A5%E9%94%99.png)

   > 从上面可以看出来，未配置hytrix开关和配置了hytrix开关报错是不同的，如果是read-timeout报错，那么则是因为hytrix开关没有打开，如果开关打开了，熔断发生那么会有两种情况，一种是配置了熔断类，一种是没有配置熔断类。那么配置了熔断类就走熔断类的代码，如果没有配置则会出现 no fallback available 报错信息提示。

---

#### 被调用方

被调用方真的是出其不意的简单，我们基本上什么都不用做，正常去声明一个接口，这个接口我们可以去实现他，最终实现一下接口的功能即可。要注意的是，被调用方的微服务一定要注册进eureka中就可以了。

其他的功能都不需要，很是方便简单。



### hytrix

> hytrix是一个用于解决服务间调用超时，异常等现象发生的时候，进行重试，以及在必要时刻进行断路处理的一种断路器。

1、在spring中，hytrix的默认配置下的熔断超时时间为1s左右。对于该配置的修改尤为关键。

问题1： 如何对hytrix的超时熔断时间进行修改？

```xml
# 单独设置这一项没有用，需要设置一下ribbon
hystrix:
  command:
    default:
      execution:
        isolation:
          thread:
            timeoutInMilliseconds: 1500 # 修改了熔断时间为10s

ribbon:
  ReadTimeout: 10000 # 连接无误，被调用方代码超时，看这个时间。
  ConnectTimeout: 10000 # 连接被调用方超时，看这个时间阈值。

# 三个配置项共同构成了一条管道，管道能限制的最大超时时间，取决于最小的那一个。
# 比方说： 如果微服务间的请求连接时间为ConnectTimeout小于配置的10s，那么进入下一个管道
# 第二个则是ReadTimeout，如果发现对端的业务代码超过了10s，那么直接发生熔断，不会进入timeoutInMilliseconds的判断。
# 如果前两个都逃过了，没问题，最后看timeoutInMilliseconds参数，如果时间超过1.5s，那么同样会触发熔断机制。
```

可以使用**@hystrixcommand**注解来自定义一个熔断方法。

问题：如何指定commandkey进行超时熔断定制化？

需求：利用**hystrix**设置一个方法，让这个方法的超时时间区别于其他方法。

```java
hystrix:
  command:
    default:
      execution:
        isolation:
          thread:
            timeoutInMilliseconds: 1000 # 修改了熔断时间为10s
    helloService: # 该配置用于解决区分超时时间的个性化配置，如果commandkey为该配置将锁定其超时时间用下面的。
      execution:
        isolation:
          thread:
            timeoutInMilliseconds: 1500
```

请求方demo：

```java
@Component
public class RestTemplateRequest {
    @HystrixCommand(
            commandKey = "helloService")
    public String getResult() {
        return new RestTemplate().getForObject("http://127.0.0.1:8081/query2", String.class);
//        return new RestTemplate().getForObject("http://SERVER-PROVIDER/query2", String.class);
    }
}
```

上述代码表示：如果请求方请求被调用方的query2接口，那么将会进行接口调用判断，如果时间超过**1.5s**，那么将会触发熔断机制，因为该方法的**commandkey**为**helloService**。



问题2： hytrix的熔断原理是什么？（底层实现）



### feign

问题1： @FeignClient注解只能用在注解上面吗？

答：**是的，必须要用在接口上面。**如果不用在接口上，则会发生报错。

![feign用在非接口类上的报错信息.png](https://image.lucfzy.com/blog/feign%E7%94%A8%E5%9C%A8%E9%9D%9E%E6%8E%A5%E5%8F%A3%E7%B1%BB%E4%B8%8A%E7%9A%84%E6%8A%A5%E9%94%99%E4%BF%A1%E6%81%AF.png)

问题2：`@FeignClient`注解中的`configuration`变量作用是什么？

1. 可以声明超时信息。
2. 可以声明log级别。
3. 可以声明超时重试时间和次数等。

示例代码：

```java
@Configuration
public class FeignConfiguration {
    @Bean
    public Retryer retryer() {
        // 每隔2s重试一次，一共重试3次
        return new Retryer.Default(100,2000,3);
    }
    @Bean
    Logger.Level feignLoggerLevel() {

        return Logger.Level.BASIC;

    }
}
```

针对client接口增加相应的配置**application.yml**中：

```yml
logging:
  level:
    com:
      example:
        server1:
          web:
            client:
              FeignClient: debug # 设置log级别为debug，试了一下设置其他的级别貌似没用。。
```

**最终效果如图**：

![logger打印日志如下.png](https://image.lucfzy.com/blog/logger%E6%89%93%E5%8D%B0%E6%97%A5%E5%BF%97%E5%A6%82%E4%B8%8B.png)

**图中打印出了最基本的请求和响应信息。**

### Spring Cloud GateWay Api

> 旨在构造新一代的智能型网关，zuul网关的替代产品，增加了更多的功能，包括鉴权（token认证），限流，日志打印，智能路由等功能。

**使用说明**

1. 引入pom依赖

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.0.6.RELEASE</version>
    <relativePath/> <!-- lookup parent from repository -->
</parent>

<properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
    <java.version>1.8</java.version>
    <spring-cloud.version>Finchley.SR2</spring-cloud.version> <!-- 重点配置 -->
</properties>

<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-gateway</artifactId>
</dependency>
```

注意此处的spring版本，建议用**2.0.x**版本的，并且项目并非是web项目，所以不需要加入**spring-boot-starter-web**。

2. 编写application.yml文件

```
server:
  port: 8080
spring:
  cloud:
    gateway:
      routes:
        - id: neo_route
          uri: http://lucfzy.com
          predicates:
            - Path=/**
```

意图： 匹配所有的请求，将路由指向http://lucfzy.com这个路径上，也就是路由重定向功能。

# 搜索框架

- ElasticSearch
- logStash
- kibana
- Beats

# 消息队列

## RocketMQ

1. **消息中间键的构成？**
   1. 发布者(provider)
   2. 主题(topic)
   3. 消息(message)
   4. 订阅者(subscriber / consumer)

2. **关键环节**

- 发布者发表消息有几种方式？适用场景有哪些？

- 发布消息为什么要用字节流，字符流不行吗？

- namespacesrv对消息如何进行接收，如何进行处理，如何进行存储的？

- namespacesrc和master和slave有什么关系？

- broker中的master和slave有什么区别？各自有什么作用？

- 发布者发布的topic存储在哪里？以什么方式进行存储的?

- 什么是感兴趣流？

- namespacesrv集群之间如何通信，通信方式有什么，通信的心跳时间又是多少，集群之间的关系是否会有slave和master，如果有的话，是热备还是冷备份？

- 同样的master集群和slave集群以及slave和master的通信机制是否和集群内的通信方式相同呢？

- master和slave单个节点可以存储的数据量为多少？如果节点的消息满了又会如何进行处理？

- mq和订阅者之间的通信方式是什么？

- 监听器的实现方式，如何做到一有消息就可以获取感兴趣流？

- mq有和数据库进行交互么，如果数据满了会不会存储到数据库当中，那么数据恢复可能也是一个问题。

  就像redis就有数据持久化的机制，不知道mq是如何保证数据的。

- 如果mq集群都挂了，数据会如何进行处理？

- 订阅者的pull和push有什么区别？

- 订阅者是和mq中的哪个节点进行通信的，通信方式又是怎么样的？

- 订阅者对于获取到的数据流如何进行处理？

- 如果在和mq的数据交互过程中，mq整体收到网络原因瘫痪，那么我们应该怎么办？

3. 进阶之旅

4. mq在分布式事务当中的应用和角色扮演及功能作用。

### 构建过程

> 访问github：https://github.com/Fzyproj/springboot-rocketmq-primary-demo

启动顺序：

1. namesrv。
2. broker。
3. 生产者或者消费者随意先哪个。
4. 访问send发送消息即可。观察控制台输出结果。