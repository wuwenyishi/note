

>文章来源
>
>* https://www.sofastack.tech/blog/seata-distributed-transaction-deep-dive/
>* 官网：http://seata.io/zh-cn/index.html
>* 推荐文章：https://www.cnblogs.com/chengxy-nds/p/14046856.html



![image-20210311152028639](https://xuemingde.com/pages/image/others/QxsNJBC7TiaLyZv.png)





### 分布式事务问题产生原因

![image-20210311153056539](https://xuemingde.com/pages/image/others/image-20210311153056539.png)

![image-20210311153133798](https://xuemingde.com/pages/image/others/image-20210311153133798.png)

### 数据一致性问题

![image-20210311153222409](https://xuemingde.com/pages/image/others/image-20210311153222409.png)

![image-20210311153254270](https://xuemingde.com/pages/image/others/image-20210311153254270.png)



### seata

####  什么是seata

官网介绍

![image-20210311152028639](https://xuemingde.com/pages/image/others/QxsNJBC7TiaLyZv.png)

![image-20210311153456333](https://xuemingde.com/pages/image/others/image-20210311153456333.png)

#### 分布式事务 Seata 解决方案

![image-20210311153638276](https://xuemingde.com/pages/image/others/image-20210311153638276.png)

##### AT 模式

![image-20210311153810577](https://xuemingde.com/pages/image/others/image-20210311153810577.png)

执行流程

![image-20210311154207648](https://xuemingde.com/pages/image/others/image-20210311154207648.png)

![image-20210311154303985](https://xuemingde.com/pages/image/others/image-20210311154303985.png)

![image-20210311154439965](https://xuemingde.com/pages/image/others/image-20210311154439965.png)



##### TCC 模式

![image-20210311154540770](https://xuemingde.com/pages/image/others/image-20210311154540770.png)

![image-20210311154603265](https://xuemingde.com/pages/image/others/image-20210311154603265.png)

**业务模型分 2 阶段设计**

![image-20210311154913051](https://xuemingde.com/pages/image/others/image-20210311154913051.png)

![image-20210311155124114](https://xuemingde.com/pages/image/others/image-20210311155124114.png)

##### Saga 模式

![image-20210311155236693](https://xuemingde.com/pages/image/others/image-20210311155236693.png)

##### XA 模式

![image-20210311155404143](https://xuemingde.com/pages/image/others/image-20210311155404143.png)

![image-20210311155447736](https://xuemingde.com/pages/image/others/image-20210311155447736.png)

![image-20210311155520511](https://xuemingde.com/pages/image/others/image-20210311155520511.png)

![image-20210311155622601](https://xuemingde.com/pages/image/others/image-20210311155622601.png)









