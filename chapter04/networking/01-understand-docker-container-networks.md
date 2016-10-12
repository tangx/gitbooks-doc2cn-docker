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

> **注意**： 你可以使用 ` ctrl-p ctrl-q ` 命令断开访问容器。


` host ` 网络添加容器到主机网络堆栈。 你可以发现容器中的网络配置信息与主机的网络配置信息相同。

除了 ` bridge ` 网络外，其他默认网络其实用的并不多。 你可以查看这些默认网络，但是不能删除， 这些默认网络是docker组件的一部分。 然而，你可以添加自定义网络并在不需要使用的时候删除他们。  在学习如何创建自定义网络之前， 有必要多了解一点 ` bridge ` 网络。

### 默认bridge网络细节

默认的 ` bridge ` 网络应用在所有的docker主机上。 使用 ` docker network inspect ` 命令查看网络信息：

```bash

$ docker network inspect bridge

[
   {
       "Name": "bridge",
       "Id": "f7ab26d71dbd6f557852c7156ae0574bbf62c42f539b50c8ebde0f728a253b6f",
       "Scope": "local",
       "Driver": "bridge",
       "IPAM": {
           "Driver": "default",
           "Config": [
               {
                   "Subnet": "172.17.0.1/16",
                   "Gateway": "172.17.0.1"
               }
           ]
       },
       "Containers": {},
       "Options": {
           "com.docker.network.bridge.default_bridge": "true",
           "com.docker.network.bridge.enable_icc": "true",
           "com.docker.network.bridge.enable_ip_masquerade": "true",
           "com.docker.network.bridge.host_binding_ipv4": "0.0.0.0",
           "com.docker.network.bridge.name": "docker0",
           "com.docker.network.driver.mtu": "9001"
       }
   }
]
```

docker自动创建了一个 ` 子网 和 网关 `。 ` docker run ` 命令自动添加新建容器到该网络中。

```bash

$ docker run -itd --name=container1 busybox

3386a527aa08b37ea9232cbcace2d2458d49f44bb05a6b775fba7ddd40d8f92c

$ docker run -itd --name=container2 busybox

94447ca479852d29aeddca75c28f7104df3c3196d7b6d83061879e339946805c


```

新启动的两个容器加入了网路，再次查看 ` bridge ` 网络， 可以看到在 “ Containers ” 部分可以看到容器ID：

```bash

$ docker network inspect bridge

{[
    {
        "Name": "bridge",
        "Id": "f7ab26d71dbd6f557852c7156ae0574bbf62c42f539b50c8ebde0f728a253b6f",
        "Scope": "local",
        "Driver": "bridge",
        "IPAM": {
            "Driver": "default",
            "Config": [
                {
                    "Subnet": "172.17.0.1/16",
                    "Gateway": "172.17.0.1"
                }
            ]
        },
        "Containers": {
            "3386a527aa08b37ea9232cbcace2d2458d49f44bb05a6b775fba7ddd40d8f92c": {
                "EndpointID": "647c12443e91faf0fd508b6edfe59c30b642abb60dfab890b4bdccee38750bc1",
                "MacAddress": "02:42:ac:11:00:02",
                "IPv4Address": "172.17.0.2/16",
                "IPv6Address": ""
            },
            "94447ca479852d29aeddca75c28f7104df3c3196d7b6d83061879e339946805c": {
                "EndpointID": "b047d090f446ac49747d3c37d63e4307be745876db7f0ceef7b311cbba615f48",
                "MacAddress": "02:42:ac:11:00:03",
                "IPv4Address": "172.17.0.3/16",
                "IPv6Address": ""
            }
        },
        "Options": {
            "com.docker.network.bridge.default_bridge": "true",
            "com.docker.network.bridge.enable_icc": "true",
            "com.docker.network.bridge.enable_ip_masquerade": "true",
            "com.docker.network.bridge.host_binding_ipv4": "0.0.0.0",
            "com.docker.network.bridge.name": "docker0",
            "com.docker.network.driver.mtu": "9001"
        }
    }
]

```

使用 ` docker network inspect ` 命令查看指定网络(bridge)，结果包含了所有加入到这个网络中的容器。 网络中的容器可以通过IP地址相互通信。 在默认的bridge网络中，docker不支持主机名自动探索； 如果你想通过容器名称相互访问，那么必须使用 ` docker run --link ` 选项启动容器。

使用 ` attach ` 进入容器并查看容器网络配置信息：

