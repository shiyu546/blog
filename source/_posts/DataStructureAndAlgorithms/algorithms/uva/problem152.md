---
title: problem152 Tree's a Crowd
date: 2025-01-22 09:41:38
mathjax: true
categories:
- [数据结构与算法,uva]
tags:
- 算法
- uva
description: "UVA problem152.Tree's a Crowd <br>
- 找最小值"
---

## 题目

Dr William Larch, noted plant psychologist and inventor of the phrase “Think like a tree — Think Fig” has invented a new classification system for trees. This is a complicated system involving a series of measurements which are then combined to produce three numbers (in the range [0, 255]) for any given tree. Thus each tree can be thought of as occupying a point in a 3-dimensional space. Because of the nature of the process, measurements for a large sample of trees are likely to be spread fairly uniformly throughout the whole of the available space. However Dr Larch is convinced that there are relationships to be found between close neighbours in this space. To test this hypothesis, he needs a histogram of the numbers of trees that have closest neighbours that lie within certain distance ranges.

Write a program that will read in the parameters of up to 5000 trees and determine how many of them have closest neighbours that are less than 1 unit away, how many with closest neighbours 1 or more but less than 2 units away, and so on up to those with closest neighbours 9 or more but less than 10 units away. Thus if di is the distance between the i’th point and its nearest neighbour(s) and $j \leq di \leq k$, with j and k integers and k = j + 1, then this point (tree) will contribute 1 to the j’th bin in the histogram (counting from zero). For example, if there were only two points 1.414 units apart, then the histogram would be 0, 2, 0, 0, 0, 0, 0, 0, 0, 0.

### Input

Input will consist of a series of lines, each line consisting of 3 numbers in the range [0; 255]. The file will be terminated by a line consisting of three zeroes.

### Output

Output will consist of a single line containing the 10 numbers representing the desired counts, each number right justified in a field of width 4.

### SampleInput

10 10 0
10 10 0
10 10 1
10 10 3
10 10 6
10 10 10
10 10 15
10 10 21
10 10 28
10 10 36
10 10 45
0 0 0

### SampleOutput

2 1 1 1 1 1 1 1 1 1

### 题意

在三维空间中给定很多坐标(坐标都是整数),对于每一个坐标P，找到离该点P最近的点的距离L，则每一个点都有一个与之关联的L。计算L在某个区间内的点的个数。

## 思路

计算任意两个点的距离，然后算出每个点的L值，再归类到指定的区间内。

### 问题

#### 实现上的问题

#### 性能上的问题

## 实现

```JAVA .{line-numbers}
//source code
import java.util.ArrayList;
import java.util.List;
import java.util.Scanner;

public class Main {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        List<Point> points = new ArrayList<>();
        while (true) {
            int x = scanner.nextInt();
            int y = scanner.nextInt();
            int z = scanner.nextInt();
            if (x == 0 && y == 0 && z == 0) {
                break;
            }
            points.add(new Point(x, y, z));
        }
        int size = points.size();

        //使用二维数组存储点之间的距离
        int[][] distance = new int[size][size];
        for (int i = 0; i < size; i++) {
            distance[i][i] = Integer.MAX_VALUE;
        }
        for (int i = 0; i < size; i++) {
            for (int j = i + 1; j < size; j++) {
                int dis = calulateDistance(points.get(i), points.get(j));
                distance[i][j] = dis;
                distance[j][i] = dis;
            }
        }
        int[] result = new int[10];
        for (int i = 0; i < result.length; i++) {
            result[i] = 0;
        }
        for (int i = 0; i < size; i++) {
            //计算每一个点的L
            int dis = distance[i][0];
            for (int j = 1; j < size; j++) {
                if (distance[i][j] < dis) {
                    dis = distance[i][j];
                }
            }

            if (dis <= 9) {
                result[dis]++;
            }
        }

        StringBuilder sb = new StringBuilder();
        for (int i = 0; i < result.length; i++) {
            String str = String.valueOf(result[i]);
            for (int j = 0; j < 4 - str.length(); j++) {
                sb.append(" ");
            }
            sb.append(str);
        }

        System.out.println(sb.toString());

    }

    public static int calulateDistance(Point pta, Point ptb) {
        int power = (pta.x - ptb.x) * (pta.x - ptb.x) + (pta.y - ptb.y) * (pta.y - ptb.y) + (pta.z - ptb.z) * (pta.z - ptb.z);
        int res = (int) Math.floor(Math.sqrt(power));
        return res;
    }

    public static class Point {
        int x, y, z;

        public Point() {
        }

        public Point(int x, int y, int z) {
            this.x = x;
            this.y = y;
            this.z = z;
        }
    }
}
```
