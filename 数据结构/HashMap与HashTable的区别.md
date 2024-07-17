## 区别

* HashTable 线程同步（线程安全），HashMap非线程同步（线程不安全）。
* HashTable 不允许键和值有空值，HashMap允许键和值有空值。
* HashTable使用Enumeration,HashMap使用iterator。
* HashTable的默认大小是11，扩容方式是old * 2 +1，HashMap的默认大小是16，扩容方式是2的指数倍。
* HashTable继承于Dictionary类，HashMap继承于AbstractMap类。


## HashMap有哪些线程安全的方式

### Conllections.synchronizedMap 
此方法返回一个新的Map，这个新的Map就是线程安全的。
特点：
通过Conllections.synchronizedMap来封装所有不安全的HashMap方法，就连toString，hashCode都进行了封装，封装的关键点有：
  * 使用了synchronized来进行互斥
  * 使用代理模式new一个新的类，这个类同样实现了Map接口。

在HashMap上面，synchronized锁住的是对象，所以第一个申请的得到锁，其他线程将进入阻塞，等到唤醒。

优点：
代码实现简单。

缺点：
从锁的角度来看，上来就加锁，基本上锁住了尽可能大的代码块，性能比较差。

### ConcurrentHashMap 
特点
重新写了HashMap，比较大的改变有以下几点：
* 使用了新的锁机制，把Hashmap进行了拆分，拆分成了多个独立的块，这样在高并发的情况下减少了锁冲突的可能。
* 使用的是NonfairSync，这个特性调用的是CAS指令来确保原子性与互斥性，当如果多个线程恰好操作到同一个segment上面，那么只会有一个线程能够运行。

优点：
需要互斥的代码段比较少，性能也好，ConcurrentHashMap 把整个Map切分成多个块，发生锁碰撞的几率大大降低了，性能会比较好。

缺点：
代码繁琐。