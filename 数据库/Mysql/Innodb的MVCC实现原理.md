理解MVCC之前，我们需要回顾了解一下数据库的一些其他相关知识点

（1）数据库为什么要有事务？

为了保证数据最终的一致性。

（2）事务包括哪几个特性？

原子性、隔离性、一致性、持久性。

（3）事务的并发引起了哪些问题？

事务并发会引起脏读、重复读、幻读问题。

（4）怎么解决事务并发出现的问题？

设置事务隔离级别，读未提交，读提交，重复读，序列化

（5）数据库通过什么方式保证了事务的隔离性？

通过加锁来实现事务的隔离性。 

（6）频繁的加锁会带来什么问题？

读数据的时候没办法修改。修改数据的时候没办法读取，极大的降低了数据库性能。

（7）数据库是如何解决加锁后的性能问题的？

MVCC 多版本控制实现读取数据不用加锁， 可以让读取数据同时修改。修改数据时同时可读取。



## 一、什么是MVCC？

MVCC是在并发访问数据库时，通过对数据做多版本管理，避免因为写锁的阻塞而造成读数据的并发阻塞问题。

通俗的讲就是MVCC通过保存数据的历史版本，根据比较版本号来处理数据的是否显示，从而达到读取数据的时候不需要加锁就可以保证事务隔离性的效果


## 二、Innodb MVCC实现的核心知识点

1、事务版本号

2、表的隐藏列。

3、undo log

4、 read view



### 2-1、事务版本号

每次事务开启前都会从数据库获得一个自增长的事务ID，可以从事务ID判断事务的执行先后顺序。



### 2-2、表格的隐藏列

**DB_TRX_ID:**        记录操作该数据事务的事务ID；

**DB_ROLL_PTR：**指向上一个版本数据在undo log 里的位置指针；

**DB_ROW_ID:**      隐藏ID ，当创建表没有合适的索引作为聚集索引时，会用该隐藏ID创建聚集索引;



### 2-3、Undo log

Undo log 主要用于记录数据被修改之前的日志，在表信息修改之前先会把数据拷贝到undo log 里，当事务进行回滚时可以通过undo log 里的日志进行数据还原。

**Undo log 的用途**

（1）保证事务进行rollback时的原子性和一致性，当事务进行回滚的时候可以用undo log的数据进行恢复。

 （2）用于MVCC快照读的数据，在MVCC多版本控制中，通过读取undo log的历史版本数据可以实现不同事务版本号都拥有自己独立的快照数据版本。

### 2-4、事务版本号、表格的隐藏列、undo log的关系

我们模拟一次数据修改的过程来让我们了解下事务版本号、表格隐藏的列和undo log他们之间的使用关系。

**（1）首先准备一张原始原始数据表**

