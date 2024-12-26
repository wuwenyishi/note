
### 支持的数据结构

1. String字符串  

存储的String数据是二进制安全的，是最进本的数据类型，一个键最大存储512M。

应用场景：很常见的场景用于统计网站访问数量，当前在线人数等（incr命令自增。decr命令自减）

![image](https://github.com/wuwenyishi/pages/raw/gh-pages/image/others/1605594981645-3560373a-80a8-4ed9-b098-bbd07f7a6daf.png)


1. Hash

Redis中的散列可以看成具有String key和String value的map容器，可以将多个key-value存储到一个key中。每一个Hash可以存储4294967295个键值对。

应用场景：例如存储、读取、修改用户属性（name，age，pwd等）

![image](https://github.com/wuwenyishi/pages/raw/gh-pages/image/others/1605594999435-52838f6c-a296-482f-bbe3-0458bc1d69dd.png)



1. List

Redis的列表允许用户从序列的两端推入或者弹出元素，列表由多个字符串值组成的有序可重复的序列，是链表结构，所以向列表两端添加元素的时间复杂度为0(1)，获取越接近两端的元素速度就越快。List中可以包含的最大元素数量是4294967295。

应用场景：1.最新消息排行榜。2.消息队列，以完成多程序之间的消息交换。可以用push操作将任务存在list中（生产者），然后线程在用pop操作将任务取出进行执行。（消费者）

![image](https://github.com/wuwenyishi/pages/raw/gh-pages/image/others/1605594989795-94de03ed-d934-447d-8503-0e9bea7b42f0.png)



1. 集合Set



Redis的集合是无序不可重复的，和列表一样，在执行插入和删除和判断是否存在某元素时，效率是很高的。集合最大的优势在于可以进行交集并集差集操作。Set可包含的最大元素数量是4294967295。

应用场景：1.利用交集求共同好友。2.利用唯一性，可以统计访问网站的所有独立IP。3.好友推荐的时候根据tag求交集，大于某个threshold（临界值的）就可以推荐。

![image](https://github.com/wuwenyishi/pages/raw/gh-pages/image/others/1605595010088-ed456003-9a56-4631-9ccd-666d4527287b.png)



1. 有序集合sorted set

和set很像，都是字符串的集合，都不允许重复的成员出现在一个set中。他们之间差别在于有序集合中每一个成员都会有一个分数(score)与之关联，Redis正是通过分数来为集合中的成员进行从小到大的排序。尽管有序集合中的成员必须是卫衣的，但是分数(score)却可以重复。

应用场景：可以用于一个大型在线游戏的积分排行榜，每当玩家的分数发生变化时，可以执行zadd更新玩家分数(score)，此后在通过zrange获取几分top ten的用户信息。



![image](https://github.com/wuwenyishi/pages/raw/gh-pages/image/others/1605595082580-7b4529d5-6260-4901-97cb-3542fc6c48a3.png)



> 最后，还有个对key的通用操作，所有的数据类型都可以使用的

![image](https://github.com/wuwenyishi/pages/raw/gh-pages/image/others/1605595240335-fef65e1b-6f04-432d-b59b-d3275920d741.png)

