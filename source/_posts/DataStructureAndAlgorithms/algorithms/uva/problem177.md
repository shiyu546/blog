---
title: problem177 Paper Folding
date: 2025-07-04 12:02:51
mathjax: true
categories:
- [数据结构与算法,uva]
tags:
- 算法
- uva
description: "UVA problem177.Paper Folding <br>
- 输出折纸痕迹"
---

## 题目

If a large sheet of paper is folded in half, then in half again, etc,with all the folds parallel, then opened up flat, there are a series of parallel creases, some pointing up and some down, dividing the paper into fractions of the original length. If the paper is only opened “half-way” up, so every crease forms a 90 degree angle, then (viewed end-on) it forms a “dragon curve”. For example, if four successive folds are made, then the following curve is see (note that it does not cross itself, but two corners touch):

{% asset_img 177_pic1.jpg 图示%}

Write a program to draw the curve which appears after N folds. The exact specification of the curve is as follows:

* The paper starts flat, with the “start edge” on the left, looking at it from above.
* The right half is folded over so it lies on top of the left half, then the right half of the new double sheet is folded on top of the left, to form a 4-thick sheet, and so on, for N folds.
* Then every fold is opened from a 180 degree bend to a 90 degree bend.
* Finally the bottom edge of the paper is viewed end-on to see the dragon curve.

From this view, the only unchanged part of the original paper is the piece containing the “start edge”, and this piece will be horizontal, with the “start edge” on the left. This uniquely defines the curve. In the above picture, the “start edge” is the left end of the rightmost bottom horizontal piece (marked ‘s’). Horizontal pieces are to be displayed with the underscore character ‘ ’, and vertical pieces with the ‘|’ character.

### Input

Input will consist of a series of lines, each with a single number N (1 ≤ N ≤ 13). The end of the input will be marked by a line containing a zero.

### Output

Output will consist of a series of dragon curves, one for each value of N in the input. Your picture must be shifted as far left, and as high as possible. Note that for large N, the picture will be greater than 80 characters wide, so it will look messy on the screen. The pattern for each different number of folds is terminated by a line containing a single ‘ˆ’.

### SampleInput

2
4
1
0

### SampleOutput

```plaintext
|_
 _|
^
   _   _
  |_|_| |_
   _|    _|
|_|
^
_|
^
```

### 题意

给定一张纸，沿中间将右半边折到左半边上面，就有一条折痕。如果在第一次折叠的基础上再将右半边折叠到左半边上，会得到两条额外的折痕，依此类推。折叠N次后将纸平铺，会得到多条平行的折痕，有的折痕向上，有的向下。现在将折痕处形成90度的角，从面向纸张的方向就可以看到一条折痕曲线，称为“龙曲线”。以左边第一条边水平为基准，输出折痕曲线。

## 思路

将折叠后的纸平铺，会得到多条折痕，按照折痕向上或向下，记录每一条边的走向，最后画出折痕图。这里涉及到两个问题：

1. 折叠N次后，共有多少条折痕，每一条的方向是什么。
2.如何最终确定折痕的图是什么样子的。

第一个问题可以递推，折叠一次有一条折痕；折叠二次在一次的基础上多两条折痕，折叠三次在二次的基础上翻倍，即每折叠一次在前一次基础上多增加一倍的折痕。折叠N次则有：

$$S(N)= 1+2+2^2+2^3+...+2^{N-1} $$ 
=>
$$ S(N)= 2^N-1 $$

折痕方向第一次是向下的，第二次在第一次基础上左边向下，右边向上。每多折叠一次，在上一次生成的折痕基础上两边会形成两条折痕，左边向下，右边向上。在已知折痕数量的基础上，按照该递归规律，计算每一条折痕的方向。

第二个问题是如果已知前一条边的方向和折痕的方向，计算下一条边的方向。这里可以通过枚举出来，边共有4种方向，折痕有2种，则共有8种可能。如果将起始边设置为坐标(0,0)，则可以计算出每一条边的坐标，最后输出该图像。

### 问题

#### 实现上的问题

#### 性能上的问题

## 实现

