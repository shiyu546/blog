---
title: problem143 Orchard Trees
date: 2024-11-12 20:22:53
mathjax: true
categories:
- [数据结构与算法,uva]
tags:
- 算法
- uva
description: "UVA problem143.Orchard Trees <br>
- 数指定区域内坐标为整数的点"
---

## 题目

An Orchardist has planted an orchard in a rectangle with trees uniformly spaced in both directions.Thus the trees form a rectangular grid and we can consider the trees to have integer coordinates. The origin of the coordinate system is at the bottom left of the following diagram:
{% asset_img 143_pic1.jpg 图示%}
Consider that we now overlay a series of triangles on to this grid. The vertices of the triangle can have any real coordinates in the range 0.0 to 100.0, thus trees can have coordinates in the range 1 to 99. Two possible triangles are shown.

Write a program that will determine how many trees are contained within a given triangle. For the purposes of this problem, you may assume that the trees are of point size, and that any tree (point) lying exactly on the border of a triangle is considered to be in the triangle.

### Input

Input will consist of a series of lines. Each line will contain 6 real numbers in the range 0.00 to 100.00 representing the coordinates of a triangle. The entire file will be terminated by a line containing 6 zeroes (0 0 0 0 0 0).

### Output

Output will consist of one line for each triangle, containing the number of trees for that triangle right justified in a field of width 4.

### SampleInput

1.5 1.5 1.5 6.8 6.8 1.5
10.7 6.9 8.5 1.5 14.5 1.5
0 0 0 0 0 0

### SampleOutput

15
17

### 题意

给定一个坐标系，坐标系的范围是[0,100],其中横纵坐标都是整数的就是整数坐标点，整数的取值范围是[1,99]，任意给定三个点，判断这三个点围成的三角形内有多少个整数坐标点，其中在边界上的整数点也算三角形内。如果三个点不构成三角形，则只算在三个点构成的线段上的点。

## 思路

计算几何问题，判断点是否在三角形内，这里采用叉积方式，如下图有三角形ABC，P是任意一点，如果$\overrightarrow{AP}\times \overrightarrow{AB}$,$\overrightarrow{BP}\times \overrightarrow{BC}$,$\overrightarrow{CP}\times \overrightarrow{CA}$三个叉积结果方向相同(即正负相同),则P在三角形ABC内部。
{% asset_img 143_pic2.jpg 图示%}

当三个向量至少有一个为0时，表示点P在边界上(无论三个点ABC是否构成三角形)。先通过向量为0判断是否在边界线段所在的直线上，再通过矩形判定法判断点是否在线段上(判断点是否在线段为对角线上的矩形内)。

### 问题

#### 实现上的问题

浮点数计算时存在误差，判断叉积为0时需要设定一个误差范围，在误差范围内则认为叉积为0.

#### 性能上的问题

## 实现

```JAVA .{line-numbers}
//source code
import java.util.Scanner;

public class Main {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        while (true) {
            double ptax = scanner.nextDouble();
            double ptay = scanner.nextDouble();
            double ptbx = scanner.nextDouble();
            double ptby = scanner.nextDouble();
            double ptcx = scanner.nextDouble();
            double ptcy = scanner.nextDouble();
            if (ptax == 0.0 &&
                    ptay == 0.0 &&
                    ptbx == 0.0 &&
                    ptby == 0.0 &&
                    ptcx == 0.0 &&
                    ptcy == 0.0) {
                break;
            }

            int minx = minVal(ptax, ptbx, ptcx);
            int maxx = maxVal(ptax, ptbx, ptcx);
            int miny = minVal(ptay, ptby, ptcy);
            int maxy = maxVal(ptay, ptby, ptcy);
            int totalNumber = 0;
            for (int i = minx; i <= maxx; i++) {
                for (int j = miny; j <= maxy; j++) {
                    if (pointIsInTrangle(i, j, ptax, ptay, ptbx, ptby, ptcx, ptcy)) {
                        totalNumber++;
                    }
                }
            }
            System.out.printf("%4d\n", totalNumber);
        }

    }

    private static boolean pointIsInTrangle(int i, int j, double ptax, double ptay,
                                            double ptbx, double ptby,
                                            double ptcx, double ptcy) {
        Vector vec1 = new Vector(ptbx - ptax, ptby - ptay);
        Vector vec2 = new Vector(ptcx - ptbx, ptcy - ptby);
        Vector vec3 = new Vector(ptax - ptcx, ptay - ptcy);
        Vector ap = new Vector(i - ptax, j - ptay);
        Vector bp = new Vector(i - ptbx, j - ptby);
        Vector cp = new Vector(i - ptcx, j - ptcy);
        double direct1 = vec1.x * ap.y - vec1.y * ap.x;
        double direct2 = vec2.x * bp.y - vec2.y * bp.x;
        double direct3 = vec3.x * cp.y - vec3.y * cp.x;
        if (Math.abs(direct1) <= 1e-6 && pointInline(i, j, ptax, ptay, ptbx, ptby)) {
            return true;
        }
        if (Math.abs(direct2) <= 1e-6 && pointInline(i, j, ptbx, ptby, ptcx, ptcy)) {
            return true;
        }
        if (Math.abs(direct3) <= 1e-6 && pointInline(i, j, ptcx, ptcy, ptax, ptay)) {
            return true;
        }

        if ((direct1 > 0 && direct2 > 0 && direct3 > 0) ||
                (direct1 < 0 && direct2 < 0 && direct3 < 0)) {
            return true;
        }

        return false;
    }

    private static boolean pointInline(int i, int j,
                                       double ptax, double ptay, double ptbx, double ptby) {
        boolean res = (Math.min(ptax, ptbx) <= i && i <= Math.max(ptax, ptbx)
                && Math.min(ptay, ptby) <= j && j <= Math.max(ptay, ptby));
        return res;
    }

    public static int minVal(double a, double b, double c) {
        double minaTob = Math.min(a, b);
        double min = Math.min(minaTob, c);
        int i = (int) Math.floor(min);
        if (i < 1) {
            return 1;
        } else if (i > 99) {
            return 99;
        }
        return i;
    }

    public static int maxVal(double a, double b, double c) {
        double maxTob = Math.max(a, b);
        double max = Math.max(maxTob, c);
        int i = (int) Math.ceil(max);
        if (i < 1) {
            return 1;
        } else if (i > 99) {
            return 99;
        }
        return i;
    }

    public static class Vector {
        double x, y;

        public Vector(double x, double y) {
            this.x = x;
            this.y = y;
        }
    }
}

```
