
#### Map集合都有哪些实现
HashMap   
HashTable    
LinkedHashMap   
TreeMap   
ConcurrentHashMap


#### **HashMap 的数据结构**
哈希表结构（链表散列：数组+链表）实现，结合数组和链表的优点。当链表长度超过 8 时，链表转换为红黑树。

#### putIfAbsent与put的区别
putIfAbsent：如果传入key对应的value已经存在，就返回存在的value，不进行替换。如果不存在，就添加key和value，返回null。
put：key存在覆盖，不存在插入。


#### 什么是哈希冲突
当输入两个不同值，根据同一散列函数计算出相同的散列值的现象，我们就把它叫做碰撞（哈希碰撞）。

#### HashMap 使用哪种方法来解决哈希冲突？

使用链表和红黑树来解决哈希冲突



#### HashMap 的扩容为什么是 2^n 
这样做的目的是为了让散列更加均匀，从而减少哈希碰撞，以提供代码的执行效率。

#### HashMap 在 JDK 7 多线程中使用会导致什么问题
HashMap 在 JDK 7 中会导致死循环的问题。因为在 JDK 7 中，多线程进行 HashMap 扩容时会导致链表的循环引用，这个时候使用 get() 获取元素时就会导致死循环，造成 CPU 100% 的情况。


#### HashMap 在 JDK 7 和 JDK 8 中有哪些不同

- 存储结构：JDK 7 使用的是数组 + 链表；JDK 8 使用的是数组 + 链表 + 红黑树。
- 存放数据的规则：JDK 7 无冲突时，存放数组；冲突时，存放链表；JDK 8 在没有冲突的情况下直接存放数组，有冲突时，当链表长度小于 8 时，存放在单链表结构中，当链表长度大于 8 时，树化并存放至红黑树的数据结构中。链表转换为红黑树的另一个条件，哈希表长度必须大于等于64才会转换，否则会扩容。
- 插入数据方式：JDK 7 使用的是头插法（先将原位置的数据移到后 1 位，再插入数据到该位置）；JDK 8 使用的是尾插法（直接插入到链表尾部/红黑树）。



#### 链表转换为红黑树
**当链表长度大于等于8的时候，链表可能会被转换为红黑树**，这里之所以说是可能，是因为还要满足另外一个条件：哈希表长度大于等于64，否则哈希表会尝试扩容。


#### 红黑树退化成链表
**当红黑树节点小于等于6的时候，红黑树会转换成普通的链表。**


#### **HashMap 扩容 **
在没有设置容量的时候，默认容量是16，而HashMap的加载因子是0.75，当存储的元素个数超过了闸值（当前容量值*0.75），就会自动触发扩容，扩容的大小事原来的一倍（当前容量值 * 2）。
扩容过程：
遍历整个数组，将同一个位置的元素按链表顺序取出，先将当前元素指向的下一个元素存起来，一个一个存放到新表的位置中（记住不一定是同一位置，因为长度变了，所有数组数据都要重新Hash，因为长度变了，Hash的规则也变了），根据新数组长度，重新生成数组索引，将当前位置的元素链表头指向即将新加入的元素，然后放入数组中，完成同一位置元素链表的拼接，最先添加的元素总在链表末尾，然后继续循环，拿出下一个元素。


