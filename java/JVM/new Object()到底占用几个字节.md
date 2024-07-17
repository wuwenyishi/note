## **Java内存模型**

对象内存中可以分为三块区域：对象头(Header)，实例数据(Instance Data)和对齐填充(Padding)，以64位操作系统为例(未开启指针压缩的情况)Java对象布局

如下图所示：

![](https://xuemingde.com/pages/image/2023/10/18/084506S4jE0c.png)

其中对象头中的Mark Word中的详细信息在文章synchronized锁升级原理中有详细介绍。上图中的对齐填充不是一定有的，如果对象头和实例数据加起来刚好是8字节的倍数，那么就不需要对齐填充。



## **Object obj=new Object()占用字节**

这是网上很多人都会提到的一个问题，那么结合上面的Java内存布局，我们来分析下，以64位操作系统为例，new Object()占用大小分为两种情况：

- 未开启指针压缩        占用大小为：8(Mark Word)+8(Class Pointer)=16字节
- 开启了指针压缩(默认是开启的)        开启指针压缩后，Class Pointer会被压缩为4字节，最终大小为：8(Mark Word)+4(Class Pointer)+4(对齐填充)=16字节

结果到底是不是这个呢？我们来验证一下。首先引入一个pom依赖：

```xml
<dependency>
            <groupId>org.openjdk.jol</groupId>
            <artifactId>jol-core</artifactId>
            <version>0.10</version>
        </dependency>

```

```java
package com.zwx.jvm;

import org.openjdk.jol.info.ClassLayout;

public class HeapMemory {
    public static void main(String[] args) {
        Object obj = new Object();
        System.out.println(ClassLayout.parseInstance(obj).toPrintable());
    }
}
```

输出结果如下：

![](https://xuemingde.com/pages/image/2023/10/18/085037Hn1vqZ.png)

最后的结果是16字节，没有问题，这是因为**`默认开启了指针压缩`**，那我们现在把指针压缩关闭之后再去试试。

```
-XX:+UseCompressedOops  开启指针压缩
-XX:-UseCompressedOops  关闭指针压缩
```

![](https://xuemingde.com/pages/image/2023/10/18/085355lpYvrI.png)

得到如下结果：

![](https://xuemingde.com/pages/image/2023/10/18/085421uRv3Tm.png)

可以看到，这时候已经没有了对齐填充部分了，但是占用大小还是16位。



> 以上是Object 内无任何成员变量的情况



如果有成员变量，以下是成员变量数据类型占用字节大小情况

| 类型      | 存储    | 取值范围               |
| ------- | ----- | ------------------ |
| int     | 4byte | -2^31 ~ 2^31 - 1   |
| short   | 2byte | -2^15 ~ 2^15 - 1   |
| long    | 8byte | (-2)^63 ~ 2^63 - 1 |
| byte    | 1byte | -128 ~ 127         |
| float   | 4byte |                    |
| double  | 8byte |                    |
| boolean | 1byte |                    |
| char    | 2byte |                    |

> Hotspot实现的JVM开启内存压缩的规则（64位机器）：

- 4G以下，直接砍掉高32位
- 4G~32G，默认开启内存压缩
- 32G以上，压缩无效，使用64位



下面我们再来演示一下如果一个对象中带有属性之后的大小。

新建一个类，内部只有一个byte属性：

```java
package com.zwx.jvm;

public class MyItem {
    byte i = 0;
}
```

然后分别在开启指针压缩和关闭指针压缩的场景下分别输出这个类的大小。

```java
package com.zwx.jvm;

import org.openjdk.jol.info.ClassLayout;

public class HeapMemory {
    public static void main(String[] args) {
        MyItem myItem = new MyItem();
        System.out.println(ClassLayout.parseInstance(myItem).toPrintable());
    }
}
```

开启指针压缩,占用16字节：

![](https://xuemingde.com/pages/image/2023/10/18/0900167BAefi.png)

关闭指针压缩，占用24字节：

![](https://xuemingde.com/pages/image/2023/10/18/090103h7bETx.png)

这个时候就能看出来开启了指针压缩的优势了，**如果不断创建大量对象，指针压缩对性能还是有一定优化的**。

