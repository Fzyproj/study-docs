# 软件准备

typora的版本，要大于等于下面版本。否则将没有自动上传的功能哦。

![image-20210124135141679](https://raw.githubusercontent.com/Fzyproj/blog-img/master/img/image-20210124135141679.png)

# 方式一：上传GitHub`图床`



依次选择，文件---》偏好设置，具体设置如下图所示

![image-20210124135524223](https://raw.githubusercontent.com/Fzyproj/blog-img/master/img/image-20210124135524223.png)

当我们完成第五步，开始第六步的时候会打开一个config.json的配置文件，一开始文件里面是空的，请复制如下信息

> 注意以下是以上传github图传为例

```json
{
  "picBed": {
    "github": {
      "repo": "your-repo/blog-img",
      "token": "your-access-token",
      "path": "img/",
      "customUrl": "",
      "branch": "master"
    },
    "current": "github",
    "uploader": "github"
  },
  "picgoPlugins": {}
}
```

要注意的是，我们需要配置`repo`和`token`属性，其他的可以默认使用即可。

github token的获取方式如下

- 注册并登录你的GitHub账号。
- 新建一个仓库名，如myblog。
- 生成一个token，用于配置到`config.json`中。

- 访问：https://github.com/settings/tokens

  > 然后点击`Generate new token`。

- 把repo的勾打上即可。

- 然后翻到页面最底部，点击`Generate token`的绿色按钮生成token。

**注意:** 这个token生成后只会显示一次！务必保存好备用。

```
Make sure to copy your new personal access token now. You won’t be able to see it again!
```

ok这个时候已经完成了设置，点击第七步，测试服务器上传的联通性即可。

![image-20210124140251055](https://raw.githubusercontent.com/Fzyproj/blog-img/master/img/image-20210124140251055.png)

看到上面有报错信息，原因很简单，因为github不支持同名文件的上传，因为之前已经测试过一次了，所以报错是正常的。**如果是第一次上传，那么不会存在问题，会显示连接服务器成功。**

### 特别关注

#### Q1：raw.githubusercontent.com 访问不了

我们的图片可以正常上传，并且正常情况下，我们的图片会上传到`raw.githubusercontent.com`，这个域名下面，那么问题来了，图片是可以正常上传，但是无法下载，这是由于我们本低的dns设置的问题，所以需要简单的修改一下`hosts文件`即可解决这个问题。

- 打开文件位置：C:\Windows\System32\drivers\etc

- 设置hosts属性，否则该文件只是**只读**状态。

  ![image-20210124140648245](https://raw.githubusercontent.com/Fzyproj/blog-img/master/img/image-20210124140648245.png)

  注意上述位置不要勾选。

- 编辑文件，追加如下内容

  ```
  # GitHub Start 
  140.82.113.3      github.com
  140.82.114.20     gist.github.com
  
  151.101.184.133    assets-cdn.github.com
  151.101.184.133    raw.githubusercontent.com
  199.232.28.133     raw.githubusercontent.com 
  151.101.184.133    gist.githubusercontent.com
  151.101.184.133    cloud.githubusercontent.com
  151.101.184.133    camo.githubusercontent.com
  199.232.96.133     avatars.githubusercontent.com
  151.101.184.133    avatars0.githubusercontent.com
  199.232.68.133     avatars0.githubusercontent.com
  199.232.28.133     avatars0.githubusercontent.com 
  199.232.28.133     avatars1.githubusercontent.com
  151.101.184.133    avatars1.githubusercontent.com
  151.101.108.133    avatars1.githubusercontent.com
  151.101.184.133    avatars2.githubusercontent.com
  199.232.28.133     avatars2.githubusercontent.com
  151.101.184.133    avatars3.githubusercontent.com
  199.232.68.133     avatars3.githubusercontent.com
  151.101.184.133    avatars4.githubusercontent.com
  199.232.68.133     avatars4.githubusercontent.com
  151.101.184.133    avatars5.githubusercontent.com
  199.232.68.133     avatars5.githubusercontent.com
  151.101.184.133    avatars6.githubusercontent.com
  199.232.68.133     avatars6.githubusercontent.com
  151.101.184.133    avatars7.githubusercontent.com
  199.232.68.133     avatars7.githubusercontent.com
  151.101.184.133    avatars8.githubusercontent.com
  199.232.68.133     avatars8.githubusercontent.com
  199.232.96.133     avatars9.githubusercontent.com
  
  # GitHub End
  ```

- 使用cmd命令来刷新dns

  ```cmd
  ipconfig /displaydns # 显示dns缓存 
  ipconfig /flushdns # 刷新DNS记录 
  ```

  注意：如果图片再次不能显示，只需要及时更新IP就行啦，这波操作不麻烦，执行下面的命令

  ```cmd
  ipconfig /renew # 重请从DHCP服务器获得IP 
  ```

  

# 方式二：上传七牛云图床

上传的步骤和**上传github**一样简单。

1. 下载[picgo](https://github.com/Molunerfinn/PicGo/releases)，注意下载最新版的即可，不是正式版的也没有关系，问题不大。

2. 正常安装picgo软件，比方我安装在了目录D:\PicGo下面，这个路径后面有用。

3. 设置picgo的七牛云访问设置

   ![image-20210124164031611](https://image.lucfzy.com/blog/img/image-20210124164031611.png)

4. 配置好了点击应用，最后确定即可。

5. 配置typora如下。

   ![image-20210629220019707](https://image.lucfzy.com/blog/img/image-20210629220019707.png)

6. 最后点击验证图片上传选项即可。

7. 测试的话，就截图然后粘贴到typora中，直接就会转换为七牛云配置的路径，而且可以正常显示照片，说明大功告成。

8. 我是来测试的呀。
