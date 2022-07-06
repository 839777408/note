---
title: DockerFile解析
categories:
  - Docker
tags:
  - Docker
  - DockerFile
top_img: 'https://cdn.jsdelivr.net/gh/839777408/tupian/img/wp10.jpg'
cover: 'https://cdn.jsdelivr.net/gh/839777408/tupian/img/wp10.jpg'
abbrlink: e53
date: 2021-06-26 00:25:59
updated: 2022-3-17 00:08:55
---

# 初识DockerFile

Dockerfile是用来**构建Docker镜像**的构建文件，是由一系列命令和参数构成的脚本。

构建发布的四个步骤：

1. 编写DockerFile文件
2. docker build
3. docker run
4. docker push发布镜像（DockerHub 、阿里云仓库)

以熟悉的CentOS7为例：

```dockerfile
FROM scratch
ADD centos-7-docker.tar.xz /

LABEL org.label-schema.schema-version="1.0" \
    org.label-schema.name="CentOS Base Image" \
    org.label-schema.vendor="CentOS" \
    org.label-schema.license="GPLv2" \
    org.label-schema.build-date="20181204"

CMD ["/bin/bash"]
```



# DockerFile构建过程

**基础知识：**

1. 每个保留关键字（指令）都是必须是大写字母，并且后面要跟随至少一个参数

2. 执行从上到下顺序

3. #表示注释

4. 每一个指令都会创建一个新的镜像层并提交

---

**Docker执行DockerFile的大致流程：**

1. docker从基础镜像运行一个容器
2. 执行一条指令并对容器作出修改
3. 执行类似docker commit的操作提交一个新的镜像层
4. docker再基于刚提交的镜像运行一个新容器
5. 执行dockerfile中的下一条指令直到所有指令都执行完成

---

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20210626120726.png)

从应用软件的角度来看，Dockerfile、Docker镜像与Docker容器分别代表软件的三个不同阶段：

- Dockerfile是软件的原材料

* Docker镜像是软件的交付品

- Docker容器则可以认为是软件的运行态

Dockerfile面向开发，Docker镜像成为交付标准，Docker容器则涉及部署与运维，三者缺一不可，合力充当Docker体系的基石。

>1. Dockerfile，需要定义一个Dockerfile，Dockerfile定义了进程需要的一切东西。Dockerfile涉及的内容包括执行代码或者是文件、环境变量、依赖包、运行时环境、动态链接库、操作系统的发行版、服务进程和内核进程(当应用进程需要和系统服务和内核进程打交道，这时需要考虑如何设计namespace的权限控制)等等;
>
>2. Docker镜像，在用Dockerfile定义一个文件之后，docker build时会产生一个Docker镜像，当运行 Docker镜像时，会真正开始提供服务;
>
>3. Docker容器，容器是直接提供服务的



# DockerFile保留字指令

| 保留字指令 | 说明                                                         |
| ---------- | ------------------------------------------------------------ |
| FROM       | 基础镜像，当前镜像是基于哪个镜像的                           |
| MAINTAINER | 镜像维护者的姓名和邮箱地址                                   |
| RUN        | 镜像构建时需要运行的命令                                     |
| EXPOSE     | 镜像对外暴露的端口                                           |
| WORKDIR    | 指定在创建容器后，终端默认登陆的进来工作目录                 |
| ENV        | 用来镜像构建过程中设置环境变量                               |
| ADD        | 将宿主机目录下的文件拷贝进镜像且ADD命令会自动**处理URL和解压tar压缩包** |
| COPY       | 类似ADD，拷贝文件和目录到镜像中。<br>将从构建上下文目录中<源路径>的文件/目录复制到新的一层的镜像内的<目标路径>位置<br>COPY src dest或**COPY ["src","dest"]** |
| VOLUME     | 容器数据卷，用于数据保存和持久化工作                         |
| CMD        | 指定这个容器启动的时候要运行的命令<br>Dockerfile中可以有多个CMD指令，但**只有最后一个生效**，CMD会被 docker run之后的参数替换 |
| ENTRYPOINT | 指定这个容器启动的时候要运行的命令，可以追加命令 。ENTRYPOINT的目的和CMD一样，都是在指定容器启动程序及参数 |
| ONBUILD    | 当构建一个被继承的Dockerfile时运行命令，父镜像在被子继承后父镜像的onbuild被触发 |

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20210905171117.png)

---



# 案例

**Base镜像：**

Docker Hub 中99%的镜像都是通过在base镜像中安装和配置需要的软件构建出来的。

例如，CentOS7的DockerFile中，首行的`FROM scratch`

---

## 自定义镜像myCentOs

1.编写DockerFile

- 准备好CentOS镜像作为Base镜像: `docker pull centos`

