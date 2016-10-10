# 管理容器的数据

> 2016-09-30  octowhale@github

[ 原文 ](https://docs.docker.com/engine/tutorials/dockervolumes/)

之前我们学习了[docker的基本操作]、[管理docker镜像]以及[管理容器网络]。本节中将学习如何管理docker容器的内部数据以及docker容器之间的数据。

docker Engine提供了两种方式管理容器的数据：
+ 数据卷
+ 数据卷容器


## 数据卷

**数据卷**是一个或多个容器绕过[Union File System](https://docs.docker.com/engine/reference/glossary/#union-file-system)而指定的一个特殊目录。数据卷为**数据持久化和共享**提供了一些有用的功能。

+ 卷在创建容器的时候被初始化。如果容器的基础镜像在卷的挂载点目录中有数据，那么这些数据会被复制到初始化后的卷上。(注意，这一条并不适用于[挂载主机目录作为数据卷](https://docs.docker.com/engine/tutorials/dockervolumes/#mount-a-host-directory-as-a-data-volume)
+ 数据卷可以在容器之间共享和复用
+ 数据卷的修改实时生效
+ 更新镜像时不会影响数据卷
+ 即使容器被删除数据卷也会存在

数据卷是为了在容器生命周期内，数据的持久化、独立性而设计。因此docker永远不会在删除容器的时候删除数据卷。也不会将数据卷放入"回收站"而不让容器重新使用。


### 添加数据卷

在使用 ` docker creaste ` 或 ` docker run ` 命令的时候，使用 ` -v ` 标识可以为容器添加数据卷。 多次使用 ` -v ` 可以为容器添加多个数据卷。

为web应用容器添加一个数据卷， 位置为 /webapp ：

```

$ docker run -d -P --name web -v /webapp training/webapp python app.py

```


> **注意**： 使用 ` Dockerfile ` 中使用 ` VOLUME ` 指令可以为新建容器添加一个或多个卷。


### 查找卷的位置

使用 ` docker inspect ` 命令查看卷在容器中的位置

```

$ docker inspect web

```


查询结果中包含了容器关于数据卷的信息，看起来有点像这样：

```

...
"Mounts": [
    {
        "Name": "fac362...80535",
        "Source": "/var/lib/docker/volumes/fac362...80535/_data",
        "Destination": "/webapp",
        "Driver": "local",
        "Mode": "",
        "RW": true,
        "Propagation": ""
    }
]
...

```


需要注意的是： 
+ ` Source ` 是卷在本机的位置。
+ ` Destination ` 是卷在容器内的位置。
+ ` RW ` 表示卷在容器内的读写状态。


### 挂载主机目录作为数据卷

使用 ` -v ` 标识可以指定运行Docker Engine的主机上的一个目录作为数据卷挂载到容器中。

```

$ docker run -d -P --name web -v /src/webapp:/opt/webapp training/webapp python app.py

```


该命令将本地目录 ` /src/webapp ` 挂载到容器内 ` /opt/webapp ` 。 如果容器使用的镜像中 `/opt/webapp ` 目录内有数据，那么这些数据会 ` /src/webapp ` 的数据覆盖， 但不会被删除。 当移除/src/webapp挂在后，原镜像中的/opt/webapp的数据可以被访问。 这种情况和普通操作系统的 ` mount ` 命令一样。


容器目录必须使用绝对路径，例如 ` /opt/webapp ` ， 但主机目录可以使用绝对路径，也可以一个 `name` 的卷名。 如果主机目录使用绝对路径，则docker会将本机目录挂在到容器中； 如果本机目录使用卷名 ` name `， 则docker会为容器创建一个数据卷，卷名为 ` name `。

```

$ docker run -d -P --name web -v webapp:/opt/webapp training/webapp python app.py
...
"Mounts": [
    {
        "Name": "webapp",
        "Source": "/var/lib/docker/volumes/webapp/_data",
        "Destination": "/opt/webapp",
        "Driver": "local",
        "Mode": "z",
        "RW": true,
        "Propagation": "rprivate"
    }
],
...

```


+ 卷名` name `必须以**数字或者字母开头，其他部分可以包含数字、字母、下划线、豆点和破折号 ( "[a-zA-Z0-9][a-zA-Z0-9_.-]" )` 。
+ 绝对路径必须以 ` / `开头。

例如，使用 /foo 或 foo 作为本地目录的值。 /foo创建一个挂载点， foo会为容器创建一个名为 foo 的数据卷。


如果在windows或者OSX上运行docker machine，那么 docker engine 被限制访问OSX或windows文件系统。docker machine共享了 /Users （OSX） 或 C:\Users （windows）目录。 
例如，你可以在OSX或windows中挂在文件或目录：

```

docker run -v /Users/<path>:/<container path> ...
docker run -v c:\<path>:/c:\<container path> ...


```


All other paths come from your virtual machine’s filesystem, so if you want to make some other host folder available for sharing, you need to do additional work. In the case of VirtualBox you need to make the host folder available as a shared folder in VirtualBox. Then, you can mount it using the Docker -v flag.


为容器挂载主机目录对于测试来说很方便。例如，挂载一个源文件到容器中，在本地修改修改后可以实时在容器中观察修改的结果。

如果容器中的挂载点不存在，启动容器后该挂载点会被自动创建。

docker卷默认可读写模式，如果需要挂载为**只读**，需要在容器挂载点后添加 **:ro** ：

```

$ docker run -d -P --name web -v /src/webapp:/opt/webapp:ro training/webapp python app.py

```


这在容器内的/opt/webapp就是只读方式了。

由于[ mount功能限制 ](http://lists.linuxfoundation.org/pipermail/containers/2015-April/035788.html), 在本地计算机内移动目录会给予容器访问本地计算机文件系统的权限。 This requires a malicious user with access to host and its mounted directory.

> **注意**： 本地目录依赖于本地计算机系统。因为dockerfile创建尽享是可移植的，而本地目录在不同文件系统上有所得不同，因此**不能在 ` Dockerfile ` 中使用mount指令**。


### 挂载一个共享存储卷作为数据卷

某些docker[ 卷插件 ](https://docs.docker.com/engine/extend/plugins_volume/)支持挂载共享存储(例如，iSCSI, NFS, FC) 到容器中，

优点是这些共享卷不依赖于本地系统。意味着只要安装了这些插件，任意容器都可以使用这些共享卷。

通过` docker run `指定 ` 使用这些卷驱动器(卷分区)其中一种方式。 通过命名方式创建这些卷驱动器，而非目录路径。

下面这条命令将创建一个名为 ` my-named-volume ` 的卷， 使用 ` flocker ` 卷驱动， 并且绑定在容器的 ` /opt/webapp `：

```

$ docker run -d -P \
  --volume-driver=flocker \
  -v my-named-volume:/opt/webapp \
  --name web training/webapp python app.py


```


你也可以先使用 ` docker volume create ` 命令预先创建卷，随后再将卷绑定在容器上：

```

$ docker volume create -d flocker -o size=20GB my-named-volume

$ docker run -d -P \
  -v my-named-volume:/opt/webapp \
  --name web training/webapp python app.py

```


更多关于卷插件的信息，访问[这里](https://docs.docker.com/engine/extend/legacy_plugins/)


### 卷的标签
标签系统，例如SELinux，要求挂在到容器的目录使用一个适当的标签。如果不使用，安全系统可能阻止容器中的进程使用这些目录。默认情况下， docker不会改变操作系统(OS)设置的标签。

如果在容器环境中改变标签，有两种后缀可以选择 ` :z ` 和 ` :Z `。 这两个标签告诉docker需要重新为共享卷中的对象设置标签。 
+ ` z ` 选项告诉docker可以在两个容器之间共享该卷。因此，如果docker在某个目录上使用了共享目录标签，那么共享卷标签允许所有容器读写该目录。
+ ` Z ` 选项告诉容器为目录设置一个私有非共享标签， 只有当前容器才能使用私有卷。


### 挂在一个主机文件作为数据卷

除了目录之外， ` -v ` 标识还可以用于挂载本地文件到容器中：

```

$ docker run --rm -it -v ~/.bash_history:/root/.bash_history ubuntu /bin/bash

```


该命令可以在宿主机和容器之间共享bash history。

> **注意**： 由于很多编辑文件的工具会导致文件的inode改变，其中包括 ` vi 和 sed --in-place `。  从docker v1.1.0开始，这将出发一些错误，类似 " *sed: cannot rename ./sedKdJ9Dy: Device or resource busy* "。 为了保证修改挂在文件的需求，最简单的方式就是挂在文件所在的目录。


## 创建于挂载数据卷容器

如果你有持久数据需要在多个容器中共享， 或在临时容器中使用这些数据。 最好的方式是创建一个数据卷容器并命名，and then to mount the data from it.

首先创建一个指定名称的数据卷容器及指定共享卷。当该容器不运行的时候，他重新使用了 ` training/webapp ` 镜像， 因此所有容器都在使用相同的layers和存储空间。

```

$ docker create -v /dbdata --name dbstore training/postgres /bin/true

```


你可以使用 ` --volumes-from ` 标识挂载 ` /dbdata ` 卷到其他容器：

```

$ docker run -d --volumes-from dbstore --name db1 training/postgres 
$ docker run -d --volumes-from dbstore --name db2 training/postgres

```


在本例中， 如果 ` postgres ` 镜像中包含了 ` /dbdata ` 目录， 将会在挂载 ` dbstore ` 容器的卷， 并隐藏 ` /dbdata `目录中的文件。这导致只有 ` dbstore ` 容器中的文件能被看到。

你同时可以使用多个 ` --volumes-from ` 参数从不同的容器中联合数据卷。 更多关于 ` --volumes-from ` 的信息， 参考[ Mount volumes from container ](https://docs.docker.com/engine/reference/commandline/run/#mount-volumes-from-container-volumes-from)的 ` run ` 命令帮助文档

你也可以级联挂载数据卷。db1或db2容器从从dbstore上挂载数据卷，其他容器再从db1或db2上挂在：

```

$ docker run -d --name db3 --volumes-from db1 training/postgres

```


即使删除了挂在数据卷的容器，包括头容器 ` dbstore ` 或者二级容器 ` db1 , db2 `， 数据卷也不会被删除。 如果确实需要从磁盘上删除数据卷，你需要在删除最后一个挂载数据卷的容器时使用 ` docker rm -v `命令。 该机制允许你在容器之间进行数据升级或迁移。

> **注意**：在不使用 ` -v ` 删除容器时，docker不会提醒你删除容器数据卷。如果不使用 ` -v ` 删除容器后，容器的数据卷成为 "dangling" 卷，这些数据卷不再关联到任何容器。 你可以使用 ` docker volume ls -f dangling=true ` 命令查询所有dangling卷， 使用 ` docker volume rm <volume name> ` 删除不再使用的卷。
> ` docker volume ls ` 默认查询所有卷。 ` docker volume ls -f dangling=false ` 查询当前关联到容器的卷。


## 备份、恢复和迁移数据卷

卷还可以与数据的备份、恢复以及迁移。 使用 ` --volumes-from ` 创建一个新容器并挂载数据卷：

```

$ docker run --rm --volumes-from dbstore -v $(pwd):/backup ubuntu tar cvf /backup/backup.tar /dbdata

```


命令创建了一个全新的容器并挂载了dbstore容器的数据卷； 然后挂载了当前目录为容器的/backup目录； 最后执行了一个 ` tar ` 命令备份 ` /dbdata ` 目录(即 dbdata的数据卷)，并保存为 ` /backup/backup.tar ` 。 当命令执行完成后， 这个容器自动停止并删除， 而我们会得到 **dbdata的数据卷** 的备份，

你可以将备份数据恢复到该容器或者其他任何容器。

另外创建一个容器，

```

$ docker run -v /dbdata --name dbstore2 ubuntu /bin/bash

```


并解压备份文件到新容器的数据卷中

```

$ docker run --rm --volumes-from dbstore2 -v $(pwd):/backup ubuntu bash -c "cd /dbdata && tar zvf /backup/bakcup.tar --strip 1 "

```


你可以通过熟悉的方式，使用上面的技巧自动完成数据的备份、迁移和恢复。


## 删除数据卷

在容器被删除之后，docker数据卷不会被删除。 docker中可以创建命名卷或匿名卷。 在容器外，可以明确指定使用命名卷，例如， ` awesome:/bar ` 。 匿名卷不能被明确指定。 在容器被删除的同时， 你可以通知docker engine将匿名卷也删除了。 使用 ` --rm ` 选项可以完成此操作，例如：

```

$ docker run --rm -v /foo -v awesome:/bar busybox top

```


该命令创建了一个 ` 匿名卷/foo `， 当容器被删除时， 匿名卷`/foo`也会被删除。但是 ` 命名卷awssome ` 不会。


## 关于使用共享卷的重要建议

多个容器可以共享使用一个或多个数据卷。然而多个容器向同一个数据卷写入数据时可能会触发数据冲突，确保你的应用在设计的时候解决了共享数据卷的写入问题。

数据卷可以被docker宿主机直接访问，这意味着你可以直接使用linux工具读写他们。如果你的容器或应用没有处理机制，那么大多数情况下直接访问数据卷会造成数据冲突。
