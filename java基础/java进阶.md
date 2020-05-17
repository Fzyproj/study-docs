# jvm调优

## jvm线程内存可视化工具

1. jvisualvm

2. jmc

3. 阿里的Arthas

   - 安装篇

   ```shell
   $ wget https://alibaba.github.io/arthas/arthas-boot.jar
   ```

   - 启用Arthas

   ```shell
   $ java -jar arthas-boot.jar --target-ip 0.0.0.0
   
   [INFO] arthas-boot version: 3.2.0
   [INFO] Found existing java process, please choose one and input the serial number of the process,eg : 1. Then hit ENTER.
   * [1]: 37 arthas-demo.jar
   1
   [INFO] Start download arthas from remote server: https://repo1.maven.org/maven2/com/taobao/arthas/arthas-packaging/3.2.0/arthas-packaging-3.2.0-bin.zip
   [INFO] Download arthas success.
   [INFO] arthas home: /root/.arthas/lib/3.2.0/arthas
   [INFO] Try to attach process 37
   [INFO] Attach process 37 success.
   [INFO] arthas-client connect 0.0.0.0 3658
     ,---.  ,------. ,--------.,--.  ,--.  ,---.   ,---.
    /  O  \ |  .--. ''--.  .--'|  '--'  | /  O  \ '   .-'
   |  .-.  ||  '--'.'   |  |   |  .--.  ||  .-.  |`.  `-.
   |  | |  ||  |\  \    |  |   |  |  |  ||  | |  |.-'    |
   `--' `--'`--' '--'   `--'   `--'  `--'`--' `--'`-----'
   
   
   wiki      https://alibaba.github.io/arthas
   tutorials https://alibaba.github.io/arthas/arthas-tutorials
   version   3.2.0
   pid       37
   time      2020-05-01 16:17:37
   ```

   - 进入Arthas

   ```shell
   $ dashboard
   
   ID      NAME                     GROUP           PRIORIT STATE   %CPU     TIME    INTERRU DAEMON
   18      Timer-for-arthas-dashboa system          10      RUNNABL 100      0:0     false   true
   8       Attach Listener          system          9       RUNNABL 0        0:0     false   true
   3       Finalizer                system          8       WAITING 0        0:0     false   true
   2       Reference Handler        system          10      WAITING 0        0:0     false   true
   4       Signal Dispatcher        system          9       RUNNABL 0        0:0     false   true
   13      arthas-shell-server      system          9       TIMED_W 0        0:0     false   true
   17      as-command-execute-daemo system          10      TIMED_W 0        0:0     false   true
   10      job-timeout              system          9       TIMED_W 0        0:0     false   true
   1       main                     main            5       TIMED_W 0        0:0     false   false
   ---more
   ```

Q： arthas每次只能查看一个java进程么？

## jvm 几种常用的垃圾回收器

> 啊啊啊啊啊啊啊啊啊啊啊啊

1. serial & serial old（JDK 1.3 及之前版本）
2. ps & po 
   1. ps
      - 相比于parNew最直接的升级：自定义参数调节。
        - 主要调节的参数有
          - -**XX:MaxGCPauseMillis**，最大垃圾回收停顿时间。（全程就一次GC回收） 
          - **-XX:GCTimeRatio**，垃圾回收时间与总时间占比。
          - **-XX:UseAdaptiveSizePolicy**，开启后，动态调整各区域大小参数。
   2. po
3. parNew & cms（concurrent mark sweep）标记清除算法
4. G1 
   1. 分块分区
   2. 初始标记；从GCROOTS开始标记可达对象（STW执行）。
   3. 并发标记；（无需STW）。并发过程。
   4. 重新标记；（STW过程）。
   5. 筛选回收；（STW过程），正式进行垃圾回收。

![image-20200501213702596](C:\Users\lucfzy\AppData\Roaming\Typora\typora-user-images\image-20200501213702596.png)

下图展示的上面三个是作用在新生代的垃圾回收器，下面三个作用在老生代的垃圾回收器，而G1是作用在两个生代的GC。

