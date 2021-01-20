# 安装篇

1. 下载`5.2.x`分支代码

   ```bash
   git clone -b 5.2.x https://gitee.com/lucfzy/spring-framework.git
   ```

2. 下载gradle 5.6.4版本

   - 路径：https://gradle.org/releases/
   - 搜索 5.6找到对应的版本
   - 下载: [binary-only](https://gradle.org/next-steps/?version=5.6.4&format=bin)版本即可

3. 配置gradle环境变量

   - 在系统变量下 添加 `GRADLE_HOME` 变量，变量值为 `D:\gradle\gradle-5.6.4`。
   - 在系统变量下 编辑 Path 新建 值为：`%GRADLE_HOME%\bin`
   - 最终点击确认，开启一个cmd窗口，输入 `gradle -v` 验证。

## 安装报错篇

### 1. Cause: zip END header not found

解决办法：

1. 修改 项目地址/gradle/wrapper/gradle-wrapper.properties的distributionUrl

   ```properties
   #distributionUrl=https://downloads.gradle-dn.com/distributions/gradle-6.5.1-bin.zip
   distributionUrl=file:///d:/gradle-5.6.4-bin.zip
   ```

2. 重新build

3. 如果还不行就应该是缓存问题，清理idea缓存并重启：文件->作废缓存/重启…->Invalidate and Restart