```bash
$ docker attach container1

root@0cb243cd1293:/# ifconfig
eth0      Link encap:Ethernet  HWaddr 02:42:AC:11:00:02
          inet addr:172.17.0.2  Bcast:0.0.0.0  Mask:255.255.0.0
          inet6 addr: fe80::42:acff:fe11:2/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:9001  Metric:1
          RX packets:16 errors:0 dropped:0 overruns:0 frame:0
          TX packets:8 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0
          RX bytes:1296 (1.2 KiB)  TX bytes:648 (648.0 B)

lo        Link encap:Local Loopback
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope:Host
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)

```

在 `bridge` 网络中使用 ` ping ` 命令检查容器联接状态。

```bash

root@0cb243cd1293:/# ping -w3 172.17.0.3

PING 172.17.0.3 (172.17.0.3): 56 data bytes
64 bytes from 172.17.0.3: seq=0 ttl=64 time=0.096 ms
64 bytes from 172.17.0.3: seq=1 ttl=64 time=0.080 ms
64 bytes from 172.17.0.3: seq=2 ttl=64 time=0.074 ms

--- 172.17.0.3 ping statistics ---
3 packets transmitted, 3 packets received, 0% packet loss
round-trip min/avg/max = 0.074/0.083/0.096 ms
```

使用 ` cat ` 命令查看 ` container1 ` 网络状态。

```bash
root@0cb243cd1293:/# cat /etc/hosts

172.17.0.2	3386a527aa08
127.0.0.1	localhost
::1	localhost ip6-localhost ip6-loopback
fe00::0	ip6-localnet
ff00::0	ip6-mcastprefix
ff02::1	ip6-allnodes
ff02::2	ip6-allrouters

```

使用 ` CTRL-p CTRL-q ` 断开  ` container1 ` ， 并进入 ` container2 ` 。

```bash

$ docker attach container2

root@0cb243cd1293:/# ifconfig
eth0      Link encap:Ethernet  HWaddr 02:42:AC:11:00:03
          inet addr:172.17.0.3  Bcast:0.0.0.0  Mask:255.255.0.0
          inet6 addr: fe80::42:acff:fe11:3/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:9001  Metric:1
          RX packets:15 errors:0 dropped:0 overruns:0 frame:0
          TX packets:13 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0
          RX bytes:1166 (1.1 KiB)  TX bytes:1026 (1.0 KiB)

lo        Link encap:Local Loopback
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope:Host
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)

root@0cb243cd1293:/# ping -w3 172.17.0.2

PING 172.17.0.2 (172.17.0.2): 56 data bytes
64 bytes from 172.17.0.2: seq=0 ttl=64 time=0.067 ms
64 bytes from 172.17.0.2: seq=1 ttl=64 time=0.075 ms
64 bytes from 172.17.0.2: seq=2 ttl=64 time=0.072 ms

--- 172.17.0.2 ping statistics ---
3 packets transmitted, 3 packets received, 0% packet loss
round-trip min/avg/max = 0.067/0.071/0.075 ms
/ # cat /etc/hosts
172.17.0.3	94447ca47985
127.0.0.1	localhost
::1	localhost ip6-localhost ip6-loopback
fe00::0	ip6-localnet
ff00::0	ip6-mcastprefix
ff02::1	ip6-allnodes
ff02::2	ip6-allrouters

```

默认 `docker0` 桥接网络支持端口映射；且 `docker run --link ` 允许容器之间在 ` docker0 ` 网络中相互通信。如果所有容器的网络都处于docker0，不仅麻烦而且容易引发错误。 使用自定义网络代替 docker0 网络可以更好的避免这些问题。


## 用户自定义网络

使用自定义网络可以更好的隔离容器。 docker提供了一些默认的**网络驱动**用于创建这些网络。你可以创建**桥接网络 bridege**、**覆盖网络 overlay**或者**MACVLAN网络**。You can also create a network plugin or remote network written to your own specifications.


你可以创建多个自定义网络。一个容器也可以加入多个不同的网络。容器可以在网络中通信，不能跨网络。如果一个容器加入了两个网络，那么可以与两个网络中的容器通信。如果一个网络链接到多个网络，那么容器外部访问将由第一个非内部网络提供。


### 桥接网络

桥接网络是最简单的自定义网络。 这种网络和之前提到的 ` docker0 ` 网络类似。 一些额外功能和老旧功能不再可用。

```bash

$ docker network create --driver bridge isolated_nw
1196a4c5af43a21ae38ef34515b6af19236a3fc48122cf585e3f3054d509679b

$ docker network inspect isolated_nw

[
    {
        "Name": "isolated_nw",
        "Id": "1196a4c5af43a21ae38ef34515b6af19236a3fc48122cf585e3f3054d509679b",
        "Scope": "local",
        "Driver": "bridge",
        "IPAM": {
            "Driver": "default",
            "Config": [
                {
                    "Subnet": "172.21.0.0/16",
                    "Gateway": "172.21.0.1/16"
                }
            ]
        },
        "Containers": {},
        "Options": {}
    }
]

$ docker network ls

NETWORK ID          NAME                DRIVER
9f904ee27bf5        none                null
cf03ee007fb4        host                host
7fca4eb8c647        bridge              bridge
c5ee82f76de3        isolated_nw         bridge

```

