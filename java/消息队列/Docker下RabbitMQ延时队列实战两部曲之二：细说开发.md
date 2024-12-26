> 原文：[Docker下RabbitMQ延时队列实战两部曲之二：细说开发](https://xie.infoq.cn/article/50307d9ccf7c5a3f9eba6cb7f)

![](https://github.com/wuwenyishi/pages/raw/gh-pages/image/2022/06/03/1124QWMauu.png)



### 本章涉及的脚本和源码下载

本章会开发一个 yml 脚本，三个基于 SpringBoot 的应用，功能如下：



1. docker-compose.yml：启动所有容器的 docker-compose 脚本；
2. delayrabbitmqconsumer：SpringBoot 框架的应用，连接 RabbitMQ 的两个队列，消费消息；
3. messagettlproducer：SpringBoot 框架的应用，收到 web 请求后向 RabbitMQ 发送消息，消息中带有过期时间（TTL）；
4. queuettlproducer：SpringBoot 框架的应用，收到 web 请求后向 RabbitMQ 发送消息，消息中不带过期时间（TTL），但是对应的消息队列已经设置了过期时间；



整体部署情况如下：

![](https://github.com/wuwenyishi/pages/raw/gh-pages/image/2022/06/03/1125YmrCjd.png) 

上述脚本和工程的源码都可以在 github 下载，地址和链接信息如下表所示：

这个 git 项目中有多个文件夹，三个 SpringBoot 工程分别在 delayrabbitmqconsumer、messagettlproducer、queuettlproducer 这三个文件夹下，如下图的三个红框所示：

![](https://github.com/wuwenyishi/pages/raw/gh-pages/image/2022/06/03/1126if9cLf.png) 

docker-compose.yml 文件在 rabbitmq_docker_files 文件夹下面的 delaymq 文件夹下，如下图：

![](https://github.com/wuwenyishi/pages/raw/gh-pages/image/2022/06/03/1126iKUR8H.png) 



### 环境信息

- 操作系统：Ubuntu 16.04.3 LTS
- Docker：1.12.6
- RabbitMQ：3.7.5-rc.1
- JDK：1.8.0_111
- SpringBoot：1.4.1.RELEASE
- Maven：3.5.0

### 开发步骤

- 本次开发实战的步骤如下：

1. 开发 messagettlproducer 应用，制作镜像；
2. 开发 queuettlproducer 应用，制作镜像；
3. 开发 delayrabbitmqconsumer 应用，制作镜像；
4. 开发 docker-compose.yml 脚本；

### messagettlproducer 应用

- messagettlproducer 是个基于 SpringBoot 的 web 工程，有一个 Controller 可以响应 web 请求，收到请求后发送一条带有过期时间的消息到 RabbitMQ 的 message.ttl.queue.source 队列；

- pom.xml 内容如下：

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
          <modelVersion>4.0.0</modelVersion>
  
          <groupId>com.bolingcavalry</groupId>
          <artifactId>messagettlproducer</artifactId>
          <version>0.0.1-SNAPSHOT</version>
          <packaging>jar</packaging>
  
          <name>messagettlproducer</name>
          <description>Demo project for Spring Boot</description>
  
          <parent>
                  <groupId>org.springframework.boot</groupId>
                  <artifactId>spring-boot-starter-parent</artifactId>
                  <version>1.4.1.RELEASE</version>
                  <relativePath/> <!-- lookup parent from repository -->
          </parent>
  
          <properties>
                  <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
                  <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
                  <java.version>1.8</java.version>
          </properties>
  
          <dependencies>
                  <dependency>
                          <groupId>org.springframework.boot</groupId>
                          <artifactId>spring-boot-starter-amqp</artifactId>
                  </dependency>
                  <dependency>
                          <groupId>org.springframework.boot</groupId>
                          <artifactId>spring-boot-starter-web</artifactId>
                  </dependency>
  
                  <dependency>
                          <groupId>org.springframework.boot</groupId>
                          <artifactId>spring-boot-starter-test</artifactId>
                          <scope>test</scope>
                  </dependency>
          </dependencies>
  
          <build>
                  <plugins>
                          <plugin>
                                  <groupId>org.springframework.boot</groupId>
                                  <artifactId>spring-boot-maven-plugin</artifactId>
                          </plugin>
                          <plugin>
                                  <groupId>com.spotify</groupId>
                                  <artifactId>docker-maven-plugin</artifactId>
                                  <version>0.4.12</version>
                                  <!--docker镜像相关的配置信息-->
                                  <configuration>
                                          <!--镜像名，这里用工程名-->
                                          <imageName>bolingcavalry/${project.artifactId}</imageName>
                                          <!--TAG,这里用工程版本号-->
                                          <imageTags>
                                                  <imageTag>${project.version}</imageTag>
                                          </imageTags>
                                          <!--镜像的FROM，使用java官方镜像-->
                                          <baseImage>java:8u111-jdk</baseImage>
                                          <!--该镜像的容器启动后，直接运行spring boot工程-->
                                          <entryPoint>["java", "-jar", "/${project.build.finalName}.jar"]</entryPoint>
                                          <!--构建镜像的配置信息-->
                                          <resources>
                                                  <resource>
                                                          <targetPath>/</targetPath>
                                                          <directory>${project.build.directory}</directory>
                                                          <include>${project.build.finalName}.jar</include>
                                                  </resource>
                                          </resources>
                                  </configuration>
                          </plugin>
                  </plugins>
          </build>
  </project>
  ```



- 上面的内容中有以下两点需要注意：a. 添加对 spring-boot-starter-amqp 的依赖，这里面是操作 RabbitMQ 所需的库；b. 添加 docker-maven-plugin 插件，可以将当前工程直接制作成 Docker 镜像；

- src/main/resources 文件夹下面创建 application.properties 文件，内容如下，只配置了应用名称和 RabbitMQ 的 virtualHost 路径：

  ```properties
  spring.application.name=messagettlproducer
  mq.rabbit.virtualHost=/
  ```

- RabbitTemplateConfig.java 文件中是应用连接 RabbitMQ 的配置信息：

  ```java
  @Configuration
  public class RabbitTemplateConfig {
  
      @Value("${mq.rabbit.address}")
      String address;
  
      @Value("${mq.rabbit.username}")
      String username;
  
      @Value("${mq.rabbit.password}")
      String password;
  
      @Value("${mq.rabbit.virtualHost}")
      String mqRabbitVirtualHost;
  
      //创建mq连接
      @Bean(name = "connectionFactory")
      public ConnectionFactory connectionFactory() {
          CachingConnectionFactory connectionFactory = new CachingConnectionFactory();
  
          connectionFactory.setUsername(username);
          connectionFactory.setPassword(password);
          connectionFactory.setVirtualHost(mqRabbitVirtualHost);
          connectionFactory.setPublisherConfirms(true);
  
          //该方法配置多个host，在当前连接host down掉的时候会自动去重连后面的host
          connectionFactory.setAddresses(address);
  
          return connectionFactory;
      }
  
      @Bean
      @Scope(ConfigurableBeanFactory.SCOPE_PROTOTYPE)
      //必须是prototype类型
      public RabbitTemplate rabbitTemplate() {
          RabbitTemplate template = new RabbitTemplate(connectionFactory());
          return template;
      }
  }
  ```

- 上面的代码有以下几点要注意：a. address、username、password 这些变量的值，是从操作系统的环境变量中获取的，我们在启动 Docker 容器的时候将这些值配置到容器的环境变量中，程序运行的时候就能取到了；b. connectionFactory()方法根据上述配置参数和 RabbitMQ 建立连接；c. rabbitTemplate()创建 RabbitTemplate 对象，我们可以在其他 Bean 中通过 Autowire 使用；

- MessageTtlRabbitConfig.java 类中是和消息队列相关的配置：

  ```java
      /**
       * 成为死信后，重新发送到的交换机的名称
       */
      @Value("${message.ttl.exchange}")
      private String MESSAGE_TTL_EXCHANGE_NAME;
  
      /**
       * 不会被消费的队列，投递到此队列的消息会成为死信
       */
      @Value("${message.ttl.queue.source}")
      private String MESSAGE_TTL_QUEUE_SOURCE;
  
      /**
       * 该队列被绑定到接收死信的交换机
       */
      @Value("${message.ttl.queue.process}")
      private String MESSAGE_TTL_QUEUE_PROCESS;
  
      /**
       * 配置一个队列，该队列的消息如果没有被消费，就会投递到死信交换机中，并且带上指定的routekey
       * @return
       */
      @Bean
      Queue messageTtlQueueSource() {
          return QueueBuilder.durable(MESSAGE_TTL_QUEUE_SOURCE)
                  .withArgument("x-dead-letter-exchange", MESSAGE_TTL_EXCHANGE_NAME)
                  .withArgument("x-dead-letter-routing-key", MESSAGE_TTL_QUEUE_PROCESS)
                  .build();
      }
  
      @Bean("messageTtlQueueProcess")
      Queue messageTtlQueueProcess() {
          return QueueBuilder.durable(MESSAGE_TTL_QUEUE_PROCESS) .build();
      }
  
      @Bean("messageTtlExchange")
      DirectExchange messageTtlExchange() {
          return new DirectExchange(MESSAGE_TTL_EXCHANGE_NAME);
      }
  
      /**
       * 绑定指定的队列到死信交换机上
       * @param messageTtlQueueProcess
       * @param messageTtlExchange
       * @return
       */
      @Bean
      Binding bindingExchangeMessage(@Qualifier("messageTtlQueueProcess") Queue messageTtlQueueProcess, @Qualifier("messageTtlExchange") DirectExchange messageTtlExchange) {
          System.out.println("11111111111111111111111111111111111111111111111111");
          System.out.println("11111111111111111111111111111111111111111111111111");
          System.out.println("11111111111111111111111111111111111111111111111111");
          System.out.println("11111111111111111111111111111111111111111111111111");
          return BindingBuilder.bind(messageTtlQueueProcess)
                  .to(messageTtlExchange)
                  .with(MESSAGE_TTL_QUEUE_PROCESS);
      }
  ```

- 上面的代码有以下几点要注意：a. MESSAGE_TTL_EXCHANGE_NAME、MESSAGE_TTL_QUEUE_SOURCE、MESSAGE_TTL_QUEUE_PROCESS 这些变量的值，是从操作系统的环境变量中获取的，我们在启动 Docker 容器的时候将这些值配置到容器的环境变量中，程序运行的时候就能取到了；b. connectionFactory()方法根据上述配置参数和 RabbitMQ 建立连接；c. rabbitTemplate()创建 RabbitTemplate 对象，我们可以在其他 Bean 中通过 Autowire 使用；d. messageTtlQueueSource()方法创建了一个队列用于投递消息，通过 **x-dead-letter-exchange** 和 **x-dead-letter-routing-key** 这两个参数，设置了队列消息过期后转发的交换机名称，以及携带的 routing key；

- 为了设置消息过期，我们还要定制一个 ExpirationMessagePostProcessor 类，作用是将给消息类设置过期时间，后面发送消息时会用到这个类：

  ```java
  package com.bolingcavalry.messagettlproducer;
  
  import org.springframework.amqp.AmqpException;
  import org.springframework.amqp.core.Message;
  import org.springframework.amqp.core.MessagePostProcessor;
  
  /**
   * @Description :
   * @Author : zq2599@gmail.com
   * @Date : 2018-06-02 23:33
   */
  public class ExpirationMessagePostProcessor implements MessagePostProcessor {
      private final Long ttl; // 毫秒
  
      public ExpirationMessagePostProcessor(Long ttl) {
          this.ttl = ttl;
      }
  
      @Override
      public Message postProcessMessage(Message message) throws AmqpException {
          message.getMessageProperties() .setExpiration(ttl.toString()); // 设置per-message的失效时间
          return message;
      }
  }
  ```

- 用于处理 web 请求的 SendMessageController 类，源码如下：

  ```java
  /**
   * @Description : 用于生产消息的web接口类
   * @Author : zq2599@gmail.com
   * @Date : 2018-06-02 23:00
   */
  @RestController
  public class SendMessageController {
  
      @Autowired
      private RabbitTemplate rabbitTemplate;
  
      @Value("${message.ttl.queue.source}")
      private String MESSAGE_TTL_QUEUE_SOURCE;
  
      /**
       * 生产一条消息，消息中带有过期时间
       * @param name
       * @param message
       * @param delaytime
       * @return
       */
      @RequestMapping(value = "/messagettl/{name}/{message}/{delaytime}", method = RequestMethod.GET)
      public @ResponseBody
      String messagettl(@PathVariable("name") final String name, @PathVariable("message") final String message, @PathVariable("delaytime") final int delaytime) {
          SimpleDateFormat simpleDateFormat=new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
          String timeStr = simpleDateFormat.format(new Date());
          String queueName = MESSAGE_TTL_QUEUE_SOURCE;
          String  sendMessage = String.format("hello, %s , %s, from queue [%s], delay %d's, %s", name, message, MESSAGE_TTL_QUEUE_SOURCE, delaytime, timeStr);
          rabbitTemplate.convertAndSend(MESSAGE_TTL_QUEUE_SOURCE,
                                          (Object)sendMessage,
                                          new ExpirationMessagePostProcessor(delaytime*1000L));
  
          return "send message to [" +  name + "] success , queue is : " + queueName + " (" + timeStr + ")";
      }
  }
  ```



- 如上所示，发送消息的代码很简单，调用 rabbitTemplate 的 convertAndSend 就能发送消息到 message.ttl.queue.source 队列（指定路由键的 Direct 方式），再传入 ExpirationMessagePostProcessor 作为处理消息的工具；
- 以上就是 messagettlproducer 应用的主要代码介绍，编码完毕后，在 pom.xml 文件所在目录执行 **mvn clean package -U -DskipTests docker:build**，即可编译、构建、制作 Docker 镜像；



### queuettlproducer 应用

- queuettlproducer 和 messagettlproducer 极为相似，都是接受 web 请求后向 RabbitMQ 发送消息，不同之处有以下两点：
  1. queuettlproducer 在绑定队列的时候，会设置队列上所有消息的过期时间，messagettlproducer 没做这个设置；
  2. queuettlproducer 在发送消息的时候，没有设置该消息的过期时间，messagettlproducer 会对每条消息都设置过期时间；

- 因此，queuettlproducer 和 messagettlproducer 这两个应用的代码大部分是相同的，这里只要关注不同的部分即可；

- 队列和交换机的配置类，QueueTtlRabbitConfig：

  ```java
  @Configuration
  public class QueueTtlRabbitConfig {
  
      /**
       * 成为死信后，重新发送到的交换机的名称
       */
      @Value("${queue.ttl.exchange}")
      private String QUEUE_TTL_EXCHANGE_NAME;
  
      /**
       * 不会被消费的队列，投递到此队列的消息会成为死信
       */
      @Value("${queue.ttl.queue.source}")
      private String QUEUE_TTL_QUEUE_SOURCE;
  
      /**
       * 该队列被绑定到接收死信的交换机
       */
      @Value("${queue.ttl.queue.process}")
      private String QUEUE_TTL_QUEUE_PROCESS;
  
      @Value("${queue.ttl.value}")
      private long QUEUE_TTL_VALUE;
  
      /**
       * 配置一个队列，该队列有消息过期时间，消息如果没有被消费，就会投递到死信交换机中，并且带上指定的routekey
       * @return
       */
      @Bean
      Queue queueTtlQueueSource() {
          return QueueBuilder.durable(QUEUE_TTL_QUEUE_SOURCE)
                  .withArgument("x-dead-letter-exchange", QUEUE_TTL_EXCHANGE_NAME)
                  .withArgument("x-dead-letter-routing-key", QUEUE_TTL_QUEUE_PROCESS)
                  .withArgument("x-message-ttl", QUEUE_TTL_VALUE)
                  .build();
      }
  
      @Bean("queueTtlQueueProcess")
      Queue queueTtlQueueProcess() {
          return QueueBuilder.durable(QUEUE_TTL_QUEUE_PROCESS) .build();
      }
  
      @Bean("queueTtlExchange")
      DirectExchange queueTtlExchange() {
          return new DirectExchange(QUEUE_TTL_EXCHANGE_NAME);
      }
  
      /**
       * 绑定
       * @param queueTtlQueueProcess
       * @param queueTtlExchange
       * @return
       */
      @Bean
      Binding bindingExchangeMessage(@Qualifier("queueTtlQueueProcess") Queue queueTtlQueueProcess, @Qualifier("queueTtlExchange") DirectExchange queueTtlExchange) {
          System.out.println("22222222222222222222222222222222222222222222222222");
          System.out.println("22222222222222222222222222222222222222222222222222");
          System.out.println("22222222222222222222222222222222222222222222222222");
          System.out.println("22222222222222222222222222222222222222222222222222");
          return BindingBuilder.bind(queueTtlQueueProcess)
                  .to(queueTtlExchange)
                  .with(QUEUE_TTL_QUEUE_PROCESS);
      }
  }
  ```

- 上述代码请注意以下两点：a. queueTtlQueueSource()方法用来设置队列，除了 **x-dead-letter-exchange** 和 **x-dead-letter-routing-key** 这两个参数，还多了 **x-message-ttl**，此参数对应的值就是进入该队列的每一条消息的过期时间；b. bindingExchangeMessage()方法将队列 queue.ttl.queue.source 绑定到 Direct 模式的交换机；

- 处理 web 请求的 SendMessageController 类：

  ```java
  @RestController
  public class SendMessageController {
  
      @Autowired
      private RabbitTemplate rabbitTemplate;
  
      @Value("${queue.ttl.queue.source}")
      private String QUEUE_TTL_QUEUE_SOURCE;
  
      /**
       * 生产一条消息，消息中不带过期时间，但是对应的队列中已经配置了过期时间
       * @param name
       * @param message
       * @return
       */
      @RequestMapping(value = "/queuettl/{name}/{message}", method = RequestMethod.GET)
      public @ResponseBody
      String queuettl(@PathVariable("name") final String name, @PathVariable("message") final String message) {
          SimpleDateFormat simpleDateFormat=new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
          String timeStr = simpleDateFormat.format(new Date());
          String queueName = QUEUE_TTL_QUEUE_SOURCE;
          String  sendMessage = String.format("hello, %s , %s, from queue [%s], %s", name, message, queueName, timeStr);
          rabbitTemplate.convertAndSend(queueName, sendMessage);
  
          return "send message to [" +  name + "] success , queue is : " + queueName + " (" + timeStr + ")";
      }
  }
  ```

- 如上所示，发送消息时只有 routing key 和消息对象这两个参数；
- 以上就是发送消息到队列的应用源码，编码完毕后，在 pom.xml 文件所在目录执行 **mvn clean package -U -DskipTests docker:build**，即可编译、构建、制作 Docker 镜像；
- 接下来我们看看消息消费者工程 delayrabbitmqconsumer 的源码；



### delayrabbitmqconsumer 应用

- delayrabbitmqconsumer 应用连接到消息队列，消费收到的每条消息；

- RabbitTemplateConfig.java 是连接到 RabbitMQ 的配置信息，和前面两个应用一样，不再赘述；

- 消费 message.ttl.queue.process 这个队列发出的消息，对应实现类是 MessageTtlReceiver：

  ```java
  /**
   * @Description : 消息接受类，接收第一类延时消息(在每条消息中指定过期时间)的转发结果
   * @Author : zq2599@gmail.com
   * @Date : 2018-06-03 9:52
   */
  @Component
  @RabbitListener(queues = "${message.ttl.queue.process}")
  public class MessageTtlReceiver {
      private static final Logger logger = LoggerFactory.getLogger(MessageTtlReceiver.class);
  
      @RabbitHandler
      public void process(String message) {
          logger.info("receive message : " + message);
      }
  }
  ```



* 如上所示，只要用注解 RabbitListener 配置好队列的名称即可，编码完毕后，在 pom.xml 文件所在目录执行 **mvn clean package -U -DskipTests docker:build**，即可编译、构建、制作 Docker 镜像；



### docker-compose.yml 配置

- 最后我们看一下所有容器的配置文件 docker-compose.yml：

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



- 上述配置文件有以下几点需要注意：
- rabbit1、rabbit2、rabbit3 是 RabbitMQ 高可用集群，如果您对 RabbitMQ 高可用集群感兴趣，推荐您请看[《Docker下RabbitMQ四部曲》](https://xie.infoq.cn/link?target=https%3A%2F%2Fblog.51cto.com%2Fu_13674465%2F5331778)系列文章；
- 三个 SpringBoot 应用都配置了 **mq.rabbit.address** 参数，值是三个 RabbitMQ server 的 IP 加端口，这样如果 RabbitMQ 集群中有一台机器故障了也不会影响正常的消息收发；
- 使用了 link 参数后，容器内就能通过 link 的参数取代对应的 IP；
- 至此，Docker 下的 RabbitMQ 延时队列实战就完成了，实战中 Docker 发挥的作用并不大，只是用来快速搭建环境，关键还是三个工程中对队列的各种操作，希望本系列能帮助您快速构建延时队列相关服务；



