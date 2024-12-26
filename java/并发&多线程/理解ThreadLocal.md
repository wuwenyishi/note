


#### ThreadLocal是什么

官方解释：

![](https://mmbiz.qlogo.cn/mmbiz_png/3eqXwttvOLtjzibSYqvuZfB4TaSYDTCb152jBhzzAMSn7gJiaU8wNL1dO5el4ibMq5dkia9OEEEIoic7dia4DJdLt7Ww/0?wx_fmt=png&from=appmsg)



- ThreadLocal是一个关于创建线程局部变量的类，主要作用是存储线程的局部变量，做到数据隔离。保存在ThreadLocal中的数据只属于当前线程，对其他线程是不可见的。在多线程环境下，可以防止自己的变量被其他线程修改而导致数据不一致性。
- ThreadLocal设计的目的就是为了能够使得当前线程拥有属于自己的变量，并不是为了解决并发或者共享变量的问题
- 底层使用ThreadLocalMap实现，每个线程都拥有自己的ThreadLocalMap，内部是继承了WeakReference的Entry数组，包含的Key为ThreadLocal，值为Object。 



#### 原理浅析

![](https://mmbiz.qlogo.cn/mmbiz_png/3eqXwttvOLtjzibSYqvuZfB4TaSYDTCb1wTpIOc8x46YYEmzjwnKwUVk6TPZvCcTLRWqlHJH09Zyic5ljZKEX21A/0?wx_fmt=png&from=appmsg)

![](https://mmbiz.qlogo.cn/mmbiz_png/3eqXwttvOLtjzibSYqvuZfB4TaSYDTCb1ibMb3RULdrsSlQMsc2SMdQicPmjIg4Pg58UPRUEhX8GDLudKVEmXvKdA/0?wx_fmt=png&from=appmsg)

![](https://mmbiz.qlogo.cn/mmbiz_png/3eqXwttvOLtjzibSYqvuZfB4TaSYDTCb1Ng1jEt8LqibXchtGiccoMGmMs2r09mWTJTQtcIpslWQqlSc1Vj9sd5Rw/0?wx_fmt=png&from=appmsg)

#### ThreadLocalMap 

![](https://mmbiz.qlogo.cn/mmbiz_png/3eqXwttvOLtjzibSYqvuZfB4TaSYDTCb104dMGjic1mKnx4yJmABR71xbMyLLvEicDGWklYWoXn0dibS9sLUXMQd8Q/0?wx_fmt=png&from=appmsg)

![](https://mmbiz.qlogo.cn/mmbiz_png/3eqXwttvOLtjzibSYqvuZfB4TaSYDTCb1DZndFlkvzoBMAcotOhtkfK5vZm769SoIcibylO2icNdoib2nAkAttl8GQ/0?wx_fmt=png&from=appmsg)

```java
private void set(ThreadLocal<?> key, Object value) {

    Entry[] tab = table;
    int len = tab.length;
    //获取 hash 值，用于数组中的下标
    int i = key.threadLocalHashCode & (len-1);

    //如果数组该位置有对象则进入
    for (Entry e = tab[i];
         e != null;
         e = tab[i = nextIndex(i, len)]) {
        ThreadLocal<?> k = e.get();

        //k 相等则覆盖旧值
        if (k == key) {
            e.value = value;
            return;
        }

        //此时说明此处 Entry 的 k 中的对象实例已经被回收了，需要替换掉这个位置的 key 和 value
        if (k == null) {
            replaceStaleEntry(key, value, i);
            return;
        }
    }

    //创建 Entry 对象
    tab[i] = new Entry(key, value);
    int sz = ++size;
    if (!cleanSomeSlots(i, sz) && sz >= threshold)
        rehash();
}


//获取 Entry
private Entry getEntry(ThreadLocal<?> key) {
    int i = key.threadLocalHashCode & (table.length - 1);
    Entry e = table[i];
    if (e != null && e.get() == key)
        return e;
    else
        return getEntryAfterMiss(key, i, e);
}
```





#### ThreadLocalMap的内存泄露问题

我们知道ThreadLocalMap使用ThreadLocal的弱引用作为key，如果一个ThreadLocal没有外部强引用时，在GC的时候，这个ThreadLocal必然会被回收，但是对应的value不会被回收掉直到线程结束才会被回收。如果当前线程一直处于运行中，那么这些Entry对象中的value就可能一直无法回收，就会发生内存泄露。实际开发中，会使用线程池维护线程的创建和复用，线程为了复用是不会主动结束的，那么ThreadLocal设置的value值就一直被引用，就会发生内存泄露。



#### 如何避免内存泄露

1. 线程执行完毕调用remove方法
2. 使用ThreadLocal，建议用static修饰 static ThreadLocal<HttpHeader> headerLocal = new ThreadLocal();
3. 可以在使用ThreadLocal的类上实现AutoCloseable接口，把需要释放的资源（remove）写在close方法里面，把此类写在try后的()里

   代码示例：

```java
public class AppContext implements AutoCloseable {

    private static final ThreadLocal<User> USER = new ThreadLocal<>();

    public AppContext(user) {
        if(user != null){
            USER.set(user);
        }
    }

    @Override
    public  void close() {
        USER.remove();
    }

    public static User getUserId() {
        return USER.get();
    }
}

//执行完线程自动释放资源
 try (AppContext appContext = new AppContext(user)) {
          filterChain.doFilter(request, response);
}
```



#### AutoCloseable是什么

此接口是在jdk1.7之后引入的，可实现自动关闭资源。

值得注意的是，在使用AutoCloseable的时候，必须配合try使用才能自动释放资源，因为利用try(AppContext appContext = AppContext(user)))才能自动调用close()方法。



> 相关文章： [文章1](https://www.justdojava.com/2019/05/12/java-threadlocal/)  