---
title: problem153 Permalex
date: 2025-02-10 20:22:19
mathjax: true
categories:
- [数据结构与算法,uva]
tags:
- 算法
- uva
description: "UVA problem153.Permalex <br>
- 计算字符串在字母全排列中的字典序序号"
---

## 题目

Given a string of characters, we can permute the individual characters to make new strings. If we can impose an ordering on the characters (say alphabetic sequence), then the strings themselves can be ordered and any given permutation can be given a unique number designating its position in that ordering. For example the string ‘acab’ gives rise to the following 12 distinct permutations:
aabc 1 acab 5 bcaa 9
aacb 2 acba 6 caab 10
abac 3 baac 7 caba 11
abca 4 baca 8 cbaa 12

Thus the string ‘acab’ can be characterised in this sequence as 5.

Write a program that will read in a string and determine its position in the ordered sequence of permutations of its constituent characters. Note that numbers of permutations can get very large;however we guarantee that no string will be given whose position is more than $2^{31}-1=2147483647$

### Input

Input will consist of a series of lines, each line containing one string. Each string will consist of up to 30 lower case letters, not necessarily distinct. The file will be terminated by a line consisting of a single ‘#’.

### Output

Output will consist of a series of lines, one for each line of the input. Each line will consist of the position of the string in its sequence, right justified in a field of width 10.

### SampleInput

bacaa
abc
cba
\#

### SampleOutput

        15
         1
         6

### 题意

给定一个字符串s，组成s的字母通过排列的方式可以构成M个不同的字符串，将这M个字符串按字典序排序，请求出s是第几个字符串。

## 思路

如果计算出所有的字符串然后排序的话，数据量会很大，不现实。所以从字典序出发，分别计算首字母比s小的字符串，第二个字母比s小的字符串...依此类推。以字符串"dcaba"为例，首字母比d小的字符串一定在d的前面，当首字母相同时，比较第二个字母,此时字母为c，则比c小的在s之前。依此类推，直到最后一个字符。

具体计算过程如下：
首字母为d，比d小的有a,b,c.以a开头时，共有4!=24种排列;以b开头时，共有4!/2!=12种排列，c开头也有4!/2!=12种排列。比d小的字符串数量就计算完毕。
现在考虑首字母相同，则比较第二个字母c，比c小的有a,b.以a开头(实际是da)有3!=6,b开头有3!/2!=3种可能。
依此类推。

排列公式如下，字符串为s，长度为t，其中共有r种字母，每个字母的数量为$n_1$,$n_2$,$n_3$...,$n_r$,则组成s的字母共有
$$ \frac{t!}{n_1!\times n_2! \times n_3! \times ... \times n_r!}$$
种排列。

### 问题

#### 实现上的问题

计算阶乘时数据量增长的很快，很容易就溢出，所以在应用排列公式时，不能先直接计算分子分母然后相除。而是在计算过程中就约分，尽量减小中间结果的大小。

#### 性能上的问题

## 实现

```JAVA .{line-numbers}
//source code
import java.util.Arrays;
import java.util.Scanner;

public class Main {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        while (true) {
            String str = scanner.nextLine();
            if ("#".equals(str)) {
                break;
            }
            if (str == null || str.length() <= 0) {
                System.out.println("         1");
                continue;
            }
            long totalNum = 0;
            for (int i = 0; i < str.length(); i++) {
                String subStr = str.substring(i);
                totalNum += calulateCnt(subStr);
            }

            StringBuilder sb = new StringBuilder();
            String val = String.valueOf(totalNum + 1);
            for (int j = 0; j < 10 - val.length(); j++) {
                sb.append(" ");
            }
            sb.append(val);
            System.out.println(sb.toString());
        }
    }

    private static long calulateCnt(String str) {
        int[] alpbets = new int[26];
        Arrays.fill(alpbets, 0);

        for (int i = 0; i < str.length(); i++) {
            int charAt = str.charAt(i) - 'a';
            alpbets[charAt]++;
        }
        int current = str.charAt(0) - 'a';
        long totalNum = 0, length = str.length();
        for (int i = 0; i < current; i++) {
            if (alpbets[i] != 0) {
                alpbets[i]--;

                //totals存储的分子部分，先不计算阶乘，而是用数组存储阶乘的每一个元素
                int[] totals = new int[50];
                for (int k = 2; k < length; k++) {
                    totals[k] = k;
                }
                for (int k = 0; k < alpbets.length; k++) {
                    for (int l = 2; l <= alpbets[k]; l++) {
                        //tmp就是分母中每一个阶乘中的一个元素，例如假设r1=2*3*4*...*k,则tmp就为2或3...k其中的一个
                        int tmp = l;
                        for (int m = 2; m < length; m++) {
                            int g = gcd(tmp, totals[m]);
                            tmp = tmp / g;
                            totals[m] = totals[m] / g;
                        }
                    }
                }

                long num = 1;
                for (int k = 2; k < length; k++) {
                    num = num * totals[k];
                }
                totalNum += num;
                alpbets[i]++;
            }
        }
        return totalNum;
    }

    private static int gcd(int x, int y) {
        int remain;
        int bigger = Math.max(x, y);
        int smaller = Math.min(x, y);
        while (bigger % smaller != 0) {
            remain = bigger % smaller;
            bigger = smaller;
            smaller = remain;
        }
        return smaller;
    }
}
```

