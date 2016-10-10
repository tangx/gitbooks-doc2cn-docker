# 容器中的Hello World

> 2016-09-26 octowhale

Docker是什么？

Docker允许你在容器中运行你创建的应用或其他任何东西。而运行一个应用只需要一个简单的命令： `docker run`。

> **注意**：跟你你docker系统的配置，可能你需要使用 `sudo`才能执行docker命令。如果不想再使用`sudo`，可以将用户添加到`docker`用户组。

## 输出hello wolrd

来，跟我一起启动一个hello world的容器。

```

$ docker run ubuntu /bin/echo "hello wolrd"
hello wolrd

```

就在刚才你启动了你的第一个容器。

刚才我们做了什么？
+ ` docker run ` 启动了一个容器。
+ ` ubuntu `是这个容器使用的镜像。如果本地没有该镜像则会从DockerHub上下载。
+ 镜像启动完成后，在容器内执行了 `/bin/echo` 命令。

这个容器启动后，Docker创建了一个新的ubuntu系统环境并在容器内执行`/bin/echo`已经打印输出

```

hello world

```


那在此之后容器有做了什么呢？Docker容器在活动时只会执行你指定的所有命令。例如，在上面这个例子中，容器在执行一次命令后就退出咯。

## 启动一个可交互的容器

让我们使用新命令启动另一个容器


```

$ docker run -t -i ubuntu /bin/bash
root@295431edfd4a:/# 

```


在本例中：
+ `docker run `启动了一个容器。
+ `ubuntu`是使用的镜像。
+ ` -t ` 在容器内部注册了一个 `pseudo-tty`或者`terminal`。
+ ` -i ` 通过容器的标准输入[STDIN]建立了一个链接，允许你与容器进行交互
+ ` /bin/bash ` 在容器中启动 bash shell。

容器启动后，我们可以看到命令行提示为：

```

root@295431edfd4a:/# 

```


让我们在容器中执行一些命令：

```

root@295431edfd4a:/# pwd
/
root@295431edfd4a:/# ls
bin  boot  dev  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
root@295431edfd4a:/# 

```


在本例中：
`pwd` 显示了当前目录
` ls ` 显示了当前目录中的文件和文件夹。

现在，你可以在container中尽情操作。当需要结束是，使用 `exit`命令或 `Ctrl+D`退出交互shell。

```

root@295431edfd4a:/# exit
exit
[octowhale@s008-docker-centos7 ~]$ 

```

> **注意**: 和我们第一个启动的容器一样，这里一旦结束了 `bash shell`进程，容器就停止了。

## 启动一个后台容器

我们启动第3个容器，该容器将在后台运行。

```

$ docker run -d ubuntu /bin/sh -c "while true; do echo hello world; sleep 1; done"

1e5535038e285177d5214659a068137486f96ee5c2e85a4ac52dc83f2ebe4147

```

+ ` -d ` 参数使容器在后台运行。

在输出中，我们看不到 hello world，但是有一串很长的字符串

```

1e5535038e285177d5214659a068137486f96ee5c2e85a4ac52dc83f2ebe4147

```

这个字符串就是`容器ID`。 该字符串唯一标识一个容器的，我们可以通过该字符串操作容器。
> **注意**：这个容器ID显然太长了。之后我们将使用更短的ID和方法标示容器。

我们可以使用这个容器ID查看`hello wolrd`做了什么。

首先， 我们需要确认容器正在运行。 使用命令 `docker ps ` 查看。 该命令将引导docker进程列出所有已知容器的信息。

```

$ docker ps

CONTAINER ID  IMAGE         COMMAND               CREATED        STATUS       PORTS NAMES
1e5535038e28  ubuntu  /bin/sh -c 'while tr  2 minutes ago  Up 1 minute        insane_babbage

```


在本例中，我们可以看到处于后台运行的容器。 `docker ps ` 另外返回了一些有用的信息：
+ `1e5535038e28` 这是一个更短的容器ID。
+ `ubuntu` 是我们所使用的镜像。
+ 另外还有容器启动后执行的命令，容器状态和容器名称。

> **注意**：如果启动容器的时候不指定容器名称，那么docker会自动为容器分配一个。

我们确认了容器正在运行，但是容器是否按照我们要求在运行呢？为了确认，我们可以使用 `docker log`命令观察容器内部。

这里我们使用容器名称

```

$ docker logs insane_babbage

hello world
hello world
hello world
. . .

```


本例中：
+ `docker log `查看了容器的内部，并返回 ` hello world`

棒极了。docker还在后台运行。另外，你刚创建了你的第一个docker化的应用。

接下来，我们使用 `docker stop`命令关闭容器。

```

$ docker stop insane_babbage

insane_babbage

```


`docker stop`命令通知docker安全的停止运行中的容器，并且在成功停止后返回容器名。

使用 `docker ps `查看容器运行状态

```

$ docker ps

CONTAINER ID  IMAGE         COMMAND               CREATED        STATUS       PORTS NAMES

```


Excellent. 所有容器都已经停止了。
