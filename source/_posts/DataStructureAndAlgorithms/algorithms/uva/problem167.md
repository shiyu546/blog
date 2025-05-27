---
title: problem167 The Sultan’s Successors
date: 2025-05-16 10:18:15
mathjax: true
categories:
- [数据结构与算法,uva]
tags:
- 算法
- uva
description: "UVA problem167.The Sultan’s Successors <br>
- 八皇后问题"
---

## 题目

The Sultan of Nubia has no children, so she has decided that the country will be split into up to k separate parts on her death and each part will be inherited by whoever performs best at some test. It is possible for any individual to inherit more than one or indeed all of the portions. To ensure that only highly intelligent people eventually become her successors, the Sultan has devised an ingenious test. In a large hall filled with the splash of fountains and the delicate scent of incense have been placed k chessboards. Each chessboard has numbers in the range 1 to 99 written on each square and is supplied with 8 jewelled chess queens. The task facing each potential successor is to place the 8 queens on the chess board in such a way that no queen threatens another one, and so that the numbers on the squares thus selected sum to a number at least as high as one already chosen by the Sultan. (For those unfamiliar with the rules of chess, this implies that each row and column of the board contains exactly
one queen, and each diagonal contains no more than one.)

Write a program that will read in the number and details of the chessboards and determine the highest scores possible for each board under these conditions. (You know that the Sultan is both a good chess player and a good mathematician and you suspect that her score is the best attainable.)

### Input

Input will consist of k (the number of boards), on a line by itself, followed by k sets of 64 numbers, each set consisting of eight lines of eight numbers. Each number will be a positive integer less than 100. There will never be more than 20 boards.

### Output

Output will consist of k numbers consisting of your k scores, each score on a line by itself and right justified in a field 5 characters wide.

### SampleInput

1
1 2 3 4 5 6 7 8
9 10 11 12 13 14 15 16
17 18 19 20 21 22 23 24
25 26 27 28 29 30 31 32
33 34 35 36 37 38 39 40
41 42 43 44 45 46 47 48
48 50 51 52 53 54 55 56
57 58 59 60 61 62 63 64

### SampleOutput

260

### 题意

八皇后问题，给出一个$8 \times 8$的棋盘，将8个皇后放到棋盘上，使得不存在任何冲突，即任意两个皇后不在同一行、同一列、同一对角线上。每一个棋盘格子都有一个分数，找出能放置8皇后的方式且使得皇后所在格子分数和最大。

## 思路

这是一个8皇后的问题，由于同一行、同一列最多一个皇后，则每一行或每一列至少一个皇后。放置皇后的方式可以采用试探法，先放置第一列，第一列则有8种可能，在放置了第一列后，第二列则采用尝试的方法，能放置则放置，不能则忽略，则在第一列已经放置的基础上，第二列也有多种可能。依此类推，整个放置过程构成了一颗树。

{% asset_img 167_pic1.jpg 皇后位置放置递推树%}

(i，j)分别表示行和列。

从递推树我们就可以列出递推关系了，即遍历每一列的所有位置。

### 问题

#### 实现上的问题

#### 性能上的问题

## 实现

```JAVA .{line-numbers}
//source code
import java.util.Scanner;

public class Main {

    public static int maxValue;

    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        int k = scanner.nextInt();
        for (int index = 0; index < k; index++) {
            int[][] numAry = new int[8][8];
            for (int i = 0; i < numAry.length; i++) {
                for (int j = 0; j < numAry[i].length; j++) {
                    numAry[i][j] = scanner.nextInt();
                }
            }
            maxValue = 0;
            int[][] tagAry = new int[8][8];
            for (int i = 0; i < tagAry.length; i++) {
                for (int j = 0; j < tagAry[i].length; j++) {
                    tagAry[i][j] = 0;
                }
            }

            fillQueue(0, tagAry, numAry);
            System.out.printf("%5d\n", maxValue);
        }
    }

    //递归的放置皇后，index表示当前放置的皇后的列，tagAry表示当前放置皇后的棋盘，numAry表示棋盘上的分数
    public static void fillQueue(int index, int[][] tagAry, int[][] numAry) {
        if (index >= 8) {
            int sum = 0;
            for (int i = 0; i < 8; i++) {
                for (int j = 0; j < 8; j++) {
                    if (tagAry[i][j] == 1) {
                        sum += numAry[i][j];
                    }
                }
            }
            if (maxValue < sum) {
                maxValue = sum;
            }
            return;
        }
        for (int i = 0; i < tagAry.length; i++) {
            if (couldPut(index, i, tagAry)) {
                tagAry[i][index] = 1;
                fillQueue(index + 1, tagAry, numAry);
                tagAry[i][index] = 0;
            }
        }
    }

    //试探的着在row行col列放置皇后，然后判断位置是否可以放置(即行列和对角线没有其他的皇后)
    public static boolean couldPut(int col, int row, int[][] tagAry) {
        for (int i = 0; i < tagAry.length; i++) {
            if (tagAry[row][i] == 1) {
                return false;
            }
            if (tagAry[i][col] == 1) {
                return false;
            }
        }
        int tmpCol = col, tmpRow = row;
        while (tmpCol >= 0 && tmpRow >= 0) {
            if (tagAry[tmpRow][tmpCol] == 1) {
                return false;
            }
            tmpRow--;
            tmpCol--;
        }
        tmpCol = col;
        tmpRow = row;
        while (tmpCol >= 0 && tmpRow < 8) {
            if (tagAry[tmpRow][tmpCol] == 1) {
                return false;
            }
            tmpRow++;
            tmpCol--;
        }

        tmpCol = col;
        tmpRow = row;
        while (tmpCol < 8 && tmpRow >= 0) {
            if (tagAry[tmpRow][tmpCol] == 1) {
                return false;
            }
            tmpRow--;
            tmpCol++;
        }

        tmpCol = col;
        tmpRow = row;
        while (tmpCol < 8 && tmpRow < 8) {
            if (tagAry[tmpRow][tmpCol] == 1) {
                return false;
            }
            tmpRow++;
            tmpCol++;
        }
        return true;
    }

}
```
