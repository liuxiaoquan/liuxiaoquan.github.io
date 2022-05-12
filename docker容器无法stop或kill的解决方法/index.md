# Docker容器无法stop或kill的解决方法


```shell
# 查看网络
docker network ls
```

bridge、none、host这 3 个网络包含在 Docker 实现中。运行一个容器时，可以使用 the –net标志指定您希望在哪个网络上运行该容器。您仍然可以使用这 3 个网络。

bridge 网络表示所有 Docker 安装中都存在的 docker0 网络。除非使用 docker run –net=选项另行指定，否则 Docker 守护进程默认情况下会将容器连接到此网络。在主机上使用 ifconfig命令，可以看到此网桥是主机的网络堆栈的一部分。
none 网络在一个特定于容器的网络堆栈上添加了一个容器。该容器缺少网络接口。
host 网络在主机网络堆栈上添加一个容器。您可以发现，容器中的网络配置与主机相同。

```shell
# 查看网络信息
docker network inspect <网络模式>
# 删除容器
docker rm -f <容器名>
# 清理网络占用
docker network disconnect --force <网络模式> <容器名>
```


