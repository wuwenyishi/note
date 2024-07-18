### linux docker 远程连接配置

修改Docker服务文件：
vi /lib/systemd/system/docker.service
在ExecStart后添加：-H tcp://0.0.0.0:2375 -H unix://var/run/docker.sock
重新加载配置文件、重启Docker服务
systemctl daemon-reload
systemctl restart docker

