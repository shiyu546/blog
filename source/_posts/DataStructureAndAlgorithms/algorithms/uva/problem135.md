---
title: problem135 No Rectangles
date: 2024-10-04 08:25:57
mathjax: true
categories:
- [数据结构与算法,uva]
tags:
- 算法
- uva
description: "UVA 135.No Rectangles <br>
- 寻找一种落点方式使得网格内点不构成矩形"
---

## 题目

Consider a grid such as the one shown. We wish to mark k intersections in each of n rows and n columns in such a way that no 4 of the selected intersections form a rectangle with sides parallel to the grid. Thus for k = 2 and n = 3, a possible solution is:
{% asset_img 135_pic1.png k=2 and n=3 %}

It can easily be shown that for any given value of k,$k^2 + k + 1$ is a lower bound on the value of n, and it can be shown further that n need never be larger than this.

Write a program that will find a solution to this problem for $k \leq 32$,$k - 1$ will be 0, 1 or prime.

### Input

Input will consist of some values for k, one per line.

### Output

For each value of k, output will consist of n lines of k points indicating the selected points on that line.

Print a blank line between two values of k.

### SampleInput

2
1

### SampleOutput

1 2
1 3
2 3

1

### 题意

给定一个$n\times n$的网格，网格共有n行，每一行有n个点。在网格的每一行n个点上标记k个点，故共标记$n\times k$个点，使得所有的这些标记的点不构成边与网格线平行的矩形。题目给出了n和k的关系：
$$ n=k^2-k+1 $$

## 思路

暂时没有找到原理去摆放这些点，通过查找他人的解决方案，得到一种摆放方式：
{% asset_img 135_pic2.png k=5 and n=21 %}

第{1,2,...,k}行，第一个点都落在n=1上，后续的依次摆放，所以每一行占k-1列,由于
$$n=k^2-k+1$$
=>
$$ k=(n-1)/(k-1) $$
所以一定可以摆放k行(见图k=5时前5行的摆放)。

后续{2,3,...k}列摆放第一个点时，每一列可以摆放k-1行，共有$(k-1)*(k-1)$行，加上前面k行，得到
$$ k^2-2*k+1+k $$
=>
$$ k^2-k+1 $$
行，与n相等。

以m列为例，每一列k-1行为一个组，依次有{1,2,...,k-1}组，每组共有{1,2,...,k-1}个正方形构成，第一组共由k-1个对角形落点构成，每组相比于上一组，第p个对角在上一组对应的p对角上右移p-1个位置。

以k=5为例，第一组都是落在对角线上(n=6,7,8,9),第二组在第一组对应的位置上右移p-1个单位，例如第二组第三个对角，就在第一组第三个对角上右移(3-1)=2个单位。
{% asset_img 135_pic3.png 第三组则右移两位 %}

### 问题

#### 实现上的问题

#### 性能上的问题

## 实现

```JAVA .{line-numbers}
//source file
import java.util.Scanner;

public class Main {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        boolean first = true;
        StringBuilder sb = new StringBuilder();
        while (scanner.hasNext()) {
            int k = scanner.nextInt();
            if (k == 1) {
                if (first) {
                    sb.append("1\n");
                    first = false;
                } else {
                    sb.append("\n1\n");
                }
                continue;
            }

            int n = k * k - k + 1;
            int[][] numbers = new int[k - 1][k - 1];
            for (int l = 0; l < numbers.length; l++) {
                for (int m = 0; m < numbers.length; m++) {
                    numbers[l][m] = (l + 1) * (k - 1) + m + 2;
                }
            }
            if (first) {
                first = false;
            } else {
                sb.append("\n");
            }
            for (int i = 1; i <= n; i++) {
                if (i == 1) {
                    for (int j = 1; j < k; j++) {
                        sb.append(j);
                        sb.append(" ");
                    }
                    sb.append(k);
                    sb.append("\n");
                } else if (i <= k) {
                    sb.append(1);
                    int start = (i - 1) * (k - 1) + 1;
                    for (int j = start + 1; j < start + k; j++) {
                        sb.append(" ");
                        sb.append(j);
                    }
                    sb.append("\n");
                } else {
                    int group = (i - k - 1) / (k - 1);
                    int remain = (i - k - 1) % (k - 1);

                    //move numbers
                    if (group > 0 && remain == 0) {
                        for (int l = 1; l < numbers.length; l++) {
                            int[] temp = new int[l];
                            for (int m = 0; m < l; m++) {
                                temp[m] = numbers[l][m];
                            }
                            for (int m = l; m < numbers.length; m++) {
                                numbers[l][m - l] = numbers[l][m];
                            }
                            for (int m = numbers.length - l; m < numbers.length; m++) {
                                numbers[l][m] = temp[m + l - numbers.length];
                            }
                        }

                    }

                    sb.append(group + 2);
                    for (int b = 1; b <= k - 1; b++) {
                        sb.append(" ");
                        sb.append(numbers[b - 1][remain]);
                    }
                    sb.append("\n");
                }
            }
        }
        System.out.printf(sb.toString());
    }
}
```
