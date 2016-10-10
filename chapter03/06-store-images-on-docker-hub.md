# 管理Docker Hub中的容器

> 2016-09-30  octowhale@github

[ 原文 ](https://docs.docker.com/engine/tutorials/dockerrepos/)


目前为止，你已经学习了如何用命令行工具启动的docker；学习了如何pull镜像；学习了如何create镜像。

接下来，你会学习如何使用 Docker Hub 来简化和加强docker工作流程。

The Docker Hub is a public registry maintained by Docker, Inc. It contains images you can download and use to build containers. It also provides authentication, work group structure, workflow tools like webhooks and build triggers, and privacy tools like private repositories for storing images you don’t want to share publicly.

Docker Hub是由Docker.Inc维护的公共仓库。提供了可用的公共镜像。也提供了认证、工作组、工作流程工具(如webhooks和构建触发器)以及私有化工具(如私有仓库)。


## docker command and Docker Hub

docker本身提供了可以访问 Docker Hub服务的命令，例如 ` docker search , pull , login , push `。 本节将为你揭露这些这些工具是如何工作的。


### 创建和登录账户

使用CLI命令之前，需要到 Docker Hub上创建用户。

在命令行界面登录账户：

```

$ docker login 

```


login 命令会将你的ID和密码保存在 ` $HOME/.docker/config.json ` 中。 windows中， `cmd` 会保存在 ` %HOME%\.docker\config.json ` ; `Powershell`会保存在 `$env:Home\.docker\config.json`。

一旦你在命令行通过验证，你就可以使用 ` commit 和 push ` 命令和你的docker hub仓库进行交互。

## 搜索镜像

通过命令行搜索docker hub上的镜像。结果将返回镜像名称，描述等信息。

```

$ docker search centos

NAME           DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
centos         The official build of CentOS                    1223      [OK]
tianon/centos  CentOS 5 and 6, created using rinse instea...   33
...

```


如 ` centos ` 这类单字镜像为docker官方管理和维护的。 如 ` tianao/centos ` 这类 ` user/repo ` 镜像是有docker用户管理并维护。

一旦确认了需要使用的镜像名，可以使用  ` docker pull ` 拉取镜像：

```

$ docker pull centos

Using default tag: latest
latest: Pulling from library/centos
f1b10cd84249: Pull complete
c852f6d61e65: Pull complete
7322fbe74aa5: Pull complete
Digest: sha256:90305c9112250c7e3746425477f1c4ef112b03b4abe78c612e092037bfecc3b7
Status: Downloaded newer image for centos:latest

```


### 镜像的版本或latest

` docker pull centos ` 实际上是  ` docker pull centos:latest `。

如果不指定镜像的TAG，那么latest为TAG的默认值。如果需要下载指定版本的镜像，需要明确指定。例如使用 `centos5` 可以使用命令 ` docker pull centos:centos5 `。

可以登录docker hub页面，查看镜像的所有可用tag和版本。


## 丰富 Docker Hub 镜像

任何人都可以从docker hub上拉取镜像，但是如要你要推送镜像，则需要注册。


### 推送镜像到docker hub

你推送到docker的镜像，镜像名必须使用你的用户名作为仓库前缀。格式为 ` your_account/repo_name `。


```

$ docker push yourname/newimage

```



## docker hub提供的功能

接下来我们详细了解一下docker hub提供的功能， 访问[这里](https://docs.docker.com/docker-hub/)查看更多信息。

+ Private repositories
+ Organizations and teams
+ Automated Builds
+ Webhooks

### 私有仓库

[私有仓库](https://hub.docker.com/account/billing-plans/)中保存的镜像不会共享给公众。 

### 组织和团队

私有仓库的一个特性是可以共享给你所在的组织或团队中的其他成员。 这样团队中就可以共同维护私有仓库。 访问[ 这里 ](https://hub.docker.com/organizations/)


### 自动构建

自动构建可以通过更新GITHUB或Bitbucket直接在docker hub上自动构建和更新镜像。 通过使用commit hook，当你在github或bitbucket提交数据的时候，会触发镜像的构建和更新。

#### 设置一个自动构建

+ 创建docker hub账户并登录。
+ 在["linked accounts & services"](https://hub.docker.com/account/authorized-services/)页面联接github或bitbucket账户。
+ 从"create"下来菜单中选择"create automated build"
+ 选择一个包含dockerfile的github或bitbucket仓库
+ 选择一个想要创建镜像的分支（默认为 master 分支）
+ 为"自动构建"命名
+ 为构建指派一个可选docker tag
+ 指定dockerfile存放位置，默认为 / 。

当自动构建配置完成，随后会自动触发一个构建动作，数分钟后，你就可以在docker hub中找到你的自动构建仓库。自动构建会一直同步你的github或bitbucket仓库，直到你手动取消自动构建。

点击 ["Your Repositorie 页面"](https://registry.hub.docker.com/repos/)检查自动构建的输出和状态。Automated Builds are indicated by a check-mark icon next to the repository name. Within the repository details page, you may click on the “Build Details” tab to view the status and output of all builds triggered by the Docker Hub.

你可以取消和删除所创建自动构建。 **不能使用 ` docker push ` 命令将镜像推送到自动构建仓库。** 你只能通过commit代码到github或bitbucket仓库管理自动构建。

你在每个仓库建立多个自动构建，并通过指定不同的dockerfile或git分支。

#### 构建触发器

自动构建也可以通过docker hub的URL被触发。这样你就可以按需求重建一个自动构建。


### webhooks

webhooks附加在你的仓库上，当镜像被push到仓库时允许你出发一个事件。 当镜像被推送到仓库时，通过webhook你可以指定一个目标URL和JSON串传递给自动构建触发器。

