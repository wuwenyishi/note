

## 分布式事务（二）

> 文章来源：
>
> * http://codin.im/2018/06/25/implementing-distributed-transaction-with-spring/
> * https://juejin.cn/post/6844903666273484814
> * https://blog.csdn.net/jiangyu1013/article/details/84397366



### @Transactional 详解

![image-20210310134240771](https://github.com/wuwenyishi/pages/raw/gh-pages/image/others/image-20210310134240771.png)

> - **编程式事务**使用TransactionTemplate或者直接使用底层的PlatformTransactionManager。对于编程式事务管理，spring推荐使用TransactionTemplate。
> - **声明式事务**是建立在AOP之上的。其本质是对方法前后进行拦截，然后在目标方法开始之前创建或者加入一个事务，在执行完目标方法之后根据执行情况提交或者回滚事务。声明式事务最大的优点就是不需要通过编程的方式管理事务，这样就不需要在业务逻辑代码中掺杂事务管理的代码，只需在配置文件中做相关的事务规则声明(或通过基于@Transactional注解的方式)，便可以将事务规则应用到业务逻辑中



![image-20210310140333912](https://github.com/wuwenyishi/pages/raw/gh-pages/image/others/image-20210310140333912.png)

![image-20210310141931318](https://github.com/wuwenyishi/pages/raw/gh-pages/image/others/image-20210310141931318.png)

![img](https://github.com/wuwenyishi/pages/raw/gh-pages/image/others/20181123164638554.png)



### JTA

![image-20210310133454787](https://github.com/wuwenyishi/pages/raw/gh-pages/image/others/image-20210310133454787.png)

![image-20210310133718647](https://github.com/wuwenyishi/pages/raw/gh-pages/image/others/image-20210310133718647.png)