# shadowsocks 镜像

shadowsocks server 可以通过 `pip` 进行安装。安装脚本参考 [安装启动shadowsocks](https://github.com/octowhale/bash-scripts/tree/master/shell_scripts/shadowsocks)

这里，只是将 ssserver 进行了容器化。


## 编辑配置文件

对 `server` 和 `local_address` 使用值 `0.0.0.0` 即可监听所有ip地址。
修改对应的 port 和 password 配置登录密钥

```json
{
    "user":"nobody",
    "server":"0.0.0.0",
    "local_address": "0.0.0.0",
    "port_password":{
                "port1":"password_for_port1",
                "port2":"password_for_port2"
        },
    "timeout":300,
    "method":"aes-256-cfb",
    "fast_open": false,
    "log-file":"/dev/null"
}

```


## 启动容器

```bash
#!/bin/bash 

source /etc/profile

docker run -itd -v /path/2/ss.json.cfg:/root/ss.json.cfg -p 66666:66666 uyinn28/centos6:ss-0.0.1 /usr/bin/ssserver -c /root/ss.json.cfg
```

> `-itd`： daemon 方式运行。
> `-v /path/2/ss.json.cfg:/root/ss.json.cfg` : 将配置文件映射到容器中
> `-p outsite_port:docker_port` : 将外部端口映射到docker内部
> `uyinn28/centos6:ss-0.0.1` : 容器镜像名与 tag 。
> `/usr/bin/ssserver -c /root/ss.json.cfg` : entrypoint, ssserver 启动命令


# 查看启动结果

```bash
# docker ps -a
CONTAINER ID        IMAGE                      COMMAND                CREATED             STATUS              PORTS                    NAMES
8612676ec288        uyinn28/centos6:ss-0.0.1   "/usr/bin/ssserver -   34 hours ago        Up 34 hours         0.0.0.0:2222->2222/tcp   pensive_hawking     
```

> `CONTAINER ID` : 容器标识符 (唯一)。 这里只是截取了部分。在没有歧义的情况下，可以使用任意长度的 `CONTAINER ID` 表示容器。
> `IMAGE` : 镜像名称
> `COMMAND` : 容器启动命令
> `CREATED` : 容器创建时间
> `STATUS` : 容器运行状态
> `PORTS` : 端口映射情况
> `NAMES` : 容器的名称 (唯一)
