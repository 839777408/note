---
title: ElasticSearch的集群部署
categories:
  - ElasticSearch
tags:
  - ElasticSearch
top_img: 'https://cdn.jsdelivr.net/npm/nan-picture/img/wp3.jpg'
cover: 'https://cdn.jsdelivr.net/npm/nan-picture/img/wp3.jpg'
abbrlink: 36216
date: 2021-11-19 22:14:16
updated: 2021-11-22 23:17:23
---

# 相关概念

## 单机 & 集群

单台 Elasticsearch 服务器提供服务，往往都有最大的负载能力，超过这个阈值，服务器性能就会大大降低甚至不可用，所以生产环境中，一般都是运行在指定服务器集群中。

除了负载能力，单点服务器也存在其他问题：

- 单台机器存储容量有限

- 单服务器容易出现单点故障，无法实现高可用

- 单服务的并发处理能力有限

配置服务器集群时，集群中节点数量没有限制，大于等于 2 个节点就可以看做是集群了。一般出于高性能及高可用方面来考虑集群中节点数量都是 3 个以上。

## 集群 Cluster

一个集群就是由一个或多个服务器节点组织在一起，共同持有整个的数据，并一起提供索引和搜索功能。一个 Elasticsearch 集群有一个唯一的名字标识，这个名字默认就是“elasticsearch”。这个名字是重要的，因为一个节点只能通过指定某个集群的名字来加入这个集群。

## 节点 Node

集群中包含很多服务器，一个节点就是其中的一个服务器。作为集群的一部分，它存储数据，参与集群的索引和搜索功能。

一个节点也是由一个名字来标识的，默认情况下，这个名字是一个随机的漫威漫画角色的名字，这个名字会在启动的时候赋予节点。这个名字对于管理工作来说挺重要的，因为在这个管理过程中，你会去确定网络中的哪些服务器对应于 Elasticsearch 集群中的哪些节点。

一个节点可以通过配置集群名称的方式来加入一个指定的集群。默认情况下，每个节点都会被安排加入到一个叫做“elasticsearch”的集群中，这意味着，如果你在你的网络中启动了若干个节点，并假定它们能够相互发现彼此，它们将会自动地形成并加入到一个叫做“elasticsearch”的集群中。

在一个集群里，只要你想，可以拥有任意多个节点。而且，如果当前你的网络中没有运行任何 Elasticsearch 节点，这时启动一个节点，会默认创建并加入一个叫做“elasticsearch”的集群。

---



# Windows集群

## 配置文件

创建 elasticsearch-cluster 文件夹，在内部复制三个 elasticsearch 服务

- node-1001
- node-1002
- node-1003

修改集群文件目录中每个节点的 config/elasticsearch.yml 配置文件

节点 node-1001 的配置信息：

```yaml
#集群名称，节点之间要保持一致
cluster.name: my-elasticsearch

#节点名称，集群内要唯一,并设置可以为主节点，存储数据
node.name: node-1001
node.master: true
node.data: true

#ip 地址
network.host: localhost
#http 端口
http.port: 1001
#tcp 监听端口
transport.tcp.port: 9301

#节点发现，第一个启动可以不用配置，后面启动的节点要配置寻找节点
#discovery.seed_hosts: ["localhost:9301", "localhost:9302","localhost:9303"]
#discovery.zen.fd.ping_timeout: 1m
#discovery.zen.fd.ping_retries: 5

#集群内的可以被选为主节点的节点列表
#cluster.initial_master_nodes: ["node-1001", "node-1002","node-1003"]

#跨域配置
#action.destructive_requires_name: true
http.cors.enabled: true
http.cors.allow-origin: "*"
```

节点 node-1002 的配置信息：

```yaml
#集群名称，节点之间要保持一致
cluster.name: my-elasticsearch

#节点名称，集群内要唯一,并设置可以为主节点，存储数据
node.name: node-1002
node.master: true
node.data: true

#ip 地址
network.host: localhost
#http 端口
http.port: 1002
#tcp 监听端口
transport.tcp.port: 9302

#节点发现
discovery.seed_hosts: ["localhost:9301"]
discovery.zen.fd.ping_timeout: 1m
discovery.zen.fd.ping_retries: 5

#集群内的可以被选为主节点的节点列表
#cluster.initial_master_nodes: ["node-1001", "node-1002","node-1003"]

#跨域配置
#action.destructive_requires_name: true
http.cors.enabled: true
http.cors.allow-origin: "*"
```

