# 第一个es6学习记录



学习网址：https://es6.ruanyifeng.com/#docs/intro

## 快速开始

新建一个`index.html`文件。

使用工具编辑该文件。

```html
<html>
    <body>
        <script>
            var input = ['1','2','3'];
            input.map(item => console.log(item));
        </script>
    </body>
</html>
```

打印如下：

![image-20210129003005897](https://image.lucfzy.com/blog/img/image-20210129003005897.png)

map的作用和java中的map是类似的，类似的还有foreach函数，可以参考学习。

'===' 在js中的作用？

这个要区别于==，那么就有一个疑问，为啥建议用===而不是==呢，最终的原因在于当变量是null或者undefined的时候，如下：

```
null == null // true
null == undifined // true (有问题)
null === null // true
null === undifined // false (正确)
```