```JAVA {.line-numbers}
import java.io.*;
import java.util.*;

public class Main {
    public static void main(String[] args) throws IOException {
        BufferedReader reader = new BufferedReader(new InputStreamReader(System.in));
        while (true) {
            int number = Integer.parseInt(reader.readLine());
            if (number == 0) {
                break;
            }
            int arysLength = 1;
            for (int i = 0; i < number; i++) {
                arysLength *= 2;
            }

            //1 means up,2 means down
            int[] folds = new int[arysLength];
            folds[arysLength / 2] = 2;
            int center = arysLength / 2, start = 0, end = arysLength;

            //这里就是计算每一条折痕的方向，1代表向上，2表示向下。
            fillArys(folds, center, start, end);

            //计算每一条折痕的位置和方向
            getFolds(folds);

        }
    }

    private static void getFolds(int[] folds) {
        //通过计算边的位置，最终确定折痕图的范围大小
        int maxptx = Integer.MIN_VALUE, minptx = Integer.MAX_VALUE;
        int maxpty = Integer.MIN_VALUE, minpty = Integer.MAX_VALUE;

        List<FoldNode> nodes = new ArrayList<>();

        FoldNode node = new FoldNode();
        node.ptx = 0;
        node.pty = 0;
        node.direction = 1;
        nodes.add(node);

        if (maxptx < node.ptx) {
            maxptx = node.ptx;
        }
        if (minptx > node.ptx) {
            minptx = node.ptx;
        }
        if (maxpty < node.pty) {
            maxpty = node.pty;
        }
        if (minpty > node.pty) {
            minpty = node.pty;
        }

        FoldNode previousNode = node;

        //根据上一条边的方向和折痕的方向确定下一条边的方向.1表示向右,2表示向左,3表示向上,4表示向下.
        for (int i = 1; i < folds.length; i++) {
            FoldNode tpNode = new FoldNode();
            if (folds[i] == 2) {
                if (previousNode.direction == 1) {
                    tpNode.ptx = previousNode.ptx + 1;
                    tpNode.pty = previousNode.pty;
                    tpNode.direction = 3;
                } else if (previousNode.direction == 2) {
                    tpNode.ptx = previousNode.ptx - 1;
                    tpNode.pty = previousNode.pty + 1;
                    tpNode.direction = 4;
                } else if (previousNode.direction == 3) {
                    tpNode.ptx = previousNode.ptx - 1;
                    tpNode.pty = previousNode.pty - 1;
                    tpNode.direction = 2;
                } else {
                    tpNode.ptx = previousNode.ptx + 1;
                    tpNode.pty = previousNode.pty;
                    tpNode.direction = 1;
                }
            } else if (folds[i] == 1) {
                if (previousNode.direction == 1) {
                    tpNode.ptx = previousNode.ptx + 1;
                    tpNode.pty = previousNode.pty + 1;
                    tpNode.direction = 4;
                } else if (previousNode.direction == 2) {
                    tpNode.ptx = previousNode.ptx - 1;
                    tpNode.pty = previousNode.pty;
                    tpNode.direction = 3;
                } else if (previousNode.direction == 3) {
                    tpNode.ptx = previousNode.ptx + 1;
                    tpNode.pty = previousNode.pty - 1;
                    tpNode.direction = 1;
                } else {
                    tpNode.ptx = previousNode.ptx - 1;
                    tpNode.pty = previousNode.pty;
                    tpNode.direction = 2;
                }
            }
            if (maxptx < tpNode.ptx) {
                maxptx = tpNode.ptx;
            }
            if (minptx > tpNode.ptx) {
                minptx = tpNode.ptx;
            }
            if (maxpty < tpNode.pty) {
                maxpty = tpNode.pty;
            }
            if (minpty > tpNode.pty) {
                minpty = tpNode.pty;
            }
            nodes.add(tpNode);
            previousNode = tpNode;
        }

        StringBuilder sb = new StringBuilder();
        for (int j = minpty; j <= maxpty; j++) {
            StringBuilder linesb = new StringBuilder();
            for (int i = minptx; i <= maxptx; i++) {
                FoldNode tmpNode = new FoldNode();
                tmpNode.ptx = i;
                tmpNode.pty = j;

                boolean searched = false;
                for (int k = 0; k < nodes.size(); k++) {
                    if (nodes.get(k).ptx == i && nodes.get(k).pty == j) {
                        int val = nodes.get(k).direction;
                        if (val == 1 || val == 2) {
                            linesb.append("_");
                        } else if (val == 3 || val == 4) {
                            linesb.append("|");
                        }
                        searched = true;
                        break;
                    }
                }

                if (!searched) {
                    linesb.append(" ");
                }
            }
            String line = linesb.toString().replaceAll("\\s+$", "");
            sb.append(line);
            sb.append("\n");
        }
        System.out.printf(sb.toString());
        System.out.println("^");
    }

    public static void fillArys(int[] folds, int center, int start, int end) {
        int leftCenter = (center + start) / 2, rightCenter = (center + end) / 2;
        if (leftCenter == center || leftCenter == start) {
            return;
        }
        folds[leftCenter] = 2;
        folds[rightCenter] = 1;
        int leftStart = start, leftEnd = center;
        int rightStart = center, rightEnd = end;
        fillArys(folds, leftCenter, leftStart, leftEnd);
        fillArys(folds, rightCenter, rightStart, rightEnd);
    }

    public static class FoldNode {
        public int ptx;
        public int pty;
        public int direction;
    }
}
```
