在终端执行yum -y install wget 时，终端提示：

```
已加载插件：fastestmirror
Loading mirror speeds from cached hostfile
docker-ce-stable                                                          
没有可用软件包 wget。
错误：无须任何处理
```

解决方法

```
curl -o /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
```



***

