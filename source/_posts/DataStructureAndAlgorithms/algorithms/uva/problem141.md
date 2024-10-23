---
title: problem141 The Spot Game
date: 2024-10-23 22:21:19
mathjax: true
categories:
- [数据结构与算法,uva]
tags:
- 算法
- uva
description: "UVA problem141.The Spot Game <br>
- 判断棋盘下棋过程中摆放方式是否重复"
---

## 题目

The game of Spot is played on an$N \times N$ board as shown below for N = 4. During the game, alternate players may either place a black counter (spot) in an empty square or remove one from the board, thus producing a variety of patterns. If a board pattern (or its rotation by 90 degrees or 180 degrees) is repeated during a game, the player producing that pattern loses and the other player wins. The game terminates in a draw after 2N moves if no duplicate pattern is produced before then.

Consider the following patterns:
{% asset_img 141_pic1.png 摆放方式 %}

If the first pattern had been produced earlier, then any of the following three patterns (plus one other not shown) would terminate the game, whereas the last one would not.

### Input

Input will consist of a series of games, each consisting of the size of the board$N(2\leq N \leq 50)$ followed, on separate lines, by 2N moves, whether they are all necessary or not. Each move will consist of the coordinates of a square (integers in the range 1..N) followed by a blank and a character ‘+’ or ‘-’ indicating the addition or removal of a spot respectively. You may assume that all moves are legal, that is there will never be an attempt to place a spot on an occupied square, nor to remove a non-existent spot. Input will be terminated by a zero (0).

### Output

Output will consist of one line for each game indicating which player won and on which move, or that the game ended in a draw. See the Sample Output below for the exact format.

### SampleInput

2
1 1 +
2 2 +
2 2 -
1 2 +
2
1 1 +
2 2 +
1 2 +
2 2 -
0

### SampleOutput

Player 2 wins on move 3
Draw

### 题意

题目给出了一个$N \times N$的棋盘，共将棋盘分为了$N^2$个小方格(上图给出了N=4的情况，共将棋盘分为了16个小方格)，现在有两位选手，轮流操作小方格(共两种操作，要么将小方格涂黑，要么将已经涂黑的小方格擦掉)，如果一位选手操作之后棋盘上的黑点摆放方式曾经出现过，那么该选手就输了。

判断棋盘黑点摆放方式是否出现不只是判断一摸一样的摆放，在一种摆放方式下，将棋盘旋转90度(有两种旋转方式，左旋和右旋)、180度得到的摆放方式仍然属于该种摆放方式(题意中没有读出来但实际上还有一种镜像摆放方式)，所以每一种摆放方式实际上代表了5种摆放(可能重复):本身、左旋90度、右旋90度、旋转180度、镜像。

## 思路

要判断摆放方式是否重复，需要记录每一步的摆放方式，每一步都是棋盘上一些点的集合，所以记录历史摆放方式构成了数组的数组。

如果两种摆放方式的点数量不一致，那这两种摆放方式一定不一样。基于此，我们定义一个map，key是点的数量，value是点集合的数组。由于$2\leq N \leq50$,点的横纵坐标不超过50，可以用一个字节存储，所以我们用Short(两字节)存储点，高8位存x坐标，低8位存y坐标。

当我们存储了历史的摆放方式后，剩下的就是比较了。对于历史摆放方式的点集合，我们要求出它的其余4种摆放方式点集合。考虑$N\times N$的情况，$A_{ij}$在操作之后的位置：

$$
\begin{cases}
A_{ij}=>A_{(N+1-j)i} \quad 左旋90度：第i行变成第i列，j列变成(N+1-j)行\\
A_{ij}=> A_{j(N+1-i)} \quad 右旋90度：第i行变成(N+1-j)列，j列变成j行\\
A_{ij}=> A_{(N+1-i)(N+1-j)} \quad 旋转180度：第i行变成(N+1-i)行，j列变成(N+1-j)列\\
A_{ij}=>A_{i(N+1-j)} \quad 镜面翻转：第i行不变，j列变成(N+1-j)列\\
\end{cases}
$$

每一个点集通过上述的一种变换后得到另一个点集，如果操作之后的点集S与这5种方式都不相等，则S没出现过。继续直至2N步都走完或者某一步出现重复的摆放方式(即点集与历史点集相同)。

### 问题

#### 实现上的问题

