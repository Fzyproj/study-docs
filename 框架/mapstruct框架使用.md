# MapStruct

> 这是一款非常牛逼的对象转换框架，完全可以用来替代BeanUtils工具，是企业高效开发之必备神器组件。

## 使用

1. 引入pom依赖

```xml
<!-- https://mvnrepository.com/artifact/org.mapstruct/mapstruct-jdk8 -->
<dependency>
    <groupId>org.mapstruct</groupId>
    <artifactId>mapstruct-jdk8</artifactId>
    <version>1.3.1.Final</version>
</dependency>

<!-- https://mvnrepository.com/artifact/org.mapstruct/mapstruct-processor -->
<dependency>
    <groupId>org.mapstruct</groupId>
    <artifactId>mapstruct-processor</artifactId>
    <version>1.3.1.Final</version>
</dependency>
```

2. 构造对象转化类，下面以一个复杂对象来作为例子。

   1. 新建`PersonBefore.java`

   ```java
   import lombok.AllArgsConstructor;
   import lombok.Builder;
   import lombok.Data;
   import lombok.NoArgsConstructor;
   
   @Data
   @Builder
   @AllArgsConstructor
   @NoArgsConstructor
   public class PersonBefore {
       private String name;
       private InnerEntity innerEntity;
   
       @Data
       @Builder
       public static class InnerEntity {
           private String id;
       }
   }
   ```

   2. 新建`PersonAfter.java`

   ```java
   import lombok.AllArgsConstructor;
   import lombok.Builder;
   import lombok.Data;
   import lombok.NoArgsConstructor;
   
   @Data
   @Builder
   @AllArgsConstructor
   @NoArgsConstructor
   public class PersonAfter {
       private String name;
       private Entity entity;
       @Data
       public static class Entity {
           private String id;
       }
   }
   ```

   3. 新建`OrderConvert.java`，**该对象的作用是为了做对象转换的核心抽象类，后面再说为什么要用抽象类或者接口的形式。**

   ```java
   import org.mapstruct.Mapper; // 是mapstruct的mapper而不是ibatis的mapper
   import org.mapstruct.Mapping;
   import org.mapstruct.Mappings;
   import org.mapstruct.factory.Mappers;
   import src.dto.PersonAfter;
   import src.dto.PersonBefore;
   
   @Mapper // 容易忽略千万不要忘记，
   public abstract class OrderConvert {
       public static final OrderConvert INSTANCE = Mappers.getMapper(OrderConvert.class);
   
       @Mappings(
           	// 将源对象中的innerEntity字段，映射为目标对象的innerEntity字段，并且两个内部类中的字段也同样符合字段名称相同即映射的映射规则。
               @Mapping(source = "innerEntity",target = "entity")
       )
       public abstract PersonAfter getAfterPerson(PersonBefore beforeDTO);
   
       public abstract PersonBefore getBeforePerson(PersonAfter beforeDTO);
   }
   ```

   4. 测试如下

   ```java
   public class main {
       public static void main(String[] args) {
           PersonBefore before = PersonBefore.builder().name("fuzeyi").innerEntity(PersonBefore.InnerEntity.builder().id("123456").build()).build();
           PersonAfter afterPerson = OrderConvert.INSTANCE.getAfterPerson(before);
           System.out.println(afterPerson);
       }
   }
   ```

   5. 打印结果如下

   ![](http://image.lucfzy.com/blog/mapstruct%E5%AF%B9%E8%B1%A1%E8%BD%AC%E6%8D%A21.png)