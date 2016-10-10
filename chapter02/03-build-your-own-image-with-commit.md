# 使用commit提交容器创建镜像

> 2016-09-26 octowhale#github


dockerfile是告诉docker按照步骤从基本镜像构建新镜像。
commit则是自己手动从基本镜像完善容器，并将容器转换为镜像。

## 获取镜像母版

1. 确认docker已经运行

2. 下载基本镜像


```bash

$ docker pull centos
Using default tag: latest
latest: Pulling from library/centos
Digest: sha256:2ae0d2c881c7123870114fb9cc7afabd1e31f9888dac8286884f6cf59373ed9b
Status: Image is up to date for centos:latest

```


## 创建自定义系统环境

3. 启动cents镜像，进入容器


```bash

$ docker run -t -i centos /bin/bash
[root@34dc47734372 /]# 

```

**注意**： -t 为容器打开一个tty。-i 则是保持。

4. 在容器内创建我们需要执行的操作，这里创建一个 [whalesay.sh文件](./script/whalesay.sh) 。 


```bash

[root@34dc47734372 ~]#  cat /root/whalesay.sh

#!/bin/bash
#
# whalesay.sh
#

# mkdir -p /tmp/whalesay/
cat  <<EOF
 _____
< $@ >
 -----
    \\
     \\
      \\     
                    ##        .            
              ## ## ##       ==            
           ## ## ## ##      ===            
       /""""""""""""""""___/ ===        
  ~~~ {~~ ~~~~ ~~~ ~~~~ ~~ ~ /  ===- ~~~   
       \\______ o          __/            
        \\    \\        __/             
          \\____\\______/   

EOF

chmod +x /root/whalesay.sh


```


5. 测试 ` whalesay.sh ` 查看效果


```bash

[root@34dc47734372 ~]# /root/whalesay.sh men turn left, cuz women always right.
 _____
< men turn left, cuz women always right. >
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


OK, 现在我们的程序可以正常运行了。使用 ` exit ` 结束 `/bin/bash ` ，退出容器


```bash

[root@34dc47734372 ~]# exit 
exit
[octowhale@s008-docker-centos7 ~]$ 

```


## 使用commit提交容器创建镜像

6. 使用 ` docker ps -a ` 查看我们所有的容器


```bash

$ docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                          PORTS               NAMES
34dc47734372        centos              "/bin/bash"         16 minutes ago      Exited (0) About a minute ago                       boring_swanson
f933921b59df        centos              "/bin/bash"         16 minutes ago      Exited (0) 16 minutes ago                           elegant_poitras

```


可以看出，` CONTAINER ID ` 为34dc47734372的容器`STATUS`是一分钟前退出的，也就是刚才我们操作的容器。

7. 使用 ` docker commit -h ` 查看帮助文档


```bash

$ docker commit -h
Flag shorthand -h has been deprecated, please use --help

Usage:  docker commit [OPTIONS] CONTAINER [REPOSITORY[:TAG]]

Create a new image from a container's changes

Options:
  -a, --author string    作者信息 (e.g., "John Hannibal Smith <hannibal@a-team.com>")
  -c, --change value     Apply Dockerfile instruction to the created image (default [])
      --help             Print usage
  -m, --message string   提交备注，可以填写提示修改内容
  -p, --pause            Pause container during commit (default true)

```


8. 使用 `docker rommit 提交镜像`


```bash

$ docker commit -a "octowhale@github " -m "CentOS7 whalesay" 34dc47734372 octowhale/centos7:whalesay
sha256:864c18200b9acd56ce2b4fec42a5cc3fb5a6ee483ab83fee9af9da2784803a4e

```


使用 `docker images`查看刚才创建的镜像


```bash

$ docker images
REPOSITORY                    TAG                     IMAGE ID            CREATED              SIZE
octowhale/centos7             whalesay                864c18200b9a        About a minute ago   196.7 MB
centos                        latest                  970633036444        8 weeks ago          196.7 MB
registry                      latest                  c6c14b3960bd        8 weeks ago          33.28 MB
hello-world                   latest                  c54a2cc56cbb        12 weeks ago         1.848 kB

```


可以看到，我们刚才创建的镜像`octowhale/centos7:whalesay`已经存在了


## 测试镜像发现问题

9. 测试创建的whalesay镜像


```bash

# 第一次
$ docker run octowhale/centos7:whalesay
# 这里好像什么都没发生

```


输入命令启动镜像后可以观察到，好像什么都没有发生。但通过 `docker ps -a `可以看到确实使用镜像创建了一个容器。只不过容器没有任何输入，所以看不到效果


```bash

