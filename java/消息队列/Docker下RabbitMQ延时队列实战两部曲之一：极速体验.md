> 原文： [Docker下RabbitMQ延时队列实战两部曲之一：极速体验](https://xie.infoq.cn/article/1140b48cb9a1e7dbfcf855698)

![](https://github.com/wuwenyishi/pages/raw/gh-pages/image/2022/06/03/1105OoFOJe.jpg)



### 关于延时队列

有的应用场景中，向 RabbitMQ 发出消息后，我们希望消费方不要立即消费，可以通过延时队列来实现，思路是将消息发送到 A  队列，此队列没有消费者，等消息过期后会进入 A 队列的 Dead Letter Exchange 中，B 队列绑定了这个 Dead Letter Exchange，消费方只要消费 B 队列的消息，就能实现延时消费了，如下图所示：

![](https://github.com/wuwenyishi/pages/raw/gh-pages/image/2022/06/03/1106Vy0IMl.png)



### 延时的时长

- 从上面的描述可以看出，延时时长就是消息的过期时间 TTL（Time To Live），这个参数可以通过以下两种方式设置：

1. 生产消息时，设置该消息的 TTL；

2. 设置整个队列的 TTL，队列中每个消息的 TTL 都是一样的；

- 接下来的实战，我们将上述两种方式都体验一次；



### 环境信息

1. 操作系统：Ubuntu 16.04.3 LTS
2. Docker：1.12.6
3. RabbitMQ：3.7.5-rc.1



### 极速体验

* 本章是《Docker 下 RabbitMQ 延时队列实战两部曲》系列的第一篇，重点是极速体验，先看一下即将体验的整体结构图：

  ![](https://github.com/wuwenyishi/pages/raw/gh-pages/image/2022/06/03/1108hShQ3h.png)

* 简单说一下上图中应用的身份：

  1. delayrabbitmqconsumer：SpringBoot 框架的应用，连接 RabbitMQ 的两个队列，消费消息；
  2. messagettlproducer：SpringBoot 框架的应用，收到 web 请求后向 RabbitMQ 发送消息，消息中带有过期时间（TTL）；
  3. queuettlproducer：SpringBoot 框架的应用，收到 web 请求后向 RabbitMQ 发送消息，消息中不带过期时间（TTL），但是对应的消息队列已经设置了过期时间；

* 开始实战：

* 确保当前电脑上 Docker 可用，创建 docker-compose.yml 文件，内容如下：

  ```yml
  version: '2'
  services:
    rabbit1:
      image: bolingcavalry/rabbitmq-server:0.0.3
      hostname: rabbit1
      ports:
        - "15672:15672"
      environment:
        - RABBITMQ_DEFAULT_USER=admin
        - RABBITMQ_DEFAULT_PASS=888888
    rabbit2:
      image: bolingcavalry/rabbitmq-server:0.0.3
      hostname: rabbit2
      depends_on:
        - rabbit1
      links:
        - rabbit1
      environment:
       - CLUSTERED=true
       - CLUSTER_WITH=rabbit1
       - RAM_NODE=true
       - HA_ENABLE=true
      ports:
        - "15673:15672"
    rabbit3:
      image: bolingcavalry/rabbitmq-server:0.0.3
      hostname: rabbit3
      depends_on:
        - rabbit2
      links:
        - rabbit1
        - rabbit2
      environment:
        - CLUSTERED=true
        - CLUSTER_WITH=rabbit1
      ports:
        - "15675:15672"
    messagettlproducer:
      image: bolingcavalry/messagettlproducer:0.0.1-SNAPSHOT
      hostname: messagettlproducer
      depends_on:
        - rabbit3
      links:
        - rabbit1:rabbitmqhost1
        - rabbit2:rabbitmqhost2
        - rabbit3:rabbitmqhost3
      ports:
        - "18080:8080"
      environment:
        - mq.rabbit.address=rabbitmqhost1:5672,rabbitmqhost2:5672,rabbitmqhost3:5672
        - mq.rabbit.username=admin
        - mq.rabbit.password=888888
        - message.ttl.exchange=message.ttl.exchange
        - message.ttl.queue.source=message.ttl.queue.source
        - message.ttl.queue.process=message.ttl.queue.process
    queuettlproducer:
      image: bolingcavalry/queuettlproducer:0.0.1-SNAPSHOT
      hostname: queuettlproducer
      depends_on:
        - messagettlproducer
      links:
        - rabbit1:rabbitmqhost1
        - rabbit2:rabbitmqhost2
        - rabbit3:rabbitmqhost3
      ports:
        - "18081:8080"
      environment:
        - mq.rabbit.address=rabbitmqhost1:5672,rabbitmqhost2:5672,rabbitmqhost3:5672
        - mq.rabbit.username=admin
        - mq.rabbit.password=888888
        - queue.ttl.exchange=queue.ttl.exchange
        - queue.ttl.queue.source=queue.ttl.queue.source
        - queue.ttl.queue.process=queue.ttl.queue.process
        - queue.ttl.value=5000
    delayrabbitmqconsumer:
      image: bolingcavalry/delayrabbitmqconsumer:0.0.1-SNAPSHOT
      hostname: delayrabbitmqconsumer
      depends_on:
        - queuettlproducer
      links:
        - rabbit1:rabbitmqhost1
        - rabbit2:rabbitmqhost2
        - rabbit3:rabbitmqhost3
      environment:
       - mq.rabbit.address=rabbitmqhost1:5672,rabbitmqhost2:5672,rabbitmqhost3:5672
       - mq.rabbit.username=admin
       - mq.rabbit.password=888888
       - message.ttl.queue.process=message.ttl.queue.process
       - queue.ttl.queue.process=queue.ttl.queue.process
  
  ```

  

* 在 docker-compose.yml 文件所在目录执行命令 **docker-compose up -d**，docker 会先去下载镜像，然后创建多个容器，请静候启动完成，如下：

  ```yml
  root@willzhao-Vostro-3267:/usr/local/work/github/blog_demos/rabbitmq_docker_files/delaymq# docker-compose up -d
  Creating network "delaymq_default" with the default driver
  Creating delaymq_rabbit1_1 ... done
  Creating delaymq_rabbit2_1 ... done
  Creating delaymq_rabbit3_1 ... done
  Creating delaymq_messagettlproducer_1 ... done
  Creating delaymq_queuettlproducer_1   ... done
  Creating delaymq_delayrabbitmqconsumer_1 ... done
  ```

- 一共启动了 5 个容器，将他们的名称和作用列举到下面表格中：

  ![](https://github.com/wuwenyishi/pages/raw/gh-pages/image/2022/06/03/1111M3kYEP.png)

- 我的电脑 IP 地址是 **192.168.31.102**，因此在浏览器输入：http://:192.168.31.102:15672 ，可以进入 RabbitMQ 的登录页面，如下图：

  ![](https://github.com/wuwenyishi/pages/raw/gh-pages/image/2022/06/03/1111USpaye.png)

- 登录用户名：**admin**，密码：**888888**，登录后页面如下图，可以见到 RabbitMQ 集群已经就绪：

  ![](https://github.com/wuwenyishi/pages/raw/gh-pages/image/2022/06/03/1112mC9tZP.png)

- 如果您的页面中的三个 RabbitMQ 还没有完全就绪（绿色状态），建议稍等几分钟后再刷新页面；

- 我的电脑 IP 地址是 **192.168.31.102**，因此在浏览器输入：http://192.168.31.102:18080/messagettl/aaa/bbb/3 ，即可向 delaymq_messagettlproducer_1 容器发起一次请求，容器会发送一条带有 TTL  的消息，然后页面提示发送成功，如下图：

  ![](https://github.com/wuwenyishi/pages/raw/gh-pages/image/2022/06/03/11120vsfud.png)

- 在浏览器输入：http://192.168.31.102:18081/queuettl/ccc/ddd ，即可向  delaymq_queuettlproducer_1 容器发起一次请求，容器会向一个已经设置了 TTL  的队列发送一条消息，然后页面提示发送成功，如下图：

  ![](https://github.com/wuwenyishi/pages/raw/gh-pages/image/2022/06/03/1113xOGFtE.png)

- 再次进入 RabbitMQ 管理页面 http://:192.168.31.102:15672 ，查看队列情况如下图，已经有四个队列了：

  ![](https://github.com/wuwenyishi/pages/raw/gh-pages/image/2022/06/03/1113gawCLe.png)

- 用下列表格对上述四个队列简单说明：

- 容器 delaymq_delayrabbitmqconsumer_1 中的 tomcat 在启动时候，由于此时队列还没创建，因此无法连接队列，会导致 tomcat 启动失败，进而导致容器退出(因为 tomcat 进程占据了控制台，它退出容器就会退出)，此时请执行 **docker restart delaymq_delayrabbitmqconsumer_1** 重启容器；

- 重启后，执行命令 **docker logs -f delaymq_delayrabbitmqconsumer_1**，开始实时打印消费者容器的日志，可以看到 SpringBoot 刚刚启动就把之前的两条消息给消费了，如下图红框所示：

  ![](https://github.com/wuwenyishi/pages/raw/gh-pages/image/2022/06/03/11140q4wiY.png)

- 在浏览器输入：http://192.168.31.102:18080/messagettl/aaa01/bbb01/10 ，url 中的 10  表示延时十秒，去看 delaymq_delayrabbitmqconsumer_1 的日志，果然，10  秒钟后日志才会打印出来，如下所示，消息中的时间和日志的时间戳相差 10 秒：

  ```shell
  2018-06-09 07:13:25.878  INFO 1 --- [cTaskExecutor-1] c.b.d.receiver.MessageTtlReceiver        : receive message : hello, aaa01 , bbb01, from queue [message.ttl.queue.source], delay 10's, 2018-06-09 07:13:15
  ```

* 在浏览器输入：http://192.168.31.102:18081/queuettl/ccc01/ddd01 ，会发送消息到队列  queue.ttl.queue.source，这个队列的 TTL 是 5 秒，在 delaymq_delayrabbitmqconsumer_1 的日志中可以看见发起和收到时间正好差 5 秒，如下：

  ```shell
  2018-06-09 07:31:05.101  INFO 1 --- [cTaskExecutor-1] c.b.d.receiver.QueueTtlReceiver          : receive message : hello, ccc01 , ddd01, from queue [queue.ttl.queue.source], 2018-06-09 07:31:00
  ```

* 至此，咱们的极速体验延时队列实战就结束了，接下来的章节，我们一起实战详细的开发过程《Docker 下 RabbitMQ 延时队列实战两部曲之二：细说开发》