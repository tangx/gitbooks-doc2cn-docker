#第一章 docker服务的安装、启动

> ` octowhale@github - 2016/09/23 16:37 `

[ CentOS官网安装说明 ](https://docs.docker.com/engine/installation/linux/centos/)

Docker在 CentOS7.X上运行。Docker可能在其他EL7的兼容版本中成功安装，但是官方并未进行测试，因此也不提供任何支持。

本文将指导你使用Docker-managed发布的安装包进行安装。这样可以确保你安装的docker是最近版本的。

##系统环境要求

docker必须运行在64-bit的系统上，对于CentOS的版本号并没有特别要求。另外，如果需要在CentOS上安装，内核版本必须高于3.10。

通过` uname -r ` 查看内核版本


```

$ uname -r
3.10.0-327.el7.x86_64

```


最后，我们建议你完全升级你的系统。请记住，你的系统应该完全修复可能存在的内核bug。任何已经被报告的内核bug可能已经在上一个内核包中被修复。

##安装

可以通过以下两种方法安装Docker Engine。使用 `yum`包管理器；使用 `curl`命令访问 `get.docker.com`网站。后者将获得一个安装脚本，该脚本将通过` yum `管理器安装Docker。

###使用yum安装

1. 登录系统，并确认用户为` root `或者用户有权限使用` sudo `命令。
2. 保证你现有的yum安装包是最新的。


```bash

$ sudo yum update 

```

3. 添加yum repo


```bash

$ sudo tee /etc/yum.repos.d/docker.repo <<-'EOF'
[dockerrepo]
name=Docker Repository
baseurl=https://yum.dockerproject.org/repo/main/centos/7/
enabled=1
gpgcheck=1
gpgkey=https://yum.dockerproject.org/gpg
EOF


```

4. 安装docker   

```bash

$ sudo yum install docker-engine

```

5. 启动docker     


```bash

$ sudo service docker start

```

6. 确认` docker `被正确安装并在容器中运行一个测试镜像(image)    


```bash

$ sudo docker run hello-world
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world

c04b14da8d14: Pull complete 
Digest: sha256:0256e8a36e2070f7bf2d0b0763dbabdd67798512411de4cdcf9431a1feb60fd9
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker Hub account:
 https://hub.docker.com

For more examples and ideas, visit:
 https://docs.docker.com/engine/userguide/
 

```


###使用脚本安装

3. 执行安装脚本


```bash

$ curl -fsSL https://get.docker.com/ | sh 

```

该脚本会添加` docker.repo `仓库并安装docker。


##创建一个docker用户组

` docker `守护进程现在绑定了一个Unix Socket，取代了之前的TCP端口。 默认情况下，这个Socket的属主用户是` root `并且用户可以通过` sudo `进行访问。因此，` docker `守护进程总是通过` root `用户运行。     

为了避免在使用docker命令的时候使用sudo，需要创建一个名为` docker `的Unix用户组，并给该用户组添加用户。当docker守护进程启动后，docker用户组的用户可以获得socket的的读写权限。

> **注意:** docker用户组等效于root用户； 关于这样做为什么为影响你的系统哦你安全，更多信息可以查看[Docker Daemon Attach Surface](https://docs.docker.com/engine/security/security/#docker-daemon-attack-surface)。

接下来，创建docker用户组并添加用户。

1. 创建一个docker用户组      

```$ sudo groupadd docker ```     

2. 为docker用户组添加用户     

```

$ sudo usermod -aG docker your_username 

```    

3. 登录并重新登录    
这样可以保证你的用户获得正确的权限     
4. 确认你可以不使用sudo启动docker容器     

```$ docker run hello-world```



##设置docker daemon为开机启动

确保docker会在开机时候启动，需要执行以下命令     

```

sudo chkconfig docker on 
# 或者
sudo systemctl enabld docker.service 

```


如果你需要使用HTTP代理，那么需要为Docker执行文件另外设置一个目录或者分区，或者使用其他定制选项。阅读关于系统的文章并学习[customize your Systemd Docker daemon options](https://docs.docker.com/engine/admin/systemd/)。

##卸载

你可以通` yum ` 卸载docker软件

1. 列出所有以及安装的程序包


```bash

$ yum list installed |grep docker 
docker-engine.x86_64                 1.12.1-1.el7.centos             @dockerrepo
docker-engine-selinux.noarch         1.12.1-1.el7.centos             @dockerrepo 

```

2. 删除软件包


```bash

$ sudo yum -y remove docker-engine.x86_64                 1.12.1-1.el7.centos

```

该命令不会删除docker镜像，容器，数据卷，或者用户创建的配置文件。          
3. 如果要删除镜像，容器和数据卷，使用以下命令    


```

$ rm -rf /var/lib/docker

```

4. 查找并删除其他任意用户创建的配置文件。