- Hub默认的CentOS镜像：
  - 初始centos运行该镜像时，登录后的默认路径是 /
  - 默认不支持vim
  - 默认不支持ifconfig

- 自定义mycentos目的使我们自己的镜像具备如下：
  -  登陆后的默认路径
  - vim编辑器
  - 查看网络配置ifconfig支持
- 在路径下新建一个以Dockerfile命名的文件，`vi Dockerfile`，也可以通过构建时 -f 参数指定文件

```dockerfile
FROM centos
MAINTAINER nan<839777408@qq.com>

ENV MYPATH /usr/local
WORKDIR $MYPATH

RUN yum -y install vim
RUN yum -y install net-tools

EXPOSE 80

CMD echo $MYPATH
CMD echo "success--------------ok"
CMD /bin/bash
 
```

2.在DockerFile所在路径下构建：`docker build -t 新镜像名字:TAG .`

**注意：**新镜像名字必须全小写，TAG 后面有个小点表示当前路径

构建结果如下：

```shell
[root@nanzx myDockerFile]# ls
Dockerfile
[root@nanzx myDockerFile]# docker build -t mycentos:1.3 .
Sending build context to Docker daemon  2.048kB
Step 1/10 : FROM centos
 ---> 300e315adb2f
Step 2/10 : MAINTAINER nan<839777408@qq.com>
 ---> Running in 7127813ad38d
Removing intermediate container 7127813ad38d
 ---> 9c0533e44a0a
Step 3/10 : ENV MYPATH /usr/local
 ---> Running in fb47fda174e9
Removing intermediate container fb47fda174e9
 ---> 82cf9fb823cc
Step 4/10 : WORKDIR $MYPATH
 ---> Running in 38de19574157
Removing intermediate container 38de19574157
 ---> 3467bda82bb7
Step 5/10 : RUN yum -y install vim
 ---> Running in 198852c5192c
CentOS Linux 8 - AppStream                      6.6 MB/s | 8.8 MB     00:01    
CentOS Linux 8 - BaseOS                         7.0 MB/s | 5.6 MB     00:00    
CentOS Linux 8 - Extras                          15 kB/s |  10 kB     00:00    
Dependencies resolved.
...
...
Installed:
  gpm-libs-1.20.7-17.el8.x86_64         vim-common-2:8.0.1763-15.el8.x86_64    
  vim-enhanced-2:8.0.1763-15.el8.x86_64 vim-filesystem-2:8.0.1763-15.el8.noarch
  which-2.21-12.el8.x86_64             

Complete!
Removing intermediate container 198852c5192c
 ---> 84b0976d75ce
Step 6/10 : RUN yum -y install net-tools
 ---> Running in d6de8e9cf44e
Last metadata expiration check: 0:00:07 ago on Sun Sep  5 12:34:50 2021.
Dependencies resolved.
...
...
Installed:
  net-tools-2.0-0.52.20160912git.el8.x86_64                                     
Complete!
Removing intermediate container d6de8e9cf44e
 ---> 45d11cc53ed5
Step 7/10 : EXPOSE 80
 ---> Running in bdc156599e08
Removing intermediate container bdc156599e08
 ---> 19fc20586285
Step 8/10 : CMD echo $MYPATH
 ---> Running in a268447d09ce
Removing intermediate container a268447d09ce
 ---> 05e469707492
Step 9/10 : CMD echo "success--------------ok"
 ---> Running in 697e6b2b7d70
Removing intermediate container 697e6b2b7d70
 ---> 2ab970b82316
Step 10/10 : CMD /bin/bash
 ---> Running in 46f76ba15387
Removing intermediate container 46f76ba15387
 ---> 9f3679e104c6
Successfully built 9f3679e104c6
Successfully tagged mycentos:1.3
[root@nanzx myDockerFile]# 
```

3.运行新镜像：`docker run -it 新镜像名字:TAG`

运行结果如下：

```shell
[root@nanzx myDockerFile]# docker images
REPOSITORY   TAG       IMAGE ID       CREATED         SIZE
mycentos     1.3       9f3679e104c6   8 minutes ago   307MB
centos       latest    300e315adb2f   9 months ago    209MB
[root@nanzx myDockerFile]# docker run -it mycentos:1.3
[root@42be0264774b local]# pwd
/usr/local
[root@42be0264774b local]# ifconfig
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
```

可以看到，我们自己的新镜像已经支持vim/ifconfig命令，扩展成功了。

4.列出镜像的变更历史：`docker history 镜像名:TAG`

