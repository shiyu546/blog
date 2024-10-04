---
title: problem136 Ugly Numbers
date: 2024-10-04 12:49:24
mathjax: true
categories:
- [数据结构与算法,uva]
tags:
- 算法
- uva
description: "UVA 136.Ugly Numbers <br>
- 按规律找出第n个数字"
---

## 题目

Ugly numbers are numbers whose only prime factors are 2, 3 or 5. The sequence
1, 2, 3, 4, 5, 6, 8, 9, 10, 12, 15, ...
shows the first 11 ugly numbers. By convention, 1 is included.

Write a program to find and print the 1500'th ugly number.

### Input

There is no input to this program.

### Output

Output should consist of a single line as shown below, with '< number >' replaced by the number computed.

### SampleOutput

The 1500'th ugly number is < number >.

### 题意

一个数如果做质因数分解之后所有的因子都由2,3,5构成，这样的数就称为ugly number.ugly number按从小到大排序，请问第1500个ugly number是多少(默认1是第一个ugly number)。

## 思路

问题关键在于找到一种方法能够一个不漏的数过去，数到第1500个。考虑ugly number的定义，如果一个数N是ugly number,那么一定可以写成:
$$ N=[2*...*2]_{0到j个2}*[3*...*3]_{0到k个3}*[5*...*5]_{0到m个5} $$
的形式，所以任何N都是2，3，5的倍数,并且N除以2，3，5之后得到的数K仍然是ugly number.所以所有的ugly number都可以从最基础的2，3，5通过倍数乘得到。找出2，3，5的所有倍数，然后按从小到大排序，取到第1500个即可。

不考虑1，假设基础序列是{2,3,5}，则2是第一个ugly number,计算2的倍数，2分别乘以2，3，5得到{4,6,8},放入基础序列得到{2,3,4,5,6,8},2的最小倍数已计算完毕，此时2的后面一个就是第二个ugly number，所以3是第二个ugly number。去掉2，得到基础序列{3,4,5,6,8}.3分别乘以2,3,5得到{6,9,15},放入序列得到{3,4,5,6,8,9,15}。则3后一个ugly number是4.依次类推，找到第1500个ugly number。

### 问题

#### 实现上的问题

#### 性能上的问题

## 实现

```JAVA .{line-numbers}
//source file
import java.util.LinkedList;

public class Main {
    public static void main(String[] args) {
        long start = 1;
        LinkedList<Long> list = new LinkedList<>();
        list.add(start);
        int index = 1;
        while (index < 1500) {
            addMulti(list, start * 2);
            addMulti(list, start * 3);
            addMulti(list, start * 5);
            list.removeFirst();
            index++;
            start = list.getFirst();
        }
        System.out.printf("The 1500'th ugly number is %d.\n", list.getFirst());
    }

    public static void addMulti(LinkedList<Long> list, long num) {
        for (int i = 1; i < list.size(); i++) {
            if (num == list.get(i)) {
                return;
            } else if (num < list.get(i)) {
                list.add(i, num);
                return;
            }
        }
        list.addLast(num);
    }
}
```
