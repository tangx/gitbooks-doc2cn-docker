# 自动启动容器

> 2016-10-13 octowhale@github

[ 原文 ](https://docs.docker.com/engine/admin/host_integration/)


从docker 1.2 开始， [restart policies](https://docs.docker.com/engine/reference/run/#restart-policies-restart)作为内建的docker机制，用来重新启动退出的容器。如果设置了， restart policies 会在docker daemon启动时被应用，通常在处于系统启动之后。 restart policies 保证相互连接的容器之间拥有正确的启动顺序。

如果你不需要 restart policies(i.e., you have non-Docker processes that depend on Docker containers)， 你可以使用进程管理器(like upstart, systemd or supervisor instead.)。


## 使用进程管理器

docker默认没有设置任何restart policies。 需要注意restart policies与大多数进程管理器相互冲突。 如果使用进程管理器就不要使用restart policies。

当你完成正在运行一个容器时，你可以使用进程管理器管理他。 当你运行 ` docker start -a ` 时， docker会自动 attach到正在运行的容器上或者在有必要的时候启动容器；并转发所有信号，这样进程管理器可以探测到容器什么时候关闭和在正确时候启动它。


## Examples

The examples below show configuration files for two popular process managers, upstart and systemd. In these examples, 假设已经存在容器运行redis服务，容器名为**redis_server** ( ` --name=redis_server `) . 这些文件定义了**一个新服务，会在docker daemon启动后自动启动**。


### upstart

```bash
description "Redis container"
author "Me"
start on filesystem and started docker
stop on runlevel [!2345]
respawn
script
  /usr/bin/docker start -a redis_server
end script

```

### systemd

```bash
[Unit]
Description=Redis container     # 此服务描述
Requires=docker.service         # 此服务依赖的服务
After=docker.service            # 在docker.server 服务后启动该服务

[Service]
Restart=always
ExecStart=/usr/bin/docker start -a redis_server
ExecStop=/usr/bin/docker stop -t 2 redis_server

[Install]
WantedBy=default.target

```

如果你打算使用这个作为系统服务， 把上面的内容保存在配置文件中 ` /etc/systemd/system/docker-redis_server.service `。

如果你需要给reids容器传递参数(such as --env)， 那么你需要使用 ` docker run ` 而非 ` docker start ` 。 这样每次服务启动后都会创建一个全新的容器， 这些容器会在服务关闭被停止与删除。

```bash
[Service]
...
ExecStart=/usr/bin/docker run --env foo=bar --name redis_server redis
ExecStop=/usr/bin/docker stop -t 2 redis_server
ExecStopPost=/usr/bin/docker rm -f redis_server
...

```

To start using the service, reload systemd and start the service:

```bash
systemctl daemon-reload
systemctl start docker-redis_server.service

```

To enable the service at system startup, execute:

```bash
systemctl enable docker-redis_server.service
```


----

## 总结

+ docker容器可以设置通过 restart policies或进程管理器设置启动
+ restart policies 与 进程管理器 互斥。 因为两者可能发生冲突
+ 使用systemd自动启动容器实质是**自定义启动服务**


