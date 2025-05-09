---
title: problem165 Stamps
date: 2025-05-08 22:02:58
mathjax: true
categories:
- [数据结构与算法,uva]
tags:
- 算法
- uva
description: "UVA problem165.Stamps <br>
- 寻找最长连续的面值组合"
---

## 题目

The government of Nova Mareterrania requires that various legal documents have stamps attached to them so that the government can derive revenue from them. In terms of recent legislation, each class of document is limited in the number of stamps that may be attached to it. The government wishes to know how many different stamps, and of what values, they need to print to allow the widest choice of values to be made up under these conditions. Stamps are always valued in units of $1.

This has been analysed by government mathematicians who have derived a formula for n(h; k), where h is the number of stamps that may be attached to a document, k is the number of denominations of stamps available, and n is the largest attainable value in a continuous sequence starting from $1. For instance, if h = 3, k = 2 and the denominations are $1 and $4, we can make all the values from $1 to $6 (as well as $8, $9 and $12). However with the same values of h and k, but using $1 and $3 stamps we can make all the values from $1 to $7 (as well as $9). This is maximal, so n(3; 2) = 7.

Unfortunately the formula relating n(h; k) to h, k and the values of the stamps has been lost — it was published in one of the government reports but no-one can remember which one, and of the three researchers who started to search for the formula, two died of boredom and the third took a job as a lighthouse keeper because it provided more social stimulation.

The task has now been passed on to you. You doubt the existence of a formula in the first place so you decide to write a program that, for given values of h and k, will determine an optimum set of stamps and the value of n(h; k).

### Input

Input will consist of several lines, each containing a value for h and k. The file will be terminated by two zeroes (0 0). For technical reasons the sum of h and k is limited to 9. (The President lost his little finger in a shooting accident and cannot count past 9).

### Output

Output will consist of a line for each value of h and k consisting of the k stamp values in ascending order right justified in fields 3 characters wide, followed by a space and an arrow (->) and the value of n(h; k) right justified in a field 3 characters wide.

### SampleInput

3 2
0 0

### SampleOutput

1 3 -> 7

### 题意

要给文件盖戳增加税收，每个戳都有不同的面值。给定两个整数h和k，h表示可以在文件上盖的戳的数量，k表示戳的面值的类型数量，例如h=3,k=2,表示共有两种面值的戳，每一个文件至多可以戳三个戳(即戳1，2，3个都可以)，这时几个戳面值的和构成了文件的戳值。问现在在给定h和k的情况下，文件戳值能连续到达的最大值是多少。这里假设连续最大戳值为max，则意思是文件上的戳值能从1，2，3...一直组合到max。并求出达到最大连续戳值的面值组合。

## 思路

假设有k种面值，不妨设为$a_1,a_2,a_3...a_k$，并且大小关系依次递增。

1. 由于文件戳值是连续的，所以$a_1$一定等于1，因为大于1则无法组成1.
2. 假设前$a_i$个面值已确定，并且前$a_i$种面值可以组合的最大连续面值是$S_i$,则可以确定$a_(i+1)$的范围：
    $$ a_i+1 \leq a_(i+1) \leq S_i+1$$
前面的不等式很显然，因为依次递增的关系，后面的则是如果$a_(i+1)$大于$S_i+1$，则$S_i+1$无法由这些面值构成，连续性就会中断。

所以在确定每个$a_i$的范围后，我们需要遍历所有可能的组合，并计算出最大连续面值。这里将整个遍历分为了两部分：面值的遍历和面值组合的遍历。

以h=3为例，

* 当k=1，$a_1=1,S_1=3$;
* 当k=2，$a_2 \in [2,4],对应的S_2为{4，4，2} $;
* 当k=3，每一个$a_2$的可能取值都会对应一个范围内的a_3取值;

所以当k越来越大，所有的取值可能性构成了一颗多叉树：
{% asset_img 165_pic1.jpg 取值情况%}

每一条从根到叶的路径构成了一种面值类型得组合，节点表示对应层级(也就是对应顺序，第一层对应第一个面值，第二层对应第二个面值，依此类推)面值的取值，以h=3，k=3为例，{1，2，3}、{1，2，4}，{1，2，5}等都是可能能得到最大连续文件戳的组合。这就是下面search函数的功能，递归获取所有面值组合。并且得到组合时将面值存入stamps数组中。

dfs函数的功能则是在给定面值类型的情况下，遍历这些面值所能构成的所有面值和，并找到最大的连续和。和serach类似，寻找所有的面值类型和也构成了一颗树。dfs在递归过程中，会将得到的面值记录到used数组中，然后通过遍历used找到最大的连续值。

### 问题

#### 实现上的问题

#### 性能上的问题

## 实现

```JAVA .{line-numbers}
//source code
import java.util.Arrays;
import java.util.Scanner;

public class Main {

    //下标从1开始，记录第i张stamp的面值
    public static int[] stamps = new int[10];

    //在i张stamp已经确定的情况下，maxVals[i]表示这i张stamp能达到的最大连续值
    public static int[] maxVals = new int[10];

    public static int[] stampRecords = new int[10];

    //用来记录组合后的数值
    public static boolean[] used = new boolean[200];

    public static int totalMaxVal = 0;

    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);

        stamps[0] = 1;
        while (true) {
            int h = scanner.nextInt();
            int k = scanner.nextInt();
            if (h == 0 && k == 0) {
                break;
            }

            totalMaxVal = 0;
            stamps[1] = 1;
            maxVals[1] = h;
            search(2, k, h);

            StringBuilder sb = new StringBuilder();
            for (int i = 1; i <= k; i++) {
                sb.append(formatNumber(stampRecords[i]));
            }
            sb.append(" ->");
            sb.append(formatNumber(totalMaxVal));
            System.out.println(sb.toString());
        }
    }

    /**
     * search用来遍历出所有的面值组合，
     */
    private static void search(int cur, int k, int h) {
        if (cur > k) {
            if (maxVals[cur - 1] > totalMaxVal) {
                totalMaxVal = maxVals[cur - 1];
                System.arraycopy(stamps, 0, stampRecords, 0, 10);
            }
            return;
        }

        //这里的遍历其实就是对应层的节点遍历，以k=2为例，遍历的第二层的节点。
        for (int i = stamps[cur - 1] + 1; i <= maxVals[cur - 1] + 1; i++) {
            stamps[cur] = i;
            Arrays.fill(used, false);
            dfs(0, cur, h, 0);
            int start = 1, num = 0;
            while (used[start++]) {
                num++;
            }
            maxVals[cur] = num;
            search(cur + 1, k, h);
        }
    }

    /**
     * 面值组合递归遍历
     *  cur当前用了几张票， hi表示当前面值的种类， sum表示面额之和
     */
    private static void dfs(int cur, int hi, int h, int sum) {
        if (cur >= h) {
            //get the limit of stamps number
            used[sum] = true;
            return;
        }
        used[sum] = true;
        for (int i = 1; i <= hi; i++) {
            dfs(cur + 1, hi, h, sum + stamps[i]);
        }
    }

    public static String formatNumber(int number) {
        String val = String.valueOf(number);
        int space = 3 - val.length();
        StringBuilder sb = new StringBuilder();
        for (int i = 0; i < space; i++) {
            sb.append(" ");
        }
        sb.append(val);
        return sb.toString();
    }
}
```