创建自定义网络后， 启动容器时使用命令 ` docker run --network=<NETWORK> ` 选项指定容器使用的网络：

```bash
$ docker run --network=isolated_nw -itd --name=container3 busybox

8c1a0a5be480921d669a073393ade66a3fc49933f08bcc5515b37b8144f6d47c

$ docker network inspect isolated_nw
[
    {
        "Name": "isolated_nw",
        "Id": "1196a4c5af43a21ae38ef34515b6af19236a3fc48122cf585e3f3054d509679b",
        "Scope": "local",
        "Driver": "bridge",
        "IPAM": {
            "Driver": "default",
            "Config": [
                {}
            ]
        },
        "Containers": {
            "8c1a0a5be480921d669a073393ade66a3fc49933f08bcc5515b37b8144f6d47c": {
                "EndpointID": "93b2db4a9b9a997beb912d28bcfc117f7b0eb924ff91d48cfa251d473e6a9b08",
                "MacAddress": "02:42:ac:15:00:02",
                "IPv4Address": "172.21.0.2/16",
                "IPv6Address": ""
            }
        },
        "Options": {}
    }
]

```

所有加入这个网络的容器都必须属于同一个docker主机。 一旦加入网络，容器可以立刻与其他容器通信。 尽管， 该网络将容器与外部网络隔离。

