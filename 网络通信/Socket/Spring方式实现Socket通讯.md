> 参考文章[【springboot】【socket】spring boot整合socket,实现服务器端两种消息推送 - Angel挤一挤 - 博客园](https://www.cnblogs.com/sxdcgaq8080/p/10690644.html) 

在SpringBoot中的代码实现

## pom文件完整示例

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



## 新建握手拦截器

```java
package socket.config;

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



## 新建webSocket的handler处理器

```java
package socket.config;

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



## 新建webSocket的config配置

```java
package socket.config;

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



## 创建chat1.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <title>测试websocket</title>
    <meta charset="UTF-8">
    <meta name="viewport"
          content="width=device-width, initial-scale=1, minimum-scale=1, maximum-scale=1, user-scalable=no">
    <meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1"/>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/twitter-bootstrap/4.1.2/css/bootstrap.min.css">
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/jquery-toast-plugin/1.3.2/jquery.toast.min.css">
</head>
<body>
<div class="container">
    <div class="input-group mb-3">
        <div class="input-group-prepend">
            <label class="input-group-text" for="inputGroupSelect01">用户</label>
        </div>
        <select class="custom-select" id="inputGroupSelect01">
            <option selected>选择一个...</option>
        </select>
    </div>
    <div class="input-group mb-3">
        <input type="text" class="form-control">
        <div class="input-group-append">
            <span class="input-group-text" id="btn1">发送给所有人</span>
        </div>
    </div>
    <div class="input-group mb-3">
        <input type="text" class="form-control">
        <div class="input-group-append">
            <span class="input-group-text" id="btn2">发送给单人</span>
        </div>
    </div>
</div>
<script src="https://cdnjs.cloudflare.com/ajax/libs/jquery/1.12.4/jquery.min.js" type="text/javascript"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/twitter-bootstrap/4.1.2/js/bootstrap.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/jquery-toast-plugin/1.3.2/jquery.toast.min.js"></script>
<script language=javascript>
    $(function () {
        var websocket;
        if ('WebSocket' in window) {
            console.log("WebSocket-->");
            //new WebSocket()在IDEA打开的任何项目中都可以 直接调用
            websocket = new WebSocket("ws://localhost:8020/ws");
        } else if ('MozWebSocket' in window) {
            console.log("MozWebSocket-->");
            websocket = new MozWebSocket("ws://ws");
        } else {
            console.log("SockJS-->");
            websocket = new SockJS("http://127.0.0.1:8020/sockjs/ws");
        }


        websocket.onopen = function (evnt) {
            console.log("链接服务器成功!", evnt.data);
        };
        websocket.onmessage = function (evnt) {
            console.log('收到消息:', evnt.data);
            var json = JSON.parse(evnt.data);
            if (json.hasOwnProperty('users')) {
                var users = json.users;
                for (var i = 0; i < users.length; i++) {
                    $("#inputGroupSelect01").append('<option value="' + users[i] + '">' + users[i] + '</option>');
                }
            } else {
                //打印消息
                toast(json.msg, 'info')
            }
        };
        websocket.onerror = function (evnt) {
        };
        websocket.onclose = function (evnt) {
            console.log("与服务器断开了链接!")
        }
        $('#btn2').bind('click', function () {
            if (websocket != null) {
                //根据勾选的人数确定是群聊还是单聊
                var value = $(this).parent().parent().find('input').val();
                //得到选择的用户
                var name = $("#inputGroupSelect01").find("option:selected").val();
                console.log('选中的用户', name);
                if (name === '选择一个...') {
                    toast('请选择一个用户', 'warning')
                } else {
                    var object = {
                        to: name,
                        msg: value,
                        type: 2
                    };
                    //将object转成json字符串发送给服务端
                    var json = JSON.stringify(object);
                    websocket.send(json);
                }
            } else {
                console.log('未与服务器链接.');
            }
        });
        $('#btn1').bind('click', function () {
            if (websocket != null) {
                //根据勾选的人数确定是群聊还是单聊
                var value = $(this).parent().parent().find('input').val();
                var object = {
                    msg: value,
                    type: 1
                };
                //将object转成json字符串发送给服务端
                var json = JSON.stringify(object);
                websocket.send(json);
            } else {
                console.log('未与服务器链接.');
            }
        });
    })

    function toast(text, icon) {
        $.toast({
            text: text,
            heading: '新消息',
            icon: icon,
            showHideTransition: 'slide',
            allowToastClose: true,
            hideAfter: 3000,
            stack: 5,
            position: 'top-right',

            bgColor: '#444444',
            textColor: '#eeeeee',
            textAlign: 'left',
            loader: true,
            loaderBg: '#006eff'
        });
    }
</script>
</body>
</html>
```

## 创建一个定时任务发送消息

```java
package socket.task;

import com.alibaba.fastjson.JSONObject;
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Component;
import org.springframework.web.socket.TextMessage;
import socket.config.MySocketHander;

import java.time.LocalDateTime;

@Component
public class ScheduledTask {

    @Scheduled(fixedRate = 3000)
    public void scheduledTask() {
        System.out.println("任务执行时间：" + LocalDateTime.now());
        JSONObject obj = new JSONObject();
        obj.put("msg", "34r35436554766787dfds");
        MySocketHander.sendMessageToUsers(new TextMessage(obj.toJSONString()));
    }

}

```

> 在启动类上添加：`@EnableScheduling`



![](https://mmbiz.qpic.cn/mmbiz_png/3eqXwttvOLvED4MbUa8NsovrpXwicGqwyM2Lhz5biaibVRvsSdgiaWiamlebVT86qWDVicmMX2ezVcb9cbuGibDv3ia0Vg/0?wx_fmt=png&from=appmsg)

