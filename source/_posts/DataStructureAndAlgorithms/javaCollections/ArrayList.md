---
title: Arraylist实现解析
date: 2024-04-20 21:19:42
categories:
- [数据结构与算法,javaCollection]
tags:
- java collection
description: "本文描述了ArrayList类在jdk1.8中的实现。包括继承体系、基本元素、构造函数以及常见操作的说明等。<br>
- [什么是List]<br>
- [ArrayList及其实现]<br>
- ..."
---


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

```plaintext
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

- line3: rangeCheckForAdd()检查index是否在[0,size]范围内，即插入的元素与数组元素在一起或相邻，不能与现有数组元素中存在空隙，否则抛出异常；
  
- line5: ensureCapacityInternal()确保添加元素后数组容量能容纳该元素。对添加单个元素来说，如果size==capbility,表示数组已经没有多余的空间能添加元素，这时就需要扩容；
  
  - line7: calculateCapacity()计算数组需要的最小容量minCapacity。
  - line9: ensureExplicitCapacity()主要实现两个功能：
    1. 递增modCount，该值用来计数该ArrayList修改的次数；
    2. 判断数组容量是否小于minCapacity,小于则调用grow()增加数组长度。
    - line11 grow()，数组的实际扩容发生在这里，通过oldCapacity + (oldCapacity >> 1)将数组容量扩充为原来的1.5倍，然后创建newCapacity容量的数组，将旧数组元素拷贝到新数组中去。
- line13: 调用native方法System.arraycopy()将index之后的元素整体后移一位，空出index处的位置。

然后在index处插入元素。函数代码如下：

```JAVA
    public void add(int index, E element) {
        rangeCheckForAdd(index);

        ensureCapacityInternal(size + 1);  // Increments modCount!!
        System.arraycopy(elementData, index, elementData, index + 1,
                            size - index);
        elementData[index] = element;
        size++;
    }

    private void rangeCheckForAdd(int index) {
        if (index > size || index < 0)
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
    }

        private void ensureCapacityInternal(int minCapacity) {
        ensureExplicitCapacity(calculateCapacity(elementData, minCapacity));
    }

    private void ensureExplicitCapacity(int minCapacity) {
        modCount++;

        // overflow-conscious code
        if (minCapacity - elementData.length > 0)
            grow(minCapacity);
    }

    private void grow(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = elementData.length;
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // minCapacity is usually close to size, so this is a win:
        elementData = Arrays.copyOf(elementData, newCapacity);
    }
```

从add操作我们可以看出，在数组元素中间插入元素涉及到对数组后部元素的移动，所以在指定位置插入元素是一个比较耗时的操作，越靠近数组起始部分，需要移动的元素的越多，可能耗时越高。

2. 删除元素remove(int index)

```JAVA
    public E remove(int index) {
        rangeCheck(index);

        modCount++;
        E oldValue = elementData(index);

        int numMoved = size - index - 1;
        if (numMoved > 0)
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        elementData[--size] = null; // clear to let GC do its work

        return oldValue;
    }
```

- line2: 检查删除元素下标是否在指定范围内；
- line9: 删除指定index的元素之后，其后的元素需要往前移动一格填补删除产生的空隙。

### 分析

ArrayList作为动态数组，本质上采用固定长度的数组来存储元素，通过记录数组元素大小size和数组容量capbility来确保增加元素时不会导致溢出。当插入元素时，判断capbility是否能容纳该元素，不能则创建新数组并将原数组元素拷贝过去。

从ArrayList的实现上可以看出，ArrayList的优势主要体现在随机读取上，读取指定位置的元素是O(1)复杂度。同时在数组尾端增加删除元素也是比较快速的操作，而在数组中部增加删除元素则涉及到数组元素移动，是一个比较耗时的操作。
