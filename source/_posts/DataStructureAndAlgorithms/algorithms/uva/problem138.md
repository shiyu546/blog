---
title: problem138 Street Numbers
date: 2024-10-12 17:33:39
mathjax: true
categories:
- [数据结构与算法,uva]
tags:
- 算法
- uva
description: "UVA 138.Street Numbers <br>
- 计算数列前n个数"
---

## 题目

A computer programmer lives in a street with houses numbered consecutively (from 1) down one side of the street. Every evening she walks her dog by leaving her house and randomly turning left or right and walking to the end of the street and back. One night she adds up the street numbers of the houses she passes (excluding her own). The next time she walks the other way she repeats this and finds, to her astonishment, that the two sums are the same. Although this is determined in part by her house number and in part by the number of houses in the street, she nevertheless feels that this is a desirable property for her house to have and decides that all her subsequent houses should exhibit it.

Write a program to find pairs of numbers that satisfy this condition. To start your list the first two pairs are: (house number, last number):
6 8
35 49

### Input

There is no input for this program.

### Output

Output will consist of 10 lines each containing a pair of numbers, each printed right justified in a field of width 10 (as shown above).

### 题意

题意大致如下：一个人居住的房子所在的街，房子是按照编号1开始逐步递增，直至最后一个房子n，主人居住在的房子编号为m。从m往左出发直至编号为1的房子，经过的所有房子编号之和(不包含自己编号为m的房子)为sum1,从m往右出发直至编号为n的房子，经过的房子(不包含自己编号为m的房子)编号之和为sum2，现在sum1等于sum2。现请找出10组满足这样条件的房子(用m和n可以唯一标记一组这样的房子)，并输出(按m递增找到数列最前面的10条)。

## 思路

假设主人所在的房子编号为x，一条街共有y个房子，分别算出sum1和sum2.sum1从房子编号1加到x-1，sum2从编号x+1加到y。

$$ sum1=1+2+3+...+x-1$$
等差数列相加，得到
$$ sum1=(x-1+1)(x-1)/2=x(x-1)/2$$
同理,
$$ sum2=x+1+x+2+...+y$$
=>
$$ sum2=(x+1+y)(y-x)/2$$
因为sum1等于sum2，得
$$ x(x-1)/2=(x+1+y)(y-x)/2$$
=>
$$ y^2+y-2x^2=0$$
=>
$$ y=\frac{-1 \pm \sqrt{1+8x^2} }{2}$$
由于x,y都是正整数，所以去掉负数根，得
$$ y=\frac{ \sqrt{1+8x^2}-1 }{2}$$

剩下的工作就是遍历x，找到满足x,y为正整数，且等式成立得x,y对，判定包括判断数能否完全开方，是否能被2整除。

### 问题

#### 实现上的问题

#### 性能上的问题

## 实现

```JAVA .{line-numbers}
//source file
public class Main {
    public static void main(String[] args) {
        int num = 10;
        long x = 6;
        StringBuilder sb = new StringBuilder();
        while (num > 0) {
            long xtmp = 1 + 8 * x * x;
            long sqrt = isPerfectSquare(xtmp);
            if (sqrt != -1) {
                if ((sqrt - 1) % 2 == 0) {
                    long y = (sqrt - 1) / 2;
                    fillNumber(x, sb);
                    fillNumber(y, sb);
                    sb.append("\n");
                    num--;
                }
            }
            x++;
        }
        System.out.printf(sb.toString());
    }

    private static void fillNumber(long x, StringBuilder sb) {
        String xstr = String.valueOf(x);
        int blankX = 10 - xstr.length();
        for (int i = 0; i < blankX; i++) {
            sb.append(" ");
        }
        sb.append(xstr);
    }

    //判断一个数能否开方
    public static long isPerfectSquare(long n) {
        if (n < 0)
            return -1;

        long tst = (long) (Math.sqrt(n) + 0.5);
        if (tst * tst == n) {
            return tst;
        }
        return -1;
    }
}
```
