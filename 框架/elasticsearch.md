> es产品表：https://www.elastic.co/cn/start

# elasticsearch-head

1. 安装es-head：https://github.com/mobz/elasticsearch-head

```shell
cd elasticsearch-head
npm install
npm run start
```

2. 提示端口占用，修改Gruntfile.js下的port即可。
3. 打开`elasticsearch.yml`解决跨域问题。

```yml
# 文件最后添加如下
http.cors.enabled: true
http.cors.allow-origin: "*"
```

4. 启动`elasticsearch.bat`即可。
5. 最终效果图

![image-20200505210155243](C:\Users\lucfzy\AppData\Roaming\Typora\typora-user-images\image-20200505210155243.png)

# kibana

1. 下载kibana：https://www.elastic.co/cn/downloads/kibana
2. 解压kibana
3. 安装nodejs：https://nodejs.org/zh-cn/download/（选择对应版本）
4. 启动es
5. 再运行kibana访问：http://localhost:5601/app/kibana#/dev_tools/console，看到ui界面成功。
6. kibana上创建一个索引。
7. 创建类型。type1
8. 创建文档document，1

```json
# POST _index/_type/_id（添加文档记录）(相当于发送post请求)
POST /person/type1/1 
# 请求体文档内容
{
  "name":"fuzeyi",
  "age":18,
  "birth":"2020-05-05"
}
# 利用Get请求获取对应索引对应类型的所有数据
GET /person/type1/_search
# 获取对应索引对应类型的文档id
GET /person/type1/id
```

# springboot集成es--模仿京东页面搜索

1. 快速开始：https://www.elastic.co/guide/en/elasticsearch/client/java-rest/current/java-rest-low-usage.html
2. **首先爬取京东搜索界面的数据**，tika包可以爬取电影。
3. **爬取数据先暂存到中台系统。**
4. **中台推数据到es数据库。**
5. 中台读取es数据库数据。
6. 中台将读取后的数据推到前台页面进行展示。

