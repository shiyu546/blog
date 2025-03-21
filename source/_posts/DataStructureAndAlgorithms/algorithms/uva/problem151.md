---
title: problem151 Power Crisis
date: 2025-01-06 11:02:31
mathjax: true
categories:
- [数据结构与算法,uva]
tags:
- 算法
- uva
description: "UVA problem151.Power Crisis <br>
- 约瑟夫问题变种"
---

## 题目

During the power crisis in New Zealand this winter (caused by a shortage of rain and hence low levels in the hydro dams), a contingency scheme was developed to turn off the power to areas of the country in a systematic, totally fair, manner. The country was divided up into N regions (Auckland was region number 1, and Wellington number 13). A number, m, would be picked ‘at random’, and the power would first be turned off in region 1 (clearly the fairest starting point) and then in every m’th region after that, wrapping around to 1 after N, and ignoring regions already turned off. For example, if N = 17 and m = 5, power would be turned off to the regions in the order:1,6,11,16,5,12,2,9,17,10,4,15,14,3,8,13,7.

The problem is that it is clearly fairest to turn off Wellington last (after all, that is where the Electricity headquarters are), so for a given N, the ‘random’ number m needs to be carefully chosen so that region 13 is the last region selected.

Write a program that will read in the number of regions and then determine the smallest number m that will ensure that Wellington (region 13) can function while the rest of the country is blacked out.

### Input

Input will consist of a series of lines, each line containing the number of regions (N) with $13\leq N < 100$. The file will be terminated by a line consisting of a single ‘0’.

### Output

Output will consist of a series of lines, one for each line of the input. Each line will consist of the number m according to the above scheme.

### SampleInput

17
0

### SampleOutput

7

### 题意

从1到N有N各区域，每个区域轮流停电，停电规则如下：从1开始，隔m个区域停电，停过电的就不再停，到N之后又从1开始数。题目给了一个示例，在N = 17 and m = 5的情况下，停电顺序为：1,6,11,16,5,12,2,9,17,10,4,15,14,3,8,13,7。现在请问最后一个停电的区域是13号区域的最小值m是多少。

## 思路

本题是一道典型的约瑟夫环问题，现在要求m，我们就从m=1开始，逐步递增，算出满足条件的第一个m，就是要求的值。

### 问题

#### 实现上的问题

#### 性能上的问题

## 实现

```JAVA .{line-numbers}
//source code
import java.util.Arrays;
import java.util.Scanner;

public class Main {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);

        label:
        while (true) {
            int n = scanner.nextInt();
            if (n == 0) {
                break;
            }
            int[] tags = new int[n];
            //从1开始，递增到满足条件的m为止
            for (int i = 1;; i++) {
                Arrays.fill(tags, 0);

                int currentPos = 0;
                tags[0] = 1;
                int num = 1;
                while (num < n) {
                    currentPos = findNextPos(tags, currentPos, i);
                    num++;
                    //找到第13号区域，并判断是不是最后一个停电的区域
                    if (currentPos == 12) {
                        if (num < n) {
                            break;
                        } else {
                            System.out.println(i);
                            continue label;
                        }
                    }
                }
            }
        }
    }

    private static int findNextPos(int[] tags, int currentPos, int m) {
        int start = 0;
        int nextPos = 0;
        while (true) {
            nextPos = currentPos + 1;
            if (nextPos >= tags.length) {
                nextPos = 0;
            }
            if (tags[nextPos] == 0) {
                start++;
                if (start == m) {
                    tags[nextPos] = 1;
                    break;
                }
            }
            currentPos = nextPos;
        }
        return nextPos;
    }
}
```
