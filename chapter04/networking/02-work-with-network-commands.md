# 使用网络命令
work-with-network-commands

> octowhale 2016-10-20
> [ 原文 ](https://docs.docker.com/engine/userguide/networking/work-with-networks/)

本节提供了一些例子讲述 `network` 子命令的用法。

+ docker network create
+ docker network connect
+ docker network ls
+ docker network rm
+ docker network disconnect
+ docker network inspect 

While not required, it is a good idea to read [Understanding Docker network](./01-understand-docker-container-networks.md) before trying the examples in this section. The examples for the rely on a bridge network so that you can try them immediately. If you would prefer to experiment with an overlay network see the [Getting started with multi-host networks](./03-get-started-with-multi-host-networking.md) instead.


## 创建网络

在安装docker engine的时候会自动创建一个名为 ` docker0 ` 的 ` bridge ` 网络。 docker engine默认使用该桥接网络。在此基础上，你可创建其他的 ` bridge ` or ` overlay ` 网络。

` bridge ` 网络只能作用在运行docker engine的主机上。 ` overlay ` 网络跨越多个运行 overlay engine 的主机。 如果你使用 ` docker network create ` 命令，将创建一个bridge网络。

```bash
$ docker network create simple-network

69568e6336d8c96bbf57869030919f7c69524f71183b44d80948bd3927c87f6a

$ docker network inspect simple-network
[
    {
        "Name": "simple-network",
        "Id": "69568e6336d8c96bbf57869030919f7c69524f71183b44d80948bd3927c87f6a",
        "Scope": "local",
        "Driver": "bridge",
        "IPAM": {
            "Driver": "default",
            "Config": [
                {
                    "Subnet": "172.22.0.0/16",
                    "Gateway": "172.22.0.1/16"
                }
            ]
        },
        "Containers": {},
        "Options": {}
    }
]

```

与 ` bridge ` 不同， 创建 ` overlay ` 网络需要一些预置条件，为：

+ 允许 ` key-value`存储。 engine 支持使用 Consul, Etcd, and ZooKeeper(分布式存储)键值存储。
+ 一个能正确管理 key-value 的集群主机。
+ 每个主机daemon经过配置，使用swarm模式。
+ Access to a key-value store. Engine supports Consul, Etcd, and ZooKeeper (Distributed store) key-value stores.
+ A cluster of hosts with connectivity to the key-value store.
+ A properly configured Engine daemon on each host in the swarm.

` dockerd ` 命令选项支持 ` overlay ` 网络的有：
+ ` --cluster-store ` 
+ ` --cluster-store-opt ` 
+ ` --cluster-advertise ` 


建议，但不是必须，你可以安装 ` docker swarm ` 来管理集群。 swarm提供精确的发现功能和服务器管理，可以帮助你管理网络。

当你创建网络时， docker engine默认创建一个非重叠子网。你可以覆盖此默认值，并使用 ` --subnet ` 徐昂想直接指定一个子网。 在 ` bridge ` 中你只能指定一个子网。 ` overlay ` 网络中支持多个子网。

> **注意**： 强烈建议创建网络时使用 ` --subnet ` 选项。 如果没有指定 ` --subnet ` ， docker Daemon自动选择并注册一个子网，可能会覆盖其他非docker管理的基础设施的子网。 在容器连接其他网络时，覆盖可能引发问题或错误。

除了 ` --subnet ` 选项之外，你还可以指定 ` --gateway, --ip-range, and --aux-address ` 选项。

```bash
$ docker network create -d overlay \
  --subnet=192.168.0.0/16 \
  --subnet=192.170.0.0/16 \
  --gateway=192.168.0.100 \
  --gateway=192.170.0.100 \
  --ip-range=192.168.1.0/24 \
  --aux-address="my-router=192.168.1.5" --aux-address="my-switch=192.168.1.6" \
  --aux-address="my-printer=192.170.1.5" --aux-address="my-nas=192.170.1.6" \
  my-multihost-network

```

`docker network create ` 帮助文档
```bash
$ docker network create -h
Flag shorthand -h has been deprecated, please use --help

Usage:  docker network create [OPTIONS] NETWORK

Create a network

Options:
      --aux-address value    网络驱动程序使用的辅助IPv4或IPv6地址 (default map[])
  -d, --driver string        Driver to manage the Network (default "bridge")
      --gateway value        IPv4 or IPv6 Gateway for the master subnet (default [])
      --help                 Print usage
      --internal             Restrict external access to the network
      --ip-range value       Allocate container ip from a sub-range (default [])
      --ipam-driver string   IP Address Management Driver (default "default")
      --ipam-opt value       Set IPAM driver specific options (default map[])
      --ipv6                 Enable IPv6 networking
      --label value          Set metadata on a network (default [])
  -o, --opt value            Set driver specific options (default map[])
      --subnet value         Subnet in CIDR format that represents a network segment (default [])
```

确保你的子网不会覆盖已有网络，如果有，会创建失败并返回一个错误。

当创建自定义网络时，默认网络驱动有很多额外选项。 以下是一些用于docker0桥接网络的选项和等价的docker daemon标识：

|  选项  |  等价值  |  描述  |
|  ----  |  -----   |  ----  | 
| com.docker.network.bridge.name | - | 创建时使用的桥接网络名称 |
| com.docker.network.bridge.enable_ip_masquerade | ` --ip-masq ` | 启用ip伪装 |
| com.docker.network.bridge.enable_icc  |  ` --icc ` | 启用或关闭容器之前连接 |
| com.docker.network.bridge.host_binding_ipv4 | ` --ip ` | 绑定容器端口时使用的默认IP |
| com.docker.network.driver.mtu  | ` --mtu ` | 设置容器网络的MTU |

下面的参数是所有网络都可以用的

| 参数 | 等价值  |  描述 |
| ---- | -----   |  ---- |
| ` --internal `  | - | 限制外部访问 | 
| ` --ipv6 ` | ` --ipv6 ` | 启用IPv6网络 |

例如， 现在我们使用 ` -o ` 或 ` --opt ` 选项来指定发布端口时的IP地址绑定：

```bash
$ docker network create -o "com.docker.network.bridge.host_binding_ipv4"="172.23.0.1" my-network

b1a086897963e6a2e7fc6868962e55e746bee8ad0c97b54a5831054b5f62672a

$ docker network inspect my-network

[
    {
        "Name": "my-network",
        "Id": "b1a086897963e6a2e7fc6868962e55e746bee8ad0c97b54a5831054b5f62672a",
        "Scope": "local",
        "Driver": "bridge",
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "172.23.0.0/16",
                    "Gateway": "172.23.0.1/16"
                }
            ]
        },
        "Containers": {},
        "Options": {
            "com.docker.network.bridge.host_binding_ipv4": "172.23.0.1"
        }
    }
]

$ docker run -d -P --name redis --network my-network redis

bafb0c808c53104b2c90346f284bda33a69beadcab4fc83ab8f2c5a4410cd129

$ docker ps

CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                        NAMES
bafb0c808c53        redis               "/entrypoint.sh redis"   4 seconds ago       Up 3 seconds        172.23.0.1:32770->6379/tcp   redis
```


## 连接容器

你可以是容器动态的连接到一个或多个网络。这些网络可以使用相同或不同的网络驱动。一旦连接成功，容器便可以使用其他容器的IP地址或容器名访问对方。

对于 ` overlay ` 网络或自定义插件网路而言，支持多主机互连。在不同主机上启动的容器加入到相同的多主机网络可以通过这种方式交互。

举例说明，创建两个容器：

```bash
$ docker run -itd --name=container1 busybox

18c062ef45ac0c026ee48a83afa39d25635ee5f02b58de4abc8f467bcaa28731

$ docker run -itd --name=container2 busybox

498eaaaf328e1018042c04b2de04036fc04719a6e39a097a4f4866043a2c2152

```

创建一个隔离的 ` bridge ` 网络用于测试，

```bash

$ docker network create -d bridge --subnet 172.25.0.0/16 isolated_nw

06a62f1c73c4e3107c0f555b7a5f163309827bfbbf999840166065a8f35455a8

```

把 ` container2 ` 加入刚才创建的网络，并使用 inspect 命令查看网络信息：

```bash
$ docker network connect isolated_nw container2

$ docker network inspect isolated_nw

[
    {
        "Name": "isolated_nw",
        "Id": "06a62f1c73c4e3107c0f555b7a5f163309827bfbbf999840166065a8f35455a8",
        "Scope": "local",
        "Driver": "bridge",
        "IPAM": {
            "Driver": "default",
            "Config": [
                {
                    "Subnet": "172.25.0.0/16",
                    "Gateway": "172.25.0.1/16"
                }
            ]
        },
        "Containers": {
            "90e1f3ec71caf82ae776a827e0712a68a110a3f175954e5bd4222fd142ac9428": {
                "Name": "container2",
                "EndpointID": "11cedac1810e864d6b1589d92da12af66203879ab89f4ccd8c8fdaa9b1c48b1d",
                "MacAddress": "02:42:ac:19:00:02",
                "IPv4Address": "172.25.0.2/16",
                "IPv6Address": ""
            }
        },
        "Options": {}
    }
]

```

你可以看到，engine自动为 container2 分配了一个我们之前创建的网络的IP地址，同时engine也获取了一个相同网络的IP地址。 现在，使用 ` docker run ` 命令和 ` --network ` 选项启动第3个容器并加入网络。

```bash
$ docker run --network=isolated_nw --ip=172.25.3.3 -itd --name=container3 busybox

467a7863c3f0277ef8e661b38427737f28099b61fa55622d6c30fb288d88c551

```

如你所见，我们可以为容器指定一个IP地址。当创建容器并指定网络时，你可以使用 ` --ip 或 --ip6 ` 为容器制定一个IPV4或IPV6地址。 指定的IP地址是容器网络陪值得一部分，因此在重启容器时，该地址会被保留。 **该功能只能用于用户自定义网络**， 因为需要确保他们的网络配置不会因为docker daemon重启而发生改变。

现在， 查看 container3 的网络资源。

```bash
{"isolated_nw":{"IPAMConfig":{"IPv4Address":"172.25.3.3"},"NetworkID":"1196a4c5af43a21ae38ef34515b6af19236a3fc48122cf585e3f3054d509679b",
"EndpointID":"dffc7ec2915af58cc827d995e6ebdc897342be0420123277103c40ae35579103","Gateway":"172.25.0.1","IPAddress":"172.25.3.3","IPPrefixLen":16,"IPv6Gateway":"","GlobalIPv6Address":"","GlobalIPv6PrefixLen":0,"MacAddress":"02:42:ac:19:03:03"}}

```

对 container2 使用相同命令。 如果你安装了python， 你可以美化输出信息。

```bash
$ docker inspect --format=''  container2 | python -m json.tool

{
    "bridge": {
        "NetworkID":"7ea29fc1412292a2d7bba362f9253545fecdfa8ce9a6e37dd10ba8bee7129812",
        "EndpointID": "0099f9efb5a3727f6a554f176b1e96fca34cae773da68b3b6a26d046c12cb365",
        "Gateway": "172.17.0.1",
        "GlobalIPv6Address": "",
        "GlobalIPv6PrefixLen": 0,
        "IPAMConfig": null,
        "IPAddress": "172.17.0.3",
        "IPPrefixLen": 16,
        "IPv6Gateway": "",
        "MacAddress": "02:42:ac:11:00:03"
    },
    "isolated_nw": {
        "NetworkID":"1196a4c5af43a21ae38ef34515b6af19236a3fc48122cf585e3f3054d509679b",
        "EndpointID": "11cedac1810e864d6b1589d92da12af66203879ab89f4ccd8c8fdaa9b1c48b1d",
        "Gateway": "172.25.0.1",
        "GlobalIPv6Address": "",
        "GlobalIPv6PrefixLen": 0,
        "IPAMConfig": null,
        "IPAddress": "172.25.0.2",
        "IPPrefixLen": 16,
        "IPv6Gateway": "",
        "MacAddress": "02:42:ac:19:00:02"
    }
}

```

你可以发现 container2 加入了两个网络。 bridge 是在创建时候加入的， isolated_nw 是之后指定加入的。

![https://docs.docker.com/engine/userguide/networking/images/working.png](https://docs.docker.com/engine/userguide/networking/images/working.png)

本例中的 container3， 使用 ` docker run ` 命令来凝结到了 isolated_nw 网络， 因此没有加入 ` bridge ` 网络。

使用 ` attach ` 命令进入 container2，并检查网络堆栈信息。

```bash
$ docker attach container2

```

如果你查看了容器的网络堆栈信息，你应该会看到网络网络接口。 一个用于默认bridge网络， 一个用于 isolated_nw 网络。

```bash
/ # ifconfig
eth0      Link encap:Ethernet  HWaddr 02:42:AC:11:00:03  
          inet addr:172.17.0.3  Bcast:0.0.0.0  Mask:255.255.0.0
          inet6 addr: fe80::42:acff:fe11:3/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:9001  Metric:1
          RX packets:8 errors:0 dropped:0 overruns:0 frame:0
          TX packets:8 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0
          RX bytes:648 (648.0 B)  TX bytes:648 (648.0 B)

eth1      Link encap:Ethernet  HWaddr 02:42:AC:15:00:02  
          inet addr:172.25.0.2  Bcast:0.0.0.0  Mask:255.255.0.0
          inet6 addr: fe80::42:acff:fe19:2/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:8 errors:0 dropped:0 overruns:0 frame:0
          TX packets:8 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0
          RX bytes:648 (648.0 B)  TX bytes:648 (648.0 B)

lo        Link encap:Local Loopback  
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope:Host
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)

```

在自定义中 isolated_nw 网络中， docker嵌入式DNS服务器实现了容器主机名解析。 在container2中，可以ping container3的主机名。

```bash
/ # ping -w 4 container3
PING container3 (172.25.3.3): 56 data bytes
64 bytes from 172.25.3.3: seq=0 ttl=64 time=0.070 ms
64 bytes from 172.25.3.3: seq=1 ttl=64 time=0.080 ms
64 bytes from 172.25.3.3: seq=2 ttl=64 time=0.080 ms
64 bytes from 172.25.3.3: seq=3 ttl=64 time=0.097 ms

--- container3 ping statistics ---
4 packets transmitted, 4 packets received, 0% packet loss
round-trip min/avg/max = 0.070/0.081/0.097 ms

```

这种方法不适用默认的`bridge`网络。 container1和container2属于默认网络。 docker在该网络上不支持自动发现。 由于此，ping container1的会失败，除非你在 `/etc/hosts`配置对应关系。

ping container1 的地址成功：

```bash
/ # ping -w 4 172.17.0.2
PING 172.17.0.2 (172.17.0.2): 56 data bytes
64 bytes from 172.17.0.2: seq=0 ttl=64 time=0.095 ms
64 bytes from 172.17.0.2: seq=1 ttl=64 time=0.075 ms
64 bytes from 172.17.0.2: seq=2 ttl=64 time=0.072 ms
64 bytes from 172.17.0.2: seq=3 ttl=64 time=0.101 ms

--- 172.17.0.2 ping statistics ---
4 packets transmitted, 4 packets received, 0% packet loss
round-trip min/avg/max = 0.072/0.085/0.101 ms

```

使用 ` docker run --link ` 命令可以连接 container1 和 container2 ， 该命令允许连个容器之间使用主机名或IP地址相互访问。

断开 congtainer2 并保持起运行使用 ` CTRL-p CTRL-q `。

在本例中， container2 连接了两个网络，因此可以与 container1 和 container3 进行交互。 但是 container3 和 container1 不在同一个网络中，因此不能相互交互。 测试，现在连接到 container3 并尝试ping container1 的ip地址。

```
$ docker attach container3

/ # ping 172.17.0.2
PING 172.17.0.2 (172.17.0.2): 56 data bytes
^C
--- 172.17.0.2 ping statistics ---
10 packets transmitted, 0 packets received, 100% packet loss

```

不论容器是否处于运行状态都可以加入到一个网络中。但是 ` docker network inspect ` 只能显示运行中的容器的信息。

### 在自定义网络中连接容器

在上例中，在自定义网络 `isolated_nw` 中， container2 可以解析 container3 的主机名。 但是在默认bridge网络中不能自动发现。这样设计是为了保持与[传统链路(legacy line)](https://docs.docker.com/engine/userguide/networking/default_network/dockerlinks/) 的向后兼容。

` legacy link ` 为默认`bridge`网络提供了4种主要功能。

+ 主机名解析
+ 主机名别名，用于容器链接，使用 ` --link=CONTAINER-NAME:ALIAS ` 指定。
+ 安全容器链接 （使用 ` --icc=false ` 隔离 ）
+ 环境变量注入

对于非默认用户自定义网络，例如之前提到的 ` isolated_nw ` ，相较于上面4种功能，在没有额外配置的情况下， dockers网络提供了：

+ 使用DNS自动主机名解析
+ 在网络中为容器提供了自动安全隔离环境
+ 可以动态的添加或退出多个网络
+ 支持 ` --link ` 选项，为容器提供主机名互联。

继续之前的例子，在 `isolated_nw`网络中创建容器` container4 `，并使用 ` --link ` 命令提供额外主机名解析的别名，供相同网络中的其他容器链接。
(这里有点拗口，原文为：create another container container4 in isolated_nw with --link to provide additional name resolution using alias for other containers in the same network.)

```bash
$ docker run --network=isolated_nw -itd --name=container4 --link container5:c5 busybox

01b5df970834b77a9eadbaff39051f237957bd35c4c56f11193e0594cfd5117c

```

在` --link container4 ` 的帮助下，我们可以使用别名 ` c5 ` 访问container5。

请注意，在我们创建 container4的时候，指定连接的名为 container5的容器还没被创建。 这是一个介于默认bridge网络中的*legacy link*与自定义网络新*link*功能之间的不同之处。 *legacy link*本质上是静态的，且难于通过别名绑定容器，另外他不允许连接的容器重启。 自定义网络中的*link*功能本质上是动态的，并支持连接的容器重启，包括容忍容器的ip地址变更。

现在，我们另外启动一台名为 ` container5 ` 的容器， 使用 c4 连接 ` container4 ` 。

```bash
$ docker run --network=isolated_nw -itd --name=container5 --link container4:c4 busybox

72eccf2208336f31e9e33ba327734125af00d1e1d2657878e2ee8154fbb23c7a

```

和预料中的一样， ` container4 ` 可以通过容器名、别名连接 ` container5 `。 反之亦然。

```bash

$ docker attach container4

/ # ping -w 4 c5
PING c5 (172.25.0.5): 56 data bytes
64 bytes from 172.25.0.5: seq=0 ttl=64 time=0.070 ms
64 bytes from 172.25.0.5: seq=1 ttl=64 time=0.080 ms
64 bytes from 172.25.0.5: seq=2 ttl=64 time=0.080 ms
64 bytes from 172.25.0.5: seq=3 ttl=64 time=0.097 ms

--- c5 ping statistics ---
4 packets transmitted, 4 packets received, 0% packet loss
round-trip min/avg/max = 0.070/0.081/0.097 ms

/ # ping -w 4 container5
PING container5 (172.25.0.5): 56 data bytes
64 bytes from 172.25.0.5: seq=0 ttl=64 time=0.070 ms
64 bytes from 172.25.0.5: seq=1 ttl=64 time=0.080 ms
64 bytes from 172.25.0.5: seq=2 ttl=64 time=0.080 ms
64 bytes from 172.25.0.5: seq=3 ttl=64 time=0.097 ms

--- container5 ping statistics ---
4 packets transmitted, 4 packets received, 0% packet loss
round-trip min/avg/max = 0.070/0.081/0.097 ms

```

```bash
$ docker attach container5

/ # ping -w 4 c4
PING c4 (172.25.0.4): 56 data bytes
64 bytes from 172.25.0.4: seq=0 ttl=64 time=0.065 ms
64 bytes from 172.25.0.4: seq=1 ttl=64 time=0.070 ms
64 bytes from 172.25.0.4: seq=2 ttl=64 time=0.067 ms
64 bytes from 172.25.0.4: seq=3 ttl=64 time=0.082 ms

--- c4 ping statistics ---
4 packets transmitted, 4 packets received, 0% packet loss
round-trip min/avg/max = 0.065/0.070/0.082 ms

/ # ping -w 4 container4
PING container4 (172.25.0.4): 56 data bytes
64 bytes from 172.25.0.4: seq=0 ttl=64 time=0.065 ms
64 bytes from 172.25.0.4: seq=1 ttl=64 time=0.070 ms
64 bytes from 172.25.0.4: seq=2 ttl=64 time=0.067 ms
64 bytes from 172.25.0.4: seq=3 ttl=64 time=0.082 ms

--- container4 ping statistics ---
4 packets transmitted, 4 packets received, 0% packet loss
round-trip min/avg/max = 0.065/0.070/0.082 ms
```

与 legacy link 功能相似， new link 别名本地化到了容器中。 对于`--link`之外的容器而言，别名是没有意义的。

同样值得注意的是，如果一个容器属于多个网络，那么连接别名被限制在给定网络中。进容器可以在不同网络间使用别名。

继续我们的案例， 创建另外一个网络，命名为 ` local_alias `

```bash
$ docker network create -d bridge --subnet 172.26.0.0/24 local_alias
76b7dc932e037589e6553f59f76008e5b76fa069638cd39776b890607f567aaa

```

现在将container4和container5加入到刚才创建的网络 `local_alias`中。

```bash
$ docker network connect --link container5:foo local_alias container4
$ docker network connect --link container4:bar local_alias container5

```

```bash

$ docker attach container4

/ # ping -w 4 foo
PING foo (172.26.0.3): 56 data bytes
64 bytes from 172.26.0.3: seq=0 ttl=64 time=0.070 ms
64 bytes from 172.26.0.3: seq=1 ttl=64 time=0.080 ms
64 bytes from 172.26.0.3: seq=2 ttl=64 time=0.080 ms
64 bytes from 172.26.0.3: seq=3 ttl=64 time=0.097 ms

--- foo ping statistics ---
4 packets transmitted, 4 packets received, 0% packet loss
round-trip min/avg/max = 0.070/0.081/0.097 ms

/ # ping -w 4 c5
PING c5 (172.25.0.5): 56 data bytes
64 bytes from 172.25.0.5: seq=0 ttl=64 time=0.070 ms
64 bytes from 172.25.0.5: seq=1 ttl=64 time=0.080 ms
64 bytes from 172.25.0.5: seq=2 ttl=64 time=0.080 ms
64 bytes from 172.25.0.5: seq=3 ttl=64 time=0.097 ms

--- c5 ping statistics ---
4 packets transmitted, 4 packets received, 0% packet loss
round-trip min/avg/max = 0.070/0.081/0.097 ms
```

可以看到，在不同的网络中ping别名都成功了。现在将container5从isolated_nw中断开，并观察结果。

```
$ docker network disconnect isolated_nw container5

$ docker attach container4

/ # ping -w 4 c5
ping: bad address 'c5'

/ # ping -w 4 foo
PING foo (172.26.0.3): 56 data bytes
64 bytes from 172.26.0.3: seq=0 ttl=64 time=0.070 ms
64 bytes from 172.26.0.3: seq=1 ttl=64 time=0.080 ms
64 bytes from 172.26.0.3: seq=2 ttl=64 time=0.080 ms
64 bytes from 172.26.0.3: seq=3 ttl=64 time=0.097 ms

--- foo ping statistics ---
4 packets transmitted, 4 packets received, 0% packet loss
round-trip min/avg/max = 0.070/0.081/0.097 ms

```

总而言之，用户定义网络中的新链路功能提供了旧有链路的所有优点，同时避免了遗留链路的大多数众所周知的问题。

与传统链接相比，一个显着的缺失功能是注入环境变量。尽管非常有用，但环境变量注入本质上是静态的，必须在容器启动时注入。 如果不付出巨大努力，是运行中的容器注入环境变量的。因此，注入与 ` docker network ` 并不匹配，后者提供了一种动态方法来连接或断开容器的网络。


### 网络范围别名

links提供的私有主机名解析是本地化到某个容器中的，而网络范围别名(network-scoped alias)为容器提供了一种使用别名方式，从而被同网络范围中的其他容器发现。 与链接别名不同，链接别名由服务的使用者定义，网络范围别名由向网络提供服务的容器定义。

> 即，网络范围别名(network-scoped alias)是容器在网络中的别名。通过 ` --network-alias ` 指定。
> ` --link ` 指定别名存在于两个容器之间。

继续之前的案例， 在 ` isolated_nw ` 网络中创建另一个容器，并指定网络别名。

```bash
$ docker run --network=isolated_nw -itd --name=container6 --network-alias app busybox

8ebe6767c1e0361f27433090060b33200aac054a68476c3be87ef4005eb1df17
```

```bash
$ docker attach container4

/ # ping -w 4 app
PING app (172.25.0.6): 56 data bytes
64 bytes from 172.25.0.6: seq=0 ttl=64 time=0.070 ms
64 bytes from 172.25.0.6: seq=1 ttl=64 time=0.080 ms
64 bytes from 172.25.0.6: seq=2 ttl=64 time=0.080 ms
64 bytes from 172.25.0.6: seq=3 ttl=64 time=0.097 ms

--- app ping statistics ---
4 packets transmitted, 4 packets received, 0% packet loss
round-trip min/avg/max = 0.070/0.081/0.097 ms

/ # ping -w 4 container6
PING container5 (172.25.0.6): 56 data bytes
64 bytes from 172.25.0.6: seq=0 ttl=64 time=0.070 ms
64 bytes from 172.25.0.6: seq=1 ttl=64 time=0.080 ms
64 bytes from 172.25.0.6: seq=2 ttl=64 time=0.080 ms
64 bytes from 172.25.0.6: seq=3 ttl=64 time=0.097 ms

--- container6 ping statistics ---
4 packets transmitted, 4 packets received, 0% packet loss
round-trip min/avg/max = 0.070/0.081/0.097 ms
```

现在将container6连接到 ` local_alias ` 中，对为容器设置一个不同的别名。

```bash
$ docker network connect --alias scoped-app local_alias container6
```

container6 在 isolated_nw 中的别名为 `app`，在 local_alias 中的别名为 `scopd-app`。

分别从container4(在两个网络中)和container5(只在isolated_nw中)访问container6的别名`scoped-app`。

```bash
$ docker attach container4

/ # ping -w 4 scoped-app
PING foo (172.26.0.5): 56 data bytes
64 bytes from 172.26.0.5: seq=0 ttl=64 time=0.070 ms
64 bytes from 172.26.0.5: seq=1 ttl=64 time=0.080 ms
64 bytes from 172.26.0.5: seq=2 ttl=64 time=0.080 ms
64 bytes from 172.26.0.5: seq=3 ttl=64 time=0.097 ms

--- foo ping statistics ---
4 packets transmitted, 4 packets received, 0% packet loss
round-trip min/avg/max = 0.070/0.081/0.097 ms

$ docker attach container5

/ # ping -w 4 scoped-app
ping: bad address 'scoped-app'

```

如你所见， **网络范围别名的影响范围是别名定义时所在的网络，因此只有那些连接到该网络中的容器才能够访问该别名**。

除了上诉特点外，多个容器可以在同一个网络中共享相同的网络范围别名。 在 isolated_nw 中创建一个 container7 ，并使用与container6相同的别名。

```bash
$ docker run --network=isolated_nw -itd --name=container7 --network-alias app busybox

3138c678c123b8799f4c7cc6a0cecc595acbdfa8bf81f621834103cd4f504554

```

当多个容器共享同一个别名时，别名的主机名解析会指向其中一个容器（通常来说是第一个使用该别名的容器）。当该别名容器关机或退出网络时，下一个使用别名的容器将会被解析。

从container4中ping别名` app `，接着关闭container6，并验证 别名` app ` 是否解析到了 congtainer7。

```bash
$ docker attach container4

/ # ping -w 4 app
PING app (172.25.0.6): 56 data bytes
64 bytes from 172.25.0.6: seq=0 ttl=64 time=0.070 ms
64 bytes from 172.25.0.6: seq=1 ttl=64 time=0.080 ms
64 bytes from 172.25.0.6: seq=2 ttl=64 time=0.080 ms
64 bytes from 172.25.0.6: seq=3 ttl=64 time=0.097 ms

--- app ping statistics ---
4 packets transmitted, 4 packets received, 0% packet loss
round-trip min/avg/max = 0.070/0.081/0.097 ms

$ docker stop container6

$ docker attach container4

/ # ping -w 4 app
PING app (172.25.0.7): 56 data bytes
64 bytes from 172.25.0.7: seq=0 ttl=64 time=0.095 ms
64 bytes from 172.25.0.7: seq=1 ttl=64 time=0.075 ms
64 bytes from 172.25.0.7: seq=2 ttl=64 time=0.072 ms
64 bytes from 172.25.0.7: seq=3 ttl=64 time=0.101 ms

--- app ping statistics ---
4 packets transmitted, 4 packets received, 0% packet loss
round-trip min/avg/max = 0.072/0.085/0.101 ms

```

## 断开容器网络

使用 ` docker network disconnect ` 命令可以指定容器退出某个网络。

```bash

$ docker network disconnect isolated_nw container2

$ docker inspect --format=''  container2 | python -m json.tool

{
    "bridge": {
        "NetworkID":"7ea29fc1412292a2d7bba362f9253545fecdfa8ce9a6e37dd10ba8bee7129812",
        "EndpointID": "9e4575f7f61c0f9d69317b7a4b92eefc133347836dd83ef65deffa16b9985dc0",
        "Gateway": "172.17.0.1",
        "GlobalIPv6Address": "",
        "GlobalIPv6PrefixLen": 0,
        "IPAddress": "172.17.0.3",
        "IPPrefixLen": 16,
        "IPv6Gateway": "",
        "MacAddress": "02:42:ac:11:00:03"
    }
}


$ docker network inspect isolated_nw

[
    {
        "Name": "isolated_nw",
        "Id": "06a62f1c73c4e3107c0f555b7a5f163309827bfbbf999840166065a8f35455a8",
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
        "Containers": {
            "467a7863c3f0277ef8e661b38427737f28099b61fa55622d6c30fb288d88c551": {
                "Name": "container3",
                "EndpointID": "dffc7ec2915af58cc827d995e6ebdc897342be0420123277103c40ae35579103",
                "MacAddress": "02:42:ac:19:03:03",
                "IPv4Address": "172.25.3.3/16",
                "IPv6Address": ""
            }
        },
        "Options": {}
    }
]

```

一旦容器从某个网络中退出， 该容器便再也不能与所退出网络中的其他容器通信。本例中， isolated_nw网络中的container2 便不再能与container3通信。

```bash

$ docker attach container2

/ # ifconfig
eth0      Link encap:Ethernet  HWaddr 02:42:AC:11:00:03  
          inet addr:172.17.0.3  Bcast:0.0.0.0  Mask:255.255.0.0
          inet6 addr: fe80::42:acff:fe11:3/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:9001  Metric:1
          RX packets:8 errors:0 dropped:0 overruns:0 frame:0
          TX packets:8 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0
          RX bytes:648 (648.0 B)  TX bytes:648 (648.0 B)

lo        Link encap:Local Loopback  
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope:Host
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)

/ # ping container3
PING container3 (172.25.3.3): 56 data bytes
^C
--- container3 ping statistics ---
2 packets transmitted, 0 packets received, 100% packet loss

```

对于，container2依旧具有`bridge`网络的完全链接。

```bash
/ # ping container1
PING container1 (172.17.0.2): 56 data bytes
64 bytes from 172.17.0.2: seq=0 ttl=64 time=0.119 ms
64 bytes from 172.17.0.2: seq=1 ttl=64 time=0.174 ms
^C
--- container1 ping statistics ---
2 packets transmitted, 2 packets received, 0% packet loss
round-trip min/avg/max = 0.119/0.146/0.174 ms
/ #

```

在实际生产环境中，多主机网络中可能会遇到docker daemon意外重启，这将导致daemon不能清楚过期的连接端点(endpoints)。例如，当新建容器的容器名与过期端点的名相同是，可能触发错误 ` container already connected to network ` 。 为了清理这些过期端点，首先需要删除容器，然后使用命令 ` docker network disconnect -f ` 强制从网络中断开该端点。 一旦端点被清楚，容器可以再次连接到该网络中。

```bash
$ docker run -d --name redis_db --network multihost redis

ERROR: Cannot start container bc0b19c089978f7845633027aa3435624ca3d12dd4f4f764b61eac4c0610f32e: container already connected to network multihost

$ docker rm -f redis_db

$ docker network disconnect -f multihost redis_db

$ docker run -d --name redis_db --network multihost redis

7d986da974aeea5e9f7aca7e510bdb216d58682faa83a9040c2f2adc0544795a

```


## 删除网络

当网络中的所有容器都关闭并退出网络后，你可以删除网络。

```bash
$ docker network disconnect isolated_nw container3 
```

```bash
$ docker network inspect isolated_nw

[
    {
        "Name": "isolated_nw",
        "Id": "06a62f1c73c4e3107c0f555b7a5f163309827bfbbf999840166065a8f35455a8",
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

$ docker network rm isolated_nw

```

列出所有网络确认 ` isolated_nw ` 被删除了。

```bash
$ docker network ls

NETWORK ID          NAME                DRIVER
72314fa53006        host                host                
f7ab26d71dbd        bridge              bridge              
0f32e83e61ac        none                null  

```