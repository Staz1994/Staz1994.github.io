---
layout: post
title: ThreadLocal(一)
categories: Java
tags: ThreadLocal
---

ThreadLocal 用于同一线程内的对象共享，在不同线程内读到的值是不同的。

##### 举个例子
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