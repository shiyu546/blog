---
title: problem154 Recycling
date: 2025-03-22 23:43:35
mathjax: true
categories:
- [数据结构与算法,uva]
tags:
- 算法
- uva
description: "UVA problem154.Recycling <br>
- 以一种垃圾桶颜色与废物对应关系为基准的最小改动方案"
---

## 题目

Kerbside recycling has come to New Zealand, and every city from Auckland to Invercargill has leapton to the band wagon. The bins come in 5 different colours — red, orange, yellow, green and blue —and 5 wastes have been identified for recycling — Plastic, Glass, Aluminium, Steel, and Newspaper.Obviously there has been no coordination between cities, so each city has allocated wastes to bins in an arbitrary fashion. Now that the government has solved the minor problems of today (such as reorganising Health, Welfare and Education), they are looking around for further challenges. The Minister for Environmental Doodads wishes to introduce the “Regularisation of Allocation of Solid Waste to Bin Colour Bill” to Parliament, but in order to do so needs to determine an allocation of his own. Being a firm believer in democracy (well some of the time anyway), he surveys all the cities that are using this recycling method. From these data he wishes to determine the city whose allocation scheme (if imposed on the rest of the country) would cause the least impact, that is would cause the smallest number of changes in the allocations of the other cities. Note that the sizes of the cities is not an issue, after all this is a democracy with the slogan “One City, One Vote”.

Write a program that will read in a series of allocations of wastes to bins and determine which city’s allocation scheme should be chosen. Note that there will always be a clear winner.

### Input

Input will consist of a series of blocks. Each block will consist of a series of lines and each line will contain a series of allocations in the form shown in the example. There may be up to 100 cities in a block. Each block will be terminated by a line starting with ‘e’. The entire file will be terminated by a line consisting of a single ‘#’.

### Output

Output will consist of a series of lines, one for each block in the input. Each line will consist of the number of the city that should be adopted as a national example.

### SampleInput

r/P,o/G,y/S,g/A,b/N
r/G,o/P,y/S,g/A,b/N
r/P,y/S,o/G,g/N,b/A
r/P,o/S,y/A,g/G,b/N
e
r/G,o/P,y/S,g/A,b/N
r/P,y/S,o/G,g/N,b/A
r/P,o/S,y/A,g/G,b/N
r/P,o/G,y/S,g/A,b/N
ecclesiastical
\#

### SampleOutput

1
4

### 题意

新西兰实施垃圾分类回收，共有五种颜色-红橙黄绿蓝的垃圾桶，分别对应回收塑料、玻璃、吕、铁和报纸五种物品。不同城市不同颜色的垃圾桶回收的物品不同。现在想实施一项政策：统一同一种颜色的垃圾桶回收的物品一致，所以需要选定一个城市的方案为基准，去修改其他城市的方案使得所有城市保持一致。现在请问选择哪个城市使得修改最小。

修改最小是指两个城市相比较，垃圾桶颜色和物品相同则算完全匹配，所以两个城市的匹配数为1-5区间，设为r，不匹配数则为5-r，假设有$c_1,c_2,c_3...c_n$共n个城市，令$c_1$与$c_2$至$c_n$的不匹配数为$r_2$,$r_3$...$r_n$,这些数之和就是以该城市为基准的改动数，现在要找出使改动数最小的城市。

## 思路

两两计算出每个城市之间的不匹配度，然后每个城市的都相加，取最小值。

### 问题

#### 实现上的问题

#### 性能上的问题

## 实现

```JAVA .{line-numbers}
//source code
import java.util.*;

public class Main {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        List<String> cities = new ArrayList<>();
        while (true) {
            String line = scanner.nextLine();
            if ("#".equals(line)) {
                break;
            }
            if (line.startsWith("e")) {
                //calculate
                int citySize = cities.size();
                int[][] matrix = new int[citySize][citySize];
                for (int i = 0; i < citySize; i++) {
                    //diagonal is zero
                    matrix[i][i] = 0;
                }
                for (int i = 0; i < citySize; i++) {
                    for (int j = i + 1; j < citySize; j++) {
                        int diff = compareDifferent(cities.get(i), cities.get(j));
                        matrix[i][j] = diff;
                        matrix[j][i] = diff;
                    }
                }
                int minNum = Integer.MAX_VALUE, index = 0;
                for (int i = 0; i < citySize; i++) {
                    int sum = 0;
                    for (int j = 0; j < citySize; j++) {
                        sum += matrix[i][j];
                    }
                    if (sum < minNum) {
                        minNum = sum;
                        index = i;
                    }
                }
                System.out.println(index + 1);
                //clear list for reuse
                cities = new ArrayList<>();
                continue;
            }
            cities.add(line);
        }
    }

    //计算两个城市的差异数
    public static int compareDifferent(String citya, String cityb) {
        String[] matcha = citya.split(",");
        String[] matchb = cityb.split(",");
        int diff = 0;
        Set<String> mtSet = new HashSet<>();
        mtSet.addAll(Arrays.asList(matcha));
        for (int i = 0; i < matchb.length; i++) {
            if (!mtSet.contains(matchb[i])) {
                diff++;
            }
        }
        return diff;
    }
}
```
