# 运行一个简单的应用

> 2016-09-28 octowhale@github

[原文](https://docs.docker.com/engine/tutorials/usingdocker/#/run-a-simple-application)

在之前[容器中的Hello World](./01-hello-world-in-a-container.md)我们学习了如何使用 ` docker run `启动容器。我们在前台运行了一个**可交互**容器；也可以在后台运行一个**分离的**容器。之前你学习了以下几个命令
+ ` docker ps ` 显示容器
+ ` docker logs ` 显示容器中的标准输出
+ ` docker stop ` 停止一个运行中的容器

## 学习如何使用docker客户端

可能你还没有意识到，你之前使用的在bash终端中输入的每一条docker命令都是在使用docker客户端。命令行客户端又被称为命令行接口(CLI)。你每一次执行的行为都是由`docker命令和一些标记和参数`组成。

```bash

# Usage:  [sudo] docker [subcommand] [flags] [arguments] ..
# Example:
$ docker run -i -t ubuntu /bin/bash

```


使用命令 ` docker version ` 可以查看你当前使用的docker客户端和服务端版本信息。

```bash

$ docker version 

```


返回结果不仅包含了docker客户端和服务端的信息，还包含了编写docker的GO语言的版本信息。

```bash

Client:
  Version:      1.8.1
  API version:  1.20
  Go version:   go1.4.2
  Git commit:   d12ea79
  Built:        Thu Aug 13 02:35:49 UTC 2015
  OS/Arch:      linux/amd64

Server:
  Version:      1.8.1
  API version:  1.20
  Go version:   go1.4.2
  Git commit:   d12ea79
  Built:        Thu Aug 13 02:35:49 UTC 2015
  OS/Arch:      linux/amd64

```


## docker命令帮助信息
使用 ` --help ` 参数， 可以查看docker命令的帮助信息。 查询docker所有子命令，如下：

```bash

$ docker --help 

```


查看具体子命令的帮助信息，则是在子命令后使用 ` --help ` 参数。

```bash

$ docker attach --help

Usage: docker attach [OPTIONS] CONTAINER

Attach to a running container

Options:
  --detach-keys string   Override the key sequence for detaching a container
  --help                 Print usage
  --no-stdin             Do not attach STDIN
  --sig-proxy            Proxy all received signals to the process (default true)

```


> **注意**：如果要了解更多命令的详细信息和用法，参考[命令手册](https://docs.docker.com/engine/reference/commandline/cli/)。


## 在docker中运行一个web应用

刚才你又学习了一点关于docker客户端的用法，现在你可以开始处理一些更做要的工作了：启动更多的容器。到目前为止，你还没有启动任何一台有实际意义的容器，现在你可以运行一台我们预先准备的web应用。

我们提供的web应用镜像中，将运行一个 ` Python Flash ` 应用。 使用 ` docker run `：

```bash

$ docker run -d -P training/webapp python app.py 

```


我们来回顾一下该命令做了些什么。 你使用了两个标记 ` -d 和 -P `。 你已经学习了 ` -d ` 是告诉docker后台运行容器；而 ` -P `则是告诉docker映射所有容器内部需要使用的网络端口到本地计算机，这让我们可以访问容器中的web应用。

你指定使用镜像：` training/webapp `。 该镜像是一个创建好的镜像，容器气候东会运行一个简单的 ` Python Flash ` web应用程序。

最后， 你为容器指定了一个命令： ` python app.py `。 该命令会启动容器内的web应用。

> **注意**： 你可以在[命令手册](https://docs.docker.com/engine/reference/commandline/run/)和[docker run 手册](https://docs.docker.com/engine/reference/run/)中查看更多关于 ` docker run ` 的信息。


## 查看web容器

现在你可以使用命令 ` docker ps ` 查看刚才启动的容器。

```bash

$ docker ps -l

CONTAINER ID  IMAGE                   COMMAND       CREATED        STATUS        PORTS                    NAMES
bc533791f3f5  training/webapp:latest  python app.py 5 seconds ago  Up 2 seconds  0.0.0.0:49155->5000/tcp  nostalgic_morse

```


你可以在 ` docker -s ` 中指定标志 ` -l ` ， 这样docker会返回 **最后一个启动的容器**的详细信息。

> **注意**：默认情况下， ` docker ps ` 只会显示正在运行的容器信息。使用 ` docker ps -a ` 则会显示所有容器信息，包括停止了的。

你可以对比[ 第一个后台容器 ](https://docs.docker.com/engine/tutorials/dockerizing/)页面中的 ` PORT `列，会发现这里多了一个很重要的信息

```bash

PORTS
0.0.0.0:49155->5000/tcp

```


当我们使用 ` -P ` 标记时， ` docker run ` 命令会映射容器中使用的所有端口到本地计算机。

> **注意**：我们将[如何创建镜像](https://docs.docker.com/engine/tutorials/dockerimages/)中学习如何在镜像` 暴露(EXPOSE) `端口。 

在上面的案例中，docker将容器的5000端口(python flask的默认端口)映射到了本地计算机的49155端口上。

端口绑定是可配置的。在上一个案例中， ` -P ` 是 ` -p 5000 ` 的缩写， 将容器内部的5000端口映射到本机计算机的高端口号(临时端口范围(ephemeral port range)，通常为32768-61000)上。我们也可以使用 ` -p `指定绑定的端口。

> **注意**： ` -p local_port:container_port ` 前面是是本机计算机端口，后面是容器端口。
> 如果省略 local_port的话，则将使用随机高位端口。


```

$ docker run -d -p 80:5000 training/webapp python app.py 

```

这样，容器的5000端口将被映射到本机计算机的80端口上。你可能会问，那我们为什么不使用`1:1`对应的端口映射，而要映射到高端口号上？ `1:1`对应端口映射的局限在于**本地计算机端口只能被映射一次，不能同时被重复映射**。

试想一下你同时测试两个python应用，容器内部都使用5000端口。在不使用docker端口映射的情况下，在同一时间内你只能访问一个docker容器。

现在，你可以在浏览器上使用49155端口访问你的容器应用了。
![python_flask_webapp1](https://docs.docker.com/engine/tutorials/webapp1.png)
python应用成功运行了。


>**注意**：如果你在OSX，Windows或Linux上使用虚拟机， 那么你就不能使用localhost，而要使用虚拟机的IP地址。你可以使用命令 ` docker-machine ip your_vm_name ` 查询例如：

> ```

> $ docker-machine ip my-docker-vm
> 192.168.99.100

> ```

> 本例中，你可以在浏览器上访问 ` http://192.168.99.100:49155 ` 访问。
>
> **译者注： 我并没有找到 ` docker-machine ` 命令。
> 


## 快速查询端口映射
使用 ` docker ps ` 命令查询容器的端口映射信息有一点不方便。因此docker提供了一个有用的快捷方式： ` docker port `。 使用 ` docker port ` 命令时，我们需要指定**容器ID或名称**以及需要进行通讯的的**容器端口号**。

```

$ docker port nostalgic_morse 5000
0.0.0.0:49155

```

本例中，你查询容器的5000端口映射到了本机计算机哪个端口上。


## 查看web应用的日志

你也可以使用 ` docker logs ` 命令查看web引用发生了什么

```

$ docker logs -f nostalgic_morse

* Running on http://0.0.0.0:5000/
10.0.2.2 - - [23/May/2014 20:16:31] "GET / HTTP/1.1" 200 -
10.0.2.2 - - [23/May/2014 20:16:31] "GET /favicon.ico HTTP/1.1" 404 -

```


这次命令后面添加了一个新的标识 ` -f `。 和 ` tail -f ` 类似， `docker logs ` 会持续观察容器的标准输出。我们可以从这些日志中看到，Flask应用使用5000端口且其访问日志不断更新。


## 查看容器内应用的进程信息
除了查看容器日志之外，还可以使用 ` docker top ` 命令查看容器内的进程信息：

```

$ docker top nostalgic_morse

PID                 USER                COMMAND
854                 root                python app.py


```

这里我们看到，容器中只有一个进程。


## 检查容器内的应用信息

最后，我们将使用 ` docker inspect ` 命令获取docker容器初级信息。该命令将返回一个JSON文档，其中包含了指定容器配置和状态信息。

```

$ docker inspect nostalgic_morse

[{
    "ID": "bc533791f3f500b280a9626688bc79e342e3ea0d528efe3a86a51ecb28ea20",
    "Created": "2014-05-26T05:52:40.808952951Z",
    "Path": "python",
    "Args": [
       "app.py"
    ],
    "Config": {
       "Hostname": "bc533791f3f5",
       "Domainname": "",
       "User": "",
. . .

```


我们可以通过指定具体的元素查询信息，例如说，查询容器的IP地址：

```

$ docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' nostalgic_morse

172.17.0.5

```


## 停止一个运行的容器

我们已经看过web应用如何工作了，现在可以使用 ` docker stop container_name|container_id` 命令停止容器。

```

$ docker stop nostalgic_morse

nostalgic_morse

```


你可以使用 ` docker ps `命令查看容器是否已经被停止了。

```

$docker ps -l

```


## 启动一个停止的容器
在你停止了容器之后，你接到一个电话说另外一个程序员需要使用web应用。现在你有两个选择：创建一个新的容器；使用 ` docker start container_name|container_id` 重启刚才关闭的容器。

```

$ docker start nostalgic_morse

nostalgic_morse

```

使用 ` docker ps -l ` 或访问浏览器查看容器和应用是否启动成功。

> **注意**： 也可以使用 ` docker restart container_name|container_id` 命令停止并启动一个容器。


## 删除一个容器

当一个容器不再被使用的时候，可以通过 ` docker rm container_name|container_id` 命令删除。

```

$ docker rm nostalgic_morse

Error: Impossible to remove a running container, please stop it first or use -f
2014/05/24 08:12:56 Error: failed to remove one or more containers

```


我们不能删除一个正在运行的容器，这样做的原因是防止删除时的误操作。因此在删除容器之前，你需要停止他。

```bash

$ docker stop nostalgic_morse

nostalgic_morse

$ docker rm nostalgic_morse

nostalgic_morse

```

现在你的容器已经被删掉了

> **注意**： 永远记住 **删除** 是一个容器操作的最后操作。



