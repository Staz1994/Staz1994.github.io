---
layout: post
title: ThreadLocal(一)
categories: ThreadLocal
tags: Java
---

ThreadLocal 用于同一线程内的对象共享，在不同线程内读到的值是不同的。

-------------------

##### 一个简单的例子。
```
/**
 * 
 * @author WynStaz
 *
 */
public class ThreadLocalTest {
     private static final ThreadLocal<String> THREAD_LOCAL = new ThreadLocal<>();

     public static void main(String[] args) throws InterruptedException {
          System.out.println(Thread.currentThread().getName() + "\t" + THREAD_LOCAL.get());

          new Thread(new Runnable() {
               @Override
               public void run() {
                    System.out.println(Thread.currentThread().getName() + "\t" + THREAD_LOCAL.get());
                    THREAD_LOCAL.set(Thread.currentThread().getName() + "Setted");
                    System.out.println(Thread.currentThread().getName() + "\t" + THREAD_LOCAL.get());
               }
          }).start();

          Thread.sleep(1000);
          System.out.println(Thread.currentThread().getName() + "\t" + THREAD_LOCAL.get());
     }
     
}
```
运行结果：

```
main null
Thread-0  null
Thread-0  Thread-0Setted
main null
```

这里main线程中，*THREAD_LOCAL.get()*获取到的值并没有改变。

##### 再写一个简单的例子。（2018年8月16日 更新）
```
public class Test {
    public static void main(String[] args) {
        ThreadLocal threadLocal = new ThreadLocal();
//        Thread mainThread = Thread.currentThread();
        ThreadLocal threadLocal2 = new ThreadLocal();
        threadLocal.set("val_1");
        threadLocal2.set("val_2");

        System.out.println(threadLocal.get());
        System.out.println(threadLocal2.get());
        System.out.println(threadLocal.get());
    }
}
```
运行结果：
```
val_1
val_2
val_1
```
这里可以看到同一个线程的不同ThreadLocal对象，获取的值是不同的。
其实也可以继承父线程的ThreadLocal,Thread类中有一个属性`inheritableThreadLocals`, 初始化ThreadLocal的时候使用这个类`InheritableThreadLocal`就可以了
```
public class TestInheritableThreadLocals {
    // 用InheritableThreadLocal 会继承父类的ThreadLocal
    public static ThreadLocal THREAD_LOCAL = new InheritableThreadLocal();
//    public static ThreadLocal THREAD_LOCAL = new ThreadLocal();

    public static void main(String[] args) {
        Map<String, String> map = Maps.newHashMap();
        map.put("key1", "val1");
        THREAD_LOCAL.set(map);
        System.out.println(JSONObject.toJSONString(THREAD_LOCAL.get()));
        MyThread thread = new MyThread();
        map.put("key2", "val2");
        thread.start();
        System.out.println(JSONObject.toJSONString(THREAD_LOCAL.get()));
    }
    static class MyThread extends Thread{
        @Override
        public void run() {
            System.out.println(Thread.currentThread().getName() + ":" + JSONObject.toJSONString(THREAD_LOCAL.get()));
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName() + ":" + JSONObject.toJSONString(THREAD_LOCAL.get()));
        }
    }
}
```

看了下代码，具体就是Thread类有一个`ThreadLocal.ThreadLocalMap`属性，调用ThreadLocal.get()方法时，会返回当前线程的这个map，这个map的key值就是ThreadLocal这个对象。