![img](https://pic.yupoo.com/crowhawk/56a02e55/3b3c42d2.jpg)

> 上述共7中常用的垃圾回收器。

## 垃圾回收算法

> 单一职责原则，一个回收器只用一种算法，干一件事情。

1. 标记复制算法（缺点： 需要耗费空间）[eg: serial,parNew,PS]
2. 标记清除算法（缺点：容易产生外碎片）[eg: CMS]
3. 标记压缩算法 (缺点：标记指针需要移动效率比较低) [eg: serial old, parallel old]

## 简述几种GC

1. full GC

   1. **新生代，老年代，永久代**都进行一次GC，耗时比较长。(jdk8之后摒弃永久代，所有只剩下新生代老年代和metaspace区域)

      · 年老代（Tenured）被写满

      · 持久代（Perm）被写满

      · System.gc()被显示调用

      ·上一次GC之后Heap的各域分配策略动态变化

   **以上会触发full GC。**

2. minor GC和Young GC

   1. 清理**新生代**。

3. major GC

   1. 清理**老年代**。

## jvm工作模式

1. Client端
2. Server端

## GC标记对象的两种方式

1. 引用计数法
2. 从GC ROOTS开始Tracing

## 各GC触发条件

Minor GC触发条件：当Eden区满时，触发Minor GC。

Full GC触发条件：

（1）调用System.gc时，系统建议执行Full GC，但是不必然执行

（2）老年代空间不足

（3）方法去空间不足

（4）通过Minor GC后进入老年代的平均大小大于老年代的可用内存

（5）由Eden区、From Space区向To Space区复制时，对象大小大于To Space可用内存，则把该对象转存到老年代，且老年代的可用内存小于该对象大小

# jvm 内存模型

jdk1.8之后的内存模型---元数据空间代替了永久代（PermGen）

![img](https://img2018.cnblogs.com/blog/285763/201912/285763-20191205113339760-1181483685.png)

# 对象分配

1. 首先考虑对象是否可以在栈上分配？（指针逃逸分析，标量替换）
   
> ==标量替换==：scalar replacement又称矢量替换，因为java对象本身就是一个若干对象的聚合量，可以对其进行对象拆解，将其成员变量恢复为分散的变量，这个过程称为标量替换。拆散后的变量便可以被单独分析与优化，可以各自分别在活动记录（栈帧或寄存器）上分配空间；原本的对象就无需整体分配空间了。

   1. 如果可以在栈上分配，那么在栈上分配，如果不可以，则进入下一步。
2. 考虑对象是否可以在TLAB中分配（Thread Local Allocation Buffer），TLAB也是eden区域的一部分，通常占用eden区域的1%的大小空间，如果对象大小小于剩余TLAB空间，那么可以在TLAB上分配。

3. 如果TLAB无法分配对象，那么根据对象的大小来判断是否可以分配到eden区域，如果不可以分配则进行下一步。

4. 进行一次Young GC（Minor GC）。

5. Yong GC之后，如果还是不能分配下对象，那么该对象直接进入olden区。

**Q：垃圾回收算法和垃圾回收器是如何搭配使用的？？**

A：针对于堆内存：

1. 新生代
   1. 拿ps和po回收器来举例。
      1. ps工作在新生代。
         1. 新生代的算法是标记复制，每次把不是垃圾的对象标记出来，并且复制到survivor区域，把其他的直接干掉即可，这样的效率很高。
2. 老生代
   1. po工作在老年代。
      1. 老年代的算法是标记清除和标记压缩算法。

# 动态代理

## 1. JDK动态代理

1. 类的准备
   1. 定义接口类Animal

```java
public interface Animal {
    public void run();
}
```

      2. 定义Animal的实现类Dog类

```java
public class Dog implements Animal {
    @Override
    public void run() {
        System.out.println("Dog can run!!!!");
    }
}
```

## 两种方式实现jdk动态代理

方式一：使用匿名内部类的方式

```java
// 使用匿名内部类的方式去实现动态代理过程
public class MainTest2 {
    public static void main(String[] args) {
        Animal proxyAnimal = (Animal)Proxy.newProxyInstance(Dog.class.getClassLoader(), Dog.class.getInterfaces(), new InvocationHandler() {
            @Override
            public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                Dog dog = new Dog();
                System.out.println("啊。。我被增强了");
                Object result = method.invoke(dog, args);
                return result;
            }
        });
        proxyAnimal.run();
    }
}
```

方式二：使用创建InvocationHandler子类的方式去实现动态代理

```java
public class MainTest {
    public static void main(String[] args) {
        // 实现类animalProxy是代理类
        Dog dog = new Dog();
        Animal animalProxy = (Animal)Proxy.newProxyInstance(Dog.class.getClassLoader(),Dog.class.getInterfaces(),new MyInvocationHandler(dog));
        animalProxy.run();
    }
}

class MyInvocationHandler implements InvocationHandler {
    private Object target;
    public MyInvocationHandler(Object target){
        this.target = target;
    }
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("啊。。我被增强了");
        Object invoke = method.invoke(target, args);
        return invoke;
    }
}
```

## 2.cglib动态代理方式

> cglib既可以对接口进行代理，又可以针对实现类进行代理。

1. 类的准备如上，都是一个Dog和Animal。

## cglib代理实现的两种方式

      1. 匿名内部类方式

   ```java
   public class MyCgLibTest {
       public static void main(String[] args) {
          Dog dog = (Dog)Enhancer.create(Dog.class, new MethodInterceptor() {
               @Override
               public Object intercept(Object o, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {
                   Object result = null;
                   if ("run".equals(method.getName())){
                       System.out.println("啊。。我被增强了");
                       result =  method.invoke(new Dog(),objects);
                   }
                   return result;
               }
           });
          dog.run();
       }
   }
   ```

      2. 构造实现类的方式

   ```java
   public class MyCgLibTest_Outermpl {
       public static void main(String[] args) {
          Dog dog = (Dog)Enhancer.create(Dog.class,new MyMethodInterceptor());
          dog.run();
       }
   }
   
   class MyMethodInterceptor implements MethodInterceptor {
       @Override
       public Object intercept(Object o, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {
           Object result = null;
           if ("run".equals(method.getName())){
               System.out.println("啊。。我被增强了");
               result = method.invoke(new Dog(),objects);
           }
           return result;
       }
   }
   ```

## 运行结果

![image-20200422233343247](C:\Users\lucfzy\AppData\Roaming\Typora\typora-user-images\image-20200422233343247.png)

​	

```java
Exception in thread "main" java.lang.ClassCastException: com.sun.proxy.$Proxy0 cannot be cast to com.lucfzy.DynamicProxyTest.Dog
	at com.lucfzy.DynamicProxyTest.MainTest.main(MainTest.java:14)
```

复杂的逻辑处理和不停的重复劳动，我们可以把ServiceImpl类中的内容进行抽取。

Autowired注解不支持注入静态变量。

```
 Autowired annotation is not supported on static fields: private static com.lucfzy.cglib.Service11 com.lucfzy.cglib.MyCgLibTest.service
```

因为Service是静态的变量所以不支持静态变量的注入。











