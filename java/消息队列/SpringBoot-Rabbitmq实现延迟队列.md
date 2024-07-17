## 安装

## MAC 系统

安装命令：brew install rabbitmq
​

安装的路径是 /usr/local/Cellar/rabbitmq/3.8.3，具体情况要视版本而定，我安装的版本是 3.8.3。
​

接下来就可以启动了，进入安装目录，执行命令：./sbin/rabbitmq-server 

接下来可以在浏览器打开 http://localhost:15672，可以看到 RabbitMQ 的管理页面。
​

登录账号密码：guest/guest

## 导入 Maven 依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency
```

## TTL 方式

### application.properties 配置信息

```yml
# 应用名称
spring.application.name=rabbitMq
# 应用服务 WEB 访问端口
server.port=8080
spring.rabbitmq.host=127.0.0.1
spring.rabbitmq.port=5672
spring.rabbitmq.username=guest
spring.rabbitmq.password=guest
# 开启消息确认机制 confirm 异步
spring.rabbitmq.publisher-confirm-type=correlated
# 之前的旧版本 开启消息确认机制的方式
# spring.rabbitmq.publisher-confirms=true
# 开启return机制
spring.rabbitmq.publisher-returns=true
# 消息开启手动确认
spring.rabbitmq.listener.direct.acknowledge-mode=manual
spring.rabbitmq.listener.simple.acknowledge-mode=manual
# 消费者每次从队列获取的消息数量。此属性当不设置时为：轮询分发，设置为1为：公平分发
spring.rabbitmq.listener.simple.prefetch=1
# 是否支持重试
spring.rabbitmq.listener.simple.retry.enabled=true

#消费者最小数量
spring.rabbitmq.listener.simple.concurrency=1
#消费之最大数量
spring.rabbitmq.listener.simple.max-concurrency=10
```

### RabbitmqConfig 配置信息

```java
package com.bean.springcloudproduct.config;
 
import org.springframework.amqp.core.*;
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.boot.autoconfigure.amqp.SimpleRabbitListenerContainerFactoryConfigurer;
import org.springframework.context.annotation.Bean;
 
import java.util.HashMap;
import java.util.Map;
 
 
/**
* @Description //TODO
* @Author huangwb
**/
public class RabbitmqConfig {
   /**
     * 死信交换机
     *
     * @return
     */
    
    public DirectExchange userOrderDelayExchange() {
        return new DirectExchange("user.order.delay_exchange");
    }
 
    /**
     * 死信队列
     *
     * @return
     */
    
    public Queue userOrderDelayQueue() {
        Map<String, Object> map = new HashMap<>(16);
        map.put("x-dead-letter-exchange", "user.order.receive_exchange");
        map.put("x-dead-letter-routing-key", "user.order.receive_key");
        return new Queue("user.order.delay_queue", true, false, false, map);
    }
 
    /**
     * 给死信队列绑定交换机
     *
     * @return
     */
    
    public Binding userOrderDelayBinding() {
        return BindingBuilder.bind(userOrderDelayQueue()).to(userOrderDelayExchange()).with("user.order.delay_key");
    }
 
    /**
     * 死信接收交换机
     *
     * @return
     */
    
    public DirectExchange userOrderReceiveExchange() {
        return new DirectExchange("user.order.receive_exchange");
    }
 
    /**
     * 死信接收队列
     *
     * @return
     */
    
    public Queue userOrderReceiveQueue() {
        return new Queue("user.order.receive_queue");
    }
 
    /**
     * 死信交换机绑定消费队列
     *
     * @return
     */
    
    public Binding userOrderReceiveBinding() {
        return BindingBuilder.bind(userOrderReceiveQueue()).to(userOrderReceiveExchange()).with("user.order.receive_key");
    }
 
}
```



### 生产者

```java


