---
title: Docker镜像和容器数据卷
categories:
  - Docker
tags:
  - Docker
top_img: 'https://cdn.jsdelivr.net/gh/839777408/tupian/img/wp9.jpg'
cover: 'https://cdn.jsdelivr.net/gh/839777408/tupian/img/wp9.jpg'
abbrlink: f978
date: 2021-06-13 16:48:26
updated: 2021-6-24 21:19:14
---

# Docker镜像

镜像是一种轻量级、可执行的独立软件包，**是用来打包软件运行环境和基于运行环境开发的软件**，它包含运行某个软件所需的所有内容，包括代码、运行时、库、环境变量和配置文件。

## 镜像加载原理

**UnionFs （联合文件系统）：**

Union文件系统（UnionFS）是一种分层、轻量级并且高性能的文件系统，它支持**对文件系统的修改作为一次提交来一层层的叠加**，同时可以将不同目录挂载到同一个虚拟文件系统下(unite several directories into a single virtual filesystem)。**Union 文件系统是 Docker 镜像的基础**。镜像可以通过分层来进行继承，基于基础镜像（没有父镜像），可以制作各种具体的应用镜像。

**特性：**一次同时加载多个文件系统，但从外面看起来，只能看到一个文件系统，联合加载会把各层文件系统叠加起来，这样最终的文件系统会包含所有底层的文件和目录。

>我们pull镜像的时候看到一层层的下载就是这个。

---

 docker的镜像实际上由一层一层的文件系统组成，这种层级的文件系统UnionFS。

bootfs(boot file system)主要包含bootloader和kernel， bootloader主要是引导加载kernel。 Linux刚启动时会加载bootfs文件系统，而在Docker镜像的最底层也是bootfs。这一层与我们典型的Linux/Unix系统是一样的，包含boot加载器和内核。当boot加载完成之后整个内核就都在内存中了，此时内存的使用权已由bootfs转交给内核，此时系统也会卸载bootfs。

rootfs (root file system) ，在bootfs之上。包含的就是典型 Linux 系统中的 /dev, /proc, /bin, /etc 等标准目录和文件。rootfs就是各种不同的操作系统发行版，比如Ubuntu，Centos等等。 

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog2/2021614125345.png)

>* **平时我们安装进虚拟机的CentOS都是好几个G，为什么docker里才200M？**
>
>对于一个精简的OS，rootfs可以很小，只需要包括最基本的命令、工具和程序库就可以了，因为底层直接用Host的kernel，自己只需要提供 rootfs 就行了。由此可见对于不同的linux发行版， bootfs基本是一致的，rootfs会有差别，因此不同的发行版可以公用bootfs。

---

**为什么Docker镜像要采用这种分层的结构呢？**

最大的一个好处就是 【共享资源】

比如：有多个镜像都从相同的 base 镜像构建而来，那么宿主机只需在磁盘上保存一份base镜像，同时内存中也只需加载一份 base 镜像，就可以为所有容器服务了。而且镜像的每一层都可以被共享。

---



## 特点

> Docker镜像都是**只读**的。当容器启动时，一个新的可写层被加载到镜像的顶部。这一层通常被称作“容器层”，“容器层”之下的都叫“镜像层”。

所有的 Docker镜像都起始于一个基础镜像层，当进行修改或添加新的内容时，就会在当前镜像层之上，创建新的镜像层。

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog2/20210626120405.png)

---

举一个简单的例子，假如基于 Ubuntu Linux16.04创建一个新的镜像，这就是新镜像的第一层；如果在该镜像中添加 Python包，就会在基础镜像层之上创建第二个镜像层；如果继续添加一个安全补丁，就会创健第三个镜像层。该镜像当前已经包含3个镜像层，如下图所示（这只是一个用于演示的很简单的例子）。

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog2/20210614183917.png)

在添加额外的镜像层的同时，镜像始终保持是当前所有镜像的组合，理解这一点非常重要。下图中举了一个简单的例子，每个镜像层包含3个文件，而镜像包含了来自两个镜像层的6个文件。

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog2/20210614231012.png)

下图中展示了一个稍微复杂的三层镜像，在外部看来整个镜像只有6个文件，这是因为最上层中的文件7是文件5的一个更新版。

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog2/20210614232723.png)

这种情況下，上层镜像层中的文件覆盖了底层镜像层中的文件。这样就使得文件的更新版本作为一个新

镜像层添加到镜像当中。

---

Docker通过存储引擎（新版本采用**快照机制**）的方式来实现镜像层堆栈，并保证多镜像层对外展示为统一的文件系统。Linux上可用的存储引撃有AUFS、 Overlay2、 Device Mapper、Btrfs以及ZFS。顾名思义，每种存储引擎都基于 Linux中对应的文件系统或者设备技术，并且每种存储引擎都有其独有的性能特点。

Docker在 Windows上仅支持 windowsfilter 一种存储引擎，该引擎基于NTFS文件系统之上实现了分层和CoW。

下图展示了与系统显示相同的三层镜像。所有镜像层堆并合并，对外提供统一的视图：

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog2/20210614232815.png)

---



## commit镜像

docker commit 提交容器使之成为一个新的镜像。

`docker commit -m="描述信息" -a="作者" 容器id 目标镜像名:[TAG]`

**案例演示：**

1. 从Hub上下载tomcat镜像到本地并成功运行`docker run -d -p 8080:8080 tomcat `
2. 发现这个默认的tomcat是没有webapps应用，官方的镜像默认webapps下面是没有文件的！ `docker exec -it 容器id /bin/bash`
3. 拷贝文件进去webapps文件夹内`docker cp /root/text.html 容器id: webapps的路径`

4. 将操作过的容器通过commit提交为一个新的镜像。我们以后就使用我们修改过的镜像即可。 `docker commit -a="阿楠" -m="add webapps app" 容器id nanzx/tomcat`

5. 通过`docker images`可以看到我们新提交的镜像。

如果你想要保存当前容器的状态，就可以通过commit来提交获得一个镜像，就好比我们我们使用虚拟机的快照。

---



# Docker容器数据卷

Docker容器产生的数据，如果不通过docker commit生成新的镜像使数据做为镜像的一部分保存下来，那么当容器删除后，数据自然也就没有了。为了能保存数据在docker中我们使用**数据卷**（类似redis里面的rdb和aof文件）。

## 介绍

卷就是目录或文件，存在于一个或多个容器中，由docker挂载到容器，但不属于联合文件系统，因此能够绕过Union File System提供一些用于**持续存储或共享数据**的特性；

卷的设计目的就是数据的持久化，完全独立于容器的生存周期，因此**Docker不会在容器删除时删除其挂载的数据卷。**

特点：

1. 数据卷可在容器之间共享或重用数据
2. 数据卷中的更改可以直接生效
3. 数据卷中的更改不会包含在镜像的更新中
4. 数据卷的生命周期一直持续到没有容器使用它为止

> **容器内挂载的文件和宿主机的文件同步修改，即使容器停止了，宿主机修改文件，也依旧同步。**

---

## 使用数据卷

**使用命令挂载(- v)：**

-  匿名挂载
  -  `docker run -d -P --name nginx1 -v /etc/nginx(容器内路径) nginx `
- 具名挂载
  - ``docker run -d -P --name nginx1 -v myVolume(自定义卷名):/etc/nginx(容器内路径) nginx ``
- 指定路径挂载
  - `docker run -d -P --name nginx1 -v /root/mynginx(宿主机路径):/etc/nginx(容器内路径) nginx`

**注意：**自定义卷名可以通过`docker volume create 卷名`创建。

---

**DockerFile添加：**

1. 根目录下新建mydocker文件夹并进入

