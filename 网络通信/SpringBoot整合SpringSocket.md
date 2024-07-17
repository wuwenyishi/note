> 纯代码分享，粘贴复制可运行

```xml 
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-websocket</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-freemarker</artifactId>
</dependency>
```


```java
import lombok.extern.slf4j.Slf4j;
import org.springframework.http.server.ServerHttpRequest;
import org.springframework.http.server.ServerHttpResponse;
import org.springframework.stereotype.Component;
import org.springframework.web.socket.WebSocketHandler;
import org.springframework.web.socket.server.HandshakeInterceptor;

import java.util.Map;

/**
 * 实现【握手拦截器】
 *
 * @Slf4j       注解使用解析 https://www.cnblogs.com/sxdcgaq8080/p/11288213.html
 * @Component   注解是将本拦截器注入为Bean给Spring容器管理
 *
 * @author sxd
 * @date 2019/8/2 13:47
 */
@Slf4j
@Component
public class MyHandshakeInterceptor implements HandshakeInterceptor{

    /**
     * 重写方法 在握手之前做的事情
     * @param serverHttpRequest
     * @param serverHttpResponse
     * @param webSocketHandler
     * @param map
     * @return
     * @throws Exception
     */
    @Override
    public boolean beforeHandshake(ServerHttpRequest serverHttpRequest, ServerHttpResponse serverHttpResponse, WebSocketHandler webSocketHandler, Map<String, Object> map) {
        return true;
    }

    /**
     * 重写方法 在握手之后做的事情
     * @param serverHttpRequest
     * @param serverHttpResponse
     * @param webSocketHandler
     * @param e
     */
    @Override
    public void afterHandshake(ServerHttpRequest serverHttpRequest, ServerHttpResponse serverHttpResponse, WebSocketHandler webSocketHandler, Exception e) {

    }
}
```

```java
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Component;
import org.springframework.web.socket.*;

import java.io.IOException;
import java.util.ArrayList;
import java.util.Collections;
import java.util.List;

/**
 * 实现WebSocketHandler接口
 *
 *
 * @author sxd
 * @date 2019/8/2 14:06
 */
@Slf4j
@Component
public class MySocketHander implements WebSocketHandler {

    /**
     * 为了保存在线用户信息，在方法中新建一个list存储一下【实际项目依据复杂度，可以存储到数据库或者缓存】
     */
    private final static List<WebSocketSession> SESSIONS = Collections.synchronizedList(new ArrayList<>());



    @Override
    public void afterConnectionEstablished(WebSocketSession webSocketSession){
        log.info("链接成功......");
        SESSIONS.add(webSocketSession);
    }

    @Override
    public void handleMessage(WebSocketSession webSocketSession, WebSocketMessage<?> webSocketMessage) {
        log.info("处理要发送的消息");
    }

    @Override
    public void handleTransportError(WebSocketSession webSocketSession, Throwable throwable) throws Exception {
        if (webSocketSession.isOpen()) {
            webSocketSession.close();
        }
        log.info("链接出错，关闭链接......");
        SESSIONS.remove(webSocketSession);
    }

    @Override
    public void afterConnectionClosed(WebSocketSession webSocketSession, CloseStatus closeStatus) throws Exception {
        log.info("链接关闭......" + closeStatus.toString());
        SESSIONS.remove(webSocketSession);
    }

    @Override
    public boolean supportsPartialMessages() {
        return false;
    }

    /**
     * 给所有在线用户发送消息
     *
     * @param message
     */
    public static void sendMessageToUsers(TextMessage message) {
        log.info("给所有在线用户发送消息");
        for (WebSocketSession user : SESSIONS) {
            try {
                if (user.isOpen()) {
                    user.sendMessage(message);
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }


}
```

```java
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.socket.config.annotation.EnableWebSocket;
import org.springframework.web.socket.config.annotation.WebSocketConfigurer;
import org.springframework.web.socket.config.annotation.WebSocketHandlerRegistry;

/**
 * @author sxd
 * @date 2019/8/2 14:43
 */
@Slf4j
@Configuration
@EnableWebSocket
public class MyWebSocketConfig  implements WebSocketConfigurer {

    @Autowired
    MyHandshakeInterceptor handshakeInterceptor;

    @Autowired
    MySocketHander socketHander;




    /**
     * 实现 WebSocketConfigurer 接口
     * 重写 registerWebSocketHandlers 方法，这是一个核心实现方法，配置 websocket 入口，允许访问的域、注册 Handler、SockJs 支持和拦截器。
     *
     * registry.addHandler()注册和路由的功能
     *      当客户端发起 websocket 连接，把 /path 交给对应的 handler 处理，而不实现具体的业务逻辑，可以理解为收集和任务分发中心。
     *
     *
     * addInterceptors() 顾名思义就是为 handler 添加拦截器
     *      可以在调用 handler 前后加入我们自己的逻辑代码。
     *
     *
     * setAllowedOrigins(String[] domains),允许指定的域名或 IP (含端口号)建立长连接
     *      如果只允许自家域名访问，这里轻松设置。如果不限时使用 * 号，如果指定了域名，则必须要以 http 或 https 开头。
     *
     * @param registry
     */
    @Override
    public void registerWebSocketHandlers(WebSocketHandlerRegistry registry) {
        //部分 支持websocket 的访问链接,允许跨域
        registry.addHandler(socketHander, "/ws").addInterceptors(handshakeInterceptor).setAllowedOrigins("*");
        //部分 不支持websocket的访问链接,允许跨域
        registry.addHandler(socketHander, "/sockjs/ws").addInterceptors(handshakeInterceptor).setAllowedOrigins("*").withSockJS();
    }
}
```

```java
@Component
public class ScheduledTask {

  //定时任务测试发送消息
    @Scheduled(fixedRate = 3000)
    public void scheduledTask() {
        System.out.println("任务执行时间：" + LocalDateTime.now());
        JSONObject obj = new JSONObject();
        obj.put("msg", "34r35436554766787dfds");
        MySocketHander.sendMessageToUsers(new TextMessage(obj.toJSONString()));
    }

}

```

