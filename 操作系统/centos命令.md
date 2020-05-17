# 常用命令

1. 查看centos的ip地址

```shell
ip addr // 查看ip地址
vi /etc/sysconfig/network-scripts/ifcfg-ens33 
// 将最后的onboot改为yes
service network start //重启网卡
```

2. 防火墙

```shell
// 停用防火墙
systemctl stop firewalld.service
// 禁止firewall开机启动
systemctl disable firewalld.service 
```

3. java安装

```shell
yum install java-1.8.0-openjdk java-1.8.0-openjdk-devel
cd /usr/lib/jvm // java默认安装路径
// 找到 java-1.8.0-openjdk-1.8.0.252.b09-2.el7_8.x86_64 类似这个名字的文件就是java_home的目录了
vi /etc/profile // 编辑环境配置文件
// 添加内容如下
---
#set java environment
JAVA_HOME=/user/lib/jvm/java-1.8.0-openjdk-1.8.0.252.b09-2.el7_8.x86_64
JRE_HOME=$JAVA_HOME/jre
CLASS_PATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib
PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin
export JAVA_HOME JRE_HOME CLASS_PATH PATH
---
source /etc/profile // 刷新配置
java -version // 查看java配置
```

# Java

1. 查看系统进程

```shell
$ jps
```

