CAS compare and swap （比较和交换）



# CAS的执行过程  

​    ![](https://xuemingde.com/pages/image/others/Foqp87.png)



过程简单描述：

A从B那里拿到一个数据是0，然后A将0改为了1，然后交给B，在交给B之前需要判断B的当前值是不是A最初拿到的0，如果是，就把1交给B，如果不是，A就会拿到B现有的值1，然后A将1改为2，然后再交给B，交给B之前再次判断是不是之前的值1，如果是把1交给B，不是再次…….



# CAS中存在的问题  

## ABA问题    

ABA问题是指在CAS操作时，其他线程将变量值A改为了B，但是又被改回了A，等到本线程使用期望值A与当前变量进行比较时，发现变量A没有变，于是CAS就将A值进行了交换操作，但是实际上该值已经被其他线程改变过。

解决方式：**加版本号** 

在JDK的 *java.util.concurrent.atomic* 包中提供了 `AtomicStampedReference` 来解决ABA问题，该类的compareAndSet是该类的核心方法，实现如下：

```java
public boolean compareAndSet(V   expectedReference,
                                 V   newReference,
                                 int expectedStamp,
                                 int newStamp) {
        Pair<V> current = pair;
        return
            expectedReference == current.reference &&
            expectedStamp == current.stamp &&
            ((newReference == current.reference &&
              newStamp == current.stamp) ||
             casPair(current, Pair.of(newReference, newStamp)));
    }
```

![bkYcDR](https://xuemingde.com/pages/image/others/bkYcDR.png)

> [参考文章](https://elsef.com/2020/03/08/%E5%A6%82%E4%BD%95%E7%90%86%E8%A7%A3ABA%E9%97%AE%E9%A2%98/)



## 修改值时的原子性问题  

底层的最终实现是使用了lock cmpxchg 指令来保证的。

cmpxchg 是非原子性的，但lock这个指令锁住了值，不允许其他cpu修改

 ![VV5fET](https://xuemingde.com/pages/image/others/VV5fET.png)