2. 可在Dockerfile中使用 VOLUME指令 来给镜像添加一个或多个数据卷。`VOLUME["/dataVolumeContainer1","/dataVolumeContainer2","/dataVolumeContainer3"]`

   说明：出于可移植和分享的考虑，用 `-v 主机目录：容器目录`这种方法不能够直接在Dockerfile中实现。因为宿主机目录是依赖于特定宿主机的，并不能够保证在所有的宿主机上都存在这样的特定目录。

3. File构建：

   ```dockerfile
   [root@localhost mydocker]#cat dockerfile1
   # volume test
   FROM centos
   VOLUME ["/dataVolumeContainer1","/dataVolumeContainer2"]
   CMD echo "finished,--------success1"
   CMD /bin/bash
   ```

4. build生成镜像：

   ```shell
   [root@localhost mydocker]#docker build -f /mydocker/dockerfile1 -t nanzx/centos
   ```

   `docker images`可以看到我们生成的镜像，-f是指定Dockerfile文件，-t是指定镜像标签

5. 启动容器：

   ```shell
   [root@localhost mydocker]#docker run -it nanzx/centos /bin/bash
   ```

   在根目录下 `ls`可以看到我们添加的数据卷，通过`docker inspect 容器id/容器名`可以发现对应的主机目录地址在/var/lib/docker/volumes/xxxx/_data 下。

---

**查看是否挂载成功：**

- 匿名挂载方式：`docker volume ls `
- 具名挂载方式：`docker volume ls`、`docker volume inspect myVolume(自定义的卷名) `

- 指定路径挂载方式：`docker inspect 容器id/容器名`

```json
"Mounts": [
            {
                "Type": "bind",
                "Source": "/root/ql/db",   #主机内地址
                "Destination": "/ql/db",   #docker容器内地址
                "Mode": "",
                "RW": true,
                "Propagation": "rprivate"
            },
]
```

>所有的docker容器内的卷，没有指定目录的情况下都是在 /var/lib/docker/volumes/xxxx/_data 下。
>
>如果指定了目录，docker volume ls 是查看不到的。

**拓展：**

- ro（readonly：只读 ） 和 rw（readwrite：可读可写 ） ，用来改变数据卷的读写权限 
  - `docker run -d -P --name nginx1 -v myVolume:/etc/nginx:ro nginx `
  - `docker run -d -P --name nginx1 -v myVolume:/etc/nginx:rw nginx`
  - 只要看到ro就说明这个路径只能通过宿主机来操作，容器内部是无法操作！ 

- Docker挂载主机目录后，Docker访问出现cannot open directory: Permission denied
  - 解决办法：在挂载目录后多加一个`--privileged=true`参数即可

---



## 数据卷容器

命名的容器挂载数据卷，其它容器通过挂载这个(父容器)实现数据共享，挂载数据卷的容器，称之为**数据卷容器**。

**容器间传递共享（--volumes-from）：**

1. 以前面新建的镜像 nanzx/centos 为模板运行容器 dc01 ，它现在具有数据卷：
   - /dataVolumeContainer1，/dataVolumeContainer2
2. 在父容器（dc01）的数据卷dataVolumeContainer2里新增文件
3. 新增 dc02、dc03容器，继承自 dc01
   - `docker run -it --name dc02 --volumes-from dc01 nanzx/centos`
   - `docker run -it --name dc03 --volumes-from dc01 nanzx/centos`

4. 可以在dc02、dc03容器的dataVolumeContainer2目录下看到操作2新增的文件，在容器dc02、dc03的数据卷里各自新增文件，三个容器都可查看到所有新增的文件。
5. 通过`docker rm -f dc01`删除容器后，dc02、dc03的数据卷还是能正常访问和使用。

> 结论：容器之间配置信息的传递，数据卷的生命周期一直持续到没有容器使用它为止。
>
> ​		     但是一旦你持久化到了本地，这个时候，本地的数据是不会删除的！
