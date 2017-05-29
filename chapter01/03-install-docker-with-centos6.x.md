# CentOS6.x 安装 docker

安装 docker 最好还是使用最新版本的centos。最新的内核可以支持最新的功能。并且最新的内核已经修复了已知的bug。

[ centos 7.x 安装 docker 及用户权限设置 ](01-install-docker-with-centos.md)

```

yum -y install http://ftp.riken.jp/Linux/fedora/epel/6Server/x86_64/epel-release-6-8.noarch.rpm


yum -y install docker-io

service docker start
service docker stop

```