节点 node-1003 的配置信息：

```yaml
#集群名称，节点之间要保持一致
cluster.name: my-elasticsearch

#节点名称，集群内要唯一,并设置可以为主节点，存储数据
node.name: node-1003
node.master: true
node.data: true

#ip 地址
network.host: localhost
#http 端口
http.port: 1003
#tcp 监听端口
transport.tcp.port: 9303

#节点发现
discovery.seed_hosts: ["localhost:9301", "localhost:9302"]
discovery.zen.fd.ping_timeout: 1m
discovery.zen.fd.ping_retries: 5

#集群内的可以被选为主节点的节点列表
#cluster.initial_master_nodes: ["node-1001", "node-1002","node-1003"]

#跨域配置
#action.destructive_requires_name: true
http.cors.enabled: true
http.cors.allow-origin: "*"
```

## 启动集群

1. 启动前先删除每个节点中的 data 和 logs 目录中所有内容（如果存在）
2. 分别双击执行 bin/elasticsearch.bat, 启动节点服务器，启动后，会自动加入指定名称的集群

查看集群状态：

![](https://cdn.jsdelivr.net/npm/nan-picture/blog/20211120232055.png)

> status 字段指示着当前集群在总体上是否工作正常。它的三种颜色含义如下：
>
> - green：所有的主分片和副本分片都正常运行。
> - yellow：所有的主分片都正常运行，但不是所有的副本分片都正常运行。
> - red有主分片没能正常运行。

向集群中的 node-1001 节点增加索引：

![](https://cdn.jsdelivr.net/npm/nan-picture/blog/20211120232533.png)

向集群中的 node-1002 节点查询索引：

![](https://cdn.jsdelivr.net/npm/nan-picture/blog/20211120232647.png)

---



# Linux单机

## 下载解压

软件下载地址：https://www.elastic.co/cn/downloads/past-releases/elasticsearch-7-8-0 

将下载好的安装包通过 Xftp 工具上传到 Linux上

将下载的软件解压缩并重命名：

- `tar -zxvf elasticsearch-7.8.0-linux-x86_64.tar.gz -C /opt/`
- `cd /opt/module ` 

- `mv elasticsearch-7.8.0 es`

## 创建用户

因为安全问题，Elasticsearch 不允许 root 用户直接运行，所以要创建新用户，在 root 用户中创建新用户：

1. 新增 es 用户：`useradd es`
2. 为 es 用户设置密码：`passwd es`
3. 如果新增错了，可以删除再加：`userdel -r es`
4. 改变文件夹所有者和群组：`chown -R es:es /opt/es`
5. 查看文件夹所有者和群组是否更改成功：`ls -ll`

## 修改配置文件

修改 /opt/es/config/elasticsearch.yml  文件，加入如下配置：

```yaml
cluster.name: elasticsearch
node.name: node-1
network.host: 0.0.0.0
http.port: 9200
cluster.initial_master_nodes: ["node-1"]
```

修改 /etc/security/limits.conf 文件，在文件末尾中增加下面内容：

```yaml
# 每个进程可以打开的文件数的限制
es soft nofile 65536
es hard nofile 65536
```

修改 /etc/security/limits.d/20-nproc.conf 文件，在文件末尾中增加下面内容：

```yaml
# 每个进程可以打开的文件数的限制
es soft nofile 65536
es hard nofile 65536
# 操作系统级别对每个用户创建的进程数的限制
* hard nproc 4096
# 注：* 代表 Linux 所有用户名称
```

修改 /etc/sysctl.conf 文件，在文件中增加下面内容：

```yaml
# 一个进程可以拥有的 VMA(虚拟内存区域)的数量,默认值为 65536
vm.max_map_count=655360
```

**重新加载：**`sysctl -p `

## 启动服务

```shell
[root@nanzx /]# cd /opt/es/
[root@nanzx es]# su es
[es@nanzx es]$ bin/elasticsearch
# 后台启动
[es@nanzx es]$ bin/elasticsearch -d
```



## 启动异常问题

```shell
[root@nanzx es]# bin/elasticsearch
Exception in thread "main" java.lang.RuntimeException: starting java failed with [1]
output:
#
# There is insufficient memory for the Java Runtime Environment to continue.
# Native memory allocation (mmap) failed to map 1073741824 bytes for committing reserved memory.
# An error report file with more information is saved as:
# logs/hs_err_pid95986.log
error:
OpenJDK 64-Bit Server VM warning: INFO: os::commit_memory(0x00000000c0000000, 1073741824, 0) failed; error='Not enough space' (errno=12)
	at org.elasticsearch.tools.launchers.JvmErgonomics.flagsFinal(JvmErgonomics.java:126)
	at org.elasticsearch.tools.launchers.JvmErgonomics.finalJvmOptions(JvmErgonomics.java:88)
	at org.elasticsearch.tools.launchers.JvmErgonomics.choose(JvmErgonomics.java:59)
	at org.elasticsearch.tools.launchers.JvmOptionsParser.jvmOptions(JvmOptionsParser.java:137)
	at org.elasticsearch.tools.launchers.JvmOptionsParser.main(JvmOptionsParser.java:95)
```

**解决方法：**

1. 执行命令` free -m `查看内存，最主要的是看有没有交换空间 swap，如果没有交换空间或者交换空间比较小，要先安装交换空间或者增大空间

2. 创建swapfile：`dd if=/dev/zero of=swapfile bs=1024 count=500000`

3. 将swapfile设置为swap空间（把这个文件变成swap文件）：  `mkswap swapfile`
4. 启用交换空间（启用这个swap文件）：`swapon swapfile` （删除交换空间是`swapoff swapfile`）
5. 执行命令` free -m `查看swap空间大小是否发生变化

---

```shell
[root@nanzx es]# bin/elasticsearch
[2021-11-21T13:25:57,949][ERROR][o.e.b.ElasticsearchUncaughtExceptionHandler] [node-1] uncaught exception in thread [main]
org.elasticsearch.bootstrap.StartupException: java.lang.RuntimeException: can not run elasticsearch as root
	at org.elasticsearch.bootstrap.Elasticsearch.init(Elasticsearch.java:174) ~[elasticsearch-7.8.0.jar:7.8.0]
	at org.elasticsearch.bootstrap.Elasticsearch.execute(Elasticsearch.java:161) ~[elasticsearch-7.8.0.jar:7.8.0]
```

**解决方法：**

1. 切换为elasticsearch文件夹所有者登录（参考第二步创建的用户）：`su es`

---

```shell
[es@nanzx es]$ bin/elasticsearch
could not find java in bundled jdk at /root/software/es/jdk/bin/java
[es@nanzx es]$ java -version
bash: java: 未找到命令
```

**解决方法：**

1. 查看本机是否安装了java环境`java -version`
2. 如果root用户显示jdk环境变量正常，而es用户显示不正常，则需要将jdk转移到一个非root目录
3. [Centos7安装配置java环境](https://nanzx.top/posts/59553/)

---

```shell
[es@nanzx es]$ bin/elasticsearch
错误: 找不到或无法加载主类 org.elasticsearch.tools.java_version_checker.JavaVersionChecker
```

**解决方法：**

1. 说明 es 没有此目录的权限，需要把Elasticsearch文件夹复制到非root目录里，建议软件不要直接安装在root目录下

---

```shell
[es@nanzx es]$ bin/elasticsearch
future versions of Elasticsearch will require Java 11; your Java version from [/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.312.b07-1.el7_9.x86_64/jre] does not meet this requirement
future versions of Elasticsearch will require Java 11; your Java version from [/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.312.b07-1.el7_9.x86_64/jre] does not meet this requirement
OpenJDK 64-Bit Server VM warning: If the number of processors is expected to increase from one, then you should configure the number of parallel GC threads appropriately using -XX:ParallelGCThreads=N
已杀死
```

**解决方法：**

启动过程中被自动killed是因为内存不够了，由于ES是运行在JVM上，JVM本身除了分配的heap内存以外，还会用到一些堆外(off heap)内存。 在小内存的机器上跑ES，如果heap划分过多，累加上堆外内存后，总的JVM使用内存量可能超过物理内存限制。 如果swap又是关闭的情况下，就会被操作系统oom killer杀掉。

我的虚拟机只有1G运行内存，而es中的jvm配置文件中就配置了1g的堆大小，导致没有足够空间分配，故es启动不起来。

修改配置文件：`vi config/jvm.options`，将`-Xmx1g`改为`-Xmx512m`，重新启动即可。

## 测试是否启动成功

新建一个窗口：

```shell
[root@nanzx ~]# curl -XGET 'localhost:9200'
{
  "name" : "node-1",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "93G_jmzxT8qKl8NI2Kyh0A",
  "version" : {
    "number" : "7.8.0",
    "build_flavor" : "default",
    "build_type" : "tar",
    "build_hash" : "757314695644ea9a1dc2fecd26d1a43856725e65",
    "build_date" : "2020-06-14T19:35:50.234439Z",
    "build_snapshot" : false,
    "lucene_version" : "8.5.1",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}

```



# Linux集群

## 下载解压

软件下载地址：https://www.elastic.co/cn/downloads/past-releases/elasticsearch-7-8-0 

将下载好的安装包通过 Xftp 工具上传到 三台Linux服务器上

将下载的软件解压缩并重命名：

- `tar -zxvf elasticsearch-7.8.0-linux-x86_64.tar.gz -C /opt/`
- `cd /opt/module ` 

- `mv elasticsearch-7.8.0 es-cluster`

## 创建用户

因为安全问题，Elasticsearch 不允许 root 用户直接运行，所以要创建新用户，在 root 用户中创建新用户：

1. 新增 es 用户：`useradd es`
2. 为 es 用户设置密码：`passwd es`
3. 如果新增错了，可以删除再加：`userdel -r es`
4. 改变文件夹所有者和群组：`chown -R es:es /opt/es-cluster`
5. 查看文件夹所有者和群组是否更改成功：`ls -ll`

## 修改配置文件

修改 /opt/es-cluster/config/elasticsearch.yml  文件，加入如下配置：

```yaml
#集群名称
cluster.name: cluster-es
#节点名称，每个节点的名称不能重复
node.name: node-1
#ip 地址，每个节点的地址不能重复
network.host: 192.168.2.110
#是不是有资格主节点
node.master: true
node.data: true
http.port: 9200
# head 插件需要这打开这两个配置，跨域配置
http.cors.allow-origin: "*"
http.cors.enabled: true
http.max_content_length: 200mb
#es7.x 之后新增的配置，初始化一个新的集群时需要此配置来选举 master
cluster.initial_master_nodes: ["node-1"]
#es7.x 之后新增的配置，节点发现
discovery.seed_hosts: ["192.168.2.110:9300","192.168.2.112:9300","192.168.2.113:9300"]
gateway.recover_after_nodes: 2
network.tcp.keep_alive: true
network.tcp.no_delay: true
transport.tcp.compress: true
#集群内同时启动的数据任务个数，默认是 2 个
cluster.routing.allocation.cluster_concurrent_rebalance: 16
#添加或删除节点及负载均衡时并发恢复的线程个数，默认 4 个
cluster.routing.allocation.node_concurrent_recoveries: 16
#初始化数据恢复时，并发恢复线程的个数，默认 4 个
cluster.routing.allocation.node_initial_primaries_recoveries: 16
```

修改 /etc/security/limits.conf 文件，在文件末尾中增加下面内容：

```yaml
# 每个进程可以打开的文件数的限制
es soft nofile 65536
es hard nofile 65536
```

修改 /etc/security/limits.d/20-nproc.conf 文件，在文件末尾中增加下面内容：

```yaml
# 每个进程可以打开的文件数的限制
es soft nofile 65536
es hard nofile 65536
# 操作系统级别对每个用户创建的进程数的限制
* hard nproc 4096
# 注：* 代表 Linux 所有用户名称
```

修改 /etc/sysctl.conf 文件，在文件中增加下面内容：

```yaml
# 一个进程可以拥有的 VMA(虚拟内存区域)的数量,默认值为 65536
vm.max_map_count=655360
```

**重新加载：**`sysctl -p `

## 启动服务

分别在不同节点上启动 ES 软件

```shell
[root@nanzx /]# cd /opt/es-cluster/
[root@nanzx es]# su es
[es@nanzx es]$ bin/elasticsearch
# 后台启动
[es@nanzx es]$ bin/elasticsearch -d
```

## 测试集群

```shell
[root@nanzx ~]# curl -XGET 'localhost:9200/_cat/nodes'
192.168.2.113 18 94 8 0.49 0.93 2.27 dilmrt - node-3
192.168.2.112 18 94 0 0.10 0.97 2.37 dilmrt - node-2
172.17.0.1 28 94 0 0.08 1.67 2.69 dilmrt * node-1
```

