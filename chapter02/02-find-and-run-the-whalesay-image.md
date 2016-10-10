# 查找并运行镜像

> 2016-09-24

[原文](https://docs.docker.com/engine/getstarted/step_three/#/find-and-run-the-whalesay-image)

全世界的使用者都能创建docker镜像并分享。你可以从Docker Hub网站上找到这些镜像。在接下来的章节中，我们将搜索查找一个镜像并运行他。

##Step 01:找到名为whalesay的镜像

搜索镜像有两种方式：
- 通过DockerHub网站查找
- 通过命令行查找

### 通过DockerHub网站查找

1. 打开[Docker Hub网站](https://hub.docker.com/)
![browse_and_search.png](https://docs.docker.com/engine/getstarted/tutimg/browse_and_search.png)
DockerHub镜像库包含了所有独立用户提交的镜像，比如说你的镜像，或者其他官方镜像(RedHat,IBM,Google)，以及更多其他的。

2. 点击 Browse & Search.
打开搜索页面

3. 在搜索框中输入`whalesay`
![image_found.png](https://docs.docker.com/engine/getstarted/tutimg/image_found.png)

4. 在查询结果中，点击**docker/whalesay**镜像
浏览器会显示关于关于**whaleway**的仓库信息
![whale_repo.png](https://docs.docker.com/engine/getstarted/tutimg/whale_repo.png)
每个镜像仓库包含了关于这个镜像的信息。这些信息应该描述：01.该镜像使用了什么软件；02.怎么使用该镜像。
从**whalesay**镜像的描述中你可以知道，该镜像是基于uhuntu的。下一步，我们将运行whalesay镜像了。

### 通过命令行查找
除了能够通过网页查找之外，docker引擎还提供了命令行` docker search `。然而，在描述上，命令还并没有页面清楚。


```

$ sudo docker search -h

Usage: docker search [OPTIONS] TERM

Search the Docker Hub for images

  --automated=false    只显示自动构建的，默认false
  --help=false         Print usage
  --no-trunc=false     Don't truncate output
  -s, --stars=0        只显示N星以上的


```



```bash

$ sudo docker search whalesay
NAME                                     DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
docker/whalesay                          An image for use in the Docker demo tutorial    421                  
firecyberice/whalesay                    Small whalesay                                  5                    [OK]
caibar/whalesay                          Builds automatizados.                           1                    [OK]
a33a/whalesay                            First automated build                           1                    [OK]
swinton/whalesay                         whalesay, innit                                 1                    
mendlik/docker-whalesay                  Docker whalesay image from training materi...   1                    [OK]
sabs1117/whalesay                        Whalesay with fortune phrases.                  1                    
ojenge/whalesay                          from docker/whalesay                            1                    
...


```


##Step 02: 启动 whalesay 镜像
确认Docker已经运行。On Docker for Mac and Docker for Windows, this is indicated by the Docker whale showing in the status bar.

1. 打开命令行终端
2. 输入` docker run docker/whalesay cowsay boo `命令，并回车
该命令会在一个容器中启动**whaleway**镜像。你的终端应该能看到一下信息


```bash

$ docker run docker/whalesay cowsay boo
Unable to find image 'docker/whalesay:latest' locally
latest: Pulling from docker/whalesay
e9e06b06e14c: Pull complete
a82efea989f9: Pull complete
37bea4ee0c81: Pull complete
07f8e8c5e660: Pull complete
676c4a1897e6: Pull complete
5b74edbcaa5b: Pull complete
1722f41ddcb5: Pull complete
99da72cfe067: Pull complete
5d5bd9951e26: Pull complete
fb434121fc77: Already exists
Digest: sha256:d6ee73f978a366cf97974115abe9c4099ed59c6f75c23d03c64446bb9cd49163
Status: Downloaded newer image for docker/whalesay:latest
 _____
< boo >
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


当你第一次使用该镜像时，` docker `命令会在本机查找该镜像是否存在。如果不存在，` docker `会从hub上下载。

3. 然后接着在终端输入 ` docker images `命令并回车
该命令会列出本机上所有的镜像。你可以看到` docker/whalesay `已经在列表中了。

```bash

$ docker images
REPOSITORY           TAG         IMAGE ID            CREATED            SIZE
docker/whalesay      latest      fb434121fc77        3 hours ago        247 MB
hello-world          latest      91c95931e552        5 weeks ago        910 B

```

当你在容器中启动一个镜像时，Docker会下载该镜像到你的计算机中。拷贝到本地的镜像会记录一个时间。只有当hub上的源镜像发生改变的时候，才会重新下载镜像。当然，你也可以自己删掉这些镜像。这个我们之后会学习。

4. 在多玩一会**whalesay**
可以尝试一下在启动` whalesay `的时候使用一些更长或更短的短语。你玩儿的转么？

```bash

$ docker run docker/whalesay cowsay boo-boo
 _________
< boo-boo >
 ---------
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

