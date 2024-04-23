---
title: Linkedlist实现解析
date: 2024-04-22 21:19:42
categories:
- [数据结构与算法,javaCollection]
tags:
- java collection
description: "本文描述了LinkedList类在jdk1.8中的实现。包括继承体系、基本元素、构造函数以及常见操作的说明等。<br>
- [什么是List]<br>
- [ArrayList及其实现]<br>
- ..."
---


## 类的原型

LinkedList类的原型如下：

```JAVA
public class LinkedList<E>
    extends AbstractSequentialList<E>
    implements List<E>, Deque<E>, Cloneable, java.io.Serializable
```

LinkedList不仅实现了List接口，也实现了Deque接口，Deque是一种双端队列，两端都可以做增删操作。

### Deque

> A linear collection that supports element insertion and removal at both ends.

从描述中Deque是一种支持在两端进行增删的队列，Deque提供了两种形式的增删操作：一种在操作失败时抛出异常，另一种则返回特定值表示操作失败。

<table>
<caption>Summary of Deque methods</caption>
 <tr>
   <td></td>
   <td ALIGN=CENTER COLSPAN = 2> <b>First Element (Head)</b></td>
   <td ALIGN=CENTER COLSPAN = 2> <b>Last Element (Tail)</b></td>
 </tr>
 <tr>
   <td></td>
   <td ALIGN=CENTER><em>Throws exception</em></td>
   <td ALIGN=CENTER><em>Special value</em></td>
   <td ALIGN=CENTER><em>Throws exception</em></td>
   <td ALIGN=CENTER><em>Special value</em></td>
 </tr>
 <tr>
   <td><b>Insert</b></td>
   <td>addFirst(e)</td>
   <td>offerFirst(e)</td>
   <td>addLast(e)</td>
   <td>offerLast(e)</td>
 </tr>
 <tr>
   <td><b>Remove</b></td>
   <td>removeFirst()</td>
   <td>pollFirst()</td>
   <td>removeLast()</td>
   <td> pollLast()</td>
 </tr>
 <tr>
   <td><b>Examine</b></td>
   <td>getFirst()</td>
   <td>peekFirst()</td>
   <td>getLast()</td>
   <td>peekLast()</td>
 </tr>
</table>

## 基本元素

从LinkedList的原型中可以看出，LinkedList实现了List和Deque接口，根据面向对象is a的特性，LinkedList既是List，也是Deque。在LinkedList中，节点通过链接形式形成List，同时Deque使得链表的首尾两端都可以插入删除元素。

LinkedList主要包含了两个分别指向链表首尾的节点first和last。

```plaintext
  +------------+        +------------+         +------------+         +------------+
  |            |------->|            |-------->|            |-------->|            |
  |   Node     |<-------|    Node    |<--------|    ...     |<--------|    Node    |
  +------------+        +------------+         +------------+         +------------+
       /|\                                                                  /|\
        |                                                                    |                  
       first                                                                last              
     
```

### Node对象

LinkedList中所有的元素都是存储在Node对象中，Node含有需要存储的数据以及指向前驱和后继节点的指针。

```JAVA

private static class Node<E> {
    E item;
    Node<E> next;
    Node<E> prev;

    Node(Node<E> prev, E element, Node<E> next) {
        this.item = element;
        this.next = next;
        this.prev = prev;
    }
}
```

## 构造函数

根据Collection接口的规范，任何Collection接口的实现类都需要实现带Collection对象的构造，LinkedList也不例外，下面看LinkedList带Collection参数的构造函数,构造函数最终调用了addAll(int,Collection)的方法。

```JAVA

public LinkedList(Collection<? extends E> c) {
    this();
    addAll(c);
}

public boolean addAll(Collection<? extends E> c) {
    return addAll(size, c);
}

public boolean addAll(int index, Collection<? extends E> c) {
    checkPositionIndex(index);

    Object[] a = c.toArray();
    int numNew = a.length;
    if (numNew == 0)
        return false;

    Node<E> pred, succ;
    if (index == size) {
        succ = null;
        pred = last;
    } else {
        succ = node(index);
        pred = succ.prev;
    }

    for (Object o : a) {
        @SuppressWarnings("unchecked") E e = (E) o;
        Node<E> newNode = new Node<>(pred, e, null);
        if (pred == null)
            first = newNode;
        else
            pred.next = newNode;
        pred = newNode;
    }

    if (succ == null) {
        last = pred;
    } else {
        pred.next = succ;
        succ.prev = pred;
    }

    size += numNew;
    modCount++;
    return true;
}
```

- line20: 初始时List中不含有元素，所以size=0,这时index==size，pred指向的是链表末尾；

- line28: 遍历Collection集合，由于pred指向的是当前尾节点，新增节点时节点前驱指向的是pred节点，所以这里是典型的尾插法。

```
  +------------+
  |            |
  |    Node    |
  +------------+
    /|\   /|\
     |     |              
    last   +----pred
    (1)pred指向last节点

  +------------+                  +------------+
  |            |<-----------------|            |
  |    Node    |                  |   newNode  |
  +------------+                  +------------+
    /|\   /|\
     |     |              
    last   +----pred
    (2)构造新插入的节点,节点prev指向pred节点

  +------------+                  +------------+
  |            |<-----------------|            |
  |    Node    |----------------->|   newNode  |
  +------------+                  +------------+
    /|\   /|\
     |     |              
    last   +----pred
    (3)pred的next指newNode

  +------------+                  +------------+
  |            |<-----------------|            |
  |    Node    |----------------->|   newNode  |
  +------------+                  +------------+
                                        /|\
                                         |              
                                         +----pred
    (3)pred指向newNode 
```

如此遍历直至所有节点都从链表尾部插入,最后将last指向pred节点.


