# 为 CentOS 安装 Docker CE

在安装 Docker CE 之前，确保你已经看了准备条件，之后在安装 docker

## 准备条件

### 安装 docker EE

如果你要安装企业版的 Docker（Docker EE），看[为 CentOS 安装 Docker EE](https://docs.docker.com/engine/installation/linux/docker-ee/centos/)

更多关于 Docker EE 的信息，看[Docker 企业版](https://www.docker.com/enterprise-edition/)


### 系统要求

安装 Dcoker CE, 需要使用 **64位的 CentOS 7**。

`centos-etras` 仓库必须为 `enabled`。该仓库默认为 `enabled`，如果你曾今关闭过，看这里[re-enabled](https://wiki.centos.org/AdditionalResources/Repositories)

### 卸载过时版本

老版本的 docker 为 `docker` 或 `docker-engine`。如果之前安装了，需要卸载他们以及关联的依赖

```bash
sudo yum remove docker docker-common docker-selinux docker-engine
```

在 `/var/lib/docker/` 下的东西，包括镜像、容器、卷和网络设备都会被保存下来。

现在Docker CE 安装包名为 `docker-ce`。

## 安装 Docker CE

有三种方法安装：
+ 使用 yum 仓库安装，推荐
+ 使用 rpm 包安装
+ 使用一键脚本安装


### 使用仓库安装

先安装 docker 仓库，之后就可以通过仓库安装或升级 docker 了。

#### 安装仓库

1. 安装依赖包。 `yum-utils` 提供 `yum-config-manager` 功能； `device-mapper-presistent-data` 和 `lvm2` 为 `devicemapper` 存储驱动的依赖包。

```bash
sudo yum install -y yum-utils device-mapper-presistent-data lvm2
```

2. 安装`稳定版`仓库。即使你使用 `edge` 或 `test` 仓库，也需要安装该 repo.

```bash
$sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```

3. **可选**: 启动 `edge` 按 `test` 仓库。这两个仓库包含在 `docker.repo`中，默认被禁用了。使用以下两个命令分别启用：

```bash
$ sudo yum-config-manager --enable docker-ce-edge

$ sudo yum-config-manager --enable docker-ce-test

```

可以使用 `yum-config-manager --disable` 警用 **edge or test** 仓库。再启用则使用上面的命令

```bash
$sudo yum-config-manager --disable docker-ce-edge

```

> 注意：从 Docker17.06 开始， stable relsease 同时包含 edge 和 test 仓库。

[关于 **stable** 和 **edge** ](https://docs.docker.com/engine/installation/)

#### 安装 Docker CE

1. 更新 `yum` 包索引

```bash
$ sudo yum makecache fast
```

如果这是你添加 docker 仓库后第一次更新包索引，系统会提示接受 GPG key。确认 fingerprint 无误，并接受。
```
060A 61C5 1B55 8A7F 742B 77AA C52F EB6B 621E 9F35
```

2. 安装最新版本的 Docker CE, 或者使用之后的步骤安装指定版本

```bash
$ sudo yum install docker-ce
```

> 警告： 如果启用了多个 Docker 仓库， 安装或更新时没有指定版本号会自动安装最高可用版本，这可能不符合你的预期。

Docker 安装完成后不会自动启动。 `docker` 用户组会被自动创建，但是不会添加用户到该组。

3. 在生产环境，可能需要安装指定版本的 Docker CE 而非最新版。 列出所有可用版本号。 该案例使用 `sort -r` 命令对结果按照版本号进行了降序排列。

>注意：下面的 `yum list` 只显示了二进制包。如果要显示源码包，省略包名中的 `.x86_64`

```bash
$ yum list docker-ce.x86_64 --showuplicates | sort -r
docker-ce.x86_64  17.06.0.el7                               docker-ce-stable
```

安装指定版本只需要在包名后面跟上版本号，使用 `-` 链接。

```bash
$ sudo yum install docker-ce-<VERSION>

```

4. 启动 Docker

```bash
$ sudo systemctl start docker
```

5. 验证 `docker` 被正确安装，启动 `hello-world` 镜像。

```bash
$ sudo docker run hello-world
```

Docker CE 已经被安装且启动。 现在需要使用 `sudo` 运行 docker 命令。 根据 [Linux postinstall](https://docs.docker.com/engine/installation/linux/linux-postinstall/) 不走，设置允许非特权用户使用docker命令。

#### 更新 Docker CE

更新之前，执行 `sudo yum makecache fast` 命令，根据之前的安装步骤选择一个需要升级的版本。

### 通过包安装

如果你不能使用 Docker 仓库安装，可以下载 `.rpm` 进行安装。每次更新你都需要下载一个对应的新文件。

1. 到 https://download.docker.com/linux/centos/7/x86_64/stable/Packages/ 下载需要的的 `.rpm` 包。

> 如果你需要安装 `edge` 包，将 URL 中的 `stable` 变为 `edge`。鞥多[关于 **stable** 和 **edge** 的内容](https://docs.docker.com/engine/installation/)

2. 安装 Docker CE.

```bash
$ sudo yum install /path/to/package.rpm
```

Docker 安装完成后不会自动启动。 `docker` 用户组会被自动创建，但是不会添加用户到该组。

3. 启动 Docker

```bash
$ sudo systemctl start docker
```

4. 运行 `hello-world` 镜像

```bash
$ sudo docker run hello-world
```

#### 更新 Docker CE

下载需要更新的 `rpm` 版本包，根据之前的步骤在执行一遍，这次使用 `yum -y upgrade` 而替代 `yum -y install`。

### 使用一键脚本安装

Docker 在 [get.docker.com](https://get.docker.com/) 和 [test.docker.com](https://test.docker.com/) 分别提供了脚本一件安装 stable 和 test 版本的 Docker CE。脚本源码放在[github 上的 `docker-install` 仓库](https://github.com/docker/docker-install)， 这样你在使用之前你可以了解潜在的风险。

+ 运行脚本需要 `root` 或 `sudo` 权限。因此，在使用之前你需要仔细检查和审计。

+ 该脚本尝试检查你的 linux 发行版及版本号，并且配置包管理系统。脚本不支持任何安装参数，This may lead to an unsupported configuration, either from Docker’s point of view or from your own organization’s guidelines and standards.
+ 该脚本会安装所有依赖包，而不需要你确认。因此可能根据系统现在的配置安装很多内容。
+ 如果你机器有正在运行的docker，不要使用该脚本。

下例是使用 `get.docker.com` 脚本安装 docker CE 最新版本的稳定包。如果要安装 `testing` 包。使用 `test.docker.com`。将列所有步骤中的 `get` 换位 `test`

> 警告：在执行网上下载的脚本之前，永远记住先检查。

```bash
$ curl -fsSL get.docker.com -o get-docker.sh
$ sudo sh get-docker.sh

<output truncated>

If you would like to use Docker as a non-root user, you should now consider
adding your user to the "docker" group with something like:

  sudo usermod -aG docker your-user

Remember that you will have to log out and back in for this to take effect!

WARNING: Adding a user to the "docker" group will grant the ability to run
         containers which can be used to obtain root privileges on the
         docker host.
         Refer to https://docs.docker.com/engine/security/security/#docker-daemon-attack-surface
         for more information.
```

Docker CE 安装后。基于 `DEB` 发行版的会自动启动， 基于 `RPM` 的需要使用 `systemctl` 或 `service` 手动启动。根据消息显示，默认情况下，非 root 用户不允许使用 docker 命令。

#### 更新使用一键脚本安装的 Docker

在使用一键脚本安装之后，你只需要使用包管理直接更新即可。 重新执行改脚本没任何好处，反而会因为重新添加仓库而导致一些问题。

### 卸载 Docker CE

1. 卸载 docker 包

```bash
$ sudo yum remove docker-ce
```

2. 镜像、容器、卷、用户配置不会被自动删除。如果你需要删除所有内容

```bash
$ sudo rm -rf /var/lib/docker
```

任何编辑后的文件必须手动删除。


## Next steps
Continue to [Post-installation steps for Linux](https://docs.docker.com/engine/installation/linux/linux-postinstall/)

Continue with the [User Guide](https://docs.docker.com/engine/userguide/).

requirements, apt, installation, centos, rpm, install, uninstall, upgrade, update
