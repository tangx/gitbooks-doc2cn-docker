# doc2cn_docker
docker官方文档，中文汉化项目


> 2016-09-23
>
> 并不是把所有文章都翻译了，进根据学习程度不定时更新
> 翻译水平有限，肯定会有翻译错误和句子不通顺的地方



## 第1章 安装运行与卸载
+ [C01S01 在CentOS7上使用二进制包安装](./chapter01/01-install-docker-with-centos.md)
  + [系统环境要求](./chapter01/01-install-docker-with-centos.md#系统环境要求)
  + [安装](./chapter01/01-install-docker-with-centos.md#安装)
    + [使用yum安装](./chapter01/01-install-docker-with-centos.md#使用yum安装)
    + [使用脚本安装](./chapter01/01-install-docker-with-centos.md#使用脚本安装)
  + [设置docker daemon为开机启动](./chapter01/01-install-docker-with-centos.md#设置docker-daemon为开机启动)
  + [卸载](./chapter01/01-install-docker-with-centos.md#卸载)
+ [C01S02 使用二进制文件安装docker](./chapter01/02-installation-from-binaries.md)


## 第2章 使用Docker
+ [C02S01 理解镜像与容器](./chapter02/01-learn-about-images-containers.md)
+ [C02S02 查找并运行镜像](./chapter02/02-find-and-run-the-whalesay-image.md)
+ C02S03 创建自定义镜像
  + [C02S03.1 使用dockerfile创建镜像](./chapter02/03-build-your-own-image-with-dockerfile.md)
  + [C02S03.2 使用commit提交容器创建镜像](./chapter02/03-build-your-own-image-with-commit.md)
  + [C02S03.3 使用import导入备份创建镜像](./chapter02/03-build-your-own-image-with-import.md)
+ [C02S04 标记、推送、下载(tag-push-pull image)](./chapter02/04-tag-push-and-pull-your-image.md)


## 第3章 通过案例学docker
+ [C03S01 容器中的Hello World](./chapter03/01-hello-world-in-a-container.md)
+ [C03S02 运行一个简单的应用](./chapter03/02-run-a-simple-application.md)
+ [C03S03 创建自定义镜像](./chapter03/03-build-your-own-images.md)
+ [C03S04 管理容器的网络连接](./chapter03/04-network-containers.md)
+ [C03S05 管理容器的数据](./chapter03/05-manage-data-in-containers.md)
+ [C03S06 管理DockerHub中的容器](./chapter03/06-store-images-on-docker-hub.md)


## 第4章 使用手册
+ Docker storage drivers
  + [理解镜像,容器和存储驱动器](./chapter04/docker_storage_drivers/understand-images-containers-and-storage-drivers.md)
