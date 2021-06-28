## Spring Aop

今天在写业务代码的时候，偶然想起用aop来小试牛刀，改造一下现有代码，但是遇到了一些问题，具体就是一些语句忘记使用，下面记录一下。

### 最基本使用

1. 定义注解

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Logger {
    
}
```

2. 定义配置类，并切入这个注解

   ```java
   import org.aspectj.lang.ProceedingJoinPoint;
   import org.aspectj.lang.annotation.Around;
   import org.aspectj.lang.annotation.Aspect;
   import org.aspectj.lang.annotation.Pointcut;
   import org.slf4j.Logger;
   import org.slf4j.LoggerFactory;
   import org.springframework.stereotype.Component;
   
   /**
    * 声明切点并且加入到Spring容器中，交给Spring管理Bean
    */
   @Aspect
   @Component
   public class LoggerConfiguration {
   
       private final Logger log = LoggerFactory.getLogger(LoggerConfiguration.class);
   
       // 定义切点，建议注解书写全限定类名
       @Pointcut("@annotation(com.lucfzy.passanger.aspect.Logger)")
       public void pointcut(){}
   
       @Around(value = "pointcut()")
       public Object around(ProceedingJoinPoint pjp) {
           log.info("开始打印日志");
           Object result = null;
           try {
               result = pjp.proceed();
           } catch (Throwable throwable) {
               log.error("系统异常，请确认");
               throwable.printStackTrace();
           }
           log.info("业务代码执行结束");
           return result;
       }
   }
   ```

3. 使用注解即可

4. 断点调试，看是否正常进入切面。

### 进阶使用

1. 如何获取当前注解类的参数值？

   我们上面的例子中，Logger注解是没有属性的，所以也不存在属性值的概念，索性，就给他加上一个属性叫做value。修改如下

   ```java
   @Target(ElementType.METHOD)
   @Retention(RetentionPolicy.RUNTIME)
   @Documented
   public @interface Logger {
       // 如果不加default，默认这个参数是必填的，否则就会编译报错。
       String value();
   }
   ```

   下一步我们在切面中的around方法进行获取用户配置的属性值。

   ```java
   @Around(value = "pointcut()")
   public Object around(ProceedingJoinPoint pjp) {
       MethodSignature signature = (MethodSignature)pjp.getSignature();
       Logger annotation = signature.getMethod().getAnnotation(Logger.class);
       // 拿到注解对象，直接获取属性值即可
       String value = annotation.value();
   }
   ```

2. 如何获取被拦截方法的参数值？

   **方法一：**

   首先我们要知道参数是一个列表，也就是入参不止一个参数，因此，我们需要获取到参数列表，第二步，找到我们的参数在哪个位置，进行获取其值。

   ```java
   @Around(value = "pointcut()")
   public Object around(ProceedingJoinPoint pjp) {
   	Object[] args = pjp.getArgs();
       for (Object arg : args) {
           // 将参数强制转化成对应类型即可
           String arg1 = (String)arg;
       }
   }
   ```

   **方法二：**

   第一种方法的缺点就是参数个数，位置等可变性，造成了多种不确定因素，导致我们的切面不能正常工作，获取到想要的结果，所以这个时候，就需要用到map结构。

   ```java
   HashMap<String, Object> paramMap = new HashMap<>();
   // 参数值
   Object[] argValues = pjp.getArgs();
   // 参数名
   String[] parameterNames = ((CodeSignature)pjp.getSignature()).getParameterNames();
   for (int i = 0; i < parameterNames.length; i++) {
       paramMap.put(parameterNames[i], argValues[i]);
   }
   ```

   debug如下

   ![image-20210628221837015](C:\Users\lucfz\AppData\Roaming\Typora\typora-user-images\image-20210628221837015.png)

   **根据map中的元素key再去获取对应的value值，那就万无一失了。**
