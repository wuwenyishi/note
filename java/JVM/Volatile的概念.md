Volatile有两大作用

1. 保证线程间可见性
2. 禁止指令重排序





## CPU的乱序执行  

![](https://xuemingde.com/pages/image/others/S8d07j.png)

CPU为什么支持乱序执行？是为了提高效率。

> CPU的速度要比内存的速度快100倍

如果按顺序执行，有两条指令，第一次指令去内存中读取数据，第二个指令要等99%的时间让第一个指令执行完再去执行，浪费CPU的时间，执行效率就很低。

在CPU的优化时候，有可能是这样执行的，在第一个指令执行的时候，会把第二个指令拿出来执行，但是有个前提条件，两个指令不能存在前后的依赖关系。



## 解释一下对象的创建过程  

```java
public static void main(String[] args) {
    Object o = new Object();
}
```

![](https://xuemingde.com/pages/image/others/hcd0hK.png)

new 出一个对象至少需要三步：

 ![](https://xuemingde.com/pages/image/others/xYGO2T.png)

第一步：new ，申请一块内存空间，用来装我new出来的对象。如上图所示，申请的内存空间里肯定有一个成员变量m，这个m的值是他的默认值，根据数据类型来定，所以这个m的默认值是0； ![](https://xuemingde.com/pages/image/others/mY2ArO.png)



第二步：给成员变量赋值，所以此时的m值变为了8。**这个过程就是对象的半初始化状态。** ![](https://xuemingde.com/pages/image/others/zdwTUK.png)



第三步：建立关联，局部变量 t ，t里面存储一个指针，这个指针就指向一个对象。 ![](https://xuemingde.com/pages/image/others/6rJ0tu.png)





## 双重检锁（DCL）单例要不要加Volatile



双重检锁（DCL）单例

```java
public class TestDemo {
    //2.要不要加 volatile
    private static volatile TestDemo INST;
    private TestDemo(){}

    public static TestDemo getInstance() {
        //1.这个地方为什么要为空判断？
        if (INST == null) {
            synchronized (TestDemo.class) {
                if (INST == null) {
                    try {
                        Thread.sleep(1);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    INST = new TestDemo();
                }
            }
        }
        return INST;
    }
}
```



**为什么要进来就为空判断？**

假如去掉这个判断，现在有1000个线程都来抢这把锁，上来就开始锁竞争，最终只有一个线程能够持有锁，去new对象，剩下的所有线程都参与了锁竞争过程，这样会降低效率。如果加上了判断，只要一个线程new出了对象，就不用去参与锁竞争过程了。

**要不要加Volatile ？？？**

答案是**必须要加**的。

为什么呢？

 ![](https://xuemingde.com/pages/image/others/UF3vK1.png)



前面讲了，new一个对象至少有三步，在线程1创建对象时，在申请内存空间的时候，发生了指令重排序（我们前面也讲了，CPU会有乱序执行），第二步和第三步发生了执行顺序的颠倒，线程1在执行完第一步去执行了第三步，这时候局部变量INST指向了创建的对象，此时线程2判断INST不再为null，而线程1还没有执行完第三步，此时线程2拿到的对象是一个半成品，而成员变量的值还是他的默认值。

Volatile的一个作用就是禁止指令重排序，用Volatile修饰的任何内存空间（对象），在他上面执行的指令不可以乱序。





## Volatile为什么会禁止重排的？

内存屏障！！



### 内存屏障 （JVM级别）

JVM级别的内存屏障分为四种

 ![](https://xuemingde.com/pages/image/others/yGGBJz.png)



JVM是一种规范，这个规范要求被Volatile修饰的东西要实现内存屏障。但是硬件级别或者hostpot级别或者操作系统级别如何实现的，实际上与JVM的规定没有直接的关系。



 ![](https://xuemingde.com/pages/image/others/vSRQKY.png)







