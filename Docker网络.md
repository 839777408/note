---
title: Docker网络
categories:
  - Docker
tags:
  - Docker
top_img: 'https://npm.elemecdn.com/nan-picture/img/wp11.jpg'
cover: 'https://npm.elemecdn.com/nan-picture/img/wp11.jpg'
abbrlink: 59415
date: 2021-09-10 21:55:40
updated: 2021-09-11 14:34:35
---

> 参考视频：狂神说的Docker网络篇 https://www.bilibili.com/video/BV1og4y1q7M4?p=34

# 理解docker0

通过`ip addr`命令可以查看linux上的网卡和IP地址等信息。

```shell
[root@nanzx ~]# ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 00:0c:29:cf:b5:e0 brd ff:ff:ff:ff:ff:ff
    inet 192.168.2.110/24 brd 192.168.2.255 scope global noprefixroute ens33
       valid_lft forever preferred_lft forever
    inet6 fe80::8aa1:d94a:624c:6979/64 scope link noprefixroute 
       valid_lft forever preferred_lft forever
3: docker0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:42:e9:4d:1f:06 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
    inet6 fe80::42:e9ff:fe4d:1f06/64 scope link 
       valid_lft forever preferred_lft forever
```

其中lo是回环接口，ens33是以太网接口，inet后为对应端口的IP地址。docker0则是docker启动时自动创建的网桥。

---



**问题：docker是如何处理容器网络访问的？**

```shell
#测试 运行一个centos
[root@nanzx ~]# docker run -it -d -p 8080:8080 --name centos01 centos

[root@nanzx ~]#ip addr
#多了以下的一个端口
13: vethba9e0e5@if12: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP group default 
    link/ether 4e:4c:bc:fa:d9:d4 brd ff:ff:ff:ff:ff:ff link-netnsid 1
    inet6 fe80::4c4c:bcff:fefa:d9d4/64 scope link 
       valid_lft forever preferred_lft forever


[root@nanzx ~]# docker exec -it centos01 ip addr
# 查看容器内部网络地址 发现容器启动的时候会得到一个 eth0@if13 ip地址，是docker分配的
12: eth0@if13: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:42:ac:11:00:03 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.17.0.3/16 brd 172.17.255.255 scope global eth0
       valid_lft forever preferred_lft forever


# 思考？ linux与容器内部能不能互相ping通？ 可以
[root@nanzx ~]# ping 172.17.0.3
PING 172.17.0.3 (172.17.0.3) 56(84) bytes of data.
64 bytes from 172.17.0.3: icmp_seq=1 ttl=64 time=0.323 ms
64 bytes from 172.17.0.3: icmp_seq=2 ttl=64 time=0.040 ms
64 bytes from 172.17.0.3: icmp_seq=3 ttl=64 time=0.075 ms
```

---

## 原理

1. 我们每启动一个docker容器，docker就会给该容器分配一个IP地址；我们只要安装了docker， 就会有一个docker0的桥接模式，使用的技术是veth-pair技术！ 
2. 再启动一个容器测试，发现又多了一对网络 ：

```shell
[root@nanzx ~]# docker run -it -d -p 8081:8081 --name centos02 centos

[root@nanzx ~]# ip addr
17: veth701cf7e@if16: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP group default 
    link/ether a6:b5:be:2d:19:f9 brd ff:ff:ff:ff:ff:ff link-netnsid 2
    inet6 fe80::a4b5:beff:fe2d:19f9/64 scope link 
       valid_lft forever preferred_lft forever

[root@nanzx ~]# docker exec -it centos02 ip addr
16: eth0@if17: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:42:ac:11:00:04 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.17.0.4/16 brd 172.17.255.255 scope global eth0
       valid_lft forever preferred_lft forever
       
#测试centos01和centos02能否相互ping通：
[root@nanzx ~]# docker exec -it centos01 ping 172.17.0.4
PING 172.17.0.4 (172.17.0.4) 56(84) bytes of data.
64 bytes from 172.17.0.4: icmp_seq=1 ttl=64 time=0.210 ms
64 bytes from 172.17.0.4: icmp_seq=2 ttl=64 time=0.062 ms
64 bytes from 172.17.0.4: icmp_seq=3 ttl=64 time=0.062 ms
```

>我们发现每启动一个容器，docker就会创建对应的网卡接口：veth701cf7e@if16和eth0@if17
>
>这一对的虚拟设备接口就是veth-pair ，它们都是成对出现的，一端连着协议，一端彼此相连 
>
>正因为有这个特性 veth-pair，docker0 充当一个桥梁来连接各种虚拟网络设备

更详细可查看：[Linux-虚拟网络设备-veth pair](https://blog.csdn.net/sld880311/article/details/77650937?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522163129071916780366583346%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=163129071916780366583346&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~sobaiduend~default-1-77650937.first_rank_v2_pc_rank_v29&utm_term=veth-pair&spm=1018.2226.3001.4187)



## 小结

![](https://npm.elemecdn.com/nan-picture/blog/20210911002732.png)

**结论**：

+ 所有的容器不指定网络的情况下，都是docker0路由的，docker会给我们的容器分配一个默认的可用 IP



![](https://npm.elemecdn.com/nan-picture/blog/20210911003146.png)

**结论**：

+  Docker使用的是Linux的桥接，宿主机是一个Docker容器的网桥
+ Docker中所有网络接口都是虚拟的，虚拟的转发效率高（内网传递文件）
+ 只要容器删除，对应的网桥一对就没了



# --link

## 演示

思考一个场景：我们编写了一个微服务，数据库url的IP地址变了，但项目不重启，我们希望可以处理这个问题，可以通过名字来进行访问容器？

```shell
[root@nanzx ~]# docker exec -it centos01 ping centos02
PING centos02 (120.240.95.35) 56(84) bytes of data.
^C
--- centos02 ping statistics ---
6 packets transmitted, 0 received, 100% packet loss, time 8ms
```

发现ping不通，我们再运行一个centos03，加上 --link参数：

```shell
[root@nanzx ~]# docker run -it -d -p 8082:8082 --name centos03 --link centos02 centos
```

这时centos03就可以ping通centos02了：

```shell
[root@nanzx ~]# docker exec -it centos03 ping centos02
PING centos02 (172.17.0.4) 56(84) bytes of data.
64 bytes from centos02 (172.17.0.4): icmp_seq=1 ttl=64 time=0.944 ms
64 bytes from centos02 (172.17.0.4): icmp_seq=2 ttl=64 time=0.067 ms
^C
--- centos02 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 3ms
rtt min/avg/max/mdev = 0.067/0.505/0.944/0.439 ms
```

**注意：**centos02 ping不通 centos03，所以--link是单向的。

## 原理

通过`docker inspcet centos03`可看到 Links 增加了映射:

```shell
            "Links": [
                "/centos02:/centos03/centos02"
            ],
```

进入centos03容器，通过`cat /etc/hosts`可看到hosts域名解析的文件里增加了centos02的映射：

```shell
127.0.0.1	localhost
::1	localhost ip6-localhost ip6-loopback
fe00::0	ip6-localnet
ff00::0	ip6-mcastprefix
ff02::1	ip6-allnodes
ff02::2	ip6-allrouters
172.17.0.4	centos02 1ba08f27a55c #centos02
172.17.0.5	b7ff4f0af138
```

所以，--link 本质就是在hosts配置中添加映射。

>现在使用Docker已经不建议使用--link了，推荐使用自定义网络（不适用docker0）。
>
>docker0问题：不支持容器名连接访问， 只能通过加 --link参数。



# 自定义网络

## 网络相关命令

```shell
[root@nanzx ~]# docker network --help

Usage:  docker network COMMAND

Manage networks

Commands:
  connect     Connect a container to a network
  create      Create a network
  disconnect  Disconnect a container from a network
  inspect     Display detailed information on one or more networks
  ls          List networks
  prune       Remove all unused networks
  rm          Remove one or more networks

Run 'docker network COMMAND --help' for more information on a command.
```

查看所有的docker网络：

```shell
[root@nanzx ~]# docker network ls
NETWORK ID     NAME      DRIVER    SCOPE
cb43c4e5baa2   bridge    bridge    local
c5f5efea2089   host      host      local
b9e8be9bb948   none      null      local
```

**网络模式：**

bridge ：桥接 docker（默认，自己创建也是用bridge模式）

none ：不配置网络，一般不用

host ：和所主机共享网络

container ：容器网络连通（用得少！局限很大）



## 演示

我们直接启动的命令默认会加--net bridge，而这个就是我们得docker0，bridge就是docker0 :

`docker run -d -P --name tomcat01 tomcat` 等价于 `docker run -d -P --name tomcat01 --net bridge tomcat`

---

docker0特点：默认，域名不能访问。( --link可以打通连接，但是很麻烦)，我们可以 自定义一个网络 ：

```shell
[root@nanzx ~]# docker network create --driver bridge --subnet 192.168.0.0/16 --gateway 192.168.0.1 mynet
88045facbfb2f67ac907189c4aee47af8861f17a66ea621b12083023d9f558cf
[root@nanzx ~]# docker network ls
NETWORK ID     NAME      DRIVER    SCOPE
cb43c4e5baa2   bridge    bridge    local
c5f5efea2089   host      host      local
88045facbfb2   mynet     bridge    local
b9e8be9bb948   none      null      local

[root@nanzx ~]# docker inspect mynet
[
    {
        "Name": "mynet",
        "Id": "88045facbfb2f67ac907189c4aee47af8861f17a66ea621b12083023d9f558cf",
        "Created": "2021-09-11T12:02:55.134949838+08:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "192.168.0.0/16",
                    "Gateway": "192.168.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {},
        "Options": {},
        "Labels": {}
    }
]
```

启动两个centos，再次查看网络情况：

```shell
[root@nanzx ~]# docker run -it -d -p 8083:8083 --name centos_net_01  --net mynet centos
[root@nanzx ~]# docker run -it -d -p 8084:8084 --name centos_net_02  --net mynet centos

[root@nanzx ~]# docker inspect mynet
[
    {
        "Name": "mynet",
        "Id": "88045facbfb2f67ac907189c4aee47af8861f17a66ea621b12083023d9f558cf",
        "Created": "2021-09-11T12:02:55.134949838+08:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "192.168.0.0/16",
                    "Gateway": "192.168.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {
            "2cf1861af8ac4359aab51a5196c1c1424b0cca9ee2f1f51ed69e645f446c9e36": {
                "Name": "centos_net_02",
                "EndpointID": "90f01a29816f3b961d8642c85011774bd612f43ace83d7a340d8a002449c0ab1",
                "MacAddress": "02:42:c0:a8:00:03",
                "IPv4Address": "192.168.0.3/16",
                "IPv6Address": ""
            },
            "45628390452a41c10987df06f1a6e3df99182af15fcfc282daf49d7306acfbea": {
                "Name": "centos_net_01",
                "EndpointID": "37941c2600d49422c13ccfb2fe01f13268b7a9b2a236e7c356118b78024049a2",
                "MacAddress": "02:42:c0:a8:00:02",
                "IPv4Address": "192.168.0.2/16",
                "IPv6Address": ""
            }
        },
        "Options": {},
        "Labels": {}
    }
]

```

在自定义的网络下，服务可以互相ping通，不用使用--link：

```shell
[root@nanzx ~]# docker exec -it centos_net_01 ping centos_net_02
PING centos_net_02 (192.168.0.3) 56(84) bytes of data.
64 bytes from centos_net_02.mynet (192.168.0.3): icmp_seq=1 ttl=64 time=0.082 ms
^C
--- centos_net_02 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 0.082/0.082/0.082/0.000 ms

[root@nanzx ~]# docker exec -it centos_net_01 ping 192.168.0.3
PING 192.168.0.3 (192.168.0.3) 56(84) bytes of data.
64 bytes from 192.168.0.3: icmp_seq=1 ttl=64 time=0.251 ms
64 bytes from 192.168.0.3: icmp_seq=2 ttl=64 time=0.065 ms
^C
--- 192.168.0.3 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 2ms
rtt min/avg/max/mdev = 0.065/0.158/0.251/0.093 ms
```

好处：不同的集群使用不同的网络，保证集群是安全和健康的，推荐我们平时这样使用网络！



# 网络连通

```shell
[root@nanzx ~]# docker network --help
Usage:  docker network COMMAND
Manage networks
Commands:
  connect     Connect a container to a network
  
  
[root@nanzx ~]# docker network connect --help
Usage:  docker network connect [OPTIONS] NETWORK CONTAINER
Connect a container to a network
Options:
      --alias strings           Add network-scoped alias for the container
      --driver-opt strings      driver options for the network
      --ip string               IPv4 address (e.g., 172.30.100.104)
      --ip6 string              IPv6 address (e.g., 2001:db8::33)
      --link list               Add link to another container
      --link-local-ip strings   Add a link-local address for the container
```

![](https://npm.elemecdn.com/nan-picture/blog/20210911142856.png)

正常情况如图所示，tomcat-01是不能直接访问tomcat-net-01的，需通过`docker network connect mynet tomcat-01 `将 tomcat01加到 mynet网络，这时tomcat01会有两个IP地址。

