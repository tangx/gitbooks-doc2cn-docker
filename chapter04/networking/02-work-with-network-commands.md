# 使用网络命令
work-with-network-commands

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