![https://docs.docker.com/engine/userguide/networking/images/bridge_network.png](https://docs.docker.com/engine/userguide/networking/images/bridge_network.png)

在自定义桥接网络中， 不支持 ` link `功能，不过，你可以暴露和公开网络中容器的端口。 如果你想让该桥接网络的部分与外部网络互通，这会很有用。

![https://docs.docker.com/engine/userguide/networking/images/network_access.png](https://docs.docker.com/engine/userguide/networking/images/network_access.png)

桥接网络在本地创建一个小网络环境非常有用。如果要创建个大型的网络，可以使用 `覆盖overlay网络`。


### An overlay network with Docker Engine swarm mode

You can create an overlay network on a manager node running in swarm mode without an external key-value store. The swarm makes the overlay network available only to nodes in the swarm that require it for a service. When you create a service that uses the overlay network, the manager node automatically extends the overlay network to nodes that run service tasks.
你可以在一台没有存储外部`key-value`的`swarm mode`的管理节点上创建一个覆盖网络。 overlay网络只对在swarm中的节点有效，因此需要绑定网络服务。 当你创建一个使用oeverlay网络的服务时，管理节点自动扩展覆盖所有节点。

To learn more about running Docker Engine in swarm mode, refer to the [Swarm mode overview](https://docs.docker.com/engine/swarm/).


本例展示了如何在swarm中使用管理节点创建一个overlay网络，并绑定网络服务。

```bash
# Create an overlay network `my-multi-host-network`.
$ docker network create \
  --driver overlay \
  --subnet 10.0.9.0/24 \
  my-multi-host-network

400g6bwzd68jizzdx5pgyoe95

# Create an nginx service and extend the my-multi-host-network to nodes where
# the service's tasks run.
# 创建一个nginx服务，并扩展 my-multi-host-network 覆盖nginx服务任务。
$ docker service create --replicas 2 --network my-multi-host-network --name my-web nginx

716thylsndqma81j6kkkb5aus
```

针对swarm的overlay网络，不能同多 ` docker run ` 为容器指定。 更新信息，参考[Docker swarm mode overlay network security model](https://docs.docker.com/engine/userguide/networking/overlay-security-model/)。

See also [Attach services to an overlay network](https://docs.docker.com/engine/swarm/networking/).


### 使用外部 key-value 的覆盖网络
An overlay network with an external key-value store

如果docker engine不使用 swarm 模式， 需要为 ` overlay ` 网络提供有效的 ` key-value ` 保存应用服务信息。 假设 key-value保存了Consul, Etcd, and ZooKeeper (Distributed store)。 在使用当前版本engine创建网络之前，必须提前安装并配置你选择的` key-value ` 保存服务(store service ) 。 你准备使用的docker主机与该服务之间必须能正常通信。

> **注意**： docker engine 的swarm模式与使用外部key-value保存的网络不兼容。
> Note: Docker Engine running in swarm mode is not compatible with networking with an external key-value store.

![https://docs.docker.com/engine/userguide/networking/images/key_value.png](https://docs.docker.com/engine/userguide/networking/images/key_value.png)

网络中的每个主机都必须运行一个 docker engine实例。 配置主机最简单的方法就是安装docker服务。

![https://docs.docker.com/engine/userguide/networking/images/engine_on_net.png](https://docs.docker.com/engine/userguide/networking/images/engine_on_net.png)

| 协议  | 端口号  |  描述  |
| ----- | ------- |  ----- |
| udp   |  4789   | 数据平台( Data plan (VXLAN)) |
| tcp/udp |  7946 | 控制平台( Control plane ) |


` key-value `保存服务可能需要额外的端口， 查看文档打开对应端口。

一旦你有几台机器准备完毕，你可以使用 Docker Swarm 的发现功能(discovery service)快速加入他们到swarm中。

为了创建一个overlay网络，你需要在使用 ` overlay ` 网络的所有 docker engine的 ` daemon ` 配置以下三个选项：

|  选项   | 描述  |
| ------  | ----- |
| ` --cluster-store=PROVIDER://URL `              | 描述 KV 服务的位置               |
| ` --cluster-advertise=HOST_IP|HOST_IFACE:PORT ` | 指定使用集群服务的IP地址或者接口 |
| ` --cluster-store-opt=KEY-VALUE OPTIONS  `      | 选项， 例如TLS证书或 tuning discovery Timers |

在swarm中创建一个 ` overlay ` 网络：

```bash 

$ docker network create --driver overlay my-multi-host-network

```

结果是创建了一个跨越多个主机的网络。 ` overlay ` 网络为容器提供了完整的网络隔离。

![https://docs.docker.com/engine/userguide/networking/images/overlay_network.png](https://docs.docker.com/engine/userguide/networking/images/overlay_network.png)

加入这个网络的容器，在启动时都需要指定网络名称

```bash
$ docker run -itd --network=my-multi-host-network busybox
```

一旦加入了这个网络， 每个容器都能访问网络中的所有容器，无论这些容器运行在哪里的docker主机。

![https://docs.docker.com/engine/userguide/networking/images/overlay-network-final.png](https://docs.docker.com/engine/userguide/networking/images/overlay-network-final.png)

If you would like to try this for yourself, see the [Getting started for overlay](https://docs.docker.com/engine/userguide/networking/get-started-overlay/).


### 定制网络插件

如果你愿意， 你可以自己写一个网络驱动插件。 网络驱动插件使用docker的插件基础设施。 该设施下，插件作为一个后台进程运行在当前docker主机上。

一旦你创建并暗疮了自定义网络驱动，你便可以想内建网络驱动一样使用，例如：

```bash 
$ docker network create --driver weave mynet
```

你可以查看该网络信息，添加容器到该网络等。 当然，不容的插件可能使用不懂的技术和框架。 自定义网络能包含docker默认不提供的网络功能。 关于写插件的更多信息，see [Extending Docker](https://docs.docker.com/engine/extend/legacy_plugins/) and [Writing a network driver plugin](https://docs.docker.com/engine/extend/plugins_network/).


### docker嵌入式DNS服务器

docker daemon启动了一个嵌入式DNS服务器提供自动探索服务，当容器加入到用户定义的网络。提供容器第一次握手嵌入式dns服务器时的域名解析。 如果嵌入式DNS服务器不能解析则会被转发到容器中配置的外部DNS服务器。 为了促进该功能， 当容器被创建时， 容器的 ` resolv.conf ` 中只有可达到的嵌入式DNS服务器(`127.0.0.11`)信息。 More information on embedded DNS server on user-defined networks can be found in the [embedded DNS server in user-defined networks] (configure-dns.md)


## links

在使用容器网络功能之前，你可以使用docker的link功能使容器相互通信。在docker网络介绍中，容器可以通过容器名称被发现。区别于自定义网络，在 ` docker0 ` 抢劫网络中，你可以使用link连接提供不同功能的容器。 For more information, please refer to [Legacy Links](https://docs.docker.com/engine/userguide/networking/default_network/dockerlinks/) for link feature in default bridge network and the [linking containers in user-defined networks](https://docs.docker.com/engine/userguide/networking/work-with-networks/#linking-containers-in-user-defined-networks) for links functionality in user-defined networks.


----

## 总结

+ bridge 网络用于单台docker主机上
+ overlay 网络用于单台或docker主机上构成一个大型局域网
  + swarm mode：可以说是overlay的主机自动发现功能。 与 ` key-value` store service 互斥。
  + ` key-value ` store service ： overlay的手动配置模式。
  + 组成overlay网络的所有主机需要安装docker服务