```shell
[root@nanzx myDockerFile]# docker history mycentos:1.3
IMAGE          CREATED          CREATED BY                                      SIZE      COMMENT
9f3679e104c6   10 minutes ago   /bin/sh -c #(nop)  CMD ["/bin/sh" "-c" "/bin…   0B        
2ab970b82316   10 minutes ago   /bin/sh -c #(nop)  CMD ["/bin/sh" "-c" "echo…   0B        
05e469707492   10 minutes ago   /bin/sh -c #(nop)  CMD ["/bin/sh" "-c" "echo…   0B        
19fc20586285   10 minutes ago   /bin/sh -c #(nop)  EXPOSE 80                    0B        
45d11cc53ed5   10 minutes ago   /bin/sh -c yum -y install net-tools             29.5MB    
84b0976d75ce   10 minutes ago   /bin/sh -c yum -y install vim                   68.1MB    
3467bda82bb7   11 minutes ago   /bin/sh -c #(nop) WORKDIR /usr/local            0B        
82cf9fb823cc   11 minutes ago   /bin/sh -c #(nop)  ENV MYPATH=/usr/local        0B        
9c0533e44a0a   11 minutes ago   /bin/sh -c #(nop)  MAINTAINER nan<839777408@…   0B        
300e315adb2f   9 months ago     /bin/sh -c #(nop)  CMD ["/bin/bash"]            0B        
<missing>      9 months ago     /bin/sh -c #(nop)  LABEL org.label-schema.sc…   0B        
<missing>      9 months ago     /bin/sh -c #(nop) ADD file:bd7a2aed6ede423b7…   209MB   
```

---



## CMD/ENTRYPOINT镜像

CMD/ENTRYPOINT都是指定一个容器要启动时运行的命令。

**CMD:**

- Dockerfile中可以有多个CMD指令，但只有最后一个生效，CMD会被 docker run之后的参数替换

```shell
# 编写dockerfile文件 
$ vim dockerfile-test-cmd 

FROM centos 
CMD ["ls","-a"] 

# 构建镜像 
$ docker build -f dockerfile-test-cmd -t cmd-test:0.1 . 

# 运行镜像 
$ docker run cmd-test:0.1 
...
bin 
dev 
root
...

# 想追加一个命令 -l 成为ls -al 
$ docker run cmd-test:0.1 -l 
docker: Error response from daemon: OCI runtime create failed: container_linux.go:349: starting container process caused "exec: \"-l\": executable file not found in $PATH": unknown. ERRO[0000] error waiting for container: context canceled 

# 可执行文件找不到的报错，executable file not found。
# 跟在镜像名后面的是 command，运行时会替换 CMD 的默认值
# cmd的情况下 -l 替换了CMD["ls","-l"]。 -l 不是命令所有报错
```

---

**ENTRYPOINT:**

- docker run之后的参数会被当做参数传递给ENTRYPOINT，之后形成新的命令组合

```shell
# 编写dockerfile文件 
$ vim dockerfile-test-entrypoint 

FROM centos 
ENTRYPOINT ["ls","-a"] 

$ docker run entrypoint-test:0.1 
...
bin 
dev 
etc 
home  
lost+found 
... 

# -l命令是直接拼接在ENTRYPOINT命令后面的 
$ docker run entrypoint-test:0.1 -l 
total 56 
drwxr-xr-x 1 root root 4096 May 16 06:32 . 
drwxr-xr-x 1 root root 4096 May 16 06:32 .. 
-rwxr-xr-x 1 root root 0 May 16 06:32 .dockerenv 
lrwxrwxrwx 1 root root 7 May 11 2019 bin -> usr/bin 
...
```

---



## 自定义镜像Tomcat

1.准备tomcat和jdk的linux安装包以及编写好的文本： apache-tomcat-9.0.35.tar.gz、jdk-8u231-linux-x64.tar.gz、README

2.编写dockerFile文件：

```shell
FROM centos 
MAINTAINER nan<839777408@qq.com> 

COPY README /usr/local/README #把宿主机当前上下文的README拷贝到容器/usr/local/路径下
ADD jdk-8u231-linux-x64.tar.gz /usr/local/ #复制解压 
ADD apache-tomcat-9.0.35.tar.gz /usr/local/ #复制解压 

RUN yum -y install vim #安装vim编辑器

ENV MYPATH /usr/local #设置环境变量 
WORKDIR $MYPATH #设置工作访问时候的WORKDIR路径，登录落脚点

ENV JAVA_HOME /usr/local/jdk1.8.0_231 #设置环境变量 
ENV CLASSPATH $JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar #设置环境变量 
ENV CATALINA_HOME /usr/local/apache-tomcat-9.0.35 #设置环境变量 
ENV PATH $PATH:$JAVA_HOME/bin:$CATALINA_HOME/lib:$CATALINA_HOME/bin #设置环境变量 分隔符是： 

EXPOSE 8080 #设置暴露的端口 

# ENTRYPOINT ["/usr/local/apache-tomcat-9.0.35/bin/startup.sh" ]
# CMD ["/usr/local/apache-tomcat-9.0.35/bin/catalina.sh","run"]
CMD /usr/local/apache-tomcat-9.0.35/bin/startup.sh && tail -F /usr/local/apache- tomcat-9.0.35/logs/catalina.out #启动时运行tomcat
```

