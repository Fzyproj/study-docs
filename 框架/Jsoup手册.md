1. 快速开始

   ```java
   // 获取一个url链接
   Connection connect = Jsoup.connect("https://lucfzy.com");
   // 解析html
   Document document2 = connect.get();
   // 输出完整html页面
   System.out.println(document);
   ```

2. 解析静态html，取出p标签下的所有标签及内容

   ```java
   // 定义document，两个p标签构成
   Document document1 = Jsoup.parse("<p id = "+"class" + ""+" >a<a href =" + ">这是我的个人主页哦</a></p></br>"+"<p id = "+"class" + ""+" >a<a href =" + ">这是我的个人主页哦1111</a></p></br>");
   // document选择器选择p标签
   Elements pElements = document1.select("p");
   // 选择所有p标签下面的a标签内容并输出
   for (Element pElement : pElements) {
       System.out.println(pElement.select("a"));
   }
   ```

3. 取出p标签中子标签a下的text文本内容，直接取a标签即可，不需要先取父标签。

   ```java
   Elements pElements = document1.select("p");
           for (Element pElement : pElements) {
               Elements a = pElement.select("a");
               for (Element element : a) {
                   System.out.println(element.text()); //这是我的个人主页哦,这是我的个人主页哦1111
               }
           }
   ```

   

