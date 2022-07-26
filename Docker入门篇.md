---
title: Docker入门篇
categories:
  - Docker
tags:
  - Docker
top_img: 'https://cdn.jsdelivr.net/npm/nan-picture/img/wp7.jpg'
cover: 'https://cdn.jsdelivr.net/npm/nan-picture/img/wp7.jpg'
abbrlink: '6883'
date: 2021-05-31 10:21:45
updated: 2021-6-19 13:38:22
---

>参考视频：周阳老师的尚硅谷_Docker核心技术（基础篇）https://www.bilibili.com/video/BV1Vs411E7AR

# 简介

[Docker官网](https://www.docker.com/)

[Docker官方文档](https://docs.docker.com/)

[Docker Hub官网](https://hub.docker.com/)

[中文参考手册](https://docker_practice.gitee.io/zh-cn/)

## 是什么

**为什么会有Docker出现？**

一款产品从开发到上线，从操作系统，到运行环境，再到应用配置。作为开发+运维之间的协作我们需要关心很多东西，这也是很多互联网公司都不得不面对的问题，特别是各种版本的迭代之后，不同版本环境的兼容，对运维人员都是考验。

环境配置如此麻烦，换一台机器，就要重来一次，费力费时。很多人想到，能不能从根本上解决问题，**软件可以带环境安装？**也就是说，安装的时候，把原始环境一模一样地复制过来。**开发人员利用 Docker 可以消除协作编码时“在我的机器上可正常工作”的问题。**

之前在服务器配置一个应用的运行环境，要安装各种软件，Java/Tomcat/MySQL/JDBC驱动包等。安装和配置这些东西有多麻烦就不说了，它还不能跨平台。假如我们是在 Windows 上安装的这些环境，到了 Linux 又得重新装。况且就算不跨操作系统，换另一台同样操作系统的服务器，要移植应用也是非常麻烦的。

---

**Docker理念：**

Docker是基于**Go语言**实现的云开源项目。

Docker的主要目标是“Build，Ship and Run Any App，Anywhere”，也就是通过对应用组件的封装、分发、部署、运行等生命周期的管理，使用户的APP（可以是一个WEB应用或数据库应用等等）及其运行环境能够做到**“一次封装，到处运行”**。

Linux 容器技术的出现就解决了这样一个问题，而 Docker 就是在它的基础上发展过来的。将应用运行在 Docker 容器上面，而 Docker 容器在任何操作系统上都是一致的，这就实现了**跨平台、跨服务器**。只需要一次配置好环境，换到别的机子上就可以一键部署好，大大简化了操作

---

一句话：**解决了运行环境和配置软件问题的容器，方便做持续集成并有助于整体发布的容器虚拟化技术。**



## 能干嘛

之前的虚拟化技术：**虚拟机**（virtual machine）就是带环境安装的一种解决方案。

它可以在一种操作系统里面运行另一种操作系统，比如在Windows 系统里面运行Linux 系统。应用程序对此毫无感知，因为虚拟机看上去跟真实系统一模一样，而对于底层系统来说，虚拟机就是一个普通文件，不需要了就删掉，对其他部分毫无影响。这类虚拟机完美的运行了另一套系统，能够使应用程序，操作系统和硬件三者之间的逻辑不变。 

![](https://cdn.jsdelivr.net/npm/nan-picture/blog/20210531162229.png)

虚拟机的缺点：资源占用多、冗余步骤多、启动慢

---

容器虚拟化技术：由于虚拟机存在这些缺点，Linux 发展出了另一种虚拟化技术：**Linux 容器**（Linux Containers，缩写为 LXC）。

Linux 容器**不是模拟一个完整的操作系统**，而是对进程进行隔离。有了容器，就可以将软件运行所需的所有资源打包到一个隔离的容器中。容器与虚拟机不同，不需要捆绑一整套操作系统，只需要软件工作所需的库资源和设置。系统因此而变得高效轻量并保证部署在任何环境中的软件都能始终如一地运行。

![](https://cdn.jsdelivr.net/npm/nan-picture/blog/20210531163236.png)

---

比较了 Docker 和传统虚拟化方式的不同之处：

- 传统虚拟机技术是虚拟出一套硬件后，在其上运行一个完整操作系统，在该系统上再运行所需应用进程；

- 而容器内的应用进程直接运行于宿主的内核，容器内**没有自己的内核，而且也没有进行硬件虚拟**。因此容器要比传统虚拟机更为轻便。

- 每个容器之间互相**隔离**，每个容器有自己的文件系统 ，容器之间进程不会相互影响，能区分计算资源。



## DevOps（开发/运维）

- 更快速的应用交付和部署

  传统的应用开发完成后，需要提供一堆安装程序和配置说明文档，安装部署后需根据配置文档进行繁杂的配置才能正常运行。Docker化之后只需要交付少量**容器镜像文件**，在正式生产环境加载镜像并运行即可，应用安装配置在镜像里已经内置好，大大节省部署配置和测试验证时间。

- 更便捷的升级和扩缩容

  随着微服务架构和Docker的发展，大量的应用会通过微服务方式架构，应用的开发构建将变成搭乐高积木一样，每个Docker容器将变成一块“积木”，应用的升级将变得非常容易。当现有的容器不足以支撑业务处理时，可通过镜像运行新的容器进行快速扩容，使应用系统的扩容从原先的天级变成分钟级甚至秒级。

- 更简单的系统运维

  应用容器化运行后，生产环境运行的应用可与开发、测试环境的应用高度一致，容器会将应用程序相关的环境和状态完全封装起来，不会因为底层基础架构和操作系统的不一致性给应用带来影响，产生新的BUG。当出现程序异常时，也可以通过测试环境的相同容器进行快速定位和修复。

- 更高效的计算资源利用

  Docker是内核级虚拟化，其不像传统的虚拟化技术一样需要额外的Hyper-visor支持，所以在一台物理机上可以运行很多个容器实例，可大大提升物理服务器的CPU和内存的利用率。



# 安装

## Docker的基本组成

* 镜像：Docker 镜像（Image）就是一个**只读**的模板。镜像可以用来创建 Docker 容器，一个镜像可以创建很多容器。**镜像与容器的关系类似于面向对象编程中的类与对象。**

- 容器：Docker 利用容器（Container）独立运行的一个或一组应用。容器是用镜像创建的运行实例。它可以被启动、开始、停止、删除。每个容器都是相互隔离的、保证安全的平台。**可以把容器看做是一个简易版的 Linux 环境**（包括root用户权限、进程空间、用户空间和网络空间等）和运行在其中的应用程序。

- 仓库：仓库是集中存放镜像文件的场所。仓库(Repository)和仓库注册服务器（Registry）是有区别的。仓库注册服务器上往往存放着多个仓库，每个仓库中又包含了多个镜像，每个镜像有不同的标签（tag）。仓库分为公开仓库（Public）和私有仓库（Private）两种形式。最大的公开仓库是 Docker Hub，存放了数量庞大的镜像供用户下载。国内的公开仓库包括阿里云 、网易云等。

- 架构图：

  ![](https://cdn.jsdelivr.net/npm/nan-picture/blog/20210531175422.png)

  >**总结：**
  >
  >Docker 本身是一个**容器运行载体或称之为管理引擎**。我们把应用程序和配置依赖打包好形成一个可交付的运行环境，这个打包好的运行环境就似乎 image镜像文件。只有通过这个镜像文件才能生成 Docker 容器。image 文件可以看作是容器的模板。Docker 根据 image 文件生成容器的实例。同一个 image 文件，可以生成多个同时运行的容器实例。
  >
  >- image 文件生成的容器实例，本身也是一个文件，称为镜像文件。
  >
  >- 一个容器运行一种服务，当我们需要的时候，就可以通过docker客户端创建一个对应的运行实例，也就是我们的容器。
  >
  >- 至于仓库，就是放了一堆镜像的地方，我们可以把镜像发布到仓库中，需要的时候从仓库中拉下来就可以了。

## 环境准备

Docker支持以下的CentOS版本：

- CentOS 7 (64-bit)系统的内核版本为 3.10 以上。

- CentOS 6.5 (64-bit) 或更高的版本系统的内核版本为 2.6.32-431 或者更高版本。

查看系统内核：uname命令用于打印当前系统相关信息（内核版本号、硬件架构、主机名称和操作系统类型等）。

```shell
uname -r
```

查看已安装的CentOS版本信息（CentOS6.8有，CentOS7无该命令）

```shell
CentOS6.8: cat /etc/redhat-release
CentOS7:  cat /etc/os-release
```

 

## 安装步骤

CentOS7安装Docker参考文档地址：https://docs.docker.com/engine/install/centos/

```shell
 #1.卸载旧版本
[root@localhost ~]yum remove docker \
                             docker-client \
                             docker-client-latest \
                             docker-common \
                             docker-latest \
                             docker-latest-logrotate \
                             docker-logrotate \
                             docker-engine
                  
#2.需要的安装包 
[root@localhost ~]yum install -y yum-utils   

#3.设置镜像仓库
[root@localhost ~]yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo

#4.更新yum软件包索引 
[root@localhost ~]yum makecache fast

#5.安装docker相关的 docker-ce 社区版 而ee是企业版 
[root@localhost ~]yum install docker-ce docker-ce-cli containerd.io 

#6.使用docker version查看是否安装成功 
[root@localhost ~]docker version 

#7.启动Docker并设置为开机自启
[root@localhost ~]systemctl start docker
[root@localhost ~]systemctl enable docker

#8.运行 hello-world 映像验证 Docker Engine 是否已正确安装。
[root@localhost ~]docker run hello-world 
Hello from Docker! 
This message shows that your installation appears to be working correctly. 

To generate this message, Docker took the following steps: 
1. The Docker client contacted the Docker daemon. 
2. The Docker daemon pulled the "hello-world" image from the Docker Hub. (amd64) 
3. The Docker daemon created a new container from that image which runs the executable that produces the output you are currently reading. 
4. The Docker daemon streamed that output to the Docker client, which sent it to your terminal. 

To try something more ambitious, you can run an Ubuntu container with: 
$ docker run -it ubuntu bash 

Share images, automate workflows, and more with a free Docker ID: 
https://hub.docker.com/ 

For more examples and ideas, visit: 
https://docs.docker.com/get-started/
```



## 卸载步骤

1. 停止Docker：

   ```shell
   systemctl stop Docker
   ```

2. 卸载 Docker Engine、CLI 和 Containerd 包：

   ```shell
   yum remove docker-ce docker-ce-cli containerd.io
   ```

3. 主机上的映像、容器、卷或自定义配置文件不会自动删除。删除所有镜像、容器和卷：

   ```shell
   rm -rf /var/lib/docker
   rm -rf /var/lib/containerd
   ```

您必须手动删除所有已编辑的配置文件。



## 配置阿里云镜像加速

1. 创建目录：`mkdir -p /etc/docker`
2. 打开配置文件：`vi /etc/docker/daemon.json`
3. 更改配置文件，设置镜像加速器(二选一)
   - 网易：`{"registry-mirrors": ["http://hub-mirror.c.163.com"]}`
   - 阿里云（推荐）：`{"registry-mirrors":["https://drkkei0a.mirror.aliyuncs.com"]}`
4. 重新加载配置文件：`systemctl daemon-reload`
5. 重启Docker：`systemctl restart docker`

>阿里云的镜像地址需要自己登陆阿里云获取。
>
>步骤是：**登陆阿里云-容器镜像服务-镜像加速器-CentOS**，找到自己的地址。
>
>（https://cr.console.aliyun.com/cn-hangzhou/instances/mirrors）



## 底层原理

`docker run hello-world`，run干了什么？

![](https://cdn.jsdelivr.net/npm/nan-picture/blog/20210601231109.png)

---

Docker是怎么工作的？

Docker是一个**Client-Server结构**的系统，Docker守护进程运行在主机上， 然后通过Socket连接从客户端访问，守护进程从客户端接受命令并管理运行在主机上的容器。 容器，是一个运行时环境，就是我们前面说到的集装箱。

![](https://cdn.jsdelivr.net/npm/nan-picture/blog/20210601232208.png)

---

为什么Docker比VM快？

1. docker有着比虚拟机更少的抽象层。由于docker不需要Hyper-visor**实现硬件资源虚拟化**，运行在docker容器上的程序直接使用的都是实际物理机的硬件资源。因此在CPU、内存利用率上docker将会在效率上有明显优势。
2. docker利用的是宿主机的内核，而不需要Guest OS。因此当新建一个容器时，docker不需要和虚拟机一样重新加载一个操作系统内核。从而避免引寻、加载操作系统内核这个比较费时费资源的过程，当新建一个虚拟机时，虚拟机软件需要加载Guest OS，这个新建过程是分钟级别的。而docker由于直接利用宿主机的操作系统,则省略了这个过程，因此新建一个docker容器只需要几秒钟。

​                   ![](https://cdn.jsdelivr.net/npm/nan-picture/blog/20210601232756.png)

|          | Dcoker容器             | 虚拟机（VM）                |
| -------- | ---------------------- | --------------------------- |
| 操作系统 | 与宿主机共享OS         | 宿主机OS上运行虚拟机OS      |
| 存储大小 | 镜像小，便于存储和传输 | 镜像庞大（vmdk，vdi等）     |
| 运行性能 | 几乎无额外性能损失     | 操作系统额外的CPU、内存消耗 |
| 移植性   | 轻便灵活，适应于Linux  | 笨重，与虚拟化技术耦合度高  |

---



# 帮助命令

```shell
docker version #显示docker的版本信息。 
docker info #显示docker的系统信息，包括镜像和容器的数量 
docker --help #帮助命令
```

---



# 镜像命令

## docker images

列出本地主机上的所有镜像

```shell
[root@localhost ~]docker images 
REPOSITORY   TAG    IMAGE ID       CREATED        SIZE 
mysql        5.7    e73346bdf465   24 hours ago   448MB
```

>REPOSITORY：表示镜像的仓库源
>
>TAG：镜像的标签
>
>IMAGE ID：镜像ID
>
>CREATED：镜像创建时间
>
>SIZE：镜像大小
>
> 同一仓库源可以有多个 TAG，代表这个仓库源的不同个版本，我们使用 REPOSITORY:TAG 来定义不同的镜像。
>
>如果你不指定一个镜像的版本标签，例如你只使用 ubuntu，docker 将默认使用 ubuntu:latest 镜像，最新镜像
>
>**可选项 Options：**
>
>- -a	列出本地所有的镜像，含中间映像层
>- -q   只显示镜像ID
>- --digests   显示镜像的摘要信息
>- --no-trunc   显示完整的镜像信息

---

## docker search

```shell
[root@localhost ~]docker search [OPTIONS] 某个镜像名字 
```

>是在远程仓库【dockerhub】中搜索：https://hub.docker.com/
>
>**可选项 Options：**
>
>- --automated    只列出automated   build类型的镜像
>
>- -s    列出收藏数不小于指定值得镜像
>
>- --no-trunc    显示完整的镜像信息

---

## docker pull

```shell
[root@localhost ~]docker pull 镜像名[:tag]
```

>docker pull tomcat:8 等价于 docker pull docker.io/library/tomcat:8（真实地址）
>
>如果不写tag，默认就是latest 

---

## docker rmi

```shell
[root@localhost ~]docker rmi -f 镜像id #删除指定的镜像 
[root@localhost ~]docker rmi -f 镜像id 镜像id 镜像id 镜像id #删除指定的镜像 
[root@localhost ~]docker rmi -f $(docker images -aq) #删除全部的镜像
```

>镜像id也可以换成镜像名[:tag]



# 容器命令

**有了镜像才能创建容器。**

## 新建并启动容器

```shell
[root@localhost ~]docker run [OPTIONS] IMAGE 

#启动一个做了端口映射的tomcat：
[root@localhost ~]docker run -d -p 8888:8080 tomcat
#以交互模式启动一个容器，在容器内执行/bin/bash命令：
[root@localhost ~]docker exec -it 容器的id或者名字 /bin/bash
```

>**可选项 Options（常用）：**有些是一个减号，有些是两个减号（一般是全称）
>
>--name="容器新名字"：为容器指定一个名称；
>
>-d：后台运行容器，并返回容器ID，也即启动守护式容器；
>
>-i：以交互模式运行容器，通常与 -t 同时使用；
>
>-e：环境配置
>
>-t：为容器重新分配一个**伪输入终端**，通常与 -i 同时使用；
>
>-P（大写）：随机端口映射；
>
>-p：指定端口映射：
>
>- -p ip:主机端口:容器端口 
>- -p ip::容器端口 
>
>- -p 主机端口:容器端口(常用) 
>
>- -p 容器端口 

---

## 列出当前所有正在运行的容器

```shell
[root@localhost ~]docker ps [OPTIONS]
```

>**可选项 Options（常用）：**
>
>-a：列出当前所有正在运行的容器+历史上运行过的(常用)
>
>-l：显示最近创建的容器。
>
>-n：显示最近n个创建的容器。
>
>-q：静默模式，只显示容器编号。
>
>--no-trunc：不截断输出。

---

## 退出交互的容器

```shell
#第一种：容器停止退出
[root@localhost /bin/bash]exit

#第二种：容器不停止退出
ctrl+p+q
```

---

## 启动、停止和重启容器

```shell
[root@localhost ~]docker start 容器的id或者名字
[root@localhost ~]docker stop 容器的id或者名字
[root@localhost ~]docker restart 容器的id或者名字

#强制停止容器：
[root@localhost ~]docker kill 容器的id或者名字
```

---

## 删除已停止容器

```shell
[root@localhost ~]docker rm 容器的id或者名字

#一次性删除多个容器：
[root@localhost ~]docker rm -f $(docker ps -aq)
#或者
[root@localhost ~]docker ps -a -q | xargs docker rm
```

---

## 其他常用命令

### 启动守护式容器

```shell
[root@localhost ~]docker run -d 镜像名

#使用镜像centos:latest以后台模式启动一个容器
[root@localhost ~]docker run -d centos
#问题：
docker ps -a 进行查看, 会发现centos容器已经退出

很重要的要说明的一点: Docker容器后台运行,就必须有一个前台进程。

容器运行的命令如果不是那些一直挂起的命令（比如运行top，tail），就会自动退出的。
centos为后台进程模式运行,就导致docker前台没有运行的应用,
这样的容器后台启动后,会立即停止因为它觉得它没有停供服务了。

所以，最佳的解决方案是，将你要运行的程序以前台进程的形式运行。
```

---

### 查看容器日志

```shell
[root@localhost ~]docker logs -f -t --tail n 容器id

#模拟日志
[root@localhost ~]docker run -d centos /bin/sh -c "while true;do echo 6666;sleep 1;done" 
```

>- -t 是加入时间戳
>
>- -f 跟随最新的日志打印
>
>- --tail 数字 显示最后多少条

---

### 查看容器内运行的进程

```shell
[root@localhost ~]docker top 容器id
```

---

### 查看容器的元数据

```shell
[root@localhost ~]docker inspect 容器id
```

---

### 进入运行的容器并以命令行交互

```shell
# 通常容器都是使用后台方式运行的，需要进入容器修改一些配置时可以使用如下命令：
[root@localhost ~]docker exec -it 容器id /bin/bash

#重新进入容器：
[root@localhost ~]docker attach 容器id
```

>区别：
>
>- exec是在容器中打开新的终端，并且可以启动新的进程
>- attach是直接进入容器启动命令的终端，不会启动新的进程

---

### 容器和主机间的文件拷贝

```shell
#从容器拷贝文件到宿主机
[root@localhost ~]docker cp 容器id:容器内路径 目的主机路径
#从宿主机拷贝文件到容器
[root@localhost ~]docker cp 目的主机路径 容器id:容器内路径

#例如
[root@localhost ~]docker cp 55321bcae33d:/usr/local/text.txt /root/
[root@localhost ~]docker cp /opt/test/file.txt 55321bcae33d:/opt/testnew/

#   需要注意的是，不管容器有没有启动，拷贝命令都会生效。
```

---



# Docker可视化

Portainer是一个可视化的容器镜像的图形管理工具，利用Portainer可以轻松构建，管理和维护Docker环境。 而且完全免费，基于容器化的安装方式，方便高效部署。

```shell
[root@localhost ~]docker run -d -p 9000:9000 --restart=always -v /var/run/docker.sock:/var/run/docker.sock --privileged=true portainer/portainer
```