#### 重新调整HashMap大小存在什么问题（JDK 7）
当重新调整HashMap大小的时候，确实存在条件竞争，因为如果两个线程都发现HashMap需要重新调整大小了，它们会同时试着调整大小。在调整大小的过程中，存储在链表中的元素的次序会反过来，因为移动到新的bucket位置的时候，HashMap并不会将元素放在链表的尾部，而是放在头部，这是为了避免尾部遍历(tail traversing)。如果条件竞争发生了，那么就死循环了。(多线程的环境下不使用HashMap）。


#### 多线程环境下使用HashMap为什么会出现死循环
![](https://xuemingde.com/pages/image/others/20210315094833.png)

![image-20210315094912288](https://xuemingde.com/pages/image/others/image-20210315094912288.png)


#### HashMap put执行过程
![image](https://xuemingde.com/pages/image/others/1606100383222-b82eb9d3-e207-4c7a-9a1f-d401c97e0a0e.png)


源码解析：
```java
/**
 * Implements Map.put and related methods
 *
 * @param hash hash for key
 * @param key the key
 * @param value the value to put
 * @param onlyIfAbsent if true, don't change existing value
 * @param evict if false, the table is in creation mode.
 * @return previous value, or null if none
 */
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
			   boolean evict) {
	Node<K,V>[] tab; Node<K,V> p; int n, i;
	//底层存储数据的数组tab为空，进行第一次扩容，同时初始化tab
	if ((tab = table) == null || (n = tab.length) == 0)
		n = (tab = resize()).length;
	//计算要存储数据的下标index，如果当前位置为null，直接插入
	if ((p = tab[i = (n - 1) & hash]) == null)
		tab[i] = newNode(hash, key, value, null);
	else {
		Node<K,V> e; K k;
		//tab[i]的首个元素就是key，直接覆盖
		if (p.hash == hash &&
			((k = p.key) == key || (key != null && key.equals(k))))
			e = p;
		//tab[i]为TreeNode，进行红黑树的插入
		else if (p instanceof TreeNode)
			e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
		//链表遍历
		else {
			for (int binCount = 0; ; ++binCount) {
				if ((e = p.next) == null) {
					p.next = newNode(hash, key, value, null);
					//链表长度大于8，转换为红黑树进行处理
					if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
						treeifyBin(tab, hash);
					break;
				}
				//找到已经存在的key，直接覆盖
				if (e.hash == hash &&
					((k = e.key) == key || (key != null && key.equals(k))))
					break;
				p = e;
			}
		}
		if (e != null) { // existing mapping for key
			V oldValue = e.value;
			if (!onlyIfAbsent || oldValue == null)
				e.value = value;
			afterNodeAccess(e);
			return oldValue;
		}
	}
	++modCount;
	//数据插入完成，size超过了扩容的阀值(容量*负载因子)，进行扩容
	if (++size > threshold)
		resize();
	afterNodeInsertion(evict);
	return null;
}
```


#### HashMap get执行过程
先得到key的hash值，再把这个hash值与length-1按位与（取余），得到table数组的下标。取出这个下标值的key，与传入的key比较，如果相同那就是这个了。如果不同呢，那就沿着这个单向链表向后找，直到找到或找到结束也找不到。


#### HashMap 在 put 时，如果数组中已经有了这个 key，我不想把 value 覆盖怎么办？取值时，如果得到的 value 是空时，想返回默认值怎么办？
如果数组有了 key，但不想覆盖 value ，可以选择 putIfAbsent 方法，这个方法有个内置变量 onlyIfAbsent，内置是 true ，就不会覆盖，我们平时使用的 put 方法，内置 onlyIfAbsent 为 false，是允许覆盖的。
取值时，如果为空，想返回默认值，可以使用 getOrDefault 方法，方法第一参数为 key，第二个参数为你想返回的默认值，如 map.getOrDefault(“2”,“0”)，当 map 中没有 key 为 2 的值时，会默认返回 0，而不是空。


#### HashMap和HashTable之间的区别

1. 继承父类不同

Hashtable继承自Dictionary类，而HashMap继承自AbstractMap类。但二者都实现了Map接口


2. 线程安全不同

HashMap不是线程安全的。
Hashtable是线程安全的，因为它每个方法中都加入了Synchronize。


3. 是否提供了contains方法

HashMap把Hashtable的contains方法去掉了，改成containsValue和containsKey，因为contains方法容易让人引起误解。
Hashtable则保留了contains，containsValue和containsKey三个方法，其中contains和containsValue功能相同。


4. key和value是否允许null值

两者的key和value都是对象，key不能重复，value能重复。
HashMap的key可以为null，但当且仅当只能有一个key为null，但可以有多个value为null。当get()方法返回null值时，可能是 HashMap中没有该键，也可能使该键所对应的值为null。因此，在HashMap中不能由get()方法来判断HashMap中是否存在某个键， 而应该用containsKey()方法来判断。
Hashtable中key和value都不允许出现null值。但是如果在Hashtable中有类似put(null,null)的操作，编译同样可以通过，因为key和value都是Object类型，但运行时会抛出NullPointerException异常，这是JDK的规范规定的。


5. 两个遍历方式的内部实现上不同

Hashtable、HashMap都使用了 Iterator（迭代器）。而由于历史原因，Hashtable还使用了Enumeration（枚举）的方式 。


6. hash值不同

HashTable直接使用对象的hashCode。
HashMap重新计算hash值。


7. 内部实现使用的数组初始化和扩容方式不同

 HashTable在不指定容量的情况下的默认容量为11，而HashMap为16。
Hashtable不要求底层数组的容量一定要为2的整数次幂，而HashMap则要求一定为2的整数次幂。
Hashtable扩容时，将容量变为原来的2倍加1，而HashMap扩容时，将容量变为原来的2倍。


#### 线程安全的Map都有哪些
线程安全的有HashTable、ConcurrentHashMap、SynchronizedMap，性能最好的是ConcurrentHashMap。


#### 为什么说HashMap是不安全的
HashMap会进行resize操作，在resize操作的时候会造成线程不安全。下面将举两个可能出现线程不安全的地方。

1. put的时候导致的多线程数据不一致。

比如有两个线程A和B，首先A希望插入一个key-value对到HashMap中，首先计算记录所要落到的桶的索引坐标，然后获取到该桶里面的链表头结点，此时线程A的时间片用完了，而此时线程B被调度得以执行，和线程A一样执行，只不过线程B成功将记录插到了桶里面，假设线程A插入的记录计算出来的桶索引和线程B要插入的记录计算出来的桶索引是一样的，那么当线程B成功插入之后，线程A再次被调度运行时，它依然持有过期的链表头但是它对此一无所知，以至于它认为它应该这样做，如此一来就覆盖了线程B插入的记录，这样线程B插入的记录就凭空消失了，造成了数据不一致的行为。

2. HashMap的get操作可能因为resize而引起死循环（cpu100%）



#### ConcurrentHashMap是如何实现线程安全的
ConcurrentHashMap的put方法和HashMap的逻辑相差不大，主要是新增了线程安全部分，在添加元素时候，采用synchronized来保证线程安全，然后计算size的时候采用CAS操作进行计算。
put的简单流程：

1. 判断key和vaule是否为空，如果为空，直接抛出异常。
2. 判断table数组是否已经初始化完毕，如果没有初始化，进行初始化。
3. 计算key的hash值，如果该位置为空，直接构造节点放入。
4. 如果table正在扩容，进入帮助扩容方法。
5. 最后开启同步锁，进行插入操作，如果开启了覆盖选项，直接覆盖，否则，构造节点添加到尾部，如果节点数超过红黑树阈值，进行红黑树转换。如果当前节点是树节点，进行树插入操作。
6. 最后统计size大小，并计算是否需要扩容。



get的简单流程：

1. 直接计算hash值，查找的节点如果key和hash值相等，直接返回该节点的value就行，value值使用了volatile关键字修饰了。
2. 如果table正在扩容，就调用ForwardingNode的find方法查找节点。
3. 如果没有扩容，直接循环遍历链表，查找到key和hash值一样的节点值即可。



扩容：
HashMap的线程安全问题大部分出在扩容(rehash)的过程中。
ConcurrentHashMap的扩容只针对每个segment中的HashEntry数组进行扩容。
由put的源码可知，ConcurrentHashMap在rehash的时候是有锁的，所以在rehash的过程中，其他线程无法对segment的hash表做操作，这就保证了线程安全。


> 具体可参考文章：[https://m.php.cn/faq/446158.html](https://m.php.cn/faq/446158.html)



#### hashCode() 和 equals() 方法的重要性体现在什么地方
哈希表（HashTable）又叫做散列表，根它通过把key映射到表中一个位置来访问记录，以加快查找速度。这个映射函数就叫做散列（哈希）函数，存放记录的数组叫做散列表。
哈希表是一个时间和空间上平衡的例子。如果没有空间的限制，我们可以直接用键来作为数组的索引，这样可以将查找时间做到最快（O(1)）。如果没有时间的限制，我们可以使用无序链表进行顺序查找，这样只需要很少的内存



> 参考文章：[深入理解JDK8 HashMap](https://cloud.tencent.com/developer/article/1609351)


