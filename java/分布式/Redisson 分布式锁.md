

> 推荐文章：[Springboot使用分布式锁（基于redisson）](https://blog.csdn.net/tianyaleixiaowu/article/details/90036180)， 



## **常见的分布式锁实现方案**

![image.png](https://xuemingde.com/pages/image/others/1616407355479-40ffe6b9-5476-488a-a8a1-6970e53f9bbc.png) 

## Redisson原理分析图解

![image](https://xuemingde.com/pages/image/others/1616405735205-372355c1-5217-48da-a0e5-370c220d189b.jpeg)



### 加锁机制

![image.png](https://xuemingde.com/pages/image/others/1616405769058-8e6279e7-68bd-4f1e-bd15-8090693c9687.png)

### watch dog自动延期机制

![image.png](https://xuemingde.com/pages/image/others/1616405802606-06d89d99-3382-4f4c-8432-40427e6f3fd8.png)



### 为啥要用lua脚本呢

![image.png](https://xuemingde.com/pages/image/others/1616405833923-f3d5a659-b9a8-4f7c-b32e-3b42fbe3e21f.png)

### 可重入加锁机制

![image.png](https://xuemingde.com/pages/image/others/1616405855881-10cb381d-4a17-415e-a61d-61e3724341fb.png)



## Redis分布式锁的缺点

![image.png](https://xuemingde.com/pages/image/others/1616405899527-5f409d65-9559-4a54-8e44-1f573b0bfd08.png)