


四种：强、软、弱、虚。

- 强引用：  
  有引用变量指向时永远不会被垃圾回收，哪怕内存不足时，JVM宁愿抛出OutOfMemory错误也不会回收这种对象。如果想中断强引用与对象之间的联系，可以显示的将强引用赋值为null，这样一来，JVM就可以适时的回收对象了
  例子：`M m = new M();` 
  `m = null;` 此时垃圾回收器才会回收。
  ​


- 软引用  
  在内存足够的时候，软引用对象不会被回收，只有在内存不足时，系统则会回收软引用对象，如果回收了软引用对象之后仍然没有足够的内存，才会抛出内存溢出异常。
  例子：`SoftReference<Byte[]> m = new SoftReference<>(new Byte[1024]);`
  ​


- 弱引用 (面试重点）  
  弱引用的引用强度比软引用要更弱一些，**无论内存是否足够，只要 JVM 开始进行垃圾回收，那些被弱引用关联的对象都会被回收。**
  例子：`WeakReference<M> m = new WeakReference<>(new M());`
  ​


- 虚引用  
  虚引用是最弱的一种引用关系，如果一个对象仅持有虚引用，那么它就和没有任何引用一样，它随时可能会被回收。
  例子：`PhantomReference phantomReference = new PhantomReference<Object>(new CanisterInfo(), new ReferenceQueue<>());`
  `phantomReference.get();` 什么时候都无法get到。 
  当垃圾回收器准备回收一个对象时，如果发现它与虚引用关联，就会在回收它之前，将这个虚引用加入到引用队列中。程序可以通过判断引用队列中是否已经加入了虚引用，来了解被引用的对象是否将要被回收，如果确实要被回收，就可以做一些回收之前的收尾工作。  


**弱引用重点示例（结合ThreadLocal）**
```java
 public static void main(String[] args) {
        ThreadLocal<CanisterInfo> tl = new ThreadLocal<>();
        tl.set(new CanisterInfo());
    }


 public void set(T value) {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null)
            //此处的this 是ThreadLocal对象
            map.set(this, value);
        else
            createMap(t, value);
    }

private void set(ThreadLocal<?> key, Object value) {
            // We don't use a fast path as with get() because it is at
            // least as common to use set() to create new entries as
            // it is to replace existing ones, in which case, a fast
            // path would fail more often than not.

            Entry[] tab = table;
            int len = tab.length;
            int i = key.threadLocalHashCode & (len-1);

            for (Entry e = tab[i];
                 e != null;
                 e = tab[i = nextIndex(i, len)]) {
                ThreadLocal<?> k = e.get();

                if (k == key) {
                    e.value = value;
                    return;
                }

                if (k == null) {
                    replaceStaleEntry(key, value, i);
                    return;
                }
            }

            tab[i] = new Entry(key, value);
            int sz = ++size;
            if (!cleanSomeSlots(i, sz) && sz >= threshold)
                rehash();
        }


//Entry对象继承了WeakReference，一个弱引用对象
static class Entry extends WeakReference<ThreadLocal<?>> {
            /** The value associated with this ThreadLocal. */
            Object value;

            Entry(ThreadLocal<?> k, Object v) {
                super(k);
                value = v;
            }
        }
```
![](https://github.com/wuwenyishi/pages/raw/gh-pages/image/others/WvXOry.png)  
此图说明了，在 `ThreadLocal<CanisterInfo> tl = new ThreadLocal<>();` 并且 `tl.set(new CanisterInfo());`后有一个强引用tl指向了ThreadLocal对象，也有一个弱引用（ThreadLocalMap的key）也指向了ThreadLocal。  
为什么Entry会使用弱引用，图中也说明了。