public class DeadLetterSenderServiceImpl implements DeadLetterSenderService {

    
    private RabbitTemplate rabbitTemplate;

    
    public void sendLetterSenderMsg() {
        User user = new User(1, "confirm", "confirm123456");
        MessagePostProcessor postProcessor = message -> {
            //在这里也可以设置超时时间,也可以设置x-message-ttl
            message.getMessageProperties().setExpiration("5000");
            return message;
        };
        rabbitTemplate.setMandatory(true);
        rabbitTemplate.setConfirmCallback(confirmCallback);
        rabbitTemplate.setReturnCallback(returnCallback);
        CorrelationData correlationData = new CorrelationData("confirm-" + System.currentTimeMillis());
        this.rabbitTemplate.convertAndSend("user.order.delay_exchange", "user.order.delay_key", user, postProcessor, correlationData);

    }

    /**
     * 配置 confirm 机制
     * 交换机错误出发
     */
    private final RabbitTemplate.ConfirmCallback confirmCallback = new RabbitTemplate.ConfirmCallback() {
        /**
         * @param correlationData 消息相关的数据，一般用于获取 唯一标识 id
         * @param b true 消息确认成功，false 失败
         * @param s 确认失败的原因
         */
        
        public void confirm(CorrelationData correlationData, boolean b, String s) {
            if (b) {
                System.out.println("confirm 消息确认成功..." + correlationData.getId() + new Date());
            } else {
                System.out.println("confirm 消息确认失败..." + correlationData.getId() + " cause: " + s);
            }
        }
    };

    /**
     * 配置 return 消息机制
     * 找不到路由才会触发
     */
    private final RabbitTemplate.ReturnCallback returnCallback = new RabbitTemplate.ReturnCallback() {
        /**
         *  return 的回调方法（找不到路由才会触发）
         * @param message 消息的相关信息
         * @param i 错误状态码
         * @param s 错误状态码对应的文本信息
         * @param s1 交换机的名字
         * @param s2 路由的key
         */
        
        public void returnedMessage(Message message, int i, String s, String s1, String s2) {
            System.out.println("消息：" + message);
            System.out.println(new String(message.getBody()));
            System.out.println("回应码：" + i);
            System.out.println("回应信息：" + s);
            System.out.println("交换机：" + s1);
            System.out.println("路由键：" + s2);
        }
    };

}
```

### 消费者

```java
Slf4j
public class DeadLetterSenderListener {

    /**
     * @Description 延迟队列
     * @Author jxb
     * @Date 2019-04-04 16:34:28
     */
    
    public void getDLMessage(User user, Channel channel, Message message) throws IOException {
        try {
            System.out.println("延迟队列参数数据 ： " + user +  new Date());
            //basicAck:表示成功确认，使用此回执方法后，消息会被rabbitmq broker 删除。
            //deliveryTag：表示消息投递序号，每次消费消息或者消息重新投递后，deliveryTag都会增加。手动消息确认模式下，我们可以对指定deliveryTag的消息进行ack、nack、reject等操作。
            //multiple(false/true)：是否批量确认，false只确认当前consumer一个消息收到,值为 true 则会一次性 ack所有小于当前消息 deliveryTag 的消息。
            //举个栗子： 假设我先发送三条消息deliveryTag分别是5、6、7，可它们都没有被确认，当我发第四条消息此时deliveryTag为8，multiple设置为 true，会将5、6、7、8的消息全部进行确认。
            channel.basicAck(message.getMessageProperties().getDeliveryTag(), false);
            System.out.println("延迟队列接受到的消息为:" + new String(message.getBody()));
        } catch (Exception e) {
            if (message.getMessageProperties().getRedelivered()) {
                log.error("延迟队列消息已重复处理失败,拒绝再次接收...");
                // 拒绝消息
                //basicReject：拒绝消息，与basicNack区别在于不能进行批量操作，其他用法很相似。
                //deliveryTag：表示消息投递序号。
                //requeue：值为 true 消息将重新入队列。
                channel.basicReject(message.getMessageProperties().getDeliveryTag(), false);
            } else {
                log.error("延迟队列消息即将再次返回队列处理...");
                //basicNack ：表示失败确认，一般在消费消息业务异常时用到此方法，可以将消息重新投递入队列。
                //deliveryTag：表示消息投递序号。
                //multiple：是否批量确认。
                //requeue：值为 true 消息将重新入队列。
                channel.basicNack(message.getMessageProperties().getDeliveryTag(), false, true);
            }
        }
    }
}
```

## DLX

> 首先我们需要下载并安装 RabbitMQ 的延迟插件。

地址：https://github.com/rabbitmq/rabbitmq-delayed-message-exchange/releases
​

将插件文件复制到 RabbitMQ 安装目录的 plugins 目录下；
​

进入 RabbitMQ 安装目录的 sbin 目录下，使用如下命令启用延迟插件

> rabbitmq-plugins enable rabbitmq_delayed_message_exchange

代码实现：
​

### 配置文件：

```java
/**
 * 延时队列交换机 DLX方式
 * 注意这里的交换机类型：CustomExchange
 *
 * @return
 */
public CustomExchange delayExchange() {
    Map<String, Object> args = new HashMap<>(1);
    args.put("x-delayed-type", "direct");
    return new CustomExchange("exchange.xdelay.delayed", "x-delayed-message", true, false, args);
}

 
/**
 * 延时队列
 *  DLX方式
 * @return
 */
public Queue immediateQueue() {
    // 第一个参数是创建的queue的名字，第二个参数是是否支持持久化
    return new Queue("queue.xdelay.immediate", true);
}

/**
 * 给延时队列绑定交换机
 *   DLX方式
 * @return
 */
public Binding bindingNotify() {
    return BindingBuilder.bind(immediateQueue()).to(delayExchange()).with("exchange.xdelay.delayed").noargs();
}
```

### 生产者

```java
    
    public void sendLetterDLXSenderMsg() {
        User user = new User(1, "confirm", "confirm123456");
        MessagePostProcessor postProcessor = message -> {
            message.getMessageProperties().setDelay(5000);
            return message;
        };
        rabbitTemplate.setMandatory(true);
        rabbitTemplate.setConfirmCallback(confirmCallback);
//        rabbitTemplate.setReturnCallback(returnCallback);
        CorrelationData correlationData = new CorrelationData("confirm-" + System.currentTimeMillis());
        this.rabbitTemplate.convertAndSend("exchange.xdelay.delayed", "exchange.xdelay.delayed", user, postProcessor, correlationData);
    }
```

### 消费者

```java
/**
   * DLX方式
   * @param user
   * @param channel
   * @param message
   * @throws IOException
   */
  
  public void getDLXMessage(User user, Channel channel, Message message) throws IOException {
      try {
          System.out.println("DLX延迟队列参数数据 ： " + user +  new Date());
          //basicAck:表示成功确认，使用此回执方法后，消息会被rabbitmq broker 删除。
          //deliveryTag：表示消息投递序号，每次消费消息或者消息重新投递后，deliveryTag都会增加。手动消息确认模式下，我们可以对指定deliveryTag的消息进行ack、nack、reject等操作。
          //multiple(false/true)：是否批量确认，false只确认当前consumer一个消息收到,值为 true 则会一次性 ack所有小于当前消息 deliveryTag 的消息。
          //举个栗子： 假设我先发送三条消息deliveryTag分别是5、6、7，可它们都没有被确认，当我发第四条消息此时deliveryTag为8，multiple设置为 true，会将5、6、7、8的消息全部进行确认。
          channel.basicAck(message.getMessageProperties().getDeliveryTag(), false);
          System.out.println("DLX延迟队列接受到的消息为:" + new String(message.getBody()));
      } catch (Exception e) {
          if (message.getMessageProperties().getRedelivered()) {
              log.error("DLX延迟队列消息已重复处理失败,拒绝再次接收...");
              // 拒绝消息
              //basicReject：拒绝消息，与basicNack区别在于不能进行批量操作，其他用法很相似。
              //deliveryTag：表示消息投递序号。
              //requeue：值为 true 消息将重新入队列。
              channel.basicReject(message.getMessageProperties().getDeliveryTag(), false);
          } else {
              log.error("DLX延迟队列消息即将再次返回队列处理...");
              //basicNack ：表示失败确认，一般在消费消息业务异常时用到此方法，可以将消息重新投递入队列。
              //deliveryTag：表示消息投递序号。
              //multiple：是否批量确认。
              //requeue：值为 true 消息将重新入队列。
              channel.basicNack(message.getMessageProperties().getDeliveryTag(), false, true);
          }
      }
  }
```

> 参考文章：https://blog.csdn.net/qq_37892957/article/details/89296157