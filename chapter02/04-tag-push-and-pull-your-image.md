# Tag,push和pull镜像

[ 原文 ](https://docs.docker.com/engine/getstarted/step_six/#/)
> 2016-09-24 uyinn28@github

本章中，你将学习如何标记和推送 ` docker-whale ` 镜像到你新创建的仓库中。推送完成后，测试从仓库中拉去镜像。

## Step 01: Tag and push 镜像

1. 打开一个都终端。

2. 使用命令 ` docker images ` 列出当前所有的镜像。

```

$ docker images
REPOSITORY           TAG          IMAGE ID            CREATED             SIZE
docker-whale         latest       7d9495d03763        38 minutes ago      273.7 MB
<none>               <none>       5dac217f722c        45 minutes ago      273.7 MB
docker/whalesay      latest       fb434121fc77        4 hours ago         247 MB
hello-world          latest       91c95931e552        5 weeks ago         910 B

```


3. 找到` docker-whale `镜像的的` IMAGE ID `。
本例中，id为7d9495d03763。
需要注意的是，现在 `REPOSITORY`显示的仓库名称为`docker-whale`，并不包含命名空间。你需要使用你的DockerHub账户名作为命名空间。该命名空间名称与你的DockerHub账号名称相同。重命名镜像为 ` YOUR_DOCKERHUB_NAME/docker-whale `。

4. 使用 ` IMAGE ID `和` docker tag `命令标记你的`docker-whale`镜像。
命令格式看起来像这样：
![tagger.png](http://files.uyinn.com/media/docker/tagger.png)

当然，账号必须是你自己的。之后使用包含镜像的ID和用户名的命令进行标记。

```

$ docker tag 7d9495d03763 maryatdocker/docker-whale:latest

```


5. 使用命令 ` docker images ` 查看新标记的镜像。

```

$ docker images
REPOSITORY                  TAG       IMAGE ID        CREATED          SIZE
maryatdocker/docker-whale   latest    7d9495d03763    5 minutes ago    273.7 MB
docker-whale                latest    7d9495d03763    2 hours ago      273.7 MB
<none>                      <none>    5dac217f722c    5 hours ago      273.7 MB
docker/whalesay             latest    fb434121fc77    5 hours ago      247 MB
hello-world                 latest    91c95931e552    5 weeks ago      910 B

```

6. 使用命令 ` docker login ` 登录DockerHub。

```

docker login 

```

跟具体是输入账号和密码，例如：

```

$ docker login
Login with your Docker ID to push and pull images from Docker Hub. If you don't have a Docker ID, head over to https://hub.docker.com to create one.
Username: 
Password:        
Login Succeeded

```


7. 使用命令 ` docker push ` 将镜像推送到你的DockerHub仓库。

```

$ docker push maryatdocker/docker-whale
    The push refers to a repository [maryatdocker/docker-whale] (len: 1)
    7d9495d03763: Image already exists
    c81071adeeb5: Image successfully pushed
    eb06e47a01d2: Image successfully pushed
    fb434121fc77: Image successfully pushed
    5d5bd9951e26: Image successfully pushed
    99da72cfe067: Image successfully pushed
    1722f41ddcb5: Image successfully pushed
    5b74edbcaa5b: Image successfully pushed
    676c4a1897e6: Image successfully pushed
    07f8e8c5e660: Image successfully pushed
    37bea4ee0c81: Image successfully pushed
    a82efea989f9: Image successfully pushed
    e9e06b06e14c: Image successfully pushed
    Digest: sha256:ad89e88beb7dc73bf55d456e2c600e0a39dd6c9500d7cd8d1025626c4b985011

```


8. 登录DockerHub网站查看你的镜像信息
![new_image.png](https://docs.docker.com/engine/getstarted/tutimg/new_image.png)

## Step 02: 下载镜像
本节中，你将下载之前推送到DockerHub中的镜像。在此之前，你需要删除本机镜像。如果不删，Docker就不会从Hub上下载。为什么？还不是两个镜像一模一样。

1. 确认Docker正在运行。

2. 使用命令 ` docker images ` 列出所有本地镜像

```

$ docker images
REPOSITORY                  TAG       IMAGE ID        CREATED          SIZE
maryatdocker/docker-whale   latest    7d9495d03763    5 minutes ago    273.7 MB
docker-whale                latest    7d9495d03763    2 hours ago      273.7 MB
<none>                      <none>    5dac217f722c    5 hours ago      273.7 MB
docker/whalesay             latest    fb434121fc77    5 hours ago      247 MB
hello-world                 latest    91c95931e552    5 weeks ago      910 B

```

如果要好好的完成测试，你需要删除本的镜像 ` maryatdocker/docker-whale ` 和 ` docker-whale ` 。只有删除了，才能强制之后的 ` docker pull ` 从DockerHub仓库下载新镜像。

3. 使用命令 ` docker rmi ` 删除镜像。

```

$ docker rmi -f 7d9495d03763
$ docker rmi -f docker-whale

```

>你可以使用`IMAGE ID`或者`REPOSITORY:TAG`来删除镜像。
>如果多个镜像具有相同 IMAGE ID，docker会提示错误。
>如果不为 REPOSITORY 指定 TAG，那么 TAG 默认之为 `latest`。

4. 使用命令 ` docker run ` 从你的仓库中下载并运行一个新镜像。
该命令必须包含你的DockerHub用户名，否则将会从官方镜像下载。

```

$ docker run yourusername/docker-whale

```

由于该镜像已经不在本地了，因此docker需要重新下载。

```

$ docker run maryatdocker/docker-whale
Unable to find image 'maryatdocker/docker-whale:latest' locally
latest: Pulling from maryatdocker/docker-whale
eb06e47a01d2: Pull complete
c81071adeeb5: Pull complete
7d9495d03763: Already exists
e9e06b06e14c: Already exists
a82efea989f9: Already exists
37bea4ee0c81: Already exists
07f8e8c5e660: Already exists
676c4a1897e6: Already exists
5b74edbcaa5b: Already exists
1722f41ddcb5: Already exists
99da72cfe067: Already exists
5d5bd9951e26: Already exists
fb434121fc77: Already exists
Digest: sha256:ad89e88beb7dc73bf55d456e2c600e0a39dd6c9500d7cd8d1025626c4b985011
Status: Downloaded newer image for maryatdocker/docker-whale:latest
 ________________________________________
/ Having wandered helplessly into a      \
| blinding snowstorm Sam was greatly     |
| relieved to see a sturdy Saint Bernard |
| dog bounding toward him with the       |
| traditional keg of brandy strapped to  |
| his collar.                            |
|                                        |
| "At last," cried Sam, "man's best      |
\ friend -- and a great big dog, too!"   /
 ----------------------------------------
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

