# 入门安装篇

1. 下载`spring-framework-5.2.x`分支代码

   ```bash
   git clone -b 5.2.x https://gitee.com/lucfzy/spring-framework.git
   ```

2. 下载gradle 5.6.4版本

   - 路径：https://gradle.org/releases/
   - 搜索 `5.6` 找到对应的版本
   - 下载: [binary-only](https://gradle.org/next-steps/?version=5.6.4&format=bin) 版本即可

3. 配置gradle环境变量

   - 在系统变量下 添加 `GRADLE_HOME` 变量，变量值为 `D:\gradle\gradle-5.6.4`。
   - 在系统变量下 编辑 Path 新建 值为：`%GRADLE_HOME%\bin`
   - 最终点击确认，开启一个cmd窗口，输入 `gradle -v` 验证。
   
4. 在安装好的gradle文件目录中的init.d中新建文件`init.gradle`，内容如下：

   ```igradle
   juallprojects{
       repositories {
           def ALIYUN_REPOSITORY_URL = &#39;http://maven.aliyun.com/nexus/content/groups/public&#39;
           def ALIYUN_JCENTER_URL = &#39;http://maven.aliyun.com/nexus/content/repositories/jcenter&#39;
           all { ArtifactRepository repo -&gt;
               if(repo instanceof MavenArtifactRepository){
                   def url = repo.url.toString()
                   if (url.startsWith(&#39;https://repo1.maven.org/maven2&#39;)) {
                       project.logger.lifecycle &quot;Repository ${repo.url} replaced by $ALIYUN_REPOSITORY_URL.&quot;
                       remove repo
                   }
                   if (url.startsWith(&#39;https://jcenter.bintray.com/&#39;)) {
                       project.logger.lifecycle &quot;Repository ${repo.url} replaced by $ALIYUN_JCENTER_URL.&quot;
                       remove repo
                   }
               }
           }
           maven {
               url ALIYUN_REPOSITORY_URL
               url ALIYUN_JCENTER_URL
           }
       }
   }j
   ```

5. 进入刚刚拉取到的项目根目录，例如我本地是这个目录：`F:\JavaProj\source\spring-framework`，配置`build.gradle`文件。

   在

   ```
   repositories {
   			mavenCentral()
   			maven { url "https://repo.spring.io/libs-spring-framework-build" }
   		}
   ```

   下添加如下一行配置，目的是为了加快下载速度。

   ```
   maven { url "http://maven.aliyun.com/nexus/content/groups/public" }
   ```

6. **预先编译spring-oxm模块**，同样在该项目根目录下面，执行cmd命令，打开命令行提示符，输入：`gradlew :spring-oxm:compileTestJava`

7. 网络情况好的情况下，大概20分钟即可build完成。

8. 使用idea导入spring-framework项目。

9. 选择gradle构建项目。

10. 导入后，等待构建完成即可，大概花费15分钟左右。【记得看一下代码是否切到了5.2.x分支哦】

11. 构建项目，选择Build > Build Project。等待构建完毕。

12. 新建一个测试的module模块，叫做：`spring-test`，构建的时候选择gradle和java来进行构建，等待构建完成。

13. 打开新建的模块，在根目录`java`目录下面新建自己的项目目录即可。输入如下内容进行测试

    ```java
    public class MyTest {
    	public static void main(String[] args) {
    		ClassPathXmlApplicationContext applicationContext = new ClassPathXmlApplicationContext("classpath:beans.xml");
    		Person person = (Person)applicationContext.getBean("person");
    		System.out.println(person);
    	}
    }
    ```

    

## 可能遇到的问题

### 1. Cause: zip END header not found

解决办法：

1. 修改 项目地址/gradle/wrapper/gradle-wrapper.properties的distributionUrl

   ```properties
   #distributionUrl=https://downloads.gradle-dn.com/distributions/gradle-6.5.1-bin.zip
   distributionUrl=file:///d:/gradle-5.6.4-bin.zip
   ```

2. 重新build

3. 如果还不行就应该是缓存问题，清理idea缓存并重启：文件->作废缓存/重启…->Invalidate and Restart

### 2. 执行gradlew命令报错：Gradle sync failed: Cause: error in opening zip file

解决办法：

1. 原因提示是打开压缩文件失败。
2. 先找一下这个文件，在`C:\Users\“本机名”\.gradle\wrapper\dists\gradle-4.1-all\bzyivzo6n839fup2jbap0tjew`（由于电脑和gradle版本不同，最后一个路径会有不同，其他的都一样）
3. 在这里面看到有压缩文件
4. 解压，提示错误：压缩文件已损坏【注意如果有多个这样的文件夹的话，就要多看一下，如果第一个文件夹中的gradle文件可以正常解压，那么就看一下第二个文件夹】
5. 找到破损的文件，到官网上下载一个最新的压缩包过来，并把破损的压缩包给覆盖掉，重新打开cmd，执行命令即可。

