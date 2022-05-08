---
title: Docker Compose
categories:
  - Docker
tags:
  - Docker Compose
top_img: 'https://cdn.jsdelivr.net/gh/839777408/tupian/img/wp12.jpg'
cover: 'https://cdn.jsdelivr.net/gh/839777408/tupian/img/wp12.jpg'
abbrlink: 51962
date: 2021-09-12 22:11:03
updated: 2021-10-27 23:45:42
---

>参考视频：编程不良人的Docker-Compose实战教程  https://www.bilibili.com/video/BV1ZT4y1K75K?p=24
>

# 简介

- Compose是 Docker 官方的开源项目，可以轻松、高效的管理容器，实现对Docker容器集群的快速编排。定位是定义和运行多个Docker容器的应用。

- 我们自定义使用 Docker 时，需先编写 Dockerfile 文件，再使用 `docker build`、`docker run` 等命令操作容器。然而微服务架构的应用系统一般包含若干个微服务，每个微服务一般都会部署多个实例，如果每个微服务都要手动操作，那么维护量会很大而且效率会很低。
- Compose项目由Python编写，实际上调用了Docker服务提供的API对容器进行管理。

---

官网相关介绍：

**Compose是一个用于定义和运行多容器Docker应用程序的工具**。使用Compose，我们可以使用YAML文件来配置应用程序的服务。然后，只需一个命令，就可以从配置中创建并启动所有服务。

使用Compose基本上分为三步：

1. 使用`Dockerfile`定义应用程序的环境，以便可以在任何地方复制。

2. 在`docker compose.yml`中定义组成应用程序的服务，以便它们可以在隔离的环境中一起运行。

