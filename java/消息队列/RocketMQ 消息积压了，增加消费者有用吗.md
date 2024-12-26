**面试官**：RocketMQ 消息积压了，增加消费者有用吗？

**我**：这个要看具体的场景，不同的场景下情况是不一样的。

**面试官**：可以详细说一下吗？

**我**：如果消费者的数量小于 MessageQueue 的数量，增加消费者可以加快消息消费速度，减少消息积压。比如一个 Topic 有 4 个 MessageQueue，2 个消费者进行消费，如果增加一个消费者，明细可以加快拉取消息的频率。如下图：

![图片](https://github.com/wuwenyishi/pages/raw/gh-pages/image/2022/04/11/1057-EAQoGV.png)

如果消费者的数量大于等于 MessageQueue 的数量，增加消费者是没有用的。比如一个 Topic 有 4 个 MessageQueue，并且有 4 个消费者进行消费。如下图

![图片](https://github.com/wuwenyishi/pages/raw/gh-pages/image/2022/04/11/1057-Uu40Ny.png)

**面试官**：你说的第一种情况，增加消费者一定能加快消息消费的速度吗？

**我**：这...，一般情况下是可以的。

**面试官**：有特殊的情况吗？

**我**：当然有。消费者消息拉取的速度也取决于本地消息的消费速度，如果本地消息消费的慢，就会延迟一段时间后再去拉取。

**面试官**：在什么情况下消费者会延迟一段时间后后再去拉取呢？

**我**：消费者拉取的消息存在 ProcessQueue，消费者是有流量控制的，如果出现下面三种情况，就不会主动去拉取：

- ProcessQueue 保存的消息数量超过阈值（默认 1000，可以配置）；
- ProcessQueue 保存的消息大小超过阈值（默认 100M，可以配置）；
- 对于非顺序消费的场景，ProcessQueue 中保存的最后一条和第一条消息偏移量之差超过阈值（默认 2000，可以配置）。

> 这部分源码请参考类：`org.apache.rocketmq.client.impl.consumer.DefaultMQPushConsumerImpl`。

**面试官**：还有其他情况吗？

**我**：对于顺序消费的场景，ProcessQueue 加锁失败，也会延迟拉取，这个延迟时间是 3s。消费者消费慢，可是能下面的原因：

- 消费者处理的业务逻辑复杂，耗时很长；
- 消费者有慢查询，或者数据库负载高导致响应慢；
- 缓存等中间件响应慢，比如 Redis 响应慢；
- 调用外部服务接口响应慢。

**面试官**：对于外部接口响应慢的情况，有什么应对措施吗？

**我**：这个要分情况讨论。

如果调用外部系统`只是一个通知，或者调用外部接口的结果并不处理`，可以采用异步的方式，异步逻辑里采用重试的方式保证接口调成功。

如果外部接口返回结果必须要处理，可以考虑接口返回的结果是否可以缓存默认值（要考虑业务可行），在调用失败后采用快速降级的方式，使用默认值替代返回接口返回值。

如果这个接口返回结果必须要处理，并且不能缓存，可以把拉取到的消息存入本地然后给 Broker 直接返回 CONSUME_SUCCESS。等外部系统恢复正常后再从本地取出来进行处理。

**面试官**：如果消费者数小于 MessageQueue 数量，并且外部系统响应正常，为了快速消费积压消息而增加消费者，有什么需要考虑的吗？

**我**：外部系统虽然响应正常，但是增加多个消费者后，外部系统的接口调用量会突增，如果达到吞吐量上限，外部系统会响应变慢，甚至被打挂。

同时也要考虑本地数据库、缓存的压力，如果数据库响应变慢，处理消息的速度就会变慢，起不到缓解消息积压的作用。

**面试官**：新增加了消费者后，怎么给它分配 MessageQueue 呢？

**我**：Consumer 在拉取消息之前，需要对 MessageQueue 进行负载操作。RocketMQ 使用一个定时器来完成负载操作，默认每间隔 20s 重新负载一次。

**面试官**：能详细说一下都有哪些负载策略吗？

**我**：RocketMQ 提供了 6 种负载策略，依次来看一下。

**平均负载策略：**

1. 把消费者进行排序；
2. 计算每个消费者可以平均分配的 MessageQueue 数量；
3. 如果消费者数量大于 MessageQueue 数量，多出的消费者就分不到；
4. 如果不可以平分，就使用 MessageQueue 总数量对消费者数量求余数 mod；
5. 对前 mod 数量消费者，每个消费者加一个，这样就获取到了每个消费者分配的 MessageQueue 数量。

比如 4 个 MessageQueue 和 3 个消费者的情况：

**面试官**：消费者延迟拉取消息，一般可能是什么原因导致的呢？

**我**：其实延迟拉取的`本质就是消费者消费慢`，导致下次去拉取的时候 ProcessQueue 中积压的消息超过阈值。以下面这张架构图为例：

![图片](https://github.com/wuwenyishi/pages/raw/gh-pages/image/2022/04/11/1059-yKOaQS.png)

消费者消费慢，可是能下面的原因：

- 消费者处理的业务逻辑复杂，耗时很长；
- 消费者有慢查询，或者数据库负载高导致响应慢；
- 缓存等中间件响应慢，比如 Redis 响应慢；
- 调用外部服务接口响应慢。

**面试官**：对于外部接口响应慢的情况，有什么应对措施吗？

**我**：这个要分情况讨论。

如果调用外部系统**只是一个通知，或者调用外部接口的结果并不处理**，可以采用异步的方式，异步逻辑里采用重试的方式保证接口调成功。

如果外部接口返回结果必须要处理，可以考虑接口返回的结果是否可以缓存默认值（要考虑业务可行），在调用失败后采用快速降级的方式，使用默认值替代返回接口返回值。

如果这个接口返回结果必须要处理，并且不能缓存，可以把拉取到的消息存入本地然后给 Broker 直接返回 CONSUME_SUCCESS。等外部系统恢复正常后再从本地取出来进行处理。

**面试官**：如果消费者数小于 MessageQueue 数量，并且外部系统响应正常，为了快速消费积压消息而增加消费者，有什么需要考虑的吗？

**我**：外部系统虽然响应正常，但是增加多个消费者后，外部系统的接口调用量会突增，如果达到吞吐量上限，外部系统会响应变慢，甚至被打挂。

同时也要考虑本地数据库、缓存的压力，如果数据库响应变慢，处理消息的速度就会变慢，起不到缓解消息积压的作用。

**面试官**：新增加了消费者后，怎么给它分配 MessageQueue 呢？

**我**：Consumer 在拉取消息之前，需要对 MessageQueue 进行负载操作。RocketMQ 使用一个定时器来完成负载操作，默认每间隔 20s 重新负载一次。

**面试官**：能详细说一下都有哪些负载策略吗？

**我**：RocketMQ 提供了 6 种负载策略，依次来看一下。

**平均负载策略：**

1. 把消费者进行排序；
2. 计算每个消费者可以平均分配的 MessageQueue 数量；
3. 如果消费者数量大于 MessageQueue 数量，多出的消费者就分不到；
4. 如果不可以平分，就使用 MessageQueue 总数量对消费者数量求余数 mod；
5. 对前 mod 数量消费者，每个消费者加一个，这样就获取到了每个消费者分配的 MessageQueue 数量。

比如 4 个 MessageQueue 和 3 个消费者的情况：

![图片](https://github.com/wuwenyishi/pages/raw/gh-pages/image/2022/04/11/1103-dHKsfY.png)

源代码的逻辑非常简单，如下：

```java
// AllocateMessageQueueAveragely 这个类
// 4 个 MessageQueue 和 3 个消费者的情况，假如第一个，index = 0
int index = cidAll.indexOf(currentCID);
// mod = 1
int mod = mqAll.size() % cidAll.size();
// averageSize = 2
int averageSize =
    mqAll.size() <= cidAll.size() ? 1 : (mod > 0 && index < mod ? mqAll.size() / cidAll.size()
                                         + 1 : mqAll.size() / cidAll.size());
// startIndex = 0
int startIndex = (mod > 0 && index < mod) ? index * averageSize : index * averageSize + mod;
// range = 2,所以第一个消费者分配到了2个
int range = Math.min(averageSize, mqAll.size() - startIndex);
for (int i = 0; i < range; i++) {
    result.add(mqAll.get((startIndex + i) % mqAll.size()));
}
```

**循环分配策略:**

这个很容易理解，遍历消费者，把 MessageQueue 分一个给遍历到的消费者，如果 MessageQueue 数量比消费者多，需要进行多次遍历，遍历次数等于  （MessageQueue 数量/消费者数量），还是以 4 个 MessageQueue 和 3 个消费者的情况，如下图：

![图片](https://github.com/wuwenyishi/pages/raw/gh-pages/image/2022/04/11/1103-4fvcvc.png)

```java
//AllocateMessageQueueAveragelyByCircle 这个类
//4 个 MessageQueue 和 3 个消费者的情况，假如第一个，index = 0
int index = cidAll.indexOf(currentCID);
for (int i = index; i < mqAll.size(); i++) {
    if (i % cidAll.size() == index) {
        //i == 0 或者 i == 3 都会走到这里
        result.add(mqAll.get(i));
    }
}
```

**自定义分配策略：**

这种策略在消费者启动的时候可以指定消费哪些 MessageQueue。可以参考下面代码：

```java
AllocateMessageQueueByConfig allocateMessageQueueByConfig = new AllocateMessageQueueByConfig();
//绑定消费 messageQueue1
allocateMessageQueueByConfig.setMessageQueueList(Arrays.asList(new MessageQueue("messageQueue1","broker1",0)));
consumer.setAllocateMessageQueueStrategy(allocateMessageQueueByConfig);
consumer.start();
```

**按照机房分配策略：**

这种方式 Consumer 只消费指定机房的 MessageQueue，如下图：Consumer0、Consumer1、Consumer2 绑定 room1 和 room2 这两个机房，而 room3 这个机房没有消费者。

![图片](https://github.com/wuwenyishi/pages/raw/gh-pages/image/2022/04/11/1104-xQcBq5.png)

Consumer 启动的时候需要绑定机房名称。可以参考下面代码：

```java
AllocateMessageQueueByMachineRoom allocateMessageQueueByMachineRoom = new AllocateMessageQueueByMachineRoom();
//绑定消费 room1 和 room2 这两个机房
allocateMessageQueueByMachineRoom.setConsumeridcs(new HashSet<>(Arrays.asList("room1","room2")));
consumer.setAllocateMessageQueueStrategy(allocateMessageQueueByMachineRoom);
consumer.start();
```

这种策略 broker 的命名必须按照格式：机房名@brokerName，因为消费者分配队列的时候，首先按照机房名称过滤出所有的 MessageQueue，然后`再按照平均分配策略进行分配`。

```java
//AllocateMessageQueueByMachineRoom 这个类
List<MessageQueue> premqAll = new ArrayList<MessageQueue>();
for (MessageQueue mq : mqAll) {
    String[] temp = mq.getBrokerName().split("@");
    if (temp.length == 2 && consumeridcs.contains(temp[0])) {
        premqAll.add(mq);
    }
}
//上面按照机房名称过滤出所有的 MessageQueue 放入premqAll，后面就是平均分配策略
```

**按照机房就近分配：**

跟按照机房分配原则相比，就近分配的好处是可以对没有消费者的机房进行分配。如下图，机房 3 的 MessageQueue 也分配到了消费者：

![图片](https://github.com/wuwenyishi/pages/raw/gh-pages/image/2022/04/11/1105-GcF8UO.png)

如果一个机房没有消费者，则会把这个机房的 MessageQueue 分配给集群中所有的消费者。

> 源码所在类：`AllocateMachineRoomNearby`。

**一致性 Hash 算法策略：**

把所有的消费者经过 Hash 计算分布到 Hash 环上，对所有的 MessageQueue  进行 Hash  计算，找到顺时针方向最近的消费者节点进行绑定。如下图：

![图片](https://github.com/wuwenyishi/pages/raw/gh-pages/image/2022/04/11/1105-5VDzq8.png)


源代码如下：

```java
//所在类 AllocateMessageQueueConsistentHash
Collection<ClientNode> cidNodes = new ArrayList<ClientNode>();
for (String cid : cidAll) {
    cidNodes.add(new ClientNode(cid));
}
//使用消费者构建 Hash 环，把消费者分布在 Hash 环节点上
final ConsistentHashRouter<ClientNode> router; //for building hash ring
if (customHashFunction != null) {
    router = new ConsistentHashRouter<ClientNode>(cidNodes, virtualNodeCnt, customHashFunction);
} else {
    router = new ConsistentHashRouter<ClientNode>(cidNodes, virtualNodeCnt);
}
//对 MessageQueue 做 Hash 运算，找到环上距离最近的消费者
List<MessageQueue> results = new ArrayList<MessageQueue>();
for (MessageQueue mq : mqAll) {
    ClientNode clientNode = router.routeNode(mq.toString());
    if (clientNode != null && currentCID.equals(clientNode.getKey())) {
        results.add(mq);
    }
}
```

**面试官**：恭喜你，通过了。



***

> 原文：[RocketMQ 消息积压了，增加消费者有用吗？](https://mp.weixin.qq.com/s/Uqy13E9qYAON43h3XFUn3g)