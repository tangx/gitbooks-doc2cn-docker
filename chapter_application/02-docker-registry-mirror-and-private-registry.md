# docker registry mirror and private registry
# 仓库镜像与私有仓库


仓库镜像与私有仓库不同

在 docker 官方版本的 `registry:2` 镜像，不支持同时将两个服务搭建在一个容器中。如果有需要，可以使用 `nexus` 实现此需求。



## 仓库镜像 registry-mirror


顾名思义，仓库镜像就是为了提高局部网络内的镜像下载速度。

现在，国内有一下几个公开仓库镜像比较广泛

+ 官方： `https://registry.docker-cn.com`
+ 清华tuna: `https://docker.mirrors.ustc.edu.cn`
+ 中科大: `https://docker.mirrors.ustc.edu.cn`


使用 `docker-compose` 搭建镜像仓库

```
# config.yml

version: 0.1
log:
    fields:
        service: registry
storage:
    cache:
        blobdescriptor: inmemory
    filesystem:
        rootdirectory: /var/lib/registry
http:
    addr: 192.168.56.205:5000
    headers:
        X-Content-Type-Options: [nosniff]
health:
    storagedriver:
        enabled: true
        interval: 10s
        threshold: 3
proxy:
    # remoteurl: https://registry-1.docker.io
    remoteurl: https://registry.docker-cn.com

```


```
# docker-compose.yml
version: '2'
services:
    mirror:
        image: registry:2
        ports:
            - "5000:5000"
        volumes:
            - ./config.yml:/etc/docker/registry/config.yml
            
```

```
$ docker-compose up -d 
```

### docker-compose


[docker-compose github 页面](https://github.com/docker/compose)

```
curl -L https://github.com/docker/compose/releases/download/1.16.1/docker-compose-`uname -s`-`uname -m` > docker-compose
chmod +x docker-compose
sudo mv docker-compose /usr/local/bin/docker-compose
```



## 私有仓库

私有仓库可以通过 `username/password` 方式和通过 `tls` 方式对外进行限制。
同样，也可以同系统层面，通过防火墙限制 `来源ip` 实现私有方式。

> https://docs.docker.com/registry/


根据官方文档，最简单的搭建私有仓库的命令为，

```
$ docker run -d -p 5000:5000 --name registry registry:2
```
