# 使用二进制文件安装Docker1.11及以后版本

> 2016-09-25 uyinn28@github

[原文](https://docs.docker.com/engine/installation/binaries/#/installation-from-binaries)

** 本文是为那些愿意在不同的环境下尝试docker的高端玩家准备的 **

在执行随后的操作之前，你确实需要确认一下当前版本的Docker是否支持你的操作系统。目前已经支持很多发行版了，以后会更多。

## 检查程序依赖

想要docker正常运行，必须安装以下软件：
+ iptables 版本 1.4 或以后
+ Git 版本 1.7 或以后
+ procps 或提供相似功能的 可执行ps
+ XZ Utils 4.9 或以后
+ 一个[适合挂载](https://github.com/tianon/cgroupfs-mount/blob/master/cgroupfs-mount)的cgroupfs分层（单一的，完全包含的 “cgroup”挂载点并[不能满足要求](https://github.com/docker/docker/issues/4568)）


## 检查内核依赖
Docker的守护模式对内核版本有特殊需求。更多细节，参考你发行版的对应[安装手册](https://docs.docker.com/engine/installation/#on-linux)

A 3.10版的Linux内核对于Docker而言是最低要求。低于3.10版的内核缺少一个Docker容器必须的功能。在某些已确认的条件下，老版内核可能因为一些已知bug会造成数据丢失或频繁的混乱。

推荐使用3.10的最新minor版本(3.x.y)，或者使用更新的已维护的版本。持续更新内核到最新minor版本，可以确保已知内核BUG被修复。

> **警告**：docker可能不支持你的linux版本供应商定制版的内核或内核包。准备在这些定制版的内核上安装docker之前，请向你的供应商确认是否支持Dcoker。
>
>**警告**：在某些操作系统发行版上安装新版本的内核也不一定管用，毕竟可能系统提供的软件跟不上不发。
>

>**注意**：Docker同时也有一个**客户端模式**，该模式可以运行在虚拟linux内核上，即使是OSX。

## 在需要的时候开启AppArmor和SELinux

如果你的操作系统支持，使用AppArmor或SELinux其中一个。这将会提高docker安全性，并且能阻挡已知类型的崩溃。你的操作系统文档应该提供了`如何开启推荐安全机制`的详细步骤。

一些linux发行版默认开启了AppArmor或SELinux，但是其运行的内核环境并不是我们所要求的最低版本[3.10或以后]。在这样的操作系统中，仅升级内核到最新版可能并不足以运行docker或容器。该系统提供的`AppArmor/SELinux用户空间组件`和升级的后的内核版本不兼容，可能会阻止Docker运行、阻止启动容器或者触发容器出现无法预测的行为。

> **警告**: 如果安全机制处于enabled状态，那么他不应该为了运行docker或容器而被设置为disbaled。这样会减少环境提供的安全保护，失去发行版商为系统提供的安全支持，and might break regulations and security policies in heavily regulated environments.

## 获取Linux版的Docker Engine二进制文件

你可以选择下载最新版本或指定版本的二进制文件。访问[docker发布页面](https://github.com/docker/docker/releases)获取稳定的docker版本。
访问以`.md5或.sha256`结尾的URL可以查看二进制文件包的MD5和SHA256信息。

### 下载linux版的二进制文件包
通过以下命令，下载最新版本的二进制文件包：

```

https://get.docker.com/builds/Linux/i386/docker-latest.tgz

https://get.docker.com/builds/Linux/x86_64/docker-latest.tgz

```


通过指定版本号，下载指定版本的二进制文件包。

```

https://get.docker.com/builds/Linux/i386/docker-<version>.tgz

https://get.docker.com/builds/Linux/x86_64/docker-<version>.tgz

```

例如：

```

https://get.docker.com/builds/Linux/i386/docker-1.11.0.tgz

https://get.docker.com/builds/Linux/x86_64/docker-1.11.0.tgz

```

> **注意**：这些操作适用于DockerEngine1.11及以后版本。1.10及之前的版本由一个单独的二进制文件组成，两种版本类型的操作方式不一样。如果要安装1.10或之前的版本，可以参考[1.10文档](https://docs.docker.com/v1.10/engine/installation/binaries/)


## 通过二进制文件安装
下载完成后，解压二进制文件包时，会在当前目录生成一个名为docker的目录。

```

$ tar -xvzf docker-latest.tgz

docker/
docker/docker
docker/docker-containerd
docker/docker-containerd-ctr
docker/docker-containerd-shim
docker/docker-proxy
docker/docker-runc
docker/dockerd

```

Docker引擎需要安装在的 ` $PATH `目录下。 例如: ` /usr/bin/ `：

```

$ mv docker/* /usr/bin/

```

> **注意**：如果之前在本地安装过DockerEngine，确保再次安装前已经停止Docker运行，` killall docker `；并且将新的二进制文件安装在和之前相同的位置。你可以通过命令 ` which docker ` 获得之前安装的目录信息。

### 启动docker守护进程
通过以下命令启动DockerEngine的守护模式

```

$ sudo dockerd &

```

在GitHub上提供了一些初始化脚本的模板，你可以通过诸如upstart或systemd的进程管理器管理docker daemon。访问[contrib directory](https://github.com/docker/docker/tree/master/contrib/init)获取这些脚本。

访问[deamon command命令行参考文档](https://docs.docker.com/engine/reference/commandline/dockerd/)，可以获得更多关于管理docker engine的信息。


## 获取MacOSX版的二进制文件
MacOSX版的二进制文件只是一个客户端。你不能在OSX下运行 `docker daemon `。 使用下面的URL获取最新版本的docker。

```

https://get.docker.com/builds/Darwin/x86_64/docker-latest.tgz

```

使用下面的URL获取指定版本的docker。

```

https://get.docker.com/builds/Darwin/x86_64/docker-<version>.tgz
https://get.docker.com/builds/Darwin/x86_64/docker-1.11.0.tgz

```

你可以通过双击` .tgz `压缩包解压下载的文件；也可以使用命令 ` tar -xvzf docker 1.11.0.tgz `。docker client可以在文件系统的任何位置被执行。

## 获取windows版的二进制文件
目前，windows版的只提供1.9.1及之后的版本下载。此外，32-bit的文件只是一个客户端，不能作为deamon运行。但64-bit的文件兼顾客户端和守护进程。

通过以下URL下载最新版本docker：

```

https://get.docker.com/builds/Windows/i386/docker-latest.zip

https://get.docker.com/builds/Windows/x86_64/docker-latest.zip

```

通过以下URL下载制定版本的docker：

```

https://get.docker.com/builds/Windows/i386/docker-<version>.zip

https://get.docker.com/builds/Windows/x86_64/docker-<version>.zip

https://get.docker.com/builds/Windows/i386/docker-1.11.0.zip

https://get.docker.com/builds/Windows/x86_64/docker-1.11.0.zip

```


> **注意**：这些操作适用于DockerEngine1.11及以后版本。1.10及之前的版本由一个单独的二进制文件组成，两种版本类型的操作方式不一样。如果要安装1.10或之前的版本，可以参考[1.10文档](https://docs.docker.com/v1.10/engine/installation/binaries/)


## 设置非root权限访问
docker daemon一直是使用root用户权限运行的，而且现在docker daemon绑定的unix socket取代了之前使用的TCP端口。默认情况下，Unix socket是属于root用的，因此你可以通过 ` sudo ` 访问使用docker。

如果创建一个叫做 `docker` 的用户组并将部分非root用户添加进该组，那么在docker daemon运行的时候，docker daemon会赋予该组用户对于unix socket的读写权限。虽然docker daemon必须使用root用户运行，但是在docker组内的用户不再需要使用 ` sudo ` 命令进行docker操作。
具体操作步骤，可以参考[创建一个docker用户组](./01-install-docker-with-centos.md#创建一个docker用户组)

> **警告**：`docker`用户组(或组通过-G指定)等同于root。更多细节可以参考[Docker Daemon Attack Surface](https://docs.docker.com/engine/security/security/#docker-daemon-attack-surface)


## 升级docker引擎
如果要在linux系统下升级docker，必须要先关闭docker daemon

```

$ killall docker

```

然后按照[常规安装步骤](#获取linux版的docker-engine二进制文件)进程安装。

