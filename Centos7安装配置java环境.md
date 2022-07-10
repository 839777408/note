---
title: Centos7安装配置java环境
categories:
  - Linux
tags:
  - Linux
top_img: 'https://npm.elemecdn.com/nan-picture/img/wp4.jpg'
cover: 'https://npm.elemecdn.com/nan-picture/img/wp4.jpg'
abbrlink: 59553
date: 2021-11-21 17:31:58
updated: 2021-11-21 22:46:41
---

# 通过yum安装

1. 查看本地是否自带java环境：`java -version` 和 `yum list installed |grep java`
2. 查看yum仓库中的java安装包：`yum -y list java*`
3. 安装java 1.8版本（可根据yum仓库选择安装）：`yum -y install java-1.8.0-openjdk*`
4. 配置环境变量，在文件末尾添加：`vi /etc/profile`

```shell
export JAVA_HOME=/usr/lib/jvm/java-1.8.0
export JRE_HOME=$JAVA_HOME/jre  
export PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib
```

5. 输入`source /etc/profile`，使配置立即生效
6. 输入`java -version`，检查Java是否安装成功



# 通过下载jdk上传安装

1. 官网下载压缩包：https://www.oracle.com/java/technologies/downloads/#java8
2. 通过xshell上传jdk安装包 
3. 解压到指定文件夹：`tar -zxvf jdk-8u181-linux-x64.tar.gz -C /opt/java/ `
4. 配置环境变量：同上
5. `source /etc/profile`，使配置立即生效



# 查找Java安装路径

```shell
[root@nanzx es]# which java
/usr/bin/java
[root@nanzx es]# ls -lrt /usr/bin/java
lrwxrwxrwx. 1 root root 22 11月 21 17:39 /usr/bin/java -> /etc/alternatives/java
[root@nanzx es]# ls -lrt /etc/alternatives/java
lrwxrwxrwx. 1 root root 73 11月 21 17:39 /etc/alternatives/java -> /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.312.b07-1.el7_9.x86_64/jre/bin/java
[root@nanzx es]# cd /usr/lib/jvm
[root@nanzx jvm]# ls
java        java-1.8.0-openjdk                               java-openjdk  jre-1.8.0          jre-1.8.0-openjdk-1.8.0.312.b07-1.el7_9.x86_64
java-1.8.0  java-1.8.0-openjdk-1.8.0.312.b07-1.el7_9.x86_64  jre           jre-1.8.0-openjdk  jre-openjdk

```

`/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.312.b07-1.el7_9.x86_64`是安装目录，也可以直接通过执行`java -verbose`，最下边会出现安装目录：

```shell
[Loaded java.lang.Shutdown$Lock from /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.312.b07-1.el7_9.x86_64/jre/lib/rt.jar]
```