$ docker ps -a 
CONTAINER ID        IMAGE                        COMMAND             CREATED             STATUS                      PORTS               NAMES
b1999e58603d        octowhale/centos7:whalesay   "/bin/bash"         2 minutes ago       Exited (0) a minutes ago                        suspicious_euler
34dc47734372        centos                       "/bin/bash"         34 minutes ago      Exited (0) 18 minutes ago                       boring_swanson
f933921b59df        centos                       "/bin/bash"         34 minutes ago      Exited (0) 34 minutes ago                       elegant_poitras

```


让我们再来是一次，这次在启动命令上多加一个参数


```bash

# 第二次
$ docker run octowhale/centos7:whalesay /root/whalesay.sh
 _____
<  >
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


# 第三次
$ docker run octowhale/centos7:whalesay /root/whalesay.sh "right brain nothing left, left brain nothing right" 
 _____
< right brain nothing left, left brain nothing right >
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

#

$ docker ps -a
CONTAINER ID        IMAGE                        COMMAND                  CREATED             STATUS                      PORTS               NAMES
f5ebc85e514f        octowhale/centos7:whalesay   "/root/whalesay.sh 'r"   42 seconds ago      Exited (0) 41 seconds ago                       awesome_pare
72eff9a6d353        octowhale/centos7:whalesay   "/root/whalesay.sh"      2 minutes ago       Exited (0) 2 minutes ago                        sharp_curie

```


第二次和第三次启动容器, 屏幕终于数据信息了. 通过观察, 我们可以看到. 这两次启动容器较第一次还多使用了一个参数 ` /root/whalesay.sh `. 而这个就是我们之前所创建的.
这也就是通过commit创建容器的一个小缺陷, **不能自动执行命令或添加entrypoint**

## commit创建镜像的不足

> **注意**: 如果一定要添加自动命令或entrypint的话, 目前好像只能再使用一次dockerfile 将刚才创建的镜像再包装一次. 方法可以参考stackoverflow上的[docker-commit-created-images-and-entrypoint](http://stackoverflow.com/questions/29015023/docker-commit-created-images-and-entrypoint/29015976#29015976)

> ** 注意**: 此问题已经在docker1.11.1及以后的版本中得到解决，[使用`docker commit -c string`实现](#commit创建镜像时添加自动运行cmd和entrypoint)

## commit创建镜像时添加自动运行CMD和entrypoint

之前的测试中我们看到，虽然通过commit创建了新的镜像，但是` whalesay.sh `并不能自己运行。因此，我们需要在commit的时候为镜像添加CMD或entrypoint

在docker 1.11.1版本后，` docker commit `增加了一个参数 `-c, --change value     Apply Dockerfile instruction to the created image (default [])`。该参数可以为创建后镜像添加一些启动操作。
使用 ` docker commit -h ` 命令，查看当前版本的docker是否支持 ` -c ; --change string ` 选项。



```bash


$ docker commit --change='ENTRYPOINT ["/root/whalesay.sh"]' d122c7798ec1 octowhale/centos7:whalesay-change-entrypoint
sha256:c0523888da6bd0149a57414d9d2bdce2019f6d4fa69690e795a675a9b3b0cbc0

$ docker commit --change='CMD ["/root/whalesay.sh"]' d122c7798ec1 octowhale/centos7:whalesay-change-cmd-v2
sha256:70d5e994e059e7bbffcf1f9ead4f17d7750c56078b2f7f2191d8d5d9a514c67f

$ docker commit --change='CMD /root/whalesay.sh "hello world"' d122c7798ec1 octowhale/centos7:whalesay-change-cmd-v4
sha256:216b5d379710d7a43457b32f0a3424258145d787335cc154a8cbaaccea1f68a9

$ docker images
REPOSITORY                    TAG                          IMAGE ID            CREATED              SIZE
octowhale/centos7             whalesay-change-cmd-v4       216b5d379710        58 seconds ago       474.6 MB
octowhale/centos7             whalesay-change-cmd-v3       79837576d3d4        About a minute ago   474.6 MB
octowhale/centos7             whalesay-change-cmd-v2       70d5e994e059        25 minutes ago       474.6 MB
octowhale/centos7             whalesay-change-cmd          94fe1aa155a7        26 minutes ago       474.6 MB
octowhale/centos7             whalesay-change-entrypoint   c0523888da6b        29 minutes ago       474.6 MB

$ docker run c0523888da6b 
 _____
<  >
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

#

$ docker run octowhale/centos7:whalesay-change-cmd-v4
 _____
< hello world >
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


