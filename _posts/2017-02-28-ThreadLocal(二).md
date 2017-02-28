---
layout: post
title: ThreadLocal(二)
categories: ThreadLocal
tags: Java
---

##### 解析部分。
1. ThreadLocal类内定义了一个静态内部类 *ThreadLocalMap*(以下简称map) 这个map 实际上是一个 *Entry* 的数组。*Entry* 是一个 WeakReference 的键值对。key则是ThreadLocal这个类的实例化对象。
2. map的引用是在Thread对象中的。

```
/* ThreadLocal values pertaining to this thread. This map is maintained
 * by the ThreadLocal class. */
ThreadLocal.ThreadLocalMap threadLocals = null;

/*
 * InheritableThreadLocal values pertaining to this thread. This map is
 * maintained by the InheritableThreadLocal class.
 */
ThreadLocal.ThreadLocalMap inheritableThreadLocals = null;
```
3. 当调用 ThreadLocal.get()方法时，会获取当前线程，从当前线程中获取map，再通过ThreadLocal作为key来获取存储的值。
```
public T get() {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null) {
        ThreadLocalMap.Entry e = map.getEntry(this);
        if (e != null) {
            @SuppressWarnings("unchecked")
            T result = (T)e.value;
            return result;
        }
    }
    return setInitialValue();
}

 ThreadLocalMap getMap(Thread t) {
    return t.threadLocals;
}
```
4. map的大小
```
/**
* The initial capacity -- MUST be a power of two.
*/
private static final int INITIAL_CAPACITY = 16;
```
初始容量是16，而且大小必须是2的N次方。当数组里面的元素个数 >= threshold (当前容量的 * 2 / 3 )时,数组扩容
```
/**
* Double the capacity of the table.
*/
private void resize() {
   Entry[] oldTab = table;
   int oldLen = oldTab.length;
   int newLen = oldLen * 2;
   Entry[] newTab = new Entry[newLen];
   int count = 0;

   for (int j = 0; j < oldLen; ++j) {
       Entry e = oldTab[j];
       if (e != null) {
           ThreadLocal<?> k = e.get();
           if (k == null) {
               e.value = null; // Help the GC
           } else {
               int h = k.threadLocalHashCode & (newLen - 1);
               while (newTab[h] != null)
                   h = nextIndex(h, newLen);
               newTab[h] = e;
               count++;
           }
       }
   }

   setThreshold(newLen);
   size = count;
   table = newTab;
}
```
也是将大小翻倍了。也就是扩容后的容量是2的N+1次。
5. 如何定位 数据存储在map中数组的位置
5.1. ThreadLocal中有一个属性
```
private final int threadLocalHashCode = nextHashCode();

private static AtomicInteger nextHashCode = new AtomicInteger();

private static final int HASH_INCREMENT = 0x61c88647;

private static int nextHashCode() {
    return nextHashCode.getAndAdd(HASH_INCREMENT);
}
```
这个就是它作为key的hashCode，每新建一个ThreadLocal,*threadLocalHashCode*会比上次的大 *0x61c88647* 。在set的时候，*threadLocalHashCode* 会与 map 的len 进行运算
```
int i = key.threadLocalHashCode & (len-1);
```
运算得到的i就是value在数组里面存放的位置entry[i]。
5.2. 有个疑问：为什么要以 0x61c88647 的倍数 & (len-1) 作为key，难道这个不会有冲突吗？
```
/**
 * 
 * @author WynStaz
 *
 */
public class HashCode {
     public static final int HASH_INCREMENT = 0x61c88647;
     
     public static void main(String[] args) {
          int[] hashs = new int[64];
          hashs[0] = HashCode.HASH_INCREMENT;
          for (int i = 1; i < hashs.length; i++) {
               hashs[i] = hashs[i-1] + HashCode.HASH_INCREMENT;
          }
          for (int i = 0; i < hashs.length; i++) {
               System.out.print(hashs[i] & 63);
               System.out.print("\t");
          }
          
     }
}
```
结果：
```
7    14   21   28   35   42   49   56   63   6    13   20   27   34   41   48   55   62   5    12   19   26   33   40   47   54   61   4    11   18   25   32   39   46   53   60   3    10   17   24   31   38   45   52   59   2    9    16   23   30   37   44   51   58   1    8    15   22   29   36   43   50   57   0    
```
