# 创建自定义镜像

> 2016-09-28  octowhale@github

[原文](https://docs.docker.com/engine/tutorials/dockerimages/)


docker images是容器的技术。每次使用 ` docker run ` 命令时都需要指定使用的镜像。之前的章节中使用的镜像都已经存在，例如 ` ubuntu 和 training/webapp `。

你也学习了如何在docker hub搜索并下载镜像到本地计算机。如果本机计算机上没有该镜像，则会从远程仓库中下载，默认仓库为[ Docker Hub Registry](https://hub.docker.com/)

在本节中，你将学习更多关于docker镜像的只是，包括：
+ 管理和使用本机镜像
+ 创建基础镜像
+ 上传镜像到Docker Hub Registry


## 查看本机镜像列表

使用 ` docker images ` 可以列出本地计算机上的所有镜像。


```bash

$ docker images

REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
ubuntu              14.04               1d073211c498        3 days ago          187.9 MB
busybox             latest              2c5ac3f849df        5 days ago          1.113 MB
training/webapp     latest              54bb4e8718e8        5 months ago        348.7 MB


```


你可以找到之前使用过的镜像。这些镜像都是之前在启动容器时从DockerHub上下载的。当你查看镜像列表时，需要注意三个重要信息：
+ 镜像所在的仓库，例如ubuntu
+ 镜像的标签(TAG)，例如14.04
+ 镜像的` IMAGE ID `

> **Tips**： 你可以使用第三方的[图形工具 dockviz tool](https://github.com/justone/dockviz)或[镜像网站](https://imagelayers.io/)查看镜像数据。


仓库中可能包含了一个镜像的不同衍生版，我们使用的ubuntu包含的衍生版有10.04,12.04,12.10,13.04,13.10和14.04。每个衍生版都有一个独立的tag。你可以通过tag指定需要使用的衍生版，例如：

```bash

ubuntu:14.04

```


因此当你的容器运行一个指定tag的镜像，命令如下：

```bash

$ docker run -t -i ubuntu:14.04 /bin/bash

```


如果你想使用12.04，则为：

```bash

$ docker run -t -i ubuntu:12.04 /bin/bash

```


如果命令中不指定tag，例如 ` ubuntu `， 那么docker默认使用 ` ubuntu:latest ` 镜像。

> **Tips**： 使用镜像时应该总是指定tag。这样你将明确知道正在使用的是什么衍生版，这将对排错很有帮助。


## 获得一个新镜像

如果你使用的镜像本机不存在，那么docker会自动下载。不过这样会增加部分启动时间。如果你需要预先下载一个镜像，可以使用 ` docker pull ` 命令。 加入你准备下载 ` centos ` 镜像：

```bash 

$ docker pull centos

Using default tag: latest
latest: Pulling from library/centos
f1b10cd84249: Pull complete
c852f6d61e65: Pull complete
7322fbe74aa5: Pull complete
Digest: sha256:90305c9112250c7e3746425477f1c4ef112b03b4abe78c612e092037bfecc3b7
Status: Downloaded newer image for centos:latest

```


你可以看到镜像的每个层(layer)。而且现在启动容器时不用在等待下载镜像了。

```bash

$ docker run -t -i centos /bin/bash
bash-4.1#

```



## 查找镜像
docker的其中一个特点在于人们基于不同目的创建了各种各样的镜像，并且很多人将他们的镜像上传到[Docker Hub](https://hub.docker.com/)。你可以访问Docker Hub搜索这些镜像。
![Docker_hub_search.png](https://docs.docker.com/engine/tutorials/search.png)

你可以在命令行界面使用 ` docker search ` 命令查找镜像。加入你想查找一个安装了Ruby和Sinatra的镜像，可以使用 ` docker search sinatra ` 命令查询包含了sinatra的所有镜像。

```bash

$ docker search sinatra
NAME                                   DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
training/sinatra                       Sinatra training image                          0                    [OK]
marceldegraaf/sinatra                  Sinatra test app                                0
mattwarren/docker-sinatra-demo                                                         0                    [OK]
luisbebop/docker-sinatra-hello-world                                                   0                    [OK]
bmorearty/handson-sinatra              handson-ruby + Sinatra for Hands on with D...   0
subwiz/sinatra                                                                         0
bmorearty/sinatra                                                                      0
. . .

```


你可以看到命令返回了很多包含了sinatra关键字的镜像；列表中包含了镜像名称，描述和stars(受欢迎程度，如果一个用户喜欢该镜像则可以为镜像加star)；以及官方版本和自动创建状态。[官方仓库](https://docs.docker.com/docker-hub/official_repos)由docker.Inc管理一系列的docker仓库。 [自动创建](https://docs.docker.com/engine/tutorials/dockerrepos/#automated-builds)允许你去验证镜像的源和内容。

目前你见过了两种镜像仓库：
+ 一种为 ` ubuntu ` 这类官方镜像，被称为基础镜像或者根镜像。这些基础镜像是有Docker Inc提交与创建、验证、支持。基础镜像可以使用一个单词作为名称。
+ 另一种为 ` training/sinatra ` 这类用户镜像。这类镜像是docker用户自己创建和维护，你可以通过前缀辨别镜像的维护者，例如这里的 ` training `。


## 拉取镜像

这里我们使用 `training/sinatra`镜像。 你可以使用 ` docker pull ` 命令下载镜像

```bash

$ docker pull training/sinatra 

```


下载完成后，就可以在容器中应用该镜像了

```bash

$ docker run -t -i training/sinatra /bin/bash

root@a8cb6ce02d85:/#

```


## 创建你自己的镜像

我们将基于 ` training/sinatra ` 创建新镜像。
+ 修改容器内容，并时候commit提交结果创建新镜像
+ 使用dockerfile指定新镜像的创建指令

### 更新容器与使用commit创建镜像

创建镜像之前需要启动一个容器

```

$ docker run -t -i training/sinatra /bin/bash

root@0b2616b0e5a8:/#

```


> **注意**： 记住所启动的容器ID , 0b2616b0e5a8 , 之后会使用。 如果你能识别容器，也可以在退出容器后使用 ` docker ps -a ` 查看容器信息。

在容器内更新Ruby
使用gem安装`json`

```

root@0b2616b0e5a8:/# apt-get install -y ruby2.0-dev
root@0b2616b0e5a8:/# gem2.0 install json

```


使用 ` Ctrol+D ` 或 ` exit ` 命令退出容器

容器已经被改变了，使用 ` docker commit ` 提交容器的一个副本为镜像

```

$ docker commit -m "Added json gem" -a "Kate Smith" \
0b2616b0e5a8 ouruser/sinatra:v2

4f177bd27a9ff0f6dc2a830403925b5360bfe0b93d476f7fc3231110e7f71b1c


```


` -m ` 表示提交的备注信息；` -a ` 表示镜像测作者或维护人员。
使用ID为0b2616b0e5a8的容器；创建的镜像为 ` ouruser/sinatra:v2 `，包含了仓库与TAG

这里镜像名由以下几部分组成：
+ 新的用户 ` ouruser ` 
+ 沿用的的镜像名 ` sinatra `
+ 新的tag ` v2 `

使用 ` docker images ` 查看镜像是否已经被创建了

```

$ docker images

REPOSITORY          TAG     IMAGE ID       CREATED       SIZE
training/sinatra    latest  5bc342fa0b91   10 hours ago  446.7 MB
ouruser/sinatra     v2      3c59e02ddd1a   10 hours ago  446.7 MB
ouruser/sinatra     latest  5db5f8471261   10 hours ago  446.7 MB

```


使用新镜像创建容器

```

$ docker run -t -i ouruser/sinatra:v2 /bin/bash

root@78e82f680994:/#

```


> 更多关于commit创建镜像的信息，可以参考[C02S03.2 使用commit提交容器创建镜像](../chapter02/03-build-your-own-image-with-commit.md)


### 使用dockerfile创建镜像

使用 ` docker commit ` 创建镜像很简单，但是很麻烦，不利于共享。 然而你可以使用 ` docker build ` 创建镜像。

` Dockerfile ` ： 包含了创建镜像时需要的基础镜像与一些列创建指令。

首先，创建一个目录和 ` Dockerfile ` 

```bash

$ mkdir sinatra

$ cd sinatra

$ touch Dockerfile

```


dockerfile内的每一条指令都会创建一个镜像layer。
创建一个简单的dockerfile来创建你的 Sinatra 镜像。

```bash

# This is a comment
FROM ubuntu:14.04
MAINTAINER Kate Smith <ksmith@example.com>
RUN apt-get update && apt-get install -y ruby ruby-dev
RUN gem install sinatra

```


检查Dockerfile，确保每行指令都以**大写字母的声明关键字**开头

```

INSTRUCTION statement

```


> **注意**：以 ` # ` 开头的行为注释行。

+ ` FROM ` 告诉docker基于哪个镜像。本例中，使用的是 ` ubuntu:14.04 `。 
+ ` MAINTAINER ` 表示谁在维护这个镜像。
+ ` RUN ` 告诉docker需要在镜像中执行什么命令。本例中是更新APT缓存和安装ruby等

> **注意**：` FROM ` 行必须位于dockerfile的第一行，否则会报错。

使用创建的 ` Dockerfile ` 和 ` docker build ` 命令创建镜像

```

$ docker build -t ouruser/sinatra:v2 .

Sending build context to Docker daemon 2.048 kB
Sending build context to Docker daemon
Step 1 : FROM ubuntu:14.04
 ---> e54ca5efa2e9
Step 2 : MAINTAINER Kate Smith <ksmith@example.com>
 ---> Using cache
 ---> 851baf55332b
Step 3 : RUN apt-get update && apt-get install -y ruby ruby-dev
 ---> Running in 3a2558904e9b
Selecting previously unselected package libasan0:amd64.
(Reading database ... 11518 files and directories currently installed.)
Preparing to unpack .../libasan0_4.8.2-19ubuntu1_amd64.deb ...
Unpacking libasan0:amd64 (4.8.2-19ubuntu1) ...
Selecting previously unselected package libatomic1:amd64.
Preparing to unpack .../libatomic1_4.8.2-19ubuntu1_amd64.deb ...
Unpacking libatomic1:amd64 (4.8.2-19ubuntu1) ...
Selecting previously unselected package libgmp10:amd64.
Preparing to unpack .../libgmp10_2%3a5.1.3+dfsg-1ubuntu1_amd64.deb ...
Unpacking libgmp10:amd64 (2:5.1.3+dfsg-1ubuntu1) ...
Selecting previously unselected package libisl10:amd64.
Preparing to unpack .../libisl10_0.12.2-1_amd64.deb ...
Unpacking libisl10:amd64 (0.12.2-1) ...
Selecting previously unselected package libcloog-isl4:amd64.
Preparing to unpack .../libcloog-isl4_0.18.2-1_amd64.deb ...
Unpacking libcloog-isl4:amd64 (0.18.2-1) ...
Selecting previously unselected package libgomp1:amd64.
Preparing to unpack .../libgomp1_4.8.2-19ubuntu1_amd64.deb ...
Unpacking libgomp1:amd64 (4.8.2-19ubuntu1) ...
Selecting previously unselected package libitm1:amd64.
Preparing to unpack .../libitm1_4.8.2-19ubuntu1_amd64.deb ...
Unpacking libitm1:amd64 (4.8.2-19ubuntu1) ...
Selecting previously unselected package libmpfr4:amd64.
Preparing to unpack .../libmpfr4_3.1.2-1_amd64.deb ...
Unpacking libmpfr4:amd64 (3.1.2-1) ...
Selecting previously unselected package libquadmath0:amd64.
Preparing to unpack .../libquadmath0_4.8.2-19ubuntu1_amd64.deb ...
Unpacking libquadmath0:amd64 (4.8.2-19ubuntu1) ...
Selecting previously unselected package libtsan0:amd64.
Preparing to unpack .../libtsan0_4.8.2-19ubuntu1_amd64.deb ...
Unpacking libtsan0:amd64 (4.8.2-19ubuntu1) ...
Selecting previously unselected package libyaml-0-2:amd64.
Preparing to unpack .../libyaml-0-2_0.1.4-3ubuntu3_amd64.deb ...
Unpacking libyaml-0-2:amd64 (0.1.4-3ubuntu3) ...
Selecting previously unselected package libmpc3:amd64.
Preparing to unpack .../libmpc3_1.0.1-1ubuntu1_amd64.deb ...
Unpacking libmpc3:amd64 (1.0.1-1ubuntu1) ...
Selecting previously unselected package openssl.
Preparing to unpack .../openssl_1.0.1f-1ubuntu2.4_amd64.deb ...
Unpacking openssl (1.0.1f-1ubuntu2.4) ...
Selecting previously unselected package ca-certificates.
Preparing to unpack .../ca-certificates_20130906ubuntu2_all.deb ...
Unpacking ca-certificates (20130906ubuntu2) ...
Selecting previously unselected package manpages.
Preparing to unpack .../manpages_3.54-1ubuntu1_all.deb ...
Unpacking manpages (3.54-1ubuntu1) ...
Selecting previously unselected package binutils.
Preparing to unpack .../binutils_2.24-5ubuntu3_amd64.deb ...
Unpacking binutils (2.24-5ubuntu3) ...
Selecting previously unselected package cpp-4.8.
Preparing to unpack .../cpp-4.8_4.8.2-19ubuntu1_amd64.deb ...
Unpacking cpp-4.8 (4.8.2-19ubuntu1) ...
Selecting previously unselected package cpp.
Preparing to unpack .../cpp_4%3a4.8.2-1ubuntu6_amd64.deb ...
Unpacking cpp (4:4.8.2-1ubuntu6) ...
Selecting previously unselected package libgcc-4.8-dev:amd64.
Preparing to unpack .../libgcc-4.8-dev_4.8.2-19ubuntu1_amd64.deb ...
Unpacking libgcc-4.8-dev:amd64 (4.8.2-19ubuntu1) ...
Selecting previously unselected package gcc-4.8.
Preparing to unpack .../gcc-4.8_4.8.2-19ubuntu1_amd64.deb ...
Unpacking gcc-4.8 (4.8.2-19ubuntu1) ...
Selecting previously unselected package gcc.
Preparing to unpack .../gcc_4%3a4.8.2-1ubuntu6_amd64.deb ...
Unpacking gcc (4:4.8.2-1ubuntu6) ...
Selecting previously unselected package libc-dev-bin.
Preparing to unpack .../libc-dev-bin_2.19-0ubuntu6_amd64.deb ...
Unpacking libc-dev-bin (2.19-0ubuntu6) ...
Selecting previously unselected package linux-libc-dev:amd64.
Preparing to unpack .../linux-libc-dev_3.13.0-30.55_amd64.deb ...
Unpacking linux-libc-dev:amd64 (3.13.0-30.55) ...
Selecting previously unselected package libc6-dev:amd64.
Preparing to unpack .../libc6-dev_2.19-0ubuntu6_amd64.deb ...
Unpacking libc6-dev:amd64 (2.19-0ubuntu6) ...
Selecting previously unselected package ruby.
Preparing to unpack .../ruby_1%3a1.9.3.4_all.deb ...
Unpacking ruby (1:1.9.3.4) ...
Selecting previously unselected package ruby1.9.1.
Preparing to unpack .../ruby1.9.1_1.9.3.484-2ubuntu1_amd64.deb ...
Unpacking ruby1.9.1 (1.9.3.484-2ubuntu1) ...
Selecting previously unselected package libruby1.9.1.
Preparing to unpack .../libruby1.9.1_1.9.3.484-2ubuntu1_amd64.deb ...
Unpacking libruby1.9.1 (1.9.3.484-2ubuntu1) ...
Selecting previously unselected package manpages-dev.
Preparing to unpack .../manpages-dev_3.54-1ubuntu1_all.deb ...
Unpacking manpages-dev (3.54-1ubuntu1) ...
Selecting previously unselected package ruby1.9.1-dev.
Preparing to unpack .../ruby1.9.1-dev_1.9.3.484-2ubuntu1_amd64.deb ...
Unpacking ruby1.9.1-dev (1.9.3.484-2ubuntu1) ...
Selecting previously unselected package ruby-dev.
Preparing to unpack .../ruby-dev_1%3a1.9.3.4_all.deb ...
Unpacking ruby-dev (1:1.9.3.4) ...
Setting up libasan0:amd64 (4.8.2-19ubuntu1) ...
Setting up libatomic1:amd64 (4.8.2-19ubuntu1) ...
Setting up libgmp10:amd64 (2:5.1.3+dfsg-1ubuntu1) ...
Setting up libisl10:amd64 (0.12.2-1) ...
Setting up libcloog-isl4:amd64 (0.18.2-1) ...
Setting up libgomp1:amd64 (4.8.2-19ubuntu1) ...
Setting up libitm1:amd64 (4.8.2-19ubuntu1) ...
Setting up libmpfr4:amd64 (3.1.2-1) ...
Setting up libquadmath0:amd64 (4.8.2-19ubuntu1) ...
Setting up libtsan0:amd64 (4.8.2-19ubuntu1) ...
Setting up libyaml-0-2:amd64 (0.1.4-3ubuntu3) ...
Setting up libmpc3:amd64 (1.0.1-1ubuntu1) ...
Setting up openssl (1.0.1f-1ubuntu2.4) ...
Setting up ca-certificates (20130906ubuntu2) ...
debconf: unable to initialize frontend: Dialog
debconf: (TERM is not set, so the dialog frontend is not usable.)
debconf: falling back to frontend: Readline
debconf: unable to initialize frontend: Readline
debconf: (This frontend requires a controlling tty.)
debconf: falling back to frontend: Teletype
Setting up manpages (3.54-1ubuntu1) ...
Setting up binutils (2.24-5ubuntu3) ...
Setting up cpp-4.8 (4.8.2-19ubuntu1) ...
Setting up cpp (4:4.8.2-1ubuntu6) ...
Setting up libgcc-4.8-dev:amd64 (4.8.2-19ubuntu1) ...
Setting up gcc-4.8 (4.8.2-19ubuntu1) ...
Setting up gcc (4:4.8.2-1ubuntu6) ...
Setting up libc-dev-bin (2.19-0ubuntu6) ...
Setting up linux-libc-dev:amd64 (3.13.0-30.55) ...
Setting up libc6-dev:amd64 (2.19-0ubuntu6) ...
Setting up manpages-dev (3.54-1ubuntu1) ...
Setting up libruby1.9.1 (1.9.3.484-2ubuntu1) ...
Setting up ruby1.9.1-dev (1.9.3.484-2ubuntu1) ...
Setting up ruby-dev (1:1.9.3.4) ...
Setting up ruby (1:1.9.3.4) ...
Setting up ruby1.9.1 (1.9.3.484-2ubuntu1) ...
Processing triggers for libc-bin (2.19-0ubuntu6) ...
Processing triggers for ca-certificates (20130906ubuntu2) ...
Updating certificates in /etc/ssl/certs... 164 added, 0 removed; done.
Running hooks in /etc/ca-certificates/update.d....done.
 ---> c55c31703134
Removing intermediate container 3a2558904e9b
Step 4 : RUN gem install sinatra
 ---> Running in 6b81cb6313e5
unable to convert "\xC3" to UTF-8 in conversion from ASCII-8BIT to UTF-8 to US-ASCII for README.rdoc, skipping
unable to convert "\xC3" to UTF-8 in conversion from ASCII-8BIT to UTF-8 to US-ASCII for README.rdoc, skipping
Successfully installed rack-1.5.2
Successfully installed tilt-1.4.1
Successfully installed rack-protection-1.5.3
Successfully installed sinatra-1.4.5
4 gems installed
Installing ri documentation for rack-1.5.2...
Installing ri documentation for tilt-1.4.1...
Installing ri documentation for rack-protection-1.5.3...
Installing ri documentation for sinatra-1.4.5...
Installing RDoc documentation for rack-1.5.2...
Installing RDoc documentation for tilt-1.4.1...
Installing RDoc documentation for rack-protection-1.5.3...
Installing RDoc documentation for sinatra-1.4.5...
 ---> 97feabe5d2ed
Removing intermediate container 6b81cb6313e5
Successfully built 97feabe5d2ed

```


上面执行的 ` docker build ` 命令中使用了 `-t ` 标识， 指定新创建的镜像就属于 ` ousruser `用户，仓库名为 ` sinatra `， tag为 ` v2 `。
> -t, --tag value               Name and optionally a tag in the 'name:tag' format (default [])

> **注意**： 你也可以指定Dockerfile的路径 
> ` -f, --file string             Name of the Dockerfile (Default is 'PATH/Dockerfile') `

` docker build ` 具体是怎么工作的呢？
首先， 向docker daemon上传本地环境信息，基本上就是用来执行building的目录信息。 docker daemon实际上是在本地环境内创建镜像的。

其次， 你可以看到 ` Dockerfile ` 中的每一条指令都依次执行。值得注意的是**每一步(每一条指令)都创建了一个容器，并在容器内执行指令，最后commit容器的变化**，就像你之前使用 ` docker commit ` 一样。 当所有指令都被执行之后，生成最终镜像。这里为 `97feabe5d2ed` ， 并且被标记为 ` ouruser/sinatra:v2 `。

最后， 所有中间容器会被清除。

> **注意**： 一个镜像不能超过127层。该限制主要是为了鼓励优化镜像。

使用刚才创建的镜像启动容器：

```

$ docker run -t -i ouruser/sinatra:v2 /bin/bash

root@8196968dac35:/#

```


> **注意**： 这只是一个简单的` docker build ` 案例，省略了很多关键字。 在以后的章节中会学习更多的创建指令。 访问[Best Practices guide](https://docs.docker.com/engine/userguide/eng-image/dockerfile_best-practices/)学习更多Dockerfile的指令。

> 更多关于dockerfile创建镜像的信息，可以参考[C02S03.1 使用dockerfile创建镜像](../chapter02/03-build-your-own-image-with-dockerfile.md)

## 为镜像设置tag
使用 ` docker tag ` 为已存在的镜像设置新的TAG。

```

$ docker tag 5db5f8471261 ouruser/sinatra:devel

```

这里 ` docker tag ` 命令使用了镜像ID、用户名、仓库名和一个新的TAG

> **注意**： 镜像ID部分可以换成镜像的其他唯一标识符，例如 `username/repo:tag` 
> `$ docker tag ouruser/sinatra:v2 ouruser/sinatra:devel`
 

现在使用 ` docker images ` 命令可以查看到新的tag镜像。

```

$ docker images ouruser/sinatra

REPOSITORY          TAG     IMAGE ID      CREATED        SIZE
ouruser/sinatra     latest  5db5f8471261  11 hours ago   446.7 MB
ouruser/sinatra     devel   5db5f8471261  11 hours ago   446.7 MB
ouruser/sinatra     v2      5db5f8471261  11 hours ago   446.7 MB

```

 
## 镜像校验符digests

docker镜像有一个属性叫`digests`，是一种内容寻址标识符。只要镜像没有被改变，那么校验值就不会改变，是可以被推算出来的。
使用 ` --digests ` 标志可以可以查校验值。

```

$ docker images --digests | head

REPOSITORY        TAG      DIGEST                                                                     IMAGE ID      CREATED       SIZE
ouruser/sinatra   latest   sha256:cbbf2f9a99b47fc460d422812b6a5adff7dfee951d8fa2e4a98caa0382cfbdbf    5db5f8471261  11 hours ago  446.7 MB

```


> **注意**： 只有被push到docker hub(或者其他registry仓库)上的镜像才拥有一串有意义的校验值。否则显示 <none>。、

> ```

> REPOSITORY                    TAG                          DIGEST               IMAGE ID            CREATED             SIZE
> octowhale/centos7             whalesay-change-cmd-v4       <none>               216b5d379710        3 days ago          474.6 MB

> ```


由于digest具有唯一性，你也可以**使用digest进行镜像操作**，包括 ` create, run, rmi ` 等命令。也可以用在 `Dockerfile中的FROM中`。


## push镜像到Docker Hub

镜像创建完成后你可以使用 ` docker push ` 命令将它们推送到 Docker Hub中分享给其他用户。

```

$ docker push ouruser/sinatra

The push refers to a repository [ouruser/sinatra] (len: 1)
Sending image list
Pushing repository ouruser/sinatra (3 tags)
. . .


```



## 从本地计算机中删除镜像

你可以使用 ` docker rmi ` 命令删除本地计算机上的[相似镜像](../chapter03/02-run-a-simple-application.md)

删除 ` training/sinatra `

```

$ docker rmi training/sinatra

Untagged: training/sinatra:latest
Deleted: 5bc342fa0b91cabf65246837015197eecfa24b2213ed6a51a8974ae250fedd8d
Deleted: ed0fffdcdae5eb2c3a55549857a8be7fc8bc4241fb19ad714364cbfd7a56b22f
Deleted: 5c58979d73ae448df5af1d8142436d81116187a7633082650549c52c3a2418f0

```


> **注意**： 删除镜像前，确保没有正在运行的容器在该镜像。
> **注意**： 如果多个镜像具有相同的镜像ID，那么在删除最后一个TAG之前，镜像都不会被真正删除而是解除TAG绑定。


