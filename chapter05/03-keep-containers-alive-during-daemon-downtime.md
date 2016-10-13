# 在守护程序停止时保持容器运行
keep-containers-alive-during-daemon-downtime

> date : 2016-10-13 
> author: octowhale@github
> [ 原文 ](https://docs.docker.com/engine/admin/live-restore/)


默认情况下， 当docker daemon停止时， 会关闭运行中的容器。 从 docker engine 1.12 开始， 你可以调整daemon参数，使daemon服务不可用的时候，容器依旧保持运行状态。 The live restore option helps reduce container downtime due to daemon crashes, planned outages, or upgrades.


## 启用 live restore 选项

有两种方式可以保持容器在docker daemon变为不可用的时候保持运行：

+ 如果daemon正在运行且但不想重启，你可以添加配置的daemon配置文档。例如， 在linux系统上，默认配置文档为 ` /etc/docker/daemon.json `。

使用编辑器修改 ` daemon.json ` 启用 ` live-restore `：

```json
{
"live-restore": true
}
```

你必须给daemon进程发送一个 ` SIGHUP ` 信号通知daemon重新加载配置文件。 更多关于如何使用config.json配置docker daemon， 查看[daemon configuration file.](https://docs.docker.com/engine/reference/commandline/dockerd/#daemon-configuration-file)。

+ 如果直接启动docker daemon，只需要传递 ` --live-restore ` 标识即可：

```bash

$ sudo dockerd --live-restore

```


## Live restore during upgrades

docker间隔一个minor release升级时(ex, from docker engine 1.12.1 to 1.13.2 )， live restore 功能支持daemon恢复容器。

如果是跳 release 升级， daemon 可能没有保存容器的连接。 如果daemon不能保存这些连接，那么会忽略这些容器，且你必须手动操作。 daemon不会关闭断开连接的容器。
If you skip releases during an upgrade, the daemon may not restore connection the containers. If the daemon is unable restore connection, it ignores the running containers and you must manage them manually. The daemon won’t shut down the disconnected containers.


## Live restore upon restart

live restore选项只会在**`daemon恢复时的配置`与`daemon停止时的配置`相同时**生效。例如， daemon重启时使用了不同的桥接IP或不同的磁盘分区， live restore可能不会生效。
The live restore option only works to restore the same set of daemon options as the daemon had before it stopped. For example, live restore may not work if the daemon restarts with a different bridge IP or a different graphdriver.


## Impact of live restore on running containers

daemon长时间不可能会影响到运行中的容器。 容器进程产生FIFO日志以便daemon可用时处理数据。 如果daemon不处理处理消耗这些输出， 缓冲区将填满并阻止对日志的进一步写入。一个完整的日志将阻塞进程，直到有更多的可用空间。默认缓冲区大小为 ` 64KB `。

你必须重启docker刷新缓冲区。

你可以修改内核缓冲区大小 ` /proc/sys/fs/pipe-max-size ` 。

## Live restore and swarm mode

The live restore option is not compatible with Docker Engine swarm mode. When the Docker Engine runs in swarm mode, the orchestration feature manages tasks and keeps containers running according to a service specification.


----

## 总结

+ docker 1.12后， daemon关闭时，容器可以继续运行。
+ docker daemon关闭后， 容器会在缓冲区中产生 FIFO 日志。
+ 缓冲区被写满后，新FIFO被丢弃。daemon进程被阻塞，直到有更多的缓冲区。
+ docker daemon重启后，从缓冲区中的FIFO日志恢复容器数据
+ 修改内核缓冲区大小 ` /proc/sys/fs/pipe-max-size `

