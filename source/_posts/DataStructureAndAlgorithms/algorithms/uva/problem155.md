---
title: problem155 All Squares
date: 2025-03-24 20:32:56
mathjax: true
categories:
- [数据结构与算法,uva]
tags:
- 算法
- uva
description: "UVA problem155.All Squares <br>
- 判断点在多少个正方形内"
---

## 题目

Geometrically, any square has a unique, well-defined centre point. On a grid this is only true if the sides of the square are an odd number of points long. Since any odd number can be written in the form 2k + 1, we can characterise any such square by specifying k, that is we can say that a square whose sides are of length 2k + 1 has size k. Now define a pattern of squares as follows.

1. The largest square is of size k (that is sides are of length 2k + 1) and is centred in a grid of size 1024 (that is the grid sides are of length 2049).
2. The smallest permissible square is of size 1 and the largest is of size 512, thus$1\leq k \leq 512$.
3. All squares of size k > 1 have a square of size ‘k div 2’ centred on each of their 4 corners. (‘div’ implies integer division, thus ‘9 div 2 = 4’).
4. The top left corner of the screen has coordinates (0,0), the bottom right has coordinates(2048,2048).

Hence, given a value of k, we can draw a unique pattern of squares according to the above rules.Furthermore any point on the screen will be surrounded by zero or more squares. (If the point is on the border of a square, it is considered to be surrounded by that square). Thus if the size of the largest square is given as 15, then the following pattern would be produced.

{% asset_img 155_pic1.jpg 图示%}

Write a program that will read in a value of k and the coordinates of a point, and will determine how many squares surround the point.

### Input

Input will consist of a series of lines. Each line will consist of a value of k and the coordinates of a point. The file will be terminated by a line consisting of three zeroes (0 0 0).

### Output

Output will consist of a series of lines, one for each line of the input. Each line will consist of the number of squares containing the specified point, right justified in a field of width 3.

### SampleInput

500 113 941
0 0 0

### SampleOutput

  5

### 题意

在每一个小网格长宽为1*1的网格中，正方形的边长与网格线重合，边长会经过网格线的交点。现在定义正方形的size，假设正方形边长覆盖网格线交点的个数为2k+1,则size为k。由于只有边长交点个数为2k+1时，才能使正方形的中心点落在网格线上。现在定义下面4个规则：

1. 最大的正方形size为k，且在一个size为1024的网格正中心；
2. k的取值范围为[1,512];
3. 当k>1时，size为k的正方形的四个顶点都会有一个正方形以该顶点为中心，该正方形size为k/2(取整).
4. 大网格左上坐标为(0,0),右下为(2048,2048).

按照这个规则，给出最大正方形的k值，我们可以生成一个图案，正方形的四个角会不断画出正方形，直到画到网格边缘或者size=1.现在给定最大的k值和一个坐标点，判断坐标点落在几个正方形内(落在正方形边长也算是落在正方形内).

## 思路

按照题中生成正方形的方式，有一个很重要的性质：由顶点生成的所有正方形不会超过该顶点所在正方形的中心位置。例如有一个正方形C，中心点为(x,y)，以该点为原点设置坐标轴，四个顶点$(x_1,y_1),(x_2,y_2),(x_3,y_3),(x_4,y_4)$分别落在一，二，三，四象限内，则对应的点生成的所有正方形不会超过该象限。

有了这个结论，计算过程就变成判断点落在哪个象限，是否在正方形内，然后以该象限的顶点为中心，size减半递归搜索。

### 问题

#### 实现上的问题

#### 性能上的问题

## 实现

```JAVA .{line-numbers}
//source code
import java.util.Scanner;

public class Main {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);

        int k, cdx, cdy, centx, centy;
        while (true) {
            //初始时正方形的中心点，且是固定的
            centx = 1024;
            centy = 1024;
            k = scanner.nextInt();
            cdx = scanner.nextInt();
            cdy = scanner.nextInt();
            if (k == 0 && cdx == 0 && cdy == 0) {
                break;
            }

            //recursive process
            int num = getPointInSquare(k, centx, centy, cdx, cdy);
            System.out.printf("%3d\n", num);
        }
    }

    private static int getPointInSquare(int k, int centx, int centy, int cdx, int cdy) {
        int result = 0;
        if (cdx >= centx - k && cdx <= centx + k &&
                cdy >= centy - k && cdy <= centy + k) {
            //point is in square
            result++;
        }
        if (cdx == centx || cdy == centy) {
            //in the middle line ,no sub square could cover the point
            return result;
        }
        if (k == 1) {
            return result;
        }

        //这里用来确定点落在哪个象限
        if (cdx > centx && cdy > centy) {
            centx = centx + k;
            centy = centy + k;
        } else if (cdx > centx && cdy < centy) {
            centx = centx + k;
            centy = centy - k;
        } else if (cdx < centx && cdy > centy) {
            centx = centx - k;
            centy = centy + k;
        } else if (cdx < centx && cdy < centy) {
            centx = centx - k;
            centy = centy - k;
        }
        k = k / 2;
        int subRes = getPointInSquare(k, centx, centy, cdx, cdy);
        return subRes + result;
    }
}
```