3. 运行`docker compose up`和[docker compose命令](https://docs.docker.com/compose/cli-command/)启动并运行整个应用程序。我们也可以使用docker compose二进制文件运行`docker compose up`。

>Compose的重要概念：
>
>- 服务(services)： 应用容器（web、redis、mysql、nginx...）
>- 项目(project)： 由一组关联的应用容器组成的一个完整业务单元，在docker-compose.yml文件中定义
>
>Compose的默认管理对象是项目，通过子命令对项目的一组容器进行便捷的生命周期管理。



# 安装与卸载

**下载：**

```shell
# 官网提供 （没有下载成功）
sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
 
# 推荐使用国内地址
curl -L https://get.daocloud.io/docker/compose/releases/download/1.29.2/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
```

可到[官网](https://docs.docker.com/compose/install/)查看最新版本号

下载成功可在/usr/local/bin/目录下看到 docker-compose

**授权：**

```shell
#使docker-compose成为可执行文件
sudo chmod +x /usr/local/bin/docker-compose
```

**查看版本号：**

```shell
[root@nanzx bin]# docker-compose version
docker-compose version 1.29.2, build 5becea4c
docker-py version: 5.0.0
CPython version: 3.7.10
OpenSSL version: OpenSSL 1.1.0l  10 Sep 2019
```

**卸载：**

```shell
sudo rm /usr/local/bin/docker-compose
```



# 体验

## 配置

定义应用程序依赖项。

在此页面上，将构建一个在 Docker Compose 上运行的简单 Python Web 应用程序。该应用程序使用 Flask 框架并在 Redis 中维护一个命中计数器。

1.为项目创建一个目录：

```shell
$ mkdir composetest
$ cd composetest
```

2.在`composetest`项目目录中创建一个名为`app.py`的文件并粘贴以下内容的文件：

```python
import time

import redis
from flask import Flask

app = Flask(__name__)
cache = redis.Redis(host='redis', port=6379)

def get_hit_count():
    retries = 5
    while True:
        try:
            return cache.incr('hits')
        except redis.exceptions.ConnectionError as exc:
            if retries == 0:
                raise exc
            retries -= 1
            time.sleep(0.5)

@app.route('/')
def hello():
    count = get_hit_count()
    return 'Hello World! I have been seen {} times.\n'.format(count)
```

3.在项目目录中创建另一个文件`requirements.txt`并粘贴以下内容的文件：

```tex
flask
redis
```

## 创建一个 Dockerfile

在此步骤中，将编写一个用于构建 Docker 映像的 Dockerfile。该图像包含 Python 应用程序所需的所有依赖项，包括 Python 本身。

在项目目录中，创建一个名为`Dockerfile`并粘贴以下内容的文件（**注意：文件名一定要大小写一致**）：

```dockerfile
#从 Python 3.7 映像开始构建映像
FROM python:3.7-alpine	

#将工作目录设置为/code
WORKDIR /code	

#设置flask命令使用的环境变量
ENV FLASK_APP=app.py								
ENV FLASK_RUN_HOST=0.0.0.0	

#安装 gcc 和其他依赖项
RUN apk add --no-cache gcc musl-dev linux-headers	

#复制requirements.txt并安装 Python 依赖项
COPY requirements.txt requirements.txt	
RUN pip install -r requirements.txt

#侦听端口 5000
EXPOSE 5000		
#将.项目中的当前目录复制到.镜像中的workdir
COPY . .			
#将容器的默认命令设置为flask run
CMD ["flask", "run"]								
```

## 在 Compose 文件中定义服务

`docker-compose.yml`在项目目录中创建一个名为的文件并粘贴以下内容：

```yaml
version: "3.9"
services:
  web:
    build: .
    ports:
      - "5000:5000"
  redis:
    image: "redis:alpine"
```

这个 Compose 文件定义了两个服务：web 和 redis

该`web`服务使用从`Dockerfile`当前目录中构建的映像。然后它将容器和主机绑定到暴露的端口`5000`， 此示例服务使用 Flask Web 服务器的默认端口`5000`。

该`redis`服务使用 从 Docker Hub 注册表中提取的公共[Redis](https://registry.hub.docker.com/_/redis/)映像。



## 使用 Compose 构建并运行应用程序

在项目目录，通过`docker-compose up`运行启动应用程序。

启动成功可以通过访问`主机地址:5000`看到：Hello World! I have been seen 1 times. 刷新页面数字应该递增。

切换到另一个终端窗口，然后键入`docker image ls`以列出本地镜像。

```shell
[root@nanzx ~]# docker images
REPOSITORY        TAG          IMAGE ID       CREATED          SIZE
composetest_web   latest       712a70388fb3   15 minutes ago   184MB
whyour/qinglong   latest       af93ec71ba86   5 days ago       364MB
python            3.7-alpine   a436fb2c575c   5 days ago       41.9MB
```

通过键入`docker ps`以列出正在运行的容器：

- 默认的服务名：项目目录名_ 服务名 _num
- 多个服务器、集群：_num 副本数量（集群状态下，服务不可能只有一个运行实例）

```shell
[root@nanzx ~]# docker ps
CONTAINER ID  IMAGE    		   COMMAND  	  CREATED         STATUS          PORTS       NAMES
b2e8d7af2c81  composetest_web  "flask run"  13 minutes ago  Up 13 minutes  0.0.0.0:5000->5000/tcp, :::5000->5000/tcp  composetest_web_1
dd96803c2f4a  redis:alpine  "docker-entrypoint.s…"  13 minutes ago  Up 13 minutes  6379/tcp      composetest_redis_1
```

通过键入`docker network ls`以列出docker的网络：

- 项目中的服务都在同个网络下，可以通过容器名进行访问

```shell
[root@nanzx ~]# docker network ls
NETWORK ID     NAME                  DRIVER    SCOPE
cb43c4e5baa2   bridge                bridge    local
09c06a901a32   composetest_default   bridge    local
c5f5efea2089   host                  host      local
b9e8be9bb948   none                  null      local

[root@nanzx ~]# docker network inspect composetest_default
[
    {
        "Name": "composetest_default",
        "Id": "09c06a901a3215d2c4d03b47f51ade034a22212bb8251133c5a32f41c605bc00",
        "Created": "2021-09-13T22:31:16.22202105+08:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "172.18.0.0/16",
                    "Gateway": "172.18.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": true,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {
            "b2e8d7af2c81f67962bc036878ef7df07ce8774ec38e53cd7bc45e857ba3ef62": {
                "Name": "composetest_web_1",
                "EndpointID": "4ee462ebcfe96a083593a10964651240023b1ea1b5a0610615935f733a80a9df",
                "MacAddress": "02:42:ac:12:00:03",
                "IPv4Address": "172.18.0.3/16",
                "IPv6Address": ""
            },
            "dd96803c2f4ac049e50a555504b01d4706e5e6a57cb686202bcc11e1349bee76": {
                "Name": "composetest_redis_1",
                "EndpointID": "4d33c44c5dbe12084bff474af9f11e0a67b4bb8c3a32c81782410615e7b4f680",
                "MacAddress": "02:42:ac:12:00:02",
                "IPv4Address": "172.18.0.2/16",
                "IPv6Address": ""
            }
        },
        "Options": {},
        "Labels": {
            "com.docker.compose.network": "default",
            "com.docker.compose.project": "composetest",
            "com.docker.compose.version": "1.29.2"
        }
    }
]

```

停止应用程序可以使用`docker-compose down` 或 Ctrl + C

**小结：以前都是单个 docker run 启动容器，现在可以通过docker-compose一键启动和停止所有服务。**



# 常用模板指令

[官网：version参考中间栏目，yaml规则具体参考右侧栏目](https://docs.docker.com/compose/compose-file/compose-file-v3/)

中文参考手册:https://docker_practice.gitee.io/zh-cn/

## yaml模板

默认的模板文件名称为 `docker-compose.yml`，格式为 YAML 格式。

```yaml
#yaml可以大致分为3层

# 1.版本
version: ''

# 2.服务
services：
	tomcat: #自定义服务名称
		#服务配置
		image
		build
		network
		...
    myRedis: 
    	...
    	
# 3.其他配置：网络/卷、全局配置等
volumes：
networks:
configs:
```

**注意：**

- 每个服务都必须通过 `image` 指令指定镜像或 `build` 指令（需要 Dockerfile）等来自动构建生成镜像。
- 如果使用 `build` 指令，在 `Dockerfile` 中设置的选项(例如：`CMD`, `EXPOSE`, `VOLUME`, `ENV` 等) 将会自动被获取，无需在 `docker-compose.yml` 中重复设置。

---

## container_name

指定容器名称。默认将会使用 `项目名称_服务名称_序号` 这样的格式。

```yaml
version: "3"
services:
  web:
	container_name: docker-web-container
```

**注意:** 指定容器名称后，该服务将无法进行扩展（scale），因为 Docker 不允许多个容器具有相同的名称。

---

## image

指定为镜像名称或镜像 ID。如果镜像在本地不存在，`Compose` 将会尝试拉取这个镜像。

```yaml
image: ubuntu 		#REPOSITORY,不加版本号拉取最新版：latest
image: mysql:5.7	#REPOSITORY:TAG
image: a4bc65fd 	#digest, https://www.jianshu.com/p/62c6ea1a0975
```

---

## build

指定 `Dockerfile` 所在文件夹的路径（可以是绝对路径，或者相对 docker-compose.yml 文件的路径）。 `Compose` 将会利用它自动构建这个镜像，然后使用这个镜像。

```yaml
version: '3'
services:
  webapp:
    build: ./dir
```

你也可以使用 `context` 指令指定 `Dockerfile` 所在文件夹的路径。

使用 `dockerfile` 指令指定 `Dockerfile` 文件名。

使用 `arg` 指令指定构建镜像时的变量。

```yaml
version: '3'
services:

  webapp:
    build:
      context: ./dir
      dockerfile: Dockerfile-alternate
      args:
        buildno: 1
```

---

## ports

暴露端口信息。使用宿主端口：容器端口 `(HOST:CONTAINER)` 格式，或者仅仅指定容器的端口（宿主将会随机选择端口）都可以。

```yaml
ports:
 - "3000"
 - "8000:8000"
 - "49100:22"
 - "127.0.0.1:8001:8001"
```

注意：当使用 `HOST:CONTAINER` 格式来映射端口时，如果你使用的容器端口小于 60 并且没放到**引号**里，可能会得到错误结果，因为 `YAML` 会自动解析 `xx:yy` 这种数字格式为 60 进制。为避免出现这种问题，建议数字串都采用引号包括起来的字符串格式。

---

## volumes

[数据卷](https://nanzx.top/posts/f978/)所挂载路径设置。可以设置为宿主机路径(`HOST:CONTAINER`)或者数据卷名称(`VOLUME:CONTAINER`)，并且可以设置访问模式 （`HOST:CONTAINER:ro`）。

该指令中路径支持相对路径。

```yaml
volumes:
 - /var/lib/mysql
 - cache/:/tmp/cache
 - ~/configs:/etc/configs/:ro
```

如果路径为数据卷名称，必须在文件中配置数据卷。

```yaml
version: "3"

services:
  my_src:
    image: mysql:8.0
    volumes:
      - mysql_data:/var/lib/mysql

volumes:
  mysql_data: #compose会自动创建该卷名并在前面加上项目（所在文件夹）名为前缀，可通过docker volumes ls查看
  	external: #可选
  		true #true时，启动容器前需先通过`docker volume create 卷名`创建自定义数据卷
```

---

## networks

配置容器连接的网络。

```yaml
version: "3"
services:
  some-service:
    networks:
     - some-network
     - other-network

networks:
  some-network:
  other-network
 	 external: #可选
  		true #true时，使用外部指定网桥，必须存在
```

---

## environment

设置环境变量，可以使用数组或字典两种格式。

只给定名称的变量会自动获取运行 Compose 主机上对应变量的值，可以用来防止泄露不必要的数据。

```yaml
version: "3"
services:
  mysql:
  	image: mysql:5.7
	environment:
		-MYSQL_ROOT_PASSWORD=123456

##########################################

environment:
  RACK_ENV: development
  SESSION_SECRET:
#等价于
environment:
  - RACK_ENV=development
  - SESSION_SECRET
```

如果变量名称或者值中用到 `true|false，yes|no` 等表达 [布尔](https://yaml.org/type/bool.html) 含义的词汇，最好放到引号里，避免 YAML 自动解析某些内容为对应的布尔语义。这些特定词汇，包括：

```bash
y|Y|yes|Yes|YES|n|N|no|No|NO|true|True|TRUE|false|False|FALSE|on|On|ON|off|Off|OFF
```

---

## env_file

从文件中获取环境变量，可以为单独的文件路径或列表。

- 如果通过 `docker-compose -f FILE` 方式来指定 Compose 模板文件，则 `env_file` 中变量的路径会基于模板文件路径。

- 如果有变量名称与 `environment` 指令冲突，则按照惯例，以后者为准。

- 文件需要以`.env`结尾。

```yaml
version: "3"
services:
  mysql:
  	image: mysql:5.7
    env_file:
      - ./msql.env
		
###############################################

env_file:
  - ./common.env
  - ./apps/web.env
  - /opt/secrets.env
```

环境变量文件中每一行必须符合格式，支持 `#` 开头的注释行。

```bash
# common.env: Set development environment
MYSQL_ROOT_PASSWORD=123456
```

---

## command

覆盖容器启动后默认执行的命令。

```yaml
version: "3"
services:
  redis:
  	image: redis:6.3
	command: "redis-server --appendonly yes" #默认启动命令redis-server
```

---

## depends_on

表示服务之间的依赖关系。服务依赖会导致以下行为：

- `docker-compose up`按依赖顺序启动服务。在下面的例子中，在 启动`web`之前先启动`db`和`redis`。
- `docker-compose up SERVICE`自动包含`SERVICE`的依赖项。在下面的示例中，`docker-compose up web`还创建并启动`db`和`redis`。
- `docker-compose stop`按依赖顺序停止服务。在以下示例中，`web`在`db`和`redis`之前停止。

简单的例子：

```yaml
version: "3.9"
services:
  web:
    build: .
    depends_on:
      - db
      - redis
  redis:
    image: redis
  db:
    image: postgres
```

> 注意：`web`不会等待`db`和`redis`「完全启动」之后才启动。
>

---

## healthcheck

通过命令检查容器是否健康运行。

有关如何进行健康检查的详细信息，请参阅[ HEALTHCHECK Dockerfile ](https://docs.docker.com/engine/reference/builder/#healthcheck)指令中的文档。

可通过`docker-compose ps`的state查看健康检查的状态

```yaml
healthcheck:
  test: ["CMD", "curl", "-f", "http://localhost"]
  interval: 1m30s
  timeout: 10s
  retries: 3
```

---

## sysctls

配置容器内核参数。

```yaml
sysctls:
  net.core.somaxconn: 1024
  net.ipv4.tcp_syncookies: 0

sysctls:
  - net.core.somaxconn=1024
  - net.ipv4.tcp_syncookies=0
```

---

## ulimits

指定容器的 ulimits 限制值。

例如，指定最大进程数为 65535，指定文件句柄数为 20000（软限制，应用可以随时修改，不能超过硬限制） 和 40000（系统硬限制，只能 root 用户提高）。

```yaml
  ulimits:
    nproc: 65535
    nofile:
      soft: 20000
      hard: 40000
```

---



# 常用命令

## 说明

**命令对象与格式：**

对于 Compose 来说，如果没有特别的说明，命令对象将是项目，这意味着项目中所有的服务都会受到命令影响。对象既可以是项目本身，**也可以指定为项目中的服务或者容器**，如：`docker-compose mysql up`

执行 `docker-compose [COMMAND] --help` 或者 `docker-compose help [COMMAND]` 可以查看具体某个命令的使用格式。

`docker-compose` 命令的基本的使用格式是

```bash
docker-compose [-f=<arg>...] [options] [COMMAND] [ARGS...]
```

**命令选项：**

- `-f, --file FILE` 指定使用的 Compose 模板文件，默认为 `docker-compose.yml`，可以多次指定。
- `-p, --project-name NAME` 指定项目名称，默认将使用所在目录名称作为项目名。
- `--x-networking` 使用 Docker 的可拔插网络后端特性
- `--x-network-driver DRIVER` 指定网络后端的驱动，默认为 `bridge`
- `--verbose` 输出更多调试信息。
- `-v, --version` 打印版本并退出。

---

## up

格式为 `docker-compose up [options] [SERVICE...]`。

- 该命令十分强大，它将尝试自动完成包括构建镜像，（重新）创建服务，启动服务，并关联服务相关容器的一系列操作。

- 链接（依赖）的服务都将会被自动启动，除非已经处于运行状态。

- 可以说，大部分时候都可以直接通过该命令来启动一个项目。

- 默认情况，`docker-compose up` 启动的容器都在前台，控制台将会同时打印所有容器的输出信息，可以很方便进行调试。

- 当通过 `Ctrl-C` 停止命令时，所有容器将会停止。

- 如果使用 `docker-compose up -d`，将会在后台启动并运行所有的容器。一般推荐生产环境下使用该选项。

- 默认情况，如果服务容器已经存在，`docker-compose up` 将会尝试停止容器，然后重新创建（保持使用 `volumes-from` 挂载的卷），以保证新启动的服务匹配 `docker-compose.yml` 文件的最新内容

---

## down

- `docker-compose down`
- 此命令将会停止 `up` 命令所启动的容器，并移除网络（external的不会移除，自动创建的就会）

----

## exec

- `docker-compose exec 服务名`
- 进入指定的容器，不能指定容器id。

----

## ps

格式为 `docker-compose ps [options] [SERVICE...]`。

列出项目中目前的所有容器。

选项：

- `-q` 只打印容器的 ID 信息。

----

## restart

格式为 `docker-compose restart [options] [SERVICE...]`。

重启项目中的服务。

选项：

- `-t, --timeout TIMEOUT` 指定重启前停止容器的超时（默认为 10 秒）。

----

## rm

格式为 `docker-compose rm [options] [SERVICE...]`。

删除所有（停止状态的）服务容器。推荐先执行 `docker-compose stop` 命令来停止容器。

选项：

- `-f, --force` 强制直接删除，包括非停止状态的容器。一般尽量不要使用该选项。
- `-v` 删除容器所挂载的数据卷。

---

## start

格式为 `docker-compose start [SERVICE...]`。

启动已经存在的服务容器。

----

## stop

格式为 `docker-compose stop [options] [SERVICE...]`。

停止已经处于运行状态的容器，但不删除它。通过 `docker-compose start` 可以再次启动这些容器。

选项：

- `-t, --timeout TIMEOUT` 停止容器时候的超时（默认为 10 秒）。

----

## top

- `docker-compose top`
- 查看各个服务容器内运行的进程。

---

## unpause

-  `docker-compose unpause [SERVICE...]`。

- 恢复处于暂停状态中的服务。

---

## logs

-  `docker-compose logs [SERVICE...]`。

- 不加服务名查看全部日志。

---



# 一键部署WordPress博客

1. 创建项目目录`mkdir my_wordpress`，进入目录`cd my_wordpress`

2. 创建一个`docker-compose.yml`文件来启动 `WordPress`博客和一个单独的`MySQL`实例，该实例具有用于数据持久性的卷挂载：

   ```yaml
   version: "3.9"
       
   services:
     db:
       image: mysql:5.7
       volumes:
         - db_data:/var/lib/mysql
       restart: always
       environment:
         MYSQL_ROOT_PASSWORD: somewordpress
         MYSQL_DATABASE: wordpress
         MYSQL_USER: wordpress
         MYSQL_PASSWORD: wordpress
       
     wordpress:
       depends_on:
         - db
       image: wordpress:latest
       volumes:
         - wordpress_data:/var/www/html
       ports:
         - "8000:80"
       restart: always
       environment:
         WORDPRESS_DB_HOST: db:3306
         WORDPRESS_DB_USER: wordpress
         WORDPRESS_DB_PASSWORD: wordpress
         WORDPRESS_DB_NAME: wordpress
   volumes:
     db_data: {}
     wordpress_data: {}
   ```

   **注意事项**：WordPress Multisite 仅适用于端口`80`和`443`

3. 通过`docker-compose up -d`运行启动应用程序，-d 参数可以在后台启动

4. 通过访问主机地址:8000就可以访问博客





