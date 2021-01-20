1. @Value注解失效情况？

   答：

2. 使用@Component注入容器和@Configuration和@bean的方式注入到容器中有什么区别？

   答： 前者注入到容器中是为了调用的是无参构造函数，如果一个想加入到容器的类中，没有动态去获取配置信息的字段，那么就可以使用无参构造器来注入，以方便后期直接使用该类的逻辑代码即可。

   而后者是因为如果这个类需要动态去获取比方yml配置文件中的配置，那么就需要去使用@Configuration和@Bean的方式去动态注入，并且搭配set方法来去进行一些注入操作，最终返回一个bean实例，给到spring容器。

3. 如何获取yml配置文件？

   答：使用默认的配置方式是无法获取到自定义的yml配置文件的。比方我们自己在根目录下配置了一个`channel.yml`配置文件。

   根据常规做法如下：

   ```java
   @PropertySource(value = "classpath:channel.yml", encoding = "gbk")
   ```

   我们会加入上面的注解来实现动态的注入。

   但是最后发现竟然报错了。

   原因是我们需要实现一个Factory来实现对指定yml文件的加载。

   ```java
   @Target({ElementType.TYPE})
   @Retention(RetentionPolicy.RUNTIME)
   @Documented
   @Repeatable(PropertySources.class)
   public @interface PropertySource {
       String name() default "";
   
       String[] value();
   
       boolean ignoreResourceNotFound() default false;
   
       String encoding() default "";
   	
       // 这里就是我们需要自定义的Fatory
       Class<? extends PropertySourceFactory> factory() default PropertySourceFactory.class;
   }
   ```

   自定义yaml读取的Fatory暂且命名为YamlPropertySourceFactory。

   ```java
   import org.springframework.beans.factory.config.YamlPropertiesFactoryBean;
   import org.springframework.core.env.PropertiesPropertySource;
   import org.springframework.core.env.PropertySource;
   import org.springframework.core.io.support.EncodedResource;
   import org.springframework.core.io.support.PropertySourceFactory;
   
   import java.io.FileNotFoundException;
   import java.io.IOException;
   import java.util.Properties;
   
   public class YamlPropertySourceFactory implements PropertySourceFactory {
       @Override
       public PropertySource<?> createPropertySource(String name, EncodedResource resource) throws IOException {
           Properties propertiesFromYaml = loadYamlIntoProperties(resource);
           String sourceName = name != null ? name : resource.getResource().getFilename();
           return new PropertiesPropertySource(sourceName, propertiesFromYaml);
       }
   
       private Properties loadYamlIntoProperties(EncodedResource resource) throws FileNotFoundException {
           try {
               YamlPropertiesFactoryBean factory = new YamlPropertiesFactoryBean();
               factory.setResources(resource.getResource());
               factory.afterPropertiesSet();
               return factory.getObject();
           } catch (IllegalStateException e) {
               // for ignoreResourceNotFound
               Throwable cause = e.getCause();
               if (cause instanceof FileNotFoundException)
                   throw (FileNotFoundException) e.getCause();
               throw e;
           }
       }
   }
   ```

   最终的配置为

   ```java
   @Setter
   @Getter
   @ToString
   @Component
   @PropertySource(value = "classpath:channel.yml", encoding = "gbk", factory = YamlPropertySourceFactory.class)
   public class ChannelYmlConfig {
       @Value("${channels.name}")
       private String name;
       @Value("${channels.id}")
       private String id;
       @Value("${channels.location}")
       private String location;
   }
   ```

   结果如下：

   ```
   ChannelYmlConfig(name=天猫华为旗舰店, id=DDDDDDDJJJJJJJJ, location=南京)
   ```

   > 扩展知识

   **多配置文件动态注入**

   ```java
   @Setter
   @Getter
   @ToString
   @Component
   @PropertySource(value = {"classpath:channel.yml","classpath:location.yml"}, encoding = "gbk", factory = YamlPropertySourceFactory.class)
   public class ChannelYmlConfig {
       @Value("${channels.name}")
       private String name;
       @Value("${channels.id}")
       private String id;
       @Value("${channels.location}")
       private String location;
       @Value("${location.id1}")
       private String id1;
       @Value("${location.name1}")
       private String name1;
       @Value("${location.location1}")
       private String location1;
   }
   ```

   前三个字段在`channel.yml`文件中，后三个字段在`location.yml`文件中。

   最终效果：

   ```
   ChannelYmlConfig(name=天猫华为旗舰店, id=DDDDDDDJJJJJJJJ, location=南京, id1=123, name1=456, location1=789)
   ```

4. 如果获取指定文件名的properties配置文件？

   答：获取指定的properties比较简单，便于记忆我们可以想像其实一开始的spring只有properties配置文件。所以一开始只为properties配置文件所设计的。

   实现如下：

   配置文件路径：`src\main\resources\config\channel.properties`

   ```properties
   channel.id=DFSRWFSDFN
   channel.name=华为终端B2C
   channel.dd=1231231321
   ```

   获取配置文件

   ```java
   @Data
   @Component
   @PropertySource(value = "classpath:/config/channel.properties", encoding = "gbk")
   @ConfigurationProperties(prefix = "channel")
   public class ChannelConfig {
       private String id;
       private String name;
       private String dd;
   }
   ```

5. 如何获取application.properties文件的配置？

   答：这里的获取配置参照application.yml的配置即可，二者将会形成对应的互补配置。

   具体实现如下：

   ```java
   @Component
   public class DBConfig {
       @Value("db.url")
       private String url;
       @Value("db.name")
       private String name;
       @Value("db.username")
       private String userName;
       @Value("db.password")
       private String passWord;
   }
   ```

   一定要记得要将类加入到容器中去。

   这和application.yml获取配置的方式是完全一致的。

6. 配置了application-dev.yml文件那么application.yml的配置会失效吗？

   答：首先如果想要dev配置文件生效，我们需要在application.yml文件中进行指定，那么需要如下配置：

   ```yml
   spring:
     profiles:
       active: dev
   ```

   让dev的配置文件生效。

   **那么此时问题就来了**

   如果application中的配置和dev中配置冲突了怎么办？

   答：dev中的配置会覆盖application中的配置。

   如果dev中没有的配置代码中引用到了application的配置，这样会读取到数据吗？

   答：当然会读取到，dev配置和application配置相当于形成了一种父子关系，儿子dev有的优先用儿子的，如果儿子没有的，到老爸application这边来看看，如果老爸这边也没有那么就返回null。
   
7. 超卖问题复现。

   1. 在单台机器上，即单jvm，如果一个请求过来，这个方法没有启用多线程，则不会出现超卖问题，如果使用一个或者多个线程去处理这个请求，就会出现超卖问题，需要使用锁进行控制，单单利用事务是无法控制的。
   2. 在多台机器上，即多jvm，如果多个请求过来，相当于是多台机器产生并发的情况，那么就会出现超卖问题，需要使用锁进行控制。

8. 事务的控制
   
   1. 事务的职责不是保证多个线程之间可以有序得去执行代码块，这个功能是锁或者分布式锁来控制的，事务保证的是在单个线程内，执行结果的一致性，如果一个线程内同时需要修改三张表，那么这三张表必须同时被修改，如果修改了其中两个，第三个表在被修改的过程中，抛出了异常，导致第三张表的数据修改失败，那么我们的诉求是需要前面两个表的数据也应该得到回滚。所以这个时候就需要用到事务来控制，普通的锁是没法控制数据修改的一致性问题。