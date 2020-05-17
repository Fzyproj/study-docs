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

6. 测试结果，如图所示

![image-20200517192630982](C:\Users\lucfzy\AppData\Roaming\Typora\typora-user-images\image-20200517192630982.png)

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

# 搜索框架

- ElasticSearch
- logStash
- kibana
- Beats

![image-20200425203729961](C:\Users\lucfzy\AppData\Roaming\Typora\typora-user-images\image-20200425203729961.png)

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