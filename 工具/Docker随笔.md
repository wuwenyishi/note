### linux docker 远程连接配置

修改Docker服务文件： 

vi /lib/systemd/system/docker.service   

在ExecStart后添加：-H tcp://0.0.0.0:2375 -H unix://var/run/docker.sock   

重新加载配置文件、重启Docker服务   

systemctl daemon-reload     

systemctl restart docker   



## 命令

###### 清理磁盘，删除关闭的容器、无用的数据卷和网络，以及dangling镜像(即无tag的镜像)

```shell
docker system prune # 命令清理得更加彻底，可以将没有容器使用Docker镜像都删掉。注意，这两个命令会把你暂时关闭的容器，以及暂时没有用到的Docker镜像都删掉了 
docker system prune -a
```

###### 删除悬空的镜像

##### 删除无用的容器

```shell
# 会清理掉所有处于stopped状态的容器 
docker container prune
```

###### 删除无用的卷

###### 删除无用的网络

###### 删除所有悬空镜像，不删除未使用镜像

```shell
docker rmi $(docker images -f "dangling=true" -q)
```

###### 删除所有未使用镜像和悬空镜像

```shell
docker rmi $(docker images -q)
```

###### 删除所有未被容器引用的卷

```shell
docker volume rm $(docker volume ls -qf dangling=true)
```

###### 删除所有已退出的容器

```shell
docker rm -v $(docker ps -aq -f status=exited)
```

###### 删除所有状态为dead的容器

```shell
docker rm -v $(docker ps -aq -f status=dead)
```

###### 查找指定目录下所有大于100M的所有文件

```shell
find /var/lib/docker/overlay2/ -type f -size +100M -print0 | xargs -0 du -h | sort -nr
```

###### 把/var目录下所有日志文件清空

```shell
for i in `find /var -name *.log*`;do >$i;done
```
