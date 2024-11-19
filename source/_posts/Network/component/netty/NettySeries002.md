---
title: Netty系列02 线程工厂ThreadFactory
date: 2024-11-05 21:43:22
mathjax: true
categories:
- [网络编程,netty]
tags:
- 原理
- 实现
description: "netty系列基础部分：DefaultThreadFactory <br>
- 线程工厂类以及ThreadLocal"
---

## 背景

网络编程模型包括了阻塞模型，每个请求每线程模型，IO多路复用模型等，除了阻塞模型，其余模型都需要线程的支持。JAVA中创建线程可以通过创建Thread对象并调用对象的start方法实现，但该方法灵活度不高。JAVA中同时提供了ThreadFactory接口，通过实现该接口创建自定义线程。

## 功能

### ThreadFactory

```JAVA .{line-numbers}
//source code
public interface ThreadFactory {
    Thread newThread(Runnable r);
}

public class DefaultThreadFactory implements ThreadFactory {
    protected Thread newThread(Runnable r, String name) {
        return new FastThreadLocalThread(threadGroup, r, name);
    }
}
```

ThreadFactory只有一个方法用来创建线程。Netty中DefaultThreadFactory实现了ThreadFactory接口，其newThread方法返回了一个FastThreadLocalThread对象。FastThreadLocalThread实现了Thread对象，在ThreadLocal上做了一定的优化。

### ThreadLocal

ThreadLocal称为线程本地存储，在了解线程本地存储前，需要先了解线程的相关机制。

>线程本质上是一系列代码的执行流，每个线程都有自己的线程栈，栈上的变量是线程私有的，其他线程不可访问。堆中的变量则是多个线程共享的，当线程访问堆中的变量时，就可能产生线程争用的情况，即多个线程同时操作一个变量，导致结果不可预知，就会产生线程安全性的问题。解决线程安全性可以采用加锁的方式，使用锁使得变量一次只能由一个线程访问，这样就实现了线程安全访问。

ThreadLocal则采用了另外一种方案，通过为每一个线程保存一份值的副本，来实现线程之间的安全访问，这些线程的值与一个ThreadLocal对象关联。每一个线程对象都有一个ThreadLocalMap对象，该类的机制类似hashmap，其Key是ThreadLocal对象，Value则是线程需要保存的值。每个ThreadLocal对象在每个线程中关联一个值，每当调用ThreadLocal的get方法时，会获取当前线程的ThreadLocalMap对象，从中取出Key为对应的ThreadLocal所对应的Value。

{% asset_img nettyseries002_001.png 图中虚线框表示ThreadLocalMap对象，每一个Map由多个key-value键值对组成，多个线程包含同一个ThreadLocal作为key，而该key所对应的value在各个线程中不相同，这样就实现了同一个ThreadLocal对象在不同线程中有不同的值。采用ThreadLocal取值时，取出的是当前执行线程的值 %}

ThreadLocalMap采用hashmap的方式实现，底层使用Entry数组存储数据，链地址法解决冲突。

```JAVA .{line-numbers}
//code segment
private Entry getEntry(ThreadLocal<?> key) {
//计算key的hash值，寻找Entry数组中对应的bucket
    int i = key.threadLocalHashCode & (table.length - 1);
    Entry e = table[i];
    if (e != null && e.get() == key)
        return e;
    else
        return getEntryAfterMiss(key, i, e);
}

private Entry getEntryAfterMiss(ThreadLocal<?> key, int i, Entry e) {
    Entry[] tab = table;
    int len = tab.length;

    while (e != null) {
        ThreadLocal<?> k = e.get();
        if (k == key)
            return e;
        if (k == null)
            expungeStaleEntry(i);
        else
            i = nextIndex(i, len);
        e = tab[i];
    }
    return null;
}
```

而在FastThreadLocalThread中则采用了自定义的ThreadLocalMap，名称为InternalThreadLocalMap，相比hashmap的实现，InternalThreadLocalMap使用下标索引的方式，即采用数组存储，并用下标索引的方式获取数据。

FastThreadLocalThread对应的ThreadLocal实现为FastThreadLocal，其get方法代码如下,每一个FastThreadLocal有一个index值，该值用来索引InternalThreadLocalMap中数组下标。

```JAVA .{line-numbers}
//code segment
private Object[] indexedVariables;
public final V get(InternalThreadLocalMap threadLocalMap) {
    Object v = threadLocalMap.indexedVariable(index);
    if (v != InternalThreadLocalMap.UNSET) {
        return (V) v;
    }
    return initialize(threadLocalMap);
}

public Object indexedVariable(int index) {
    Object[] lookup = indexedVariables;
    return index < lookup.length? lookup[index] : UNSET;
}
```

通过使用数组下标索引替代hash table，实现了一定程度上的性能提升。
