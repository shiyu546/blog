---
title: problem160 Factors and Factorials
date: 2025-04-08 14:31:50
mathjax: true
categories:
- [数据结构与算法,uva]
tags:
- 算法
- uva
description: "UVA problem160.Factors and Factorials <br>
- 计算数字由几个质数组合而来，每个又有多少个"
---

## 题目

The factorial of a number N (written N!) is defined as the product of all the integers from 1 to N. It is often defined recursively as follows:
$$1! = 1$$
$$N! = N \times (N - 1)!$$

Factorials grow very rapidly — 5! = 120, 10! = 3,628,800. One way of specifying such large numbers is by specifying the number of times each prime number occurs in it, thus 825 could be specified as (0 1 2 0 1) meaning no twos, 1 three, 2 fives, no sevens and 1 eleven.

Write a program that will read in a number $N (2 \leq N \leq 100)$ and write out its factorial in terms of the numbers of the primes it contains.

### Input

Input will consist of a series of lines, each line containing a single integer N. The file will be terminated by a line consisting of a single ‘0’.

### Output

Output will consist of a series of blocks of lines, one block for each line of the input. Each block will start with the number N, right justified in a field of width 3, and the characters ‘!’, space, and ‘=’. This will be followed by a list of the number of times each prime number occurs in N!.

These should be right justified in fields of width 3 and each line (except the last of a block, which may be shorter) should contain fifteen numbers. Any lines after the first should be indented.

Follow the layout of the example shown below exactly.

### SampleInput

5
53
0

### SampleOutput

  5! = 3 1 1
 53! = 49 23 12 8 4 4 3 2 2 1 1 1 1 1 1
        1

### 题意

给定一个数字N,N可以表示为质数的乘积，例如$825=2^0*3^1*5^2*7^0*11^1$.现在给定一个数N，要求分解N的阶乘N!,输出每个质数的指数部分，没有则输出0.

## 思路

按照题意，相当于要计算每一个质数的指数部分，所以依次取质数去阶乘中的每一个数去除，获取能整除的次数，则得到了该阶乘该质数的指数部分。例如
$$N!=1*2*3*...*(N-1)*N$$
要获取有多少个2，用2分别取除1，2，3，4...(N-1)和N，总共除的次数就是2的指数部分，依次类推。由于N不大于100，质数也不会超过100，可枚举所有的质数。

### 问题

#### 实现上的问题

#### 性能上的问题

## 实现

```JAVA .{line-numbers}
//source code
import java.util.*;


public class Main {

    private static List<Integer> primes = Arrays.asList(2, 3, 5, 7, 11, 13, 17, 19, 23, 29,
            31, 37, 41, 43, 47, 53, 59, 61, 67, 71,
            73, 79, 83, 89, 97);

    private static Map<Integer, Integer> initMap = new HashMap<>();

    static {
        initMap.put(2, 0);
        initMap.put(3, 1);
        initMap.put(5, 2);
        initMap.put(7, 3);
        initMap.put(11, 4);
        initMap.put(13, 5);
        initMap.put(17, 6);
        initMap.put(19, 7);
        initMap.put(23, 8);
        initMap.put(29, 9);
        initMap.put(31, 10);
        initMap.put(37, 11);
        initMap.put(41, 12);
        initMap.put(43, 13);
        initMap.put(47, 14);
        initMap.put(53, 15);
        initMap.put(59, 16);
        initMap.put(61, 17);
        initMap.put(67, 18);
        initMap.put(71, 19);
        initMap.put(73, 20);
        initMap.put(79, 21);
        initMap.put(83, 22);
        initMap.put(89, 23);
        initMap.put(97, 24);
    }

    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        while (true) {
            int num = scanner.nextInt();
            if (num == 0) {
                break;
            }

            //数组用来存放阶乘的每一个乘积数字
            int[] numbers = new int[num + 1];
            numbers[0] = 1;
            numbers[1] = 1;
            for (int i = 2; i < numbers.length; i++) {
                numbers[i] = i;
            }

            //用来存放<质数,指数>对
            Map<Integer, Integer> countsMap = new HashMap<>();

            int i = 0;
            while (!allHandles(numbers)) {
                if (i >= primes.size()) {
                    break;
                }
                int divide = primes.get(i);
                for (int j = 2; j < numbers.length; j++) {
                    int number = numbers[j];
                    //一个数字可能分解为质数的多次放，所以需要循环去除
                    while (number % divide == 0) {
                        if (countsMap.containsKey(divide)) {
                            countsMap.put(divide, countsMap.get(divide) + 1);
                        } else {
                            countsMap.put(divide, 1);
                        }
                        number = number / divide;
                    }
                    numbers[j] = number;
                }
                i++;
            }

            int[] arys = new int[25];
            for (Integer prime : countsMap.keySet()) {
                arys[initMap.get(prime)] = countsMap.get(prime);
            }

            //用来记录最大的质数的索引
            int lastIndex = 24;
            for (; lastIndex >= 0; lastIndex--) {
                if (arys[lastIndex] != 0) {
                    break;
                }
            }

            StringBuilder sb = new StringBuilder();
            sb.append(fillNumber(num, 3));
            sb.append("! =");
            int len = sb.length();
            for (int j = 0; j <= lastIndex; j++) {
                if (j > 0 && j % 15 == 0) {
                    sb.append("\n");
                    for (int k = 0; k < len; k++) {
                        sb.append(" ");
                    }
                }
                sb.append(fillNumber(arys[j], 3));
            }
            System.out.println(sb.toString());
        }
    }

    private static String fillNumber(int num, int width) {
        String str = String.valueOf(num);
        int len = str.length();
        StringBuilder sb = new StringBuilder();
        for (int i = 0; i < width - len; i++) {
            sb.append(" ");
        }
        sb.append(str);
        return sb.toString();
    }

    /**
     * 如果阶乘每一个乘积都被除到只有1，表示已经除完
     */
    public static boolean allHandles(int[] numbers) {
        for (int i = 0; i < numbers.length; i++) {
            if (numbers[i] != 1) {
                return false;
            }
        }
        return true;
    }
}
```
