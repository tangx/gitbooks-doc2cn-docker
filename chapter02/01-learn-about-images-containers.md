#了解docker镜像与容器

> 2016-09-24

[原文](https://docs.docker.com/engine/getstarted/step_two/#/learn-about-images-containers)


docker引擎提供核心docker技术，从而实现镜像和容器。正如安装的最后一步，你执行` docker run hello-world `命令。这条命令分为三个部分：

![container_explainer](http://files.uyinn.com/media/docker/container_explainer.png)

**镜像**是在运行时使用的文件系统和参数构成的。他没有状态且用还不会改变。**容器**是镜像运行时的一个实例。当你执行命令时，Docker引擎会：
- 检查当前系统环境是否有` hello-world `软件镜像
- 如果没有则从Docker Hub上下载镜像
- 加载镜像到容器并“执行”他

根据构建方式的不同，一个镜像可能在执行一个简单的、单一的命令后退出。正如` hello-world `镜像。

一个docker镜像可以做事情还有很多。一个镜像可能启动一个例如数据库的复杂程序，并等待你(或其他人)添加、存储数据并在之后使用他；进而等待之后的人员继续使用。

那么是谁创建了 `hello-world`镜像呢？这里是Docker，但是其他任何人也可以。Docker引擎允许其他人或公司通过镜像创建并分享软件。使用docker时，你不用担心你的计算机是否能够运行镜像中的软件 -- 在dock容器中，永远可以成功运行。

