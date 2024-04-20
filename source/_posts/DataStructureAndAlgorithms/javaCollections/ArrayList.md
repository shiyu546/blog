---
title: Arraylist实现解析
date: 2024-04-20 21:19:42
categories:
- [数据结构与算法,javaCollection]
tags:
- java collection
---

# Arraylist实现解析

## 什么是List

List是一种比较常见的数据结构，表示有序的数据集合(sequence)。java里List接口实现了Collections接口：

```JAVA
public interface List<E> extends Collection<E> {
```

在Collection提供的iterator, add, remove, equals, and hashCode等方法上，List提供了get,set等存取元素的操作。

## ArrayList及其实现

### 继承体系

{% asset_img inherit.png inherit %}

ArrayList的类继承图如上，其实现了List接口，继承了AbstractList<E>抽象类以及标记接口:RandomAccess,Cloneable,java.io.Serializable，分别表示随机存取特性，clone特性以及可序列化。

```JAVA
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
```

### 基本元素

ArrayList有三个比较关键的概念：存储元素的数组elementData，元素个数size以及数组容量capbility.

### 构造函数

ArrayList有三个构造函数:
1.当入参为空时，构造一个空数组；
2.当入参代表数组初始容量时，数组初始化为指定容量大小；
3.当传入Collections时，数组初始化为包含Collections中所有元素的数组。

```JAVA
    public ArrayList();
    public ArrayList(int initialCapacity);
    public ArrayList(Collection<? extends E> c);
```

### 操作

1. 插入元素

插入操作有两个函数，不指定index则是插在数组末尾，指定index则插在数组下标index处。

```JAVA
    public boolean add(E e);
    public void add(int index, E element);
```

``` ascii
add(int, E)
 |
 o---> rangeCheckForAdd()
 |
 o---> ensureCapacityInternal()
 |             |
 |             o---> calculateCapacity()
 |             |
 |             o---> ensureExplicitCapacity()
 |                             |
 |                             o---> grow()
 |
 o---> System.arraycopy()
```

add(int index, E element)的函数调用关系如上图:

- 3行rangeCheckForAdd()检查index是否在[0,size]范围内，即插入的元素与数组元素在一起或相邻，不能与现有数组元素中存在空隙，否则抛出异常。
  
- 5行ensureCapacityInternal()









当ArrayList插入元素时，需要判断数组当前容量是否满足能插入元素，不行则需要扩容。


