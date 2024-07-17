
## 查看Redis当前的内存淘汰策略（本地） 

进入redis安装目录

```shell
//进入redis安装目录
cd /Users/yons/Redis-service/redis-6.2.6
//进入客户端
redis-cli
//获取当前内存淘汰策略
config get maxmemory-policy
//获取设置的Redis能使用的最大内存大小
config get maxmemory
//设置Redis最大占用内存大小为100M
config set maxmemory 100mb
```

> 内存大小：如果不设置最大内存大小或者设置最大内存大小为0，在64位操作系统下不限制内存大小，在32位操作系统下最多使用3GB内存

## Redis定义的内存淘汰策略  

- **noeviction(默认策略)** ：对于写请求不再提供服务，直接返回错误（DEL请求和部分特殊请求除外）
- **allkeys-lru**：从所有key中使用LRU算法进行淘汰
- **volatile-lru**：从设置了过期时间的key中使用LRU算法进行淘汰
- **allkeys-random**：从所有key中随机淘汰数据
- **volatile-random**：从设置了过期时间的key中随机淘汰
- **volatile-ttl**：在设置了过期时间的key中，根据key的过期时间进行淘汰，越早过期的越优先被淘汰

> 当使用**volatile-lru**、**volatile-random**、**volatile-ttl**这三种策略时，如果没有key可以被淘汰，则和**noeviction**一样返回错误

## 修改淘汰策略 

### 通过命令修改  

```shell
config set maxmemory-policy allkeys-lru
```

### 通过配置文件修改  

修改redis.conf文件

`maxmemory-policy allkeys-lru`

