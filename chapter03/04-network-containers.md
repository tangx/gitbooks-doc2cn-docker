# 管理容器的网络连接

> 2016-09-29  octowhale@github

[ 原文 ](https://docs.docker.com/engine/tutorials/networkingcontainers/)

> 原文中出现很多network，翻译的时候有很多错误和误差。以后修改

如果之前你按照我们的引导进行学习，那么你已经学习了如何运行一个简单的程序和如何创建一个镜像。本节中，你将学习如何配置容器的网络。


## 设置容器名称

如果没有特别指明，容器在创建的时候会被自动分配一个随机名称。例如教程之前出现的 ` nostalgic_morse `。 当然，你也可以自己为容器指定一个名称。 手动命名有两个好处：

+ 让容器更容易被识别和记忆。例如运行web应用的容器名称为 ` web `
+ 容器之间可通过容器名相互连接。docker中有几个命令支持使用容器名访问，稍后你将学习其中一个。

使用 ` --flag ` 标记命名容器。例如，启动一个新容器，并命名为 ` web `：

```

$ docker run -d -P --name web training/webapp python app.py

```


使用 ` docker ps ` 命令查看容器名称：

```

$ docker ps -l

CONTAINER ID  IMAGE                  COMMAND        CREATED       STATUS       PORTS                    NAMES
aed84ee21bde  training/webapp:latest python app.py  12 hours ago  Up 2 seconds 0.0.0.0:49154->5000/tcp  web

```


也可以使用 ` docker inspect ` 命令：

```

$ docker inspect web

[
   {
       "Id": "3ce51710b34f5d6da95e0a340d32aa2e6cf64857fb8cdb2a6c38f7c56f448143",
       "Created": "2015-10-25T22:44:17.854367116Z",
       "Path": "python",
       "Args": [
           "app.py"
       ],
       "State": {
           "Status": "running",
           "Running": true,
           "Paused": false,
           "Restarting": false,
           "OOMKilled": false,
  ...

```


**容器名称具有唯一性**，这意味着只有一个容器能使用了`web`。如果需要其他容器也使用这个名称，必须提前删除旧容器。

移除并删除 ` web ` 容器：

```bash

$ docker stop web

web

$ docker rm web

web

```


## 使用默认网络配置启动容器
通过使用**网络驱动**，docker包含了对容器网络的支持。默认情况下，docker为容器提供了两种网络模式： ` 桥接(bridge)网络 和 覆盖(overlay)网络 `。 你也可以使用自定义网络驱动插件创建自定义的网络支持，不过这属于高级行为。

> 维基百科中关于 [overlay network](https://en.wikipedia.org/wiki/Overlay_network) 的页面
> An overlay network is a computer network that is built on top of another network. Nodes in the overlay network can be thought of as being connected by virtual or logical links, each of which corresponds to a path, perhaps through many physical links, in the underlying network. 

Docker Engine包含三种网络模式，使用 ` docker network ` 命令查看：

```

$ docker network ls

NETWORK ID          NAME                DRIVER
18a2866682b8        none                null                
c288470c46f6        host                host                
7b369448dccb        bridge              bridge  


```


` bridege ` 是一种特殊的网络模式。 除非你指定使用其他的，否则docker启动容器时默认使用bridege。例如：

```

$ docker run -itd --name=networktest ubuntu

74695c9cea6d9810718fddadc01a727a5dd3ce6a69d09752239736c030599741

```


查看网络信息是一种查找容器IP地址的简单方式：

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


你可以通过 ` docker network disconnect ` 命令关闭一个容器的网络链接。 使用命令是，需要提供**网络名称**和**容器名称**，也可以使用容器ID。

```

$ docker network disconnect bridge networktest

```


当你关闭容器的网络连接之后，将不能再删除名为`bridge`的内建的` bridge `网络。 网络是用于隔离容器和容器、容器和其他网络的自然方式。当你拥有更多经验之后，你可能会创建自己的网络。


## 自定义bridge网络

Docker Engine 默认支持 `bridege 和 overlay` 网络。 bridge网络被限制用在运行docker engine的主机上； 而 overlay网络可以包含多台主机。

例如，创建一个bridge网络：

```

$ docker network create -d bridge my-bridge-network

```


` -d ` 标识告诉docker使用 ` bridge ` 驱动创建一个名为 ` my-bridge-network ` 的网络。 你可以省略 ` -d ` 标识， 因为 ` bridge ` 是默认值。 查看所有网络链接：

```

$ docker network ls

NETWORK ID          NAME                DRIVER
7b369448dccb        bridge              bridge              
615d565d498c        my-bridge-network   bridge              
18a2866682b8        none                null                
c288470c46f6        host                host

```


如果你查看 ` my-bridge-network ` 的信息，你会发现里面什么都么没有：

```bash

$ docker network inspect my-bridge-network

[
    {
        "Name": "my-bridge-network",
        "Id": "5a8afc6364bccb199540e133e63adb76a557906dd9ff82b94183fc48c40857ac",
        "Scope": "local",
        "Driver": "bridge",
        "IPAM": {
            "Driver": "default",
            "Config": [
                {
                    "Subnet": "172.18.0.0/16",
                    "Gateway": "172.18.0.1/16"
                }
            ]
        },
        "Containers": {},
        "Options": {}
    }
]

```



## 指定容器使用的网络

创建自定义网络，可以增加web应用的安全性。 理论上，容器在不同网络之间完全隔离。 可以在容器初次启动的时候指定加入的网络。

启动一个 PostgreSQL database容器，并使用 ` --network=my-bridge-network `标识将容器加入my-bridge-network网络：

```bash

$ docker run -d --network=my-bridge-network --name db training/postgres

```


查看 `my-bridge-network` 的信息， 可以观察到有个容器加入。 你也可以查看你的容器加入了哪个网络：

```

$ docker network inspect  my-bridge-network

##

$ docker inspect --format='{{json .NetworkSettings.Networks}}'  db

{"my-bridge-network":{"NetworkID":"7d86d31b1478e7cca9ebed7e73aa0fdeec46c5ca29497431d3007d2d9e15ed99",
"EndpointID":"508b170d56b2ac9e4ef86694b0a76a22dd3df1983404f7321da5649645bf7043","Gateway":"172.18.0.1","IPAddress":"172.18.0.2","IPPrefixLen":16,"IPv6Gateway":"","GlobalIPv6Address":"","GlobalIPv6PrefixLen":0,"MacAddress":"02:42:ac:11:00:02"}}

```


现在，重新启动之前的web应用。 不过这是不使用 ` -P ` 、也不指定网络名。

```

$ docker run -d --name web training/webapp python app.py

```


查看容器信息，确认容器运行在哪个网络下：

```

$ docker inspect --format='{{json .NetworkSettings.Networks}}'  web

{"bridge":{"NetworkID":"7ea29fc1412292a2d7bba362f9253545fecdfa8ce9a6e37dd10ba8bee7129812",
"EndpointID":"508b170d56b2ac9e4ef86694b0a76a22dd3df1983404f7321da5649645bf7043","Gateway":"172.17.0.1","IPAddress":"172.17.0.2","IPPrefixLen":16,"IPv6Gateway":"","GlobalIPv6Address":"","GlobalIPv6PrefixLen":0,"MacAddress":"02:42:ac:11:00:02"}}

```


获取容器ID地址

```

$ docker inspect --format='{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' web

172.17.0.2

```


进入 ` db ` 容器：

```

$ docker exec -it db bash

root@a205f0dd33b2:/# ping 172.17.0.2
ping 172.17.0.2
PING 172.17.0.2 (172.17.0.2) 56(84) bytes of data.
^C
--- 172.17.0.2 ping statistics ---
44 packets transmitted, 0 received, 100% packet loss, time 43185ms

```


在db中ping web将会失败。因为两个容器在不容的网络中。

docker允许一个容器加入多个不同的网络，可以在容器运行时添加容器网络。 使用命令将web容器添加到 my-bridge-network中。

```

$ docker network connect my-bridge-network 

```


重新进入db容器，并ping web容器，这次使用容器名web替代使用容器IP地址：

```

$ docker exec -it db bash

root@a205f0dd33b2:/# ping web
PING web (172.18.0.3) 56(84) bytes of data.
64 bytes from web (172.18.0.3): icmp_seq=1 ttl=64 time=0.095 ms
64 bytes from web (172.18.0.3): icmp_seq=2 ttl=64 time=0.060 ms
64 bytes from web (172.18.0.3): icmp_seq=3 ttl=64 time=0.066 ms
^C
--- web ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2000ms
rtt min/avg/max/mdev = 0.060/0.073/0.095/0.018 ms

```


ping的结果显示，web使用了一个不同的IP地址，该IP属于 ` my-bridge-network ` 网络。




