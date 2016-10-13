# 使用systemd控制和配置docker
control-and-configure-docker-with-systemd

> date: 2016-10-13
> author: octowhale@github
> [ 原文 ](https://docs.docker.com/engine/admin/systemd/)

Many Linux distributions use systemd to start the Docker daemon. This document shows a few examples of how to customize Docker’s settings.


## Starting the Docker daemon

启动docker

```bash
$ sudo systemctl start docker

# or on older distributions, you may need to use
$ sudo service docker start
```

开机启动docker

```bash
$ sudo systemctl enable docker

# or on older distributions, you may need to use
$ sudo chkconfig docker on
```


## 定制docker daemon选项

有很多中方法可以配置docker daemon的标识(flags)和环境变量。

我们推荐的方法是使用 `插入文件(drop-in file)` (as described in the [systemd.unit](https://www.freedesktop.org/software/systemd/man/systemd.unit.html) documentation)。 这些文件命名规则为 ` <something>.conf ` ， 保存在 ` /etc/systemd/system/docker.service.d ` 目录中。 drop-in文件也可以是 ` /etc/systemd/system/docker.service ` ， 不过这会覆盖默认配置 ` /lib/systemd/system/docker.service ` 。

然而， 如果你之前使用的安装包包含了 ` EnvironmentFile `(通常指向 ` /etc/sysconfig/docker ` )， 那么为了向后兼容，你需要在 ` /etc/systemd/system/docker.service.d ` 目录中放入一个以 ` .conf ` 结尾的文件， 文件内容包含，

```bash

[Service]
EnvironmentFile=-/etc/sysconfig/docker
EnvironmentFile=-/etc/sysconfig/docker-storage
EnvironmentFile=-/etc/sysconfig/docker-network
ExecStart=
ExecStart=/usr/bin/dockerd $OPTIONS \
          $DOCKER_STORAGE_OPTIONS \
          $DOCKER_NETWORK_OPTIONS \
          $BLOCK_REGISTRY \
          $INSECURE_REGISTRY

```

使用命令检查 ` docker.service ` 是否是使用了 ` EnvironmentFile `：

```bash

$ systemctl show docker | grep EnvironmentFile
EnvironmentFile=-/etc/sysconfig/docker (ignore_errors=yes)

```

另一种方法是找到service文件的位置，查看是否使用EnvironmentFile：

```bash
$ systemctl show --property=FragmentPath docker
FragmentPath=/usr/lib/systemd/system/docker.service
$ grep EnvironmentFile /usr/lib/systemd/system/docker.service
EnvironmentFile=-/etc/sysconfig/docker

```

你可以和下面[HTTP Proxy example]中一样，使用覆盖文件(override file)定制的docker daemon参数选项。 该文件为在 ` /usr/lib/systemd/system ` 或 ` /lib/systemd/system `中， 包含了默认参数且不会被更改。


### 运行目录和存储驱动
Runtime directory and storage driver

你可能想将docker 镜像，容器和卷的储存目录放在独立分区。

本例中，我们假设你的 ` docker.service ` 如下：

```bash
[Unit]
Description=Docker Application Container Engine
Documentation=https://docs.docker.com
After=network.target

[Service]
Type=notify
# the default is not to use systemd for cgroups because the delegate issues still
# exists and systemd currently does not support the cgroup feature set required
# for containers run by docker
ExecStart=/usr/bin/dockerd
ExecReload=/bin/kill -s HUP $MAINPID
# Having non-zero Limit*s causes performance problems due to accounting overhead
# in the kernel. We recommend using cgroups to do container-local accounting.
LimitNOFILE=infinity
LimitNPROC=infinity
LimitCORE=infinity
# Uncomment TasksMax if your systemd version supports it.
# Only systemd 226 and above support this version.
#TasksMax=infinity
TimeoutStartSec=0
# set delegate yes so that systemd does not reset the cgroups of docker containers
Delegate=yes
# kill only the docker process, not all processes in the cgroup
KillMode=process

[Install]
WantedBy=multi-user.target


```

这将允许你通过 drop-in 文件添加额外的标识， drop-in文件放在 ` /etc/systemd/system/docker.service.d ` 目录：

```bash

[Service]
ExecStart=
ExecStart=/usr/bin/dockerd --graph="/mnt/docker-data" --storage-driver=overlay

```

你也可以在该文件中设置其他的的环境变量， 例如下面将提到的` HTTP_PROXY ` 。

为了使修改的 ExecStart 变量生效， 必须首先指定一个空配置行， 然后再指定新配置，如下：

```bash
[Service]
ExecStart=
ExecStart=/usr/bin/dockerd --bip=172.17.42.1/16
```

如果你忘记指定空配置文件行，docker会报错，如下：

```
docker.service has more than one ExecStart= setting, which is only allowed for Type=oneshot services. Refusing.

```

### HTTP proxy

本例展示如何覆盖默认 ` docker.service ` 文件。

如果你的docker在一个 HTTP 代理服务器后面， 那么你需要在 docker systemd service 文件中添加一些配置。

首先， 创建一个 systemd drop-in 目录：

```bash
mkdir /etc/systemd/system/docker.service.d 
```

然后，创建一个名为 ` /etc/systemd/system/docker.service.d/http-proxy.conf ` 的文件，用来添加 ` HTTP_PROXY ` 环境变量：

```bash
[Service]
Environment="HTTP_PROXY=http://proxy.example.com:80/"

```

如果你有一个内部docker仓库，不需要使用代理访问，你可以通过 ` NO_PROXY ` 环境变量指定：

```bash
Environment="HTTP_PROXY=http://proxy.example.com:80/" "NO_PROXY=localhost,127.0.0.1,docker-registry.somecorporation.com"

```

刷新变更

```bash
$ sudo systemctl daemon-reload 
```

验证配置是否被加载

```bash
$ systemctl show --property=Environment docker
Environment=HTTP_PROXY=http://proxy.example.com:80/
```

重启docker

```bash
$ sudo systemctl restart docker

```


## 手动创建systemd unit files

When installing the binary without a package, you may want to integrate Docker with systemd. For this, simply install the two unit files (service and socket) from [the github repository](https://github.com/docker/docker/tree/master/contrib/init/systemd) to /etc/systemd/system.



----

## 总结

+ docker的配置文件可以不做修改，使用systemd管理drop-in文件进行变量覆盖
  + ` /etc/systemd/system/docker.service ` 将完全覆盖docker默认配置
  + ` /etc/systemd/system/docker.service.d/ ` 目录下的 ` .conf ` 文件将增加配置或覆盖相同部分默认配置
  + ` HTTP_PROXY ` 设置代理； ` NO_PROXY ` 跳过代理
  + 如果之前的配置使用了 ` EnvironmentFile ` ， 为了向后兼容，也必须在` /etc/systemd/system/docker.service.d/somefile.conf` 使用
  + 通过systemd的drop-in文件覆盖 ExecStart 变量值的时候，必须要有一个空行。


