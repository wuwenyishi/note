

> 推荐文章：[消息Ack确认机制](https://www.pianshen.com/article/5851218267/) ，  
> [RabbitMQ如何处理消息丢失](https://segmentfault.com/a/1190000019125512) ，  
> [RabbitMQ防止数据丢失](https://developer.aliyun.com/article/769882)   



## 数据丢失的原因

![image.png](https://github.com/wuwenyishi/pages/raw/gh-pages/image/others/1616125932796-190b38de-df34-41af-89dc-b05abf8e6603.png)

## 消息持久化

![image.png](https://github.com/wuwenyishi/pages/raw/gh-pages/image/others/1616126025417-8eb9b06d-f0fa-43c0-ad8f-d04b8065ffde.png)

![image.png](https://github.com/wuwenyishi/pages/raw/gh-pages/image/others/1616126007166-5079fa45-6134-4ab5-ae37-abe0b0ae1ceb.png)





## 消息确认机制

### confirm机制

![image.png](https://github.com/wuwenyishi/pages/raw/gh-pages/image/others/1616126107133-56e7d1aa-f8d3-40c4-b32a-8b4ad1ead2b0.png)

![image.png](https://github.com/wuwenyishi/pages/raw/gh-pages/image/others/1616126141908-248347ce-82e1-4e93-a90b-944f0ce1f663.png)



**![image.png](https://github.com/wuwenyishi/pages/raw/gh-pages/image/others/1616061939812-fd18b222-64ad-499a-8077-c70510157ba5.png)**

#### 生产者确认模式实现原理

![image.png](https://github.com/wuwenyishi/pages/raw/gh-pages/image/others/1616120057595-7d700569-69fc-4dd4-81f3-a4028f8222bc.png)

###  

#### 理解RabbitMQ Confirm消息确认机制

![image.png](https://github.com/wuwenyishi/pages/raw/gh-pages/image/others/1616119562697-fcbf2cdd-7abe-4797-8e75-54ae2c4e4c5e.png)

#### RabbitMQ Confirm确认机制流程图

![image.png](https://github.com/wuwenyishi/pages/raw/gh-pages/image/others/1616119588730-57000588-f975-418a-854c-11c80ccee25b.png)

### 事务机制(ACK)

![image.png](https://github.com/wuwenyishi/pages/raw/gh-pages/image/others/1616126238189-76f84851-3cee-4279-adf9-83c13dee0230.png)

![image.png](https://github.com/wuwenyishi/pages/raw/gh-pages/image/others/1616126256931-80467414-4679-44ae-8446-b50df502dc49.png)

![image.png](https://github.com/wuwenyishi/pages/raw/gh-pages/image/others/1616123500213-61b0b20e-ba8f-497d-aba7-5f92e0f26fe2.png)



![image.png](https://github.com/wuwenyishi/pages/raw/gh-pages/image/others/1616121122072-5f1d10e2-8701-4070-8af1-b06a1a725e6d.png)

#### ACK工作原理

![image.png](https://github.com/wuwenyishi/pages/raw/gh-pages/image/others/1616121190328-6e7acd85-85e6-461a-a49b-8b294e3c19b4.png)



> 推荐文章：[消息Ack确认机制](https://www.pianshen.com/article/5851218267/) ， [RabbitMQ如何处理消息丢失](https://segmentfault.com/a/1190000019125512) ， [RabbitMQ防止数据丢失](https://developer.aliyun.com/article/769882)   