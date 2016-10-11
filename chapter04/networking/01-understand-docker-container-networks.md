# 理解容器网络

> 2016-10-10 octowhale@github

[ 原文 ](https://docs.docker.com/engine/userguide/networking/)


本节主要介绍docker的默认网络行为。描述了docker默认网络类型及如何创建自定义网络。也描述了创建一个单一网络或覆盖集群主机时所需要的资源。


## 默认网络

安装docker时会自动创建三个网络。使用 ` docker network ls ` 命令查看：

```bash

$ docker network ls

NETWORK ID          NAME                DRIVER
7fca4eb8c647        bridge              bridge
9f904ee27bf5        none                null
cf03ee007fb4        host                host

```

使用 ` --network ` 标识为容器选择网络。

容器启动时默认使用 ` bridge ` 网络。使用 ` docker run --network=<NETWORK> ` 选项可以更改容器的默认网络。 在docker主机上使用 ` ifconfig ` 命令可以看到 ` bridge ` 为主机网络堆栈的一部分。

```bash 
$ ifconfig

docker0   Link encap:Ethernet  HWaddr 02:42:47:bc:3a:eb
          inet addr:172.17.0.1  Bcast:0.0.0.0  Mask:255.255.0.0
          inet6 addr: fe80::42:47ff:febc:3aeb/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:9001  Metric:1
          RX packets:17 errors:0 dropped:0 overruns:0 frame:0
          TX packets:8 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0
          RX bytes:1100 (1.1 KB)  TX bytes:648 (648.0 B)

```

` none ` 网络是**容器特殊网络堆栈( ` container-specific network stack ` )**。 这些容器缺少网络接口。 进入容器中查看堆栈信息：

```bash
$ docker attach nonenetcontainer

root@0cb243cd1293:/# cat /etc/hosts
127.0.0.1	localhost
::1	localhost ip6-localhost ip6-loopback
fe00::0	ip6-localnet
ff00::0	ip6-mcastprefix
ff02::1	ip6-allnodes
ff02::2	ip6-allrouters
root@0cb243cd1293:/# ifconfig
lo        Link encap:Local Loopback
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope:Host
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)

root@0cb243cd1293:/#

```

> **注意**： 你可以使用 ` ctrl-p ctrl-q ` 命令断网容器。




