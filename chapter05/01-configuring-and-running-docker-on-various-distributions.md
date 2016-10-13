# docker在各种发行版中的配置与启动

> 2016-10-13 octowhale@github

[ 原文 ](https://docs.docker.com/engine/admin/)


docker安装完成后， ` docker daemon ` 使用默认配置启动。

在生产环境中， 系统管理员通常按照企业要求配置 ` docker daemon `运行环境。 在大多数情况下，系统管理员使用进程管理工具(ex. ` SysVinit, Upstart, systemd `) 配置并管理docker。


### 直接启动docker daemon

docker daemon可以通过使用 ` dockerd ` 命令直接启动。 默认监听 unix socket ` unix:///var/run/docker.sock ` 。

```bash
$ dockerd

INFO[0000] +job init_networkdriver()
INFO[0000] +job serveapi(unix:///var/run/docker.sock)
INFO[0000] Listening for HTTP on unix (/var/run/docker.sock)
...
...

```

### 直接配置docker daemom

如果你不用进程管理器，而是直接使用 ` dockerd ` 命令直接启动docker daemon， 你可以在 ` dockerd ` 命令后天面颊配置选项。 其他选项会被传递到docker daemon并配置。

部分 docker daemom 选项：

|  Flag  |  描述  |
|  ----  |  ----  |
| ` -D, --debug=false `  | 是否启动debug模式， 默认为否 |
| ` -H, --host=[]  `     | 监听docker服务的地址和端口 |
| ` --tls=false  `       | 是否启用 TLS, 默认为否  | 

下面是一个直接启动docker daemon的例子：

```bash

$ dockerd -D --tls=true --tlscert=/var/docker/server.pem --tlskey=/var/docker/serverkey.pem -H tcp://192.168.59.3:2376

```

+ ` -D `: 启动debug模式
+ ` --tls=true ` ： 使用TSL。
  + ` --tlscert ` 指定证书
  + ` --tlskey ` 指定密钥。
+ 监听 ` tcp://192.168.59.3:2376 ` 的连接

dockerd命令解释参考[complete list of daemon flags](https://docs.docker.com/engine/reference/commandline/dockerd/)


### Daemon debugging

正如之前提到的， 设置daemon的日志等级为 ` debug ` 或者启动debug模式( `-D , --debug=ture` )， 管理员可以获得更多的关于daemon运行时的信息。 如果是一个无应答deamon(` non-responsive deamon`)， 管理员可以向docker daemon 发送 ` SIGUSR1 `信号，强制对所有线程进行全堆栈追踪(stack trace)，添加daemon log。 通常是使用 ` kill ` 命令发送该信号。 例如  ` kill -USR1 <daemon-pid> `。

> **注意**： 使用堆栈追踪(stack trace)方式设置日志等级，日志等级必须为 info 及以上 。 默认 daemon的日志等级设置为 info。

The daemon will continue operating after handling the SIGUSR1 signal and dumping the stack traces to the log. The stack traces can be used to determine the state of all goroutines and threads within the daemon. 

## Ubuntu

从ubuntu 14.04开始， 使用 Upstart 作为进程管理器。默认的， Upstart任务都放在 ` /etc/init ` 中， ` docker `的哦upstart任务为 ` /etc/init/docker.conf ` 。

成功在 [ubuntu上安装docker](https://docs.docker.com/engine/installation/linux/ubuntulinux/) 之后， 使用 status 查看docker运行状态：

```bash
$ sudo status docker

docker start/running, process 989

```

### 启动docker

You can start/stop/restart the docker daemon using

```bash
$ sudo start docker

$ sudo stop docker

$ sudo restart docker

```

### 配置docker运行参数

以下指令描述了在使用 `upstart` 的系统上配置docker参数。从Ubuntu 15.04开始， 使用 `systemd`作为进程管理器。 ubuntu 15.04及以后的版本，参考 [使用systemd控制和配置docker](./control-and-configure-docker-with-systemd.md)。

配置docker daemon需要修改文件 ` /etc/default/docker ` 。 在文件中指定 ` DOCKER_OPTS ` 变量值。

配置docker选项：
1. 登录具有sudo或root权限的用户。
2. 如果不存在 ` /etc/default/docker ` 则创建一个。 根据docker的安装方式不同，可以自带。
3. 使用编辑器打开文件 ` $ sudo vi /etc/default/docker ` 。
4. 为 ` DOCKER_OPTS ` 变量添加下列选项。 这些选项将被添加到 docker daemon 的启动命令中。

```bash

DOCKER_OPTS="-D --tls=true --tlscert=/var/docker/server.pem --tlskey=/var/docker/serverkey.pem -H tcp://192.168.59.3:2376"

```

+ ` -D `: 启动debug模式
+ ` --tls=true ` ： 使用TSL。
  + ` --tlscert ` 指定证书
  + ` --tlskey ` 指定密钥。
+ 监听 ` tcp://192.168.59.3:2376 ` 的连接

dockerd命令解释参考[complete list of daemon flags](https://docs.docker.com/engine/reference/commandline/dockerd/)

5. 保存并关闭文件
6. 重启docker daemon,

```bash
$ sudo restart docker 
```

7. 验证docker daemon是否使用新参数

```bash
$ ps aux | grep docker | grep -v grep
```

### 日志

默认的， Upstart任务的日志存放在 ` /var/log/upstart ` ， docker daemon的日志存放在 ` /var/log/upstart/docker.log ` 。

```bash
$ tail -f /var/log/upstart/docker.log
INFO[0000] Loading containers: done.
INFO[0000] Docker daemon commit=1b09a95-unsupported graphdriver=aufs version=1.11.0-dev
INFO[0000] +job acceptconnections()
INFO[0000] -job acceptconnections() = OK (0)
INFO[0000] Daemon has completed initialization

``` 


## CentOS / RHEL / Fedora

从 CentOS / RHEL 7.x 或 Fedora 21 开始， 使用 `systemd` 作为进程管理器。 

成功在 CentOS/RHEL/Fedora上安装docker后，使用以下命令检查状态：

```bash
$ sudo systemctl status docker

```

### 启动docker
You can start/stop/restart the docker daemon using

```bash
$ sudo systemctl start docker

$ sudo systemctl stop docker

$ sudo systemctl restart docker
```

开机启动docker, you should also:

```bash
$ sudo systemctl enable docker
```


### 配置docker运行参数

在 CentOS 7.x 和 RHEL 7.x中，你可以通过 [control and configure Docker with systemd](https://docs.docker.com/engine/admin/systemd/.

在之前的 CentOS6.x中配置docker daemon，你可以在文件 ` /etc/sysconfig/docker ` 中指定 ` other_args ` 的值。在很短的一段时间内， CentOS 7.x中，你可以通过指定 ` OPTIONS ` 的值。 然而，现在不在支持使用 systemd 配置了。

本节中，我们将学习一个在CentOS 7.x中配置docker的案例。

配置docker选项：

1. 使用具有sodu或root权限的用户的呢公路
2. 创建配置文件目录

```bash
$ sudo mkdir /etc/systemd/system/docker.service.d
```

3. 创建配置文件 ` /etc/systemd/system/docker.service.d/docker.conf `

4. 使用编辑器编辑配置文件

```bash
$ sudo vi /etc/systemd/system/docker.service.d/docker.conf
```

5. 在 ` docker.service ` 文件中重载 ` ExecStart ` 配置以定制的docker daemon。 为了使 ` ExecStart ` 配置生效， 你需要在配置行前面指定一个空配置行。

```bash
[Service]
ExecStart=
ExecStart=/usr/bin/dockerd -H fd:// -D --tls=true --tlscert=/var/docker/server.pem --tlskey=/var/docker/serverkey.pem -H tcp://192.168.59.3:2376

```

6. 保存并推出配置文件爱你

7. 刷新改变

```bash
$sudo systemctl daemon-reload
```

8. 重启docker daemon

```bash
$ sudo systemctl restart docker 
```

9 验证docker daemon是否使用新配置

```bash
$ ps aux | grep docker |grep -v grep 
```

### 日志

systemd 拥有自己的日志系统，被称为 ` journal `。  docker daemon的日志可以使用  ` journalctl -u docker ` 查看：

```bash
$ sudo journalctl -u docker
May 06 00:22:05 localhost.localdomain systemd[1]: Starting Docker Application Container Engine...
May 06 00:22:05 localhost.localdomain docker[2495]: time="2015-05-06T00:22:05Z" level="info" msg="+job serveapi(unix:///var/run/docker.sock)"
May 06 00:22:05 localhost.localdomain docker[2495]: time="2015-05-06T00:22:05Z" level="info" msg="Listening for HTTP on unix (/var/run/docker.sock)"
May 06 00:22:06 localhost.localdomain docker[2495]: time="2015-05-06T00:22:06Z" level="info" msg="+job init_networkdriver()"
May 06 00:22:06 localhost.localdomain docker[2495]: time="2015-05-06T00:22:06Z" level="info" msg="-job init_networkdriver() = OK (0)"
May 06 00:22:06 localhost.localdomain docker[2495]: time="2015-05-06T00:22:06Z" level="info" msg="Loading containers: start."
May 06 00:22:06 localhost.localdomain docker[2495]: time="2015-05-06T00:22:06Z" level="info" msg="Loading containers: done."
May 06 00:22:06 localhost.localdomain docker[2495]: time="2015-05-06T00:22:06Z" level="info" msg="Docker daemon commit=1b09a95-unsupported graphdriver=aufs version=1.11.0-dev"
May 06 00:22:06 localhost.localdomain docker[2495]: time="2015-05-06T00:22:06Z" level="info" msg="+job acceptconnections()"
May 06 00:22:06 localhost.localdomain docker[2495]: time="2015-05-06T00:22:06Z" level="info" msg="-job acceptconnections() = OK (0)"

```

*注意： 使用配置 journal 不是本文讨论范围*

----

## 总结：

+ docker daemon 配置参数管理分为两种：
  + dockerd 直接在后面添加参数
    + 例如 ubuntu 14.04   ( `/etc/default/docker` )
    + 例如 CentOS/RHEL 6.x  ( `/etc/systemd/system/docker.service.d/docker.conf` )
  + 通过 类似于 Upstart， systemd 等进程管理软件管理
    + CentOS 7 中， 修改docker daemon参数，配置文件需要多一个空配置行  ` ExecStart= `。
  + docker现在更倾向于时候后者


