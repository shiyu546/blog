---
title: problem133 The Dole Queue
date: 2024-09-24 22:49:44
mathjax: true
categories:
- [数据结构与算法,uva]
tags:
- 算法
- uva
description: "UVA 133.The Dole Queue <br>
- 约瑟夫环问题的变种"
---

## 题目

In a serious attempt to downsize (reduce) the dole queue, The New National Green Labour Rhinoceros Party has decided on the following strategy. Every day all dole applicants will be placed in a large circle, facing inwards. Someone is arbitrarily chosen as number 1, and the rest are numbered counterclockwise up to N (who will be standing on 1’s left). Starting from 1 and moving counter-clockwise, one labour official counts off k applicants, while another official starts from N and moves clockwise,counting m applicants. The two who are chosen are then sent off for retraining; if both officials pick the same person she (he) is sent off to become a politician. Each official then starts counting again at the next available person and the process continues until no-one is left. Note that the two victims(sorry, trainees) leave the ring simultaneously, so it is possible for one official to count a person already selected by the other official.

### Input

Write a program that will successively read in (in that order) the three numbers (N, k and m; k，m > 0,0 < N < 20) and determine the order in which the applicants are sent off for retraining. Each set of three numbers will be on a separate line and the end of data will be signalled by three zeroes (0 0 0).

### Output

For each triplet, output a single line of numbers specifying the order in which people are chosen. Each number should be in a field of 3 characters. For pairs of numbers list the person chosen by the counterclockwise official first. Separate successive pairs (or singletons) by commas (but there should not be a trailing comma).

Note: The symbol ⊔ in the Sample Output below represents a space.

### SampleInput

10 4 3
0 0 0

### SampleOutput

⊔⊔4⊔⊔8,⊔⊔9⊔⊔5,⊔⊔3⊔⊔1,⊔⊔2⊔⊔6,⊔10,⊔⊔7

### 题意

有N个人围成一圈面朝里，任意一个人编号为1，按逆时针方向从编号1给人按顺序编号，直到1号左手边的人编号为N。现在有两个人A和B，A从编号1开始顺时针数，数到k则数到的编号为s的人出列；B从编号为N的人逆时针开始数，数到m则编号为t的人出列。注意每数一圈A和B都是同时数的，所以A和B在数到s和t时，s和t都在圈里。当s和t出列之后，A从s下一个开始数，直到第k个人；B从t下一个数，直到第m个人。重复该过程直到圈里面所有的人都出列。如果A和B数到同一个人，则该回合只有一个人出列。

现要求出人员出列的顺序。

## 思路

此题是约瑟夫环的变种，按照题意标记出列的人即可。注意同时数和输出格式。

### 问题

#### 实现上的问题

#### 性能上的问题

## 实现

```JAVA .{line-numbers}
//file
import java.util.Scanner;

public class Main {
    public static void main(String[] args) {
        canner scanner = new Scanner(System.in);
        while (scanner.hasNext()) {
            int n = scanner.nextInt();
            int k = scanner.nextInt();
            int m = scanner.nextInt();
            if (n == 0 && k == 0 && m == 0) {
                break;
            }
            int[] queue = new int[n + 1];
            for (int i = 1; i <= n; i++) {
                queue[i] = 1;
            }
            StringBuilder sb = new StringBuilder();
            int cnt = n, curCounterClock = 1, curClock = n;
            while (cnt > 0) {
                //顺时针数
                int ktmp = 0;
                while (true) {
                    if (queue[curCounterClock] == 1) {
                        ktmp++;
                        if (ktmp == k) {
                            break;
                        }
                    }
                    curCounterClock = (curCounterClock + 1) > n ? (curCounterClock + 1) % n : (curCounterClock + 1);
                }
                //逆时针数
                int mtmp = 0;
                while (true) {
                    if (queue[curClock] == 1) {
                        mtmp++;
                        if (mtmp == m) {
                            break;
                        }
                    }
                    curClock = curClock - 1 > 0 ? curClock - 1 : curClock - 1 + n;
                }

                //标记这两个位置的人出列
                queue[curCounterClock] = 0;
                queue[curClock] = 0;
                if (curCounterClock == curClock) {
                    cnt--;
                    sb.append(formatNumber(curCounterClock));
                } else {
                    sb.append(formatNumber(curCounterClock));
                    sb.append(formatNumber(curClock));
                    cnt -= 2;
                }
                if (cnt > 0) {
                    sb.append(",");
                }
            }
            System.out.println(sb.toString());

        }
    }

    public static String formatNumber(int n) {
        String nStr = String.valueOf(n);
        int remain = 3 - nStr.length();
        StringBuilder sb = new StringBuilder();
        for (int i = 0; i < remain; i++) {
            sb.append(" ");
        }
        sb.append(nStr);
        return sb.toString();
    }

}
```