#### 性能上的问题

## 实现

```JAVA .{line-numbers}
//source file
import java.util.*;

public class Main {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        while (true) {
            String sizeStr = scanner.nextLine();
            if ("0".equals(sizeStr)) {
                break;
            }
            int size = Integer.valueOf(sizeStr);
            int step = 2 * size;
            Map<Integer, List<List<Short>>> map = new HashMap<>();
            List<Short> currentPoints = new ArrayList<>();
            List<String> steps = new ArrayList<>();

            for (int i = 1; i <= step; i++) {
                steps.add(scanner.nextLine());
            }

            boolean hasResult = false;
            for (int i = 1; i <= step; i++) {
                String pointOp = steps.get(i - 1);
                String[] opAry = pointOp.split(" ");
                String xStr = opAry[0];
                String yStr = opAry[1];
                String op = opAry[2];
                //用一个short存放一个点
                Short point = (short) ((Short.parseShort(xStr) << 8) | Short.parseShort(yStr));
                if (op.equals("+")) {
                    currentPoints.add(point);
                } else {
                    int index = currentPoints.indexOf(point);
                    currentPoints.remove(index);
                }
                int pointSize = currentPoints.size();
                //pointList存放历史点集
                List<List<Short>> pointList = map.get(pointSize);
                if (pointList == null) {
                    List<Short> tmp = new ArrayList<>(currentPoints);
                    List<List<Short>> tmpList = new ArrayList<>();
                    tmpList.add(tmp);
                    map.put(pointSize, tmpList);
                } else {
                    boolean cmpRes = comparePoints(currentPoints, pointList, (short) size);
                    if (cmpRes) {
                        System.out.printf("Player %d wins on move %d\n", i % 2 == 1 ? 2 : 1, i);
                        hasResult = true;
                        break;
                    } else {
                        List<Short> tmp = new ArrayList<>(currentPoints);
                        map.get(pointSize).add(tmp);
                    }
                }
            }
            if (!hasResult) {
                System.out.printf("Draw\n");
            }
        }
    }

    /**
     * 比较相同点数量的当前点集和历史点集
     */
    private static boolean comparePoints(List<Short> currentPoints, List<List<Short>> pointList, short size) {
        for (int i = 0; i < pointList.size(); i++) {
            List<Short> points = pointList.get(i);
            List<Short> leftRotate = new ArrayList<>();
            List<Short> rightRotate = new ArrayList<>();
            List<Short> right180Rotate = new ArrayList<>();
            List<Short> mirrorRotate = new ArrayList<>();
            getLeftRotate(points, size, leftRotate, rightRotate, right180Rotate, mirrorRotate);
            if (comparePointList(currentPoints, points) ||
                    comparePointList(currentPoints, leftRotate) ||
                    comparePointList(currentPoints, rightRotate) ||
                    comparePointList(currentPoints, right180Rotate) ||
                    comparePointList(currentPoints, mirrorRotate)) {
                return true;
            }
        }
        return false;
    }

    /**
     * 历史点集计算其操作(左旋，右旋，旋转180度，镜像)之后的点集
     */
    private static void getLeftRotate(List<Short> points, short size,
                                      List<Short> leftRotate,
                                      List<Short> rightRotate,
                                      List<Short> right180Rotate,
                                      List<Short> mirrorRotate) {
        for (int i = 0; i < points.size(); i++) {
            short point = points.get(i);
            short x = (short) ((point & 0xFF00) >> 8);
            short y = (short) (point & 0x00FF);
            short leftPt = (short) ((size + 1 - y) << 8 | x);
            short rightPt = (short) (y << 8 | (size + 1 - x));
            short right180Pt = (short) ((size + 1 - x) << 8 | (size + 1 - y));
            short mirrorPt = (short) (x << 8 | (size + 1 - y));

            leftRotate.add(leftPt);
            rightRotate.add(rightPt);
            right180Rotate.add(right180Pt);
            mirrorRotate.add(mirrorPt);
        }
    }

    private static boolean comparePointList(List<Short> ary1, List<Short> ary2) {
        ary1.sort(Comparator.naturalOrder());
        ary2.sort(Comparator.naturalOrder());
        for (int i = 0; i < ary1.size(); i++) {
            if (!Objects.equals(ary1.get(i), ary2.get(i))) {
                return false;
            }
        }
        return true;
    }
}
```
