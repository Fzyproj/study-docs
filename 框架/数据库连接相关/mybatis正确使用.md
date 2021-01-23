# mybatis教程

## 快速入门

1. 新建一个maven项目。

2. 导入pom包

   ```xml
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-web</artifactId>
   </dependency>
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-test</artifactId>
   </dependency>
   <dependency>
       <groupId>org.mybatis.spring.boot</groupId>
       <artifactId>mybatis-spring-boot-starter</artifactId>
       <version>2.0.0</version>
   </dependency>
   <dependency>
       <groupId>mysql</groupId>
       <artifactId>mysql-connector-java</artifactId>
       <version>5.1.48</version>
   </dependency>
   <!--lombok插件，可选-->
   <dependency>
       <groupId>org.projectlombok</groupId>
       <artifactId>lombok</artifactId>
       <version>1.18.12</version>
   </dependency>
   ```

3. 配置数据源`application.yml`

   ```yaml
   server:
     port: 8080
   
   spring:
     datasource:
       url: jdbc:mysql://localhost:3306/good?serverTimezone=UTC&characterEncoding=utf-8
       username: root
       password: root
       driver-class-name: com.mysql.jdbc.Driver
       hikari:
         maxLifetime: 1765000 #一个连接的生命时长（毫秒），超时而且没被使用则被释放（retired），缺省:30分钟，建议设置比数据库超时时长少30秒以上
         maximumPoolSize: 15 #连接池中允许的最大连接数。缺省值：10；推荐的公式：((core_count * 2) + effective_spindle_count)
         pool-name: my-hikari-pool
   
   mybatis:
     mapper-locations: classpath:mapper/*.xml
     typeAliasesPackage: com.lucfzy
   ```

4. 编写`classpath:mapper/mapper.xml`配置文件，注意路径要和配置中的`mapper-locations`对应

   ```xml-dtd
   <?xml version="1.0" encoding="UTF-8" ?>
   <!DOCTYPE mapper
           PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
           "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
   
   <mapper namespace="com.lucfzy.GoodInfoMapper">
   
       <resultMap id="getUserByIdMap" type="com.lucfzy.GoodInfo">
           <result property="skuCode" column="sku_code"></result>
       </resultMap>
   
       <select id="findAll" resultType="com.lucfzy.GoodInfo">
           select * from good_info
       </select>
   
   </mapper>
   
   ```

5. 编写mapper dao层接口

   ```java
   import com.github.pagehelper.Page;
   import org.apache.ibatis.annotations.Mapper;
   import org.springframework.stereotype.Repository;
   @Mapper
   @Repository
   public interface GoodInfoMapper {
       Page<GoodInfo> findAll();
   }
   ```

6. 编写启动类

   ```java
   @SpringBootApplication
   public class SpringApplicationRunner {
       public static void main(String[] args) {
           SpringApplication.run(SpringApplicationRunner.class, args);
       }
   }
   ```

7. 启动项目或者编写测试类进行测试皆可。

## 强大的分页插件 PageHepler

1. 引入pom依赖

   ```xml
   <!--分页插件的实现-->
   <dependency>
       <groupId>com.github.pagehelper</groupId>
       <artifactId>pagehelper-spring-boot-starter</artifactId>
       <version>1.2.5</version>
   </dependency>
   ```

2. 使用相关的分页类

   ```java
   // 设置初始页，注意该配置，只对下面的第一次查询起作用。
   PageHelper.startPage(1,3);
   // 展示分页结果，实际上就是一个list没有任何分页信息。
   Page<GoodInfo> result = goodInfoMapper.findAll();
   // 里面包装了各种分页信息
   PageInfo<GoodInfo> pagedResult = PageInfo.of(result);
   ```

3. 查看结果即可。

