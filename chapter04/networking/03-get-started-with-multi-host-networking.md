# docker的多主机网络-overlay网络
# get started with multi-host network

> octowhale@github 2016-10-28
> [ 原文 ](https://docs.docker.com/engine/userguide/networking/get-started-overlay/)


本文通过案例讲解如何创建多主机网络的基础只是。docker engine通过`overlay`网络驱动支持多主机网络互联。与`bridge`网络不同，创建overlay网络之前主要一些前置准备。

+ [docker engine运行在swarm模式](./03-get-started-with-multi-host-networking.md#overlay-networking-and-swarm-mode)
或者
+ [集群主机使用键值对存储](./03-get-started-with-multi-host-networking.md#overlay-networking-with-an-external-key-value-store)

## 覆盖网络与swarm模式
## Overlay networking and swarm mode

docker engine使用[swarm模式](https://docs.docker.com/engine/swarm/swarm-mode/)时，你可以在overlay网络中创建一个管理节点。

该群使得覆盖网络只提供给在群需要它为服务节点。当你创建了一个使用覆盖网络的服务节点时，管理节点会自动将覆盖网络扩展到裕兴服务的节点。