![1014-F3zJiu](https://github.com/wuwenyishi/pages/raw/gh-pages/image/2022/04/07/1014-pZjl2q.jpg)



（2）开启一个事务A： 对user_info表执行 update  user_info set name =“李四”where id=1 会进行如下流程操作

1、首先获得一个事务编号 104

2、把user_info表修改前的数据拷贝到undo log

3、修改user_info表 id=1的数据

4、把修改后的数据事务版本号改成 当前事务版本号，并把DB_ROLL_PTR 地址指向undo log数据地址。

（3）最后执行完结果如图：

![img](https://github.com/wuwenyishi/pages/raw/gh-pages/image/2022/04/07/1017-gKgh48.jpg)

### 2-5、Read view

在innodb 中每个事务开启后都会得到一个read_view。副本主要保存了当前数据库系统中正处于活跃（没有commit）的事务的ID号，其实简单的说这个副本中保存的是系统中当前不应该被本事务看到的其他事务id列表。



### Read view 的几个重要属性

**trx_ids:**       当前系统活跃(未提交)事务版本号集合。

**low_limit_id:**   创建当前read view 时“当前系统最大**事务版本号**+1”。

**up_limit_id:**    创建当前read view 时“系统正处于**活跃事务**最小版本号”

**creator_trx_id:**  创建当前read view的事务版本号；



### Read view 匹配条件



（1）数据事务ID <up_limit_id 则显示

如果数据事务ID小于read view中的最小活跃事务ID，则可以肯定该数据是在当前事务启之前就已经存在了的,所以可以显示。

（2）数据事务ID>=low_limit_id 则不显示

如果数据事务ID大于read view 中的当前系统的最大事务ID，则说明该数据是在当前read view 创建之后才产生的，所以数据不予显示。

（3） up_limit_id <=**数据事务ID<**low_limit_id  则与活跃事务集合**trx_ids**里匹配

如果数据的事务ID大于最小的活跃事务ID,同时又小于等于系统最大的事务ID，这种情况就说明这个数据有可能是在当前事务开始的时候还没有提交的。

所以这时候我们需要把数据的事务ID与当前read view 中的活跃事务集合trx_ids 匹配:

**情况1:**    如果事务ID不存在于trx_ids 集合（则说明read view产生的时候事务已经commit了），这种情况数据则可以显示。

**情况2：**  如果事务ID存在trx_ids则说明read view产生的时候数据还没有提交，但是如果数据的事务ID等于creator_trx_id ，那么说明这个数据就是当前事务自己生成的，自己生成的数据自己当然能看见，所以这种情况下此数据也是可以显示的。

**情况3：**  如果事务ID既存在trx_ids而且又不等于creator_trx_id那就说明read view产生的时候数据还没有提交，又不是自己生成的，所以这种情况下此数据不能显示。



（4）不满足read view条件时候，从undo log里面获取数据

当数据的事务ID不满足read view条件时候，从undo log里面获取数据的历史版本，然后数据历史版本事务号回头再来和read view 条件匹配 ，直到找到一条满足条件的历史数据，或者找不到则返回空结果；



## 三、Innodb实现MCC的原理

![img](https://github.com/wuwenyishi/pages/raw/gh-pages/image/2022/04/07/1017-C85dvp.jpg)

### 3-1、模拟MVCC实现流程

下面我们通过开启两个同时进行的事务来模拟MVCC的工作流程。

**（1）创建user_info表，插入一条初始化数据**

![img](https://github.com/wuwenyishi/pages/raw/gh-pages/image/2022/04/07/1017-S1wPS3.jpg)



**（2）事务A和事务B同时对user_info进行修改和查询操作**

事务A：update user_info set name =”李四”

事务B：select * fom user_info where id=1



**问题：**

先开启事务A ，在事务A修改数据后但未进行commit，此时执行事B。最后返回结果如何。



**执行流程如下图：**

![img](https://github.com/wuwenyishi/pages/raw/gh-pages/image/2022/04/07/1018-WDgWpW.jpg)

**执行流程说明：**

**（1）事务A：开启事务，首先得到一个事务编号102；**

**（2）事务B：开启事务，得到事务编号103；**

**（3）事务A：进行修改操作，首先把原数据拷贝到undolog,然后对数据进行修改，标记事务编号和上一个数据版本在undo log的地址。**



![img](https://github.com/wuwenyishi/pages/raw/gh-pages/image/2022/04/07/1018-1vklPX.jpg)



**（4）事务B： 此时事务B获得一个read view ，read view对应的值如下**

![1018-OSba8r](https://github.com/wuwenyishi/pages/raw/gh-pages/image/2022/04/07/1018-O8GlSP.jpg)

**（5）事务B： 执行查询语句，此时得到的是事务A修改后的数据**

![img](https://github.com/wuwenyishi/pages/raw/gh-pages/image/2022/04/07/1019-GRx57G.jpg)



**（6）事务B： 把数据与read view进行匹配，**

数据事务ID为102 等于up_limit_id （这里不小于up_limit_id）

数据事务ID为102  小于low_limit_id

数据事务ID为102存在于  trx_ids，

数据事务ID为102不等于creator_trx_id

发现不满足read view显示条件，所以从undo lo获取历史版本的数据再和read view进行匹配，最后返回数据如下。

![img](https://github.com/wuwenyishi/pages/raw/gh-pages/image/2022/04/07/1019-YvOauz.jpg)


## 四、补充

### 各种事务隔离级别下的Read view 工作方式

RC(read commit) 级别下同一个事务里面的每一次查询都会获得一个新的read view副本。这样就可能造成同一个事务里前后读取数据可能不一致的问题（重复读）

![img](https://github.com/wuwenyishi/pages/raw/gh-pages/image/2022/04/07/1019-x1NF8M.jpg)



RR(重复读)级别下的一个事务里只会获取一次read view副本，从而保证每次查询的数据都是一样的。

![img](https://github.com/wuwenyishi/pages/raw/gh-pages/image/2022/04/07/1019-EwbKQo.jpg)



READ_UNCOMMITTED 级别的事务不会获取read view 副本。

### 快照读和当前读



**快照读**

快照读是指读取数据时不是读取最新版本的数据，而是基于历史版本读取的一个快照信息（mysql读取undo log历史版本) ，快照读可以使普通的SELECT 读取数据时不用对表数据进行加锁，从而解决了因为对数据库表的加锁而导致的两个如下问题

1、解决了因加锁导致的修改数据时无法对数据读取问题;

2、解决了因加锁导致读取数据时无法对数据进行修改的问题;



**当前读**

当前读是读取的数据库最新的数据，当前读和快照读不同，因为要读取最新的数据而且要保证事务的隔离性，所以当前读是需要对数据进行加锁的（Update    delete    insert   select ....lock in share mode   select for update 为当前读）

## 五、讨论

### MVCC是否有解决幻读问题？

看到有很多网友对这个话题有讨论，这里补充一下和大家理一理这个问题，首先我通过验证得出来的结论是MVCC不存在幻读问题的，但也并不是说MVCC解决了幻读的问题，经过理论的推断和验证得到的结论是在**快照读的情况下可以避免幻读问题，在当前读的情况下则需要使用间隙锁来解决幻读问题的**。





### MVCC不存在幻读问题（RR级别的情况下）

首先确认一点MVCC属于快照读的，在进行快照读的情况下是不会对数据进行加锁，而是基于事务版本号和undo历史版本读取数据，其实上面的文章已经说得很清楚了，我们根据上面的MVCC流程来推导，无论如何在MVCC的情况下都是不会出现幻读的问题的，如下图。

1、开启事务1，获得事务ID为1。

2、事务1执行查询，得到readview。

3、开始事务2。

4、执行insert。

5、提交事务2。

6、执行事务1的第二次查询 (因为这里是RR级别，所以不会再去获得readview,还是使用第一次获得的readview)

7、最后得到的结果是，插入的数据不会显示，因为插入的数据事务ID大于等于 readview里的最大活跃事务ID。

![img](https://github.com/wuwenyishi/pages/raw/gh-pages/image/2022/04/07/1020-iIrKAA.jpg)



**实际案例：**

首先关闭数据库的自动提交事务功能。

![img](https://github.com/wuwenyishi/pages/raw/gh-pages/image/2022/04/07/1020-8492ES.jpg)



 然后使用手动提交事务的方式，进行一次快照读，但是不提交事务，然后启动插入数据的事务，进行数据插入 commit，结果我发现在使用快照读的时候，数据是可以插入成功的，那这也就说明了一个问题，**快照读的时候就根本没加锁，否则的话数据是不可能插入成功的，而且在插入数据提交成功后，我们执行第二条查询 语句是读取不到中间插入的这条数据的，这也就说明在没有加锁的情况下，基于历史版本的MVCC快照读是可以避免幻读问题的**。

![1020-gEDxYX](https://github.com/wuwenyishi/pages/raw/gh-pages/image/2022/04/07/1020-iUMz8J.jpg)



### 当前读存的幻读问题解决方案

上面我们已经论证了在RR级别快照读的情况下，是不存在幻读问题的。  因为会基于历史版本读取数据，但是当前读的话就不同了，当前读每次都会读取最新的数据。所以两次读取中间如果可以插入数据，那么就肯定会造成幻读问题，所以在当前读的情况下就必须通过一种方式来解决幻读问题，而这种方式就是采用加锁来解决。



**实际案例：**

首先关闭数据库的自动提交事务功能，  使用当前读的方式演示和上面一样的流程，结果发现在当前读的时候没有提交事务之前是根本无法进行数据插入的，所以这也就说明了，使用当前读的时候会对这个范围内的数据进行加锁，所以无法在查询的范围内进行数据插入，这无疑也证明了在当前读的情况下mysql是使用锁的机制来避免出现重复读和幻读问题的。

![img](https://github.com/wuwenyishi/pages/raw/gh-pages/image/2022/04/07/1021-hFAdty.jpg)


> 原文： [数据库基础（四）Innodb MVCC实现原理 - 知乎](https://zhuanlan.zhihu.com/p/52977862)