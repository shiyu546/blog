---
title: problem147 Dollars
date: 2024-12-18 09:40:43
mathjax: true
categories:
- [数据结构与算法,uva]
tags:
- 算法
- uva
description: "UVA problem147.Dollars <br>
- 给定多枚面值的硬币，计算总共有多少种组合可以构成指定金额"
---

## 题目

New Zealand currency consists of $100, $50, $20, $10, and $5 notes and $2, $1, 50c, 20c, 10c and 5c coins. Write a program that will determine, for any given amount, in how many ways that amount may be made up. Changing the order of listing does not increase the count. Thus 20c may be made up in
4 ways: 1 $\times$ 20c, 2 $\times$ 10c, 10c+2 $\times$ 5c, and 4 $\times$ 5c.

### Input

Input will consist of a series of real numbers no greater than $300.00 each on a separate line. Each amount will be valid, that is will be a multiple of 5c. The file will be terminated by a line containing zero (0.00).

### Output

Output will consist of a line for each of the amounts in the input, each line consisting of the amount of money (with two decimal places and right justified in a field of width 6), followed by the number of ways in which that amount may be made up, right justified in a field of width 17.

### SampleInput

0.20
2.00
0.00

### SampleOutput

0.20 4
2.00 293

### 题意

钱的面值总共有100，50，20，10，5，2，1块和50，20，10，5分共11种，现给定金额，计算共有多少种面值组合方式构成该金额。

## 思路

该问题是一个典型的动态规划问题，问题的关键是找出递推式，所以需要找到一种划分组合数量的方法，假设使用的面值有5,10,20美分三种，金额为m，则组合方式可以划分为包含20美分的部分和不包含20美分的部分。即
$$ C_m=C_{m1}+C_{m2} \\
\text{m1为包含20美分的组合，m2为不包含20美分的组合}
$$
从$C_{m1}$中我们减去一个20美分，得到的组合数仍然保持不变，这样就得到了递推关系。

$$\begin{bmatrix}
{a_{11}}&{a_{12}}&{\cdots}&{a_{1n}}\\
{a_{21}}&{a_{22}}&{\cdots}&{a_{2n}}\\
{\vdots}&{\vdots}&{\ddots}&{\vdots}\\
{a_{m1}}&{a_{m2}}&{\cdots}&{a_{mn}}\\
\end{bmatrix}$$

矩阵的列表示金额P，第一列表示0，第二列表示1，依此类推直到第n列表示金额P。行表示用来组合的面值，例如第一行表示面值为0，第二行表示用5分组合成对应的金额的组合数，第三行表示用5和10美分组合成对应金额的组合数。所以a_{ij}表示的含义是使用前i种面值的钱币组合成金额j的组合方式。递推关系为：
$$ a_{ij}=a_{(i-1)j}+a_{i(j-k)} \tag{k表示第i行钱币面值}  $$
第一行的值为
$$\begin{bmatrix}
{1}&{0}&{\cdots}&{0}\\
{a_{21}}&{a_{22}}&{\cdots}&{a_{2n}}\\
{\vdots}&{\vdots}&{\ddots}&{\vdots}\\
{a_{m1}}&{a_{m2}}&{\cdots}&{a_{mn}}\\
\end{bmatrix}$$

在i,j,k的值有意义的情况下，我们可以递推出金额P的所有组合数。

### 问题

#### 实现上的问题

矩阵表示的递推关系可以简化为数组形式，因为下一行只依赖本行之前的值和上一行的值，所以可以用一个数组来表示矩阵。

#### 性能上的问题

输出和接收double类型的数据可能存在性能问题。

## 实现

```JAVA .{line-numbers}
//source code
import java.util.Scanner;

public class Main {
    private static int[] coins = {5, 10, 20, 50, 100, 200, 500, 1000, 2000, 5000, 10000};

    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);

        StringBuilder sb = new StringBuilder();

        while (true) {
            String numStr = scanner.next();
            double num = Double.valueOf(numStr);
            if (Math.abs(num) < 1e-6) {
                break;
            }
            int total = (int) (num * 100);
            if (total % 5 != 0) {
                int remain = total % 5;
                total += 5 - remain;
            }
            //初始化状态转义矩阵
            long[] arys = new long[total + 1];
            arys[0] = 1;
            for (int i = 1; i < args.length; i++) {
                arys[i] = 0;
            }
            long totalNumbers = 0;
            for (int i = 0; i < coins.length; i++) {
                int coinVal = coins[i];
                //如果金额小于当前行钱币面值，直接返回
                if (total < coinVal) {
                    totalNumbers = arys[total];
                    break;
                }
                //逐步推导递推矩阵(数组形式表示)
                for (int j = coinVal; j < arys.length; j++) {
                    arys[j] = arys[j - coinVal] + arys[j];
                }
            }
            totalNumbers = arys[total];
            formatPrint(numStr, totalNumbers, sb);
        }
        System.out.printf(sb.toString());
    }

    public static void formatPrint(String numStr, long totalNumber, StringBuilder sb) {
        int length = numStr.length();
        for (int i = 0; i < 6 - length; i++) {
            sb.append(" ");
        }
        sb.append(numStr);
        int length2 = String.valueOf(totalNumber).length();
        for (int i = 0; i < 17 - length2; i++) {
            sb.append(" ");
        }
        sb.append(totalNumber);
        sb.append("\n");
    }
}
```
