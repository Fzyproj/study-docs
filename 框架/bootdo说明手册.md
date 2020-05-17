1. 首页
   - url ： http://localhost/blog
   - 请求过程
     - 映射com.bootdo.blog.controller.BlogController
     - @GetMapping()
     - `blog/index/main`themleaf模板引擎去搜索该html

2. 首页---点击BootDo
   - url: http://localhost/blog
   - 请求过程和上一个类似，比较简单。
3. 首页---点击主页
   - 和上一个相同，只是换了一个名字而已。

4. 首页---点击关于
   - url : http://localhost/blog/open/page/about
   - 请求过程：
     - `@GetMapping("/open/page/{categories}")`
     - 首先将路径变量中的categories加入到map集合中。