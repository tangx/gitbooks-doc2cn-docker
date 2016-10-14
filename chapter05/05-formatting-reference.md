# 格式化输出信息参考

> data: 2016-10-14
> author: octowhale@github
> [ 原文 ](https://docs.docker.com/engine/admin/formatting/)

docker 使用 [go语言模板](https://golang.org/pkg/text/template/) ，允许用户控制指定命令的输出的格式。详细信息参考各自命令：

Docker Images formatting
Docker Inspect formatting
Docker Log Tag formatting
Docker Network Inspect formatting
Docker PS formatting
Docker Volume Inspect formatting
Docker Version formatting


## 模板功能

docker提供了一系列基本功能来指定模板元素。This is the complete list of the available functions with examples:


### Join

join连接所有字符串组成一个新的字符串。 原始字符串之间使用指定的分隔符进行分割。分割符可以是多个字符组成的字符串。

```bash
$ docker ps --format '{{join .Names " or "}}'
```


### Json

使用Json格式输出字符串

```bash
$ docker inspect --format '{{json .Mounts}}' container
```


### Lower

使用小写字母输出字符串

```bash
$ docker inspect --format "{{lower .Name}}" container

```


### Split

通过指定分隔符，将一个字符串拆分为多个字符串。

```bash
# docker inspect --format '{{split (join .Names "/") "/"}}' container

```


### Title

字符串首字母大写

```bash
$ docker inspect --format "{{title .Name}}" container
```


### Upper

使用大写字母输出字符串

```
$ docker inspect --format "{{upper .Name}}" container
```