3.构建镜像：`docker build -t mytomcat:0.1 . `

4.运行镜像：

```shell
docker run 
-d 
-p 8080:8080 
--name tomcat01 
-v /home/nan/build/tomcat/test:/usr/local/apache-tomcat-9.0.35/webapps/test 
-v /home/nan/build/tomcat/tomcatlogs/:/usr/local/apache-tomcat-9.0.35/logs 
mytomcat:0.1
```

> Docker挂载主机目录Docker访问出现cannot open directory : Permission denied
>
> 解决办法：在挂载目录后多加一个--privileged=true参数即可

---



# 镜像发布

**镜像的生成方法：**

1. DockerFile
2. 从容器创建一个新的镜像：`docker commit [OPTIONS] 容器ID [REPOSITORY[：TAG]]`
3. OPTIONS说明：-a :提交的镜像作者；-m :提交时的说明文字；

---

**Docker Hub发布：**

1. 官网：https://hub.docker.com/

2. 登录

   ```shell
   $ docker login --help 
   Usage: docker login [OPTIONS] [SERVER] 
   
   Log in to a Docker registry. 
   If no server is specified, the default is defined by the daemon. 
   
   Options: 
   	-p, --password string Password 
   	--password-stdin Take the password from stdin 
       -u, --username string Username
      
   $docker login -u nan #nan改成用户名
   ```

3. 提交 push镜像：`docker push mytomcat`

> 会发现push不上去，因为如果没有前缀的话默认是push到 官方的library 
>
> 解决方法 
>
> 1. build的时候添加你的dockerhub用户名，然后在push就可以放到自己的仓库了 
>
>    $ docker build -t nan/mytomcat:0.1 . 
>
> 2. 使用docker tag #然后再次push 
>
>    $ docker tag 容器id nan/mytomcat:1.0 #然后再次push 

---

**阿里云发布：**

1. 官网详细教程：https://cr.console.aliyun.com/repository/ 
2. 阿里云开发者平台：https://dev.aliyun.com/search.html

```shell
$ sudo docker login --username= registry.cn-shenzhen.aliyuncs.com 
$ sudo docker tag [ImageId] registry.cn-shenzhen.aliyuncs.com/nanzx/mytomcat:[镜像版本号]
$ sudo docker push registry.cn-shenzhen.aliyuncs.com/nanzx/mytomcat:[镜像版本号]
```

3. 接下来就可以在阿里云开发者平台的公有云查询到
4. 将镜像下载下来

```shell
$ sudo docker pull registry.cn-shenzhen.aliyuncs.com/nanzx/mytomcat:[镜像版本号]
```



![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20220706215002.png)

---



# Docker常用安装

## Mysql

```shell
docker run -p 12345:3306 --name mysql 
-v /root/mysql/conf:/etc/mysql/conf.d 
-v /root/mysql/logs:/logs 
-v /root/mysql/data:/var/lib/mysql 
-e MYSQL_ROOT_PASSWORD=123456 
-d mysql:5.6 
```

命令说明：

-p 12345:3306：将主机的12345端口映射到docker容器的3306端口。

--name mysql：运行容器的名字

-v /root/mysql/conf:/etc/mysql/conf.d ：将主机/root/mysql录下的conf/my.cnf 挂载到容器的 /etc/mysql/conf.d

-v /root/mysql/logs:/logs：将主机/root/mysql目录下的 logs 目录挂载到容器的 /logs

-v /root/mysql/data:/var/lib/mysql ：将主机/root/mysql目录下的data目录挂载到容器的 /var/lib/mysql 

-e MYSQL_ROOT_PASSWORD=123456：初始化 root 用户的密码。

-d mysql:5.6 : 后台程序运行mysql5.6

---

## Redis

```shell
docker run -p 6379:6379 
-v /root/redis/data:/data 
-v /root/redis/conf/redis.conf:/usr/local/etc/redis/redis.conf  
-d redis:3.2 redis-server /usr/local/etc/redis/redis.conf --appendonly yes
```

命令说明：

redis-server /usr/local/etc/redis/redis.conf --appendonly yes ：表示启动持久化存储

主机/root/redis/conf/redis.conf需要自行配置

连接：` docker exec -it 运行着Rediis服务的容器ID redis-cli`

---

## CentOS

```shell
docker run -it -d -p 8081:8081 --name centos01 centos
```

命令说明：

需要加 -it 保持伪终端的运行，防止容器因没有进程Kill itself

---

## Zookeeper

```shell
docker run -d
-p 2181:2181
-v /root/zookeeper/data/:/data/
--name=zookeeper
--privileged zookeeper
```



# 总结

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20210905223219.png)

---

