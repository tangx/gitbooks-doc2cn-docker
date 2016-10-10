# 使用import导入备份创建镜像

> 2016-09-26



有些时候, 镜像的母本不一定是docker镜像或容器, 也有可能是虚拟机或者实体机, 这个时候就需要用到 import 创建镜像了.
当然, 这种方法也可以用在docker容器内.

这里我们就以docker容器做示范

参考文章 [从ISO到DOCKER](http://wrfly.kfd.me/%E4%BB%8Eiso%E5%88%B0docker/)

## 使用tar命令备份备份系统

1. 进入容器并备份系统


```

$ docker run -t -i octowhale/centos7:whalesay /bin/bash
[root@d122c7798ec1 /]# 
[root@d122c7798ec1 /]#  tar -zcpf /tmp/centos7_whalesay.tar.gz --directory=/ \
    --exclude=proc --exclude=sys \
    --exclude=dev --exclude=run \
    --exclude=boot --exclude=tmp .
    
[root@d122c7798ec1 /]# ls -l /tmp/centos7_whalesay.tar.gz 
-rw-r--r--. 1 root root 70701506 Sep 26 03:11 /tmp/centos7_whalesay.tar.gz


```

[如何打包系统](http://www.aboutdebian.com/tar-backup.htm)


2. 使用你喜欢的方法，将打包好的备份传到运行docker的主机上

```

[root@d122c7798ec1 /]# scp /tmp/centos7_whalesay.tar.gz root@172.17.0.1:/tmp/
centos7_whalesay.tar.gz                        100%   67MB  67.4MB/s   00:01    
[root@d122c7798ec1 /]# exit
exit
[octowhale@s008-docker-centos7 ~]$ ls -l /tmp/centos7_whalesay.tar.gz 
-rw-r--r--. 1 root root 70701506 Sep 26 11:15 /tmp/centos7_whalesay.tar.gz


```


## 使用import导入备份创建镜像

3. 查看 ` docker import ` 使用方法

```

$ docker import -h
Flag shorthand -h has been deprecated, please use --help

Usage:  docker import [OPTIONS] file|URL|- [REPOSITORY[:TAG]]

Import the contents from a tarball to create a filesystem image

Options:
  -c, --change value     Apply Dockerfile instruction to the created image (default [])
      --help             Print usage
  -m, --message string   Set commit message for imported image

```

三种数据源：
+ file: 直接使用本地备份文件
+ URL：使用远程备份文件
+ - ： 使用标准输出(stdout)

4. 使用import导入备份创建镜像

```

# 使用文件
$ docker import -m " whalesay import with file " /tmp/centos7_whalesay.tar.gz octowhale/centos7:whalesay-import-file
sha256:b0a659356ddf65a8cd75129093961d3c41ff2d1c42d95b7ac854551596718492
# 使用标准输出
$ cat /tmp/centos7_whalesay.tar.gz | docker import -m " whalesay import with stdout " - octowhale/centos7:whalesay-import-stdout
sha256:dc6a4bb774a154a96a36f527d62c2e27810e377f3109f0e2403efacca4bde44e

```


使用 ` docker images `查看结果

```

$ docker images
REPOSITORY                    TAG                      IMAGE ID            CREATED             SIZE
octowhale/centos7             whalesay-import-stdout   dc6a4bb774a1        2 minutes ago       196.7 MB
octowhale/centos7             whalesay-import-file     b0a659356ddf        3 minutes ago       196.7 MB
octowhale/centos7             whalesay                 864c18200b9a        About an hour ago   196.7 MB

```


## 测试镜像
使用run命令测试刚才import创建的镜像


```

$ docker run octowhale/centos7:whalesay-import-file /root/whalesay.sh "that's all"
 _____
< that's all >
 -----
    \
     \
      \     
                    ##        .            
              ## ## ##       ==            
           ## ## ## ##      ===            
       /""""""""""""""""___/ ===        
  ~~~ {~~ ~~~~ ~~~ ~~~~ ~~ ~ /  ===- ~~~   
       \______ o          __/            
        \    \        __/             
          \____\______/   


```

输出成功， 通过备份文件创建镜像完成

## 存在的问题
> 和通过容器commit创建镜像一样，使用import创建镜像也不能指定自动运行命令或entrypint。
> 如有有需要，则再次使用dockerfile包装；或者使用系统本身的功能完成。


> **注意**：此问题已经在docker1.11.1及之后的版本得到解决，具体操作方式参考[使用`docker commit -c string`实现](./03-build-your-own-image-with-commit.md##commit创建镜像时添加自动运行cmd和entrypoint)


## 系统模板

访问[openvz系统模板页面](https://openvz.org/Download/template/precreated)。该页面提供主流linux发行版的系统模板。使用 ` docker import ` 命令可制作镜像。