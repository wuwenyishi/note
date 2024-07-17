> 原文：[Redis 高可用集群原理和实践 - 掘金](https://juejin.cn/post/7103551714879340580)

Redis 集群是 Redis 提供的分布式数据库方案，集群痛殴分片（sharding）来进行数据共享，并提供复制和故障转移能力。

![31104759](https://xuemingde.com/pages/image/2022/05/31104759.jpg)

# 集群环境搭建

Redis 集群最少需要 3 个 master 节点，这里我们搭建 3 个master 节点，3 个 slave 及节点（由于我机器配置受限，直接通过端口的方式模拟集群搭建，本处只是实验方便，**生产环境不可采取此方案**）。
环境搭建步骤如下：

1. 简单说明，首先我们先要定义集群节点的端口 `7000-7005` 然后配置文件复制 redis.conf 到对应的配置文件名。

   ![31104954](https://xuemingde.com/pages/image/2022/05/31104954.png)

   

2. 编辑 redis.conf 文件，主要修改以下的几个配置（如果需要设置密码需要配置 `requirepass`和  `masterauth`）

```shell
daemonize yes
# 这个端口和上面的配置清单一致即可
port 7000 
# 启动集群模式
cluster-enabled yes
# 集群节点信息文件，这里700x最好和port对应上
cluster-config-file nodes-7000.conf
cluster-node-timeout 5000
appendonly yes
```

3. 服务启动，注意我们需要启动所有的节点，命令如下：

   ```shell
   # 启动所有的服务 7000-7005
   cd 7000
   redis-server ./redis-7000.conf
   ```

4. 初始化集群，通过 `redis-cli --cluster create`命令初始化集群，命令如下(如果是生产环境，需要节点间 IP 以及端口是否可互相访问)：

   ```shell
   redis-cli --cluster create 127.0.0.1:7000 127.0.0.1:7001 \
   127.0.0.1:7002 127.0.0.1:7003 127.0.0.1:7004 127.0.0.1:7005 \
   --cluster-replicas 1
   ```

   

5. 集群状态查询  

   - 登录节点 `redis-cli -c -h 127.0.0.1 -p 7000`, 注意一定要加 `-c` 表示集群模式。
   - 查询集群状态  `cluster info`

   ```shell
   ➜  etc redis-cli -c -h 127.0.0.1 -p 7000                         
   127.0.0.1:7000> 
   127.0.0.1:7000> cluster info  
   cluster_state:ok
   cluster_slots_assigned:16384
   cluster_slots_ok:16384
   cluster_slots_pfail:0
   cluster_slots_fail:0
   cluster_known_nodes:6
   cluster_size:3
   cluster_current_epoch:6
   cluster_my_epoch:1
   cluster_stats_messages_ping_sent:263
   cluster_stats_messages_pong_sent:270
   cluster_stats_messages_sent:533
   cluster_stats_messages_ping_received:270
   cluster_stats_messages_pong_received:263
   cluster_stats_messages_received:533
   127.0.0.1:7000> 
   ```

   其他的集群创建方案 `utils/create-cluster`我们可以在参考资料中找到创建方式（参考文档：[redis.io/docs/manual…](https://link.juejin.cn?target=https%3A%2F%2Fredis.io%2Fdocs%2Fmanual%2Fscaling%2F%23redis-cluster-101)）



# 集群原理

## 槽指派机制

Redis Cluster 将所有数据划分为 16384 个 slots(槽位)，每个节点负责其中一部分槽位。槽位的信息存储于每个节点中。
当 Redis Cluster 的客户端来连接集群时，它也会得到一份集群的槽位配置信息并将其缓存在客户端本地。这样当客户端要查找某个 key 时，可以直接定位到目标节点。同时因为槽位的信息可能会存在客户端与服务器不一致的情况，还需要纠正机制来实现槽位信息的校验调整。

### 槽位定位算法

Cluster 默认会对 key 值使用 crc16 算法进行 hash 得到一个整数值，然后用这个整数值对 16384 进行与操作来得到具体槽位。
源码位置 `src/cluster.c` 中的 keyHashSlot 方法

```c
crc16(key,keylen) & 0x3FFF
```

为什是 16384 可以看看这篇文章 ：[Redis 为什么是 16384 个槽 ？](https://link.juejin.cn?target=https%3A%2F%2Fwww.csdn.net%2Ftags%2FMtzakg5sNDg3NzctYmxvZwO0O0OO0O0O.html)

查询某个 key 那个节点上，方法如下：

```shell
# 查询 key 所在的槽位
127.0.0.1:7000> cluster keyslot wahaha
(integer) 12318
# 查询所有的 槽分布信息
# 可以得出结论： wahaha 这个 key 在 7002 这个节点上
127.0.0.1:7000> cluster slots
1) 1) (integer) 10923
   2) (integer) 16383
   3) 1) "127.0.0.1"
      2) (integer) 7002
      3) "de631f8ac9649f5d9fb12013dc01407f953c3299"
   4) 1) "127.0.0.1"
      2) (integer) 7004
      3) "6ac6065d07eb44b674a181da897401ec4cea9571"
2) 1) (integer) 5461
   2) (integer) 10922
   3) 1) "127.0.0.1"
      2) (integer) 7001
      3) "e36fc81472afb04b9c88af1504a8e02647de1b13"
   4) 1) "127.0.0.1"
      2) (integer) 7003
      3) "a89dcff90147f6cc9425ff0c6e4bead7017dc1e1"
3) 1) (integer) 0
   2) (integer) 5460
   3) 1) "127.0.0.1"
      2) (integer) 7000
      3) "0acfc8b3dd2223333a03bbcf856dd2a839d2e072"
   4) 1) "127.0.0.1"
      2) (integer) 7005
      3) "57ddd19f38d8b748386944d15d32671fb5cb1570"
127.0.0.1:7000> 
```

其实没有这么麻烦，集群模式支持 **跳转重定位** 我们直接 get 就可以跳转过去。

### 跳转重定位

当客户端向一个错误的节点发出了指令，该节点会发现指令的 key 所在的槽位并不归自己管理，这时它会向客户端发送一个特殊的跳转指令携带目标操作的节点地址，告诉客户端去连这个节点去获取数据。客户端收到指令后除了跳转到正确的节点上去操作，还会同步更新纠正本地的槽位映射表缓存，后续所有 key 将使用新的槽位映射表。

> 大白话说：就是如果当前 key 是自己的节点的槽位就自己处理，如果不是自己的槽位，就转向目标槽位的节点。

演示一下：

```shell
set abc sdl
set sbc sdl
```

![31105832](https://xuemingde.com/pages/image/2022/05/31105832.jpg)



## 集群通讯机制

redis cluster 节点间采取 gossip 协议进行通信 
维护集群的元数据(集群节点信息，主从角色，节点数量，各节点共享的数据等)有两种方式：

- 集中式
- gossip

### 集中式

优点在于元数据的更新和读取，时效性非常好，一旦元数据出现变更立即就会更新到集中式的存储中，其他节点读取的时候立即就可以立即感知到；不足在于所有的元数据的更新压力全部集中在一个地方，可能导致元数据的存储压力。 很多中间件都会借助 zookeeper 集中式存储元数据。

![31105938](https://xuemingde.com/pages/image/2022/05/31105938.jpg)

### gossip

![31110007](https://xuemingde.com/pages/image/2022/05/31110007.jpg)



gossip 协议包含多种消息，包括ping，pong，meet，fail等等。 
meet：某个节点发送meet给新加入的节点，让新节点加入集群中，然后新节点就会开始与其他节点进行通信；
ping：每个节点都会频繁给其他节点发送ping，其中包含自己的状态还有自己维护的集群元数据，互相通过ping交换元数据(类似自己感知到的集群节点增加和移除，hash slot信息等)； 
pong: 对ping和meet消息的返回，包含自己的状态和其他信息，也可以用于信息广播和更新； 
fail: 某个节点判断另一个节点fail之后，就发送fail给其他节点，通知其他节点，指定的节点宕机了。

**gossip协议的优点在于元数据的更新比较分散，不是集中在一个地方，更新请求会陆陆续续，打到所有节点上去更新，有一定的延时，降低了压力；缺点在于元数据更新有延时可能导致集群的一些操作会有一些滞后。**

#### gossip 通信的 10000 端口

每个节点都有一个专门用于节点间gossip通信的端口，就是自己提供服务的端口号+10000，比如7001，那么用于节点间通信的就是17001端口。 每个节点每隔一段时间都会往另外几个节点发送ping消息，同时其他几点接收到ping消息之后返回pong消息。

#### 网络抖动

真实世界的机房网络往往并不是风平浪静的，它们经常会发生各种各样的小问题。比如网络抖动就是非常常见的一种现象，突然之间部分连接变得不可访问，然后很快又恢复正常。
为解决这种问题，Redis Cluster 提供了一种选项cluster-node-timeout，表示当某个节点持续 timeout 的时间失联时，才可以认定该节点出现故障，需要进行主从切换。如果没有这个选项，网络抖动会导致主从频繁切换 (数据的重新复制)。



## 集群选举原理

当slave发现自己的master变为FAIL状态时，便尝试进行Failover，以期成为新的master。由于挂掉的master可能会有多个slave，从而存在多个slave竞争成为master节点的过程， 其过程如下：

1. slave发现自己的master变为FAIL
2. 将自己记录的集群currentEpoch加1，并广播FAILOVER_AUTH_REQUEST 信息
3. 其他节点收到该信息，只有master响应，判断请求者的合法性，并发送FAILOVER_AUTH_ACK，对每一个epoch只发送一次ack
4. 尝试failover的slave收集master返回的FAILOVER_AUTH_ACK
5. slave收到超过半数master的ack后变成新Master(这里解释了集群为什么至少需要三个主节点，如果只有两个，当其中一个挂了，只剩一个主节点是不能选举成功的)
6. slave广播Pong消息通知其他集群节点。

从节点并不是在主节点一进入 FAIL 状态就马上尝试发起选举，而是有一定延迟，一定的延迟确保我们等待FAIL状态在集群中传播，slave如果立即尝试选举，其它masters或许尚未意识到FAIL状态，可能会拒绝投票

延迟计算公式：

> DELAY = 500ms + random(0 ~ 500ms) + SLAVE_RANK * 1000ms

SLAVE_RANK表示此slave已经从master复制数据的总量的rank。Rank越小代表已复制的数据越新。这种方式下，持有最新数据的slave将会首先发起选举（理论上）。

### 集群脑裂数据丢失问题

Redis 集群没有过半机制会有脑裂问题，网络分区导致脑裂后多个主节点对外提供写服务，一旦网络分区恢复，会将其中一个主节点变为从节点，这时会有大量数据丢失。

规避方法可以在 redis 配置里加上参数(这种方法不可能百分百避免数据丢失，参考集群leader选举机制)：

```shell
// 写数据成功最少同步的slave数量，这个数量可以模仿大于半数机制配置，
// 比如集群总共三个节点可以配置1，加上leader就是2，超过了半数
min-replicas-to-write 1  
```

注意：这个配置在一定程度上会影响集群的可用性，比如slave要是少于1个，这个集群就算leader正常也不能提供服务了，需要具体场景权衡选择。

### 集群是否完整才能对外提供服务

当redis.conf的配置cluster-require-full-coverage为no时，表示当负责一个插槽的主库下线
且没有相应的从库进行故障恢复时，集群仍然可用，如果为yes则集群不可用(默认为 yes)。

![31110357](https://xuemingde.com/pages/image/2022/05/31110357.jpg)

### Redis 集群为什么至少需要三个master节点，并且推荐节点数为奇数？

因为新master的选举需要大于半数的集群master节点同意才能选举成功，如果只有两个master节点，当其中一个挂了，是达不到选举新master的条件的。

奇数个master节点可以在满足选举该条件的基础上节省一个节点，比如三个master节点和四个master节点的集群相比，大家如果都挂了一个master节点都能选举新master节点，如果都挂了两个master节点都没法选举新master节点了，所以奇数的master节点更多的是**从节省机器资源角度出发**说的。

### Redis 集群对批量操作命令的支持

![31110442](https://xuemingde.com/pages/image/2022/05/31110442.jpg)

如何让多个 key 落到一个槽里面 ？
对于类似mset，mget这样的多个key的原生批量操作命令，redis集群只支持所有key落在同一slot的情况，如果有多个key**一定要用mset命令在redis集群上操作，则可以在key的前面加上{XX}，这样参数数据分片hash计算的只会是大括号里的值，这样能确保不同的key能落到同一slot里去**，示例如下：

> mset {user1}:1:name zhangsan {user1}:1:age 18

![31110518](https://xuemingde.com/pages/image/2022/05/31110518.jpg)

假设name和age计算的hash slot值不一样，但是这条命令在集群下执行，redis只会用大括号里的 user1 做hash slot计算，所以算出来的slot值肯定相同，最后都能落在同一slot。

### 对比: 哨兵leader选举流程

当一个master服务器被某sentinel视为下线状态后，该sentinel会与其他sentinel协商选出sentinel的leader进行故障转移工作。每个发现master服务器进入下线的sentinel都可以要求其他sentinel选自己为sentinel的leader，选举是先到先得。同时每个sentinel每次选举都会自增配置纪元(选举周期)，每个纪元中只会选择一个sentinel的leader。如果所有超过一半的sentinel选举某sentinel作为leader。之后该sentinel进行故障转移操作，从存活的slave中选举出新的master，这个选举过程跟集群的master选举很类似。

哨兵集群只有一个哨兵节点，redis的主从也能正常运行以及选举master，如果master挂了，那唯一的那个哨兵节点就是哨兵leader了，可以正常选举新master。

不过为了高可用一般都推荐至少部署三个哨兵节点。为什么推荐奇数个哨兵节点原理跟集群奇数个master节点类似。

# 集群故障转移

Redis 集群中的节点分为主节点（master）和从节点（slave）,其中主节点用于处理槽，而从节点则用于复制某个主节点，并且在复制主节点下线时，代替主节点继续处理命令请求。

## 故障检测

集群中的每个节点都会**定期**地向集群中的其他节点发送PING消息，以此来检测对方是否在线，如果接收PING消息的节点没有在规定的时间内，向发送PING消息的节点返回PONG消息，那么发送PING消息的节点就会将接收PING消息的节点标记为**疑似下线（probable fail，PFAIL）**。

如果在一个集群里面，半数以上负责处理槽的主节点都将某个主节点x报告为疑似下线，那么这个主节点x将被标记为**已下线（FAIL）**，将主节点x标记为已下线的节点会向集群广播一条关于主节点x的FAIL消息，所有收到这条FAIL消息的节点都会立即将主节点x标记为已下线。

举个例子，主节点7002和主节点7003都认为主节点7000进入了下线状态，并且主节点7001也认为主节点7000进入了疑似下线状态，在集群四个负责处理槽的主节点里面，有三个都将主节点7000标记为下线，数量已经超过了半数，所以主节点7001会将主节点7000标记为已下线，并向集群广播一条关于主节点7000的FAIL消息，如图所示：

![31110629](https://xuemingde.com/pages/image/2022/05/31110629.jpg)

## 故障转移

当一个从节点发现自己正在复制的主节点进入了已下线状态时，从节点将开始对下线主节点进行故障转移，以下是故障转移的执行步骤：

1. 复制下线主节点的所有从节点里面，会有一个从节点被选中。
2. 被选中的从节点会执行`SLAVEOF no one`命令，成为新的主节点。
3. 新的主节点会撤销所有对已下线主节点的槽指派，并将这些槽全部指派给自己。
4. 新的主节点向集群广播一条PONG消息，这条PONG消息可以让集群中的其他节点立即知道这个节点已经由从节点变成了主节点，并且这个主节点已经接管了原本由已下线节点负责处理的槽。
5. 新的主节点开始接收和自己负责处理的槽有关的命令请求，故障转移完成。



## 重新选择新节点

新的主节点是通过选举产生的。以下是集群选举新的主节点的方法：

1. 集群的配置纪元是一个自增计数器，它的初始值为0。
2. 当集群里的某个节点开始一次故障转移操作时，集群配置纪元的值会被增一。
3. 对于每个配置纪元，集群里每个负责处理槽的主节点都有一次投票的机会，而第一个向主节点要求投票的从节点将获得主节点的投票。
4. 当从节点发现自己正在复制的主节点进入已下线状态时，从节点会向集群广播一条CLUSTERMSG_TYPE_FAILOVER_AUTH_REQUEST消息，要求所有收到这条消息、并且具有投票权的主节点向这个从节点投票。
5. 如果一个主节点具有投票权（它正在负责处理槽），并且这个主节点尚未投票给其他从节点，那么主节点将向要求投票的从节点返回一条CLUSTERMSG_TYPE_FAILOVER_AUTH_ACK消息，表示这个主节点支持从节点成为新的主节点。
6. 每个参与选举的从节点都会接收CLUSTERMSG_TYPE_FAILOVER_AUTH_ACK消息，并根据自己收到了多少条这种消息来统计自己获得了多少主节点的支持。
7. 如果集群里有N个具有投票权的主节点，那么当一个从节点收集到大于等于N/2+1张支持票时，这个从节点就会当选为新的主节点。
8. 因为在每一个配置纪元里面，每个具有投票权的主节点只能投一次票，所以如果有N个主节点进行投票，那么具有大于等于N/2+1张支持票的从节点只会有一个，这确保了新的主节点只会有一个。
9. 如果在一个配置纪元里面没有从节点能收集到足够多的支持票，那么集群进入一个新的配置纪元，并再次进行选举，直到选出新的主节点为止。

Cluster 选举新主节点的方法和 Sentinel 的方法非常相似，因为两者都是基于 Raft 算法的领头选举（leader election）方法来实现的。

# 水平拓展



## 拓展主节点

集群状态查询

```shell
127.0.0.1:7000> cluster nodes 
6ac6065d07eb44b674a181da897401ec4cea9571 127.0.0.1:7004@17004 slave de631f8ac9649f5d9fb12013dc01407f953c3299 0 1653897401000 3 connected
de631f8ac9649f5d9fb12013dc01407f953c3299 127.0.0.1:7002@17002 master - 0 1653897402475 3 connected 10923-16383
a89dcff90147f6cc9425ff0c6e4bead7017dc1e1 127.0.0.1:7003@17003 slave e36fc81472afb04b9c88af1504a8e02647de1b13 0 1653897401956 2 connected
e36fc81472afb04b9c88af1504a8e02647de1b13 127.0.0.1:7001@17001 master - 0 1653897401541 2 connected 5461-10922
0acfc8b3dd2223333a03bbcf856dd2a839d2e072 127.0.0.1:7000@17000 myself,master - 0 1653897400000 1 connected 0-5460
57ddd19f38d8b748386944d15d32671fb5cb1570 127.0.0.1:7005@17005 slave 0acfc8b3dd2223333a03bbcf856dd2a839d2e072 0 1653897401000 1 connected
127.0.0.1:7000> 

```



### 拓展规划

我们在原始集群基础上再增加一主(7006)一从(7007)

![31110722](https://xuemingde.com/pages/image/2022/05/31110722.png)

启动节点

```shell
redis-server redis-7006.conf
redis-server redis-7007.conf        

```



### 集群中加入节点

加入集群

```shell
➜  etc redis-cli --cluster add-node 127.0.0.1:7006 127.0.0.1:7001
>>> Adding node 127.0.0.1:7006 to cluster 127.0.0.1:7001
>>> Performing Cluster Check (using node 127.0.0.1:7001)
M: e36fc81472afb04b9c88af1504a8e02647de1b13 127.0.0.1:7001
   slots:[5461-10922] (5462 slots) master
   1 additional replica(s)
S: a89dcff90147f6cc9425ff0c6e4bead7017dc1e1 127.0.0.1:7003
   slots: (0 slots) slave
   replicates e36fc81472afb04b9c88af1504a8e02647de1b13
S: 6ac6065d07eb44b674a181da897401ec4cea9571 127.0.0.1:7004
   slots: (0 slots) slave
   replicates de631f8ac9649f5d9fb12013dc01407f953c3299
S: 57ddd19f38d8b748386944d15d32671fb5cb1570 127.0.0.1:7005
   slots: (0 slots) slave
   replicates 0acfc8b3dd2223333a03bbcf856dd2a839d2e072
M: de631f8ac9649f5d9fb12013dc01407f953c3299 127.0.0.1:7002
   slots:[10923-16383] (5461 slots) master
   1 additional replica(s)
M: 0acfc8b3dd2223333a03bbcf856dd2a839d2e072 127.0.0.1:7000
   slots:[0-5460] (5461 slots) master
   1 additional replica(s)
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
>>> Send CLUSTER MEET to node 127.0.0.1:7006 to make it join the cluster.
[OK] New node added correctly.
➜  etc 

```

如下，7006加入了集群，并且默认是一个master节点：

```shell
➜  etc redis-cli -p 7001 cluster nodes
2109c2832177e8514174c6ef8fefd681076e28df 127.0.0.1:7006@17006 master - 0 1653919429534 0 connected
a89dcff90147f6cc9425ff0c6e4bead7017dc1e1 127.0.0.1:7003@17003 slave e36fc81472afb04b9c88af1504a8e02647de1b13 0 1653919430000 2 connected
6ac6065d07eb44b674a181da897401ec4cea9571 127.0.0.1:7004@17004 slave de631f8ac9649f5d9fb12013dc01407f953c3299 0 1653919430549 3 connected
57ddd19f38d8b748386944d15d32671fb5cb1570 127.0.0.1:7005@17005 slave 0acfc8b3dd2223333a03bbcf856dd2a839d2e072 0 1653919430549 1 connected
de631f8ac9649f5d9fb12013dc01407f953c3299 127.0.0.1:7002@17002 master - 0 1653919429129 3 connected 10923-16383
0acfc8b3dd2223333a03bbcf856dd2a839d2e072 127.0.0.1:7000@17000 master - 0 1653919430549 1 connected 0-5460
e36fc81472afb04b9c88af1504a8e02647de1b13 127.0.0.1:7001@17001 myself,master - 0 1653919429000 2 connected 5461-10922

```



### 设置和迁移分片

为集群分配分片

```shell
redis-cli --cluster reshard 127.0.0.1:7001

```

在执行过程中会询问，计划迁移槽数，迁移数据目标，以及迁移数据来源。

![31110818](https://xuemingde.com/pages/image/2022/05/31110818.jpg)

重新分配后的结果查询 `redis-cli -p 7001 cluster nodes`

![31110858](https://xuemingde.com/pages/image/2022/05/31110858.jpg)

配置从 7006 的从节点 7007,  同样也是先执行加入集群的命令

```shell
redis-cli --cluster add-node 127.0.0.1:7007 127.0.0.1:7001

```



### 设置从节点

我们需要执行 replicate 命令来指定当前节点(从节点)的主节点id为哪个,首先需要连接新加的7007节点的客户端，然后使用集群命令进行操作，把当前的7007(slave)节点指定到一个主节点下(这里使用之前创建的7006主节点)

具体的命令如下：

```shell
# 查询集群节点
➜  etc redis-cli -p 7001 cluster nodes                           
2109c2832177e8514174c6ef8fefd681076e28df 127.0.0.1:7006@17006 master - 0 1653920562718 7 connected 0-1332 5461-6794 10923-12255
a89dcff90147f6cc9425ff0c6e4bead7017dc1e1 127.0.0.1:7003@17003 slave e36fc81472afb04b9c88af1504a8e02647de1b13 0 1653920561710 2 connected
6ac6065d07eb44b674a181da897401ec4cea9571 127.0.0.1:7004@17004 slave de631f8ac9649f5d9fb12013dc01407f953c3299 0 1653920562000 3 connected
57ddd19f38d8b748386944d15d32671fb5cb1570 127.0.0.1:7005@17005 slave 0acfc8b3dd2223333a03bbcf856dd2a839d2e072 0 1653920562516 1 connected
8d935918d877a63283e1f3a1b220cdc8cb73c414 127.0.0.1:7007@17007 master - 0 1653920563000 0 connected
de631f8ac9649f5d9fb12013dc01407f953c3299 127.0.0.1:7002@17002 master - 0 1653920563000 3 connected 12256-16383
0acfc8b3dd2223333a03bbcf856dd2a839d2e072 127.0.0.1:7000@17000 master - 0 1653920561506 1 connected 1333-5460
e36fc81472afb04b9c88af1504a8e02647de1b13 127.0.0.1:7001@17001 myself,master - 0 1653920562000 2 connected 6795-10922

# 登录到新加节点
➜  etc redis-cli -p 7007
127.0.0.1:7007> 
# 制定当前当前节点的主节点 7006
127.0.0.1:7007> cluster replicate 2109c2832177e8514174c6ef8fefd681076e28df
OK
# 重新查询集群状态
127.0.0.1:7007> cluster nodes
57ddd19f38d8b748386944d15d32671fb5cb1570 127.0.0.1:7005@17005 slave 0acfc8b3dd2223333a03bbcf856dd2a839d2e072 0 1653920688403 1 connected
8d935918d877a63283e1f3a1b220cdc8cb73c414 127.0.0.1:7007@17007 myself,slave 2109c2832177e8514174c6ef8fefd681076e28df 0 1653920685000 7 connected
2109c2832177e8514174c6ef8fefd681076e28df 127.0.0.1:7006@17006 master - 0 1653920688000 7 connected 0-1332 5461-6794 10923-12255
de631f8ac9649f5d9fb12013dc01407f953c3299 127.0.0.1:7002@17002 master - 0 1653920687000 3 connected 12256-16383
0acfc8b3dd2223333a03bbcf856dd2a839d2e072 127.0.0.1:7000@17000 master - 0 1653920687000 1 connected 1333-5460
6ac6065d07eb44b674a181da897401ec4cea9571 127.0.0.1:7004@17004 slave de631f8ac9649f5d9fb12013dc01407f953c3299 0 1653920688099 3 connected
a89dcff90147f6cc9425ff0c6e4bead7017dc1e1 127.0.0.1:7003@17003 slave e36fc81472afb04b9c88af1504a8e02647de1b13 0 1653920688504 2 connected
e36fc81472afb04b9c88af1504a8e02647de1b13 127.0.0.1:7001@17001 master - 0 1653920687392 2 connected 6795-10922

```

最后我们再次查询，发现 7007 成功变成了 7008 的主节点

## 主节点下线

彻底删除主节点，因为主节点中存在数据，所以我们可以分为两个步骤操作

- 数据迁移
- 节点下线

为了方便验证，我先设置一个数据

```shell
127.0.0.1:7001> set sdl 123
-> Redirected to slot [11164] located at 127.0.0.1:7006
OK
127.0.0.1:7006> get sdl
"123"
```

先下先从节点, 执行一下命令：
`redis-cli --cluster del-node 127.0.0.1:7007 8d935918d877a63283e1f3a1b220cdc8cb73c414`

### 数据迁出

```shell
➜  etc redis-cli --cluster reshard 127.0.0.1:7001
>>> Performing Cluster Check (using node 127.0.0.1:7001)
M: e36fc81472afb04b9c88af1504a8e02647de1b13 127.0.0.1:7001
   slots:[10212-10922] (711 slots) master
   1 additional replica(s)
M: 2109c2832177e8514174c6ef8fefd681076e28df 127.0.0.1:7006
   slots:[11471-12255] (785 slots) master
S: a89dcff90147f6cc9425ff0c6e4bead7017dc1e1 127.0.0.1:7003
   slots: (0 slots) slave
   replicates e36fc81472afb04b9c88af1504a8e02647de1b13
S: 6ac6065d07eb44b674a181da897401ec4cea9571 127.0.0.1:7004
   slots: (0 slots) slave
   replicates de631f8ac9649f5d9fb12013dc01407f953c3299
S: 57ddd19f38d8b748386944d15d32671fb5cb1570 127.0.0.1:7005
   slots: (0 slots) slave
   replicates 0acfc8b3dd2223333a03bbcf856dd2a839d2e072
M: de631f8ac9649f5d9fb12013dc01407f953c3299 127.0.0.1:7002
   slots:[15624-16383] (760 slots) master
   1 additional replica(s)
M: 0acfc8b3dd2223333a03bbcf856dd2a839d2e072 127.0.0.1:7000
   slots:[0-10211],[10923-11470],[12256-15623] (14128 slots) master
   1 additional replica(s)
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
How many slots do you want to move (from 1 to 16384)? 785
What is the receiving node ID? 0acfc8b3dd2223333a03bbcf856dd2a839d2e072
Please enter all the source node IDs.
  Type 'all' to use all the nodes as source nodes for the hash slots.
  Type 'done' once you entered all the source nodes IDs.
Source node #1: 2109c2832177e8514174c6ef8fefd681076e28df
Source node #2: done

```

查询节点槽信息
![31111010](https://xuemingde.com/pages/image/2022/05/31111010.jpg)

节点下线

```shell
redis-cli --cluster del-node 127.0.0.1:7006 2109c2832177e8514174c6ef8fefd681076e28df
```

执行后结果如下：

![31111039](https://xuemingde.com/pages/image/2022/05/31111039.jpg)

# 参考资料

- 《Redis 设计与实现》黄健宏
- [Redis 为什么是 16384 个槽 ？](https://link.juejin.cn?target=https%3A%2F%2Fwww.csdn.net%2Ftags%2FMtzakg5sNDg3NzctYmxvZwO0O0OO0O0O.html)
- [blog.csdn.net/wanderstarr…](https://link.juejin.cn?target=https%3A%2F%2Fblog.csdn.net%2Fwanderstarrysky%2Farticle%2Fdetails%2F118157751)
- [segmentfault.com/a/119000003…](https://link.juejin.cn?target=https%3A%2F%2Fsegmentfault.com%2Fa%2F1190000038373546)
- 部分图片来源于网络，如有侵权请留言



