---
title: problem144 Student Grants
date: 2024-11-19 11:18:21
mathjax: true
categories:
- [数据结构与算法,uva]
tags:
- 算法
- uva
description: "UVA problem144.Student Grants <br>
- 按规律输出人员序号"
---

## 题目

The Government of Impecunia has decided to discourage tertiary students by making the payments of tertiary grants a long and time-consuming process. Each student is issued a student ID card which has a magnetically encoded strip on the back which records the payment of the student grant. This is initially set to zero. The grant has been set at $40 per year and is paid to the student on the working day nearest to his birthday. (Impecunian society is still somewhat medieval and only males continue with tertiary education.) Thus on any given working day up to 25 students will appear at the nearest office of the Department of Student Subsidies to collect their grant.

The grant is paid by an Automatic Teller Machine which is driven by a reprogrammed 80851/2 chip originally designed to run the state slot machine. The ATM was built in the State Workshops and is designed to be difficult to rob. It consists of an interior vault where it holds a large stock of $1 coins and an output store from which these coins are dispensed. To limit possible losses it will only move coins from the vault to the output store when that is empty. When the machine is switched on in the morning, with an empty output store, it immediately moves 1 coin into the output store. When that has been dispensed it will then move 2 coins, then 3, and so on until it reaches some preset limit k. It then recycles back to 1, then 2 and so on.

The students form a queue at this machine and, in turn, each student inserts his card. The machine dispenses what it has in its output store and updates the amount paid to that student by writing the new total on the card. If the student has not received his full grant, he removes his card and rejoins the queue at the end. If the amount in the store plus what the student has already received comes to more than $40, the machine only pays out enough to make the total up to $40. Since this fact is recorded on the card, it is pointless for the student to continue queuing and he leaves. The amount remaining in the store is then available for the next student.

Write a program that will read in values of N (the number of students, $1 \leq N \leq 25$) and k (the limit for that machine, $1 \leq k \leq 40$) and calculate the order in which the students leave the queue.

### Input

Input will consist of a series of lines each containing a value for N and k as integers. The list will be terminated by two zeroes (0 0).

### Output

Output will consist of a line for each line of input and will contain the list of students in the order in which they leave the queue. Students are ordered according to their position in the queue at the start of the day. All numbers must be right justified in a field of width 3.

### SampleInput

5 3
0 0

### SampleOutput

1 3 5 2 4

### 题意

题目比较长，大致意思如下：每个学生每年会有一笔奖金，为40元，在学生过生日的头一个工作日发放。发放只能通过一台机器，该机器会设置一个值K，每一次可以发放1，2，3...直至K块钱，到达K块之后又从1开始发放，之后递增到K，又从1开始循环。所有当天可以领钱的学生在机器前排队，领到机器当前发放的钱后，如果达到40，就离开；没有则回到队尾继续排队；如果机器当前的钱发给当前学生之后超过了40，则只发放到40，剩下的钱发放给下一个排队的学生。

现在要输出学生离开队伍的顺序。

## 思路

按照机器发钱的思路，每一次机器的钱都在1到K循环，将当前的钱发下去，有多的就发给下一个，发完了就取下一次的钱发给下一个学生，直至所有的学生都拿到了40.

### 问题

#### 实现上的问题

#### 性能上的问题

## 实现

```JAVA .{line-numbers}
//source file
import java.util.ArrayList;
import java.util.List;
import java.util.Scanner;

public class Main {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        while (true) {
            int number = scanner.nextInt();
            int k = scanner.nextInt();
            if (number == 0 && k == 0) {
                break;
            }
            int[] numbers = new int[number];
            for (int i = 0; i < number; i++) {
                numbers[i] = 40;
            }
            List<Integer> leaveQueue = new ArrayList<>();
            int leaveNum = 0;
            int index = 0, curStore = 1, curStorTemp = 0;
            while (leaveNum < number) {
                curStorTemp = curStore;
                //curStorTemp表示当前机器的钱，直到发放完毕就发放机器下一批钱
                while (curStorTemp > 0 && leaveNum < number) {
                    if (numbers[index] == 0) {
                        index = (index + 1) % number;
                        continue;
                    }
                    if (numbers[index] > curStorTemp) {
                        numbers[index] -= curStorTemp;
                        curStorTemp = 0;
                    } else if (numbers[index] == curStorTemp) {
                        curStorTemp = 0;
                        numbers[index] = 0;
                        leaveNum++;
                        leaveQueue.add(index + 1);
                    } else {
                        curStorTemp -= numbers[index];
                        numbers[index] = 0;
                        leaveNum++;
                        leaveQueue.add(index + 1);
                    }
                    index = (index + 1) % number;
                }
                if (curStore + 1 > k) {
                    curStore = 1;
                } else {
                    curStore++;
                }

            }
            formatPrint(leaveQueue);
        }
    }

    private static void formatPrint(List<Integer> leaveQueue) {
        StringBuilder sb = new StringBuilder();
        for (int i = 0; i < leaveQueue.size(); i++) {
            String num = String.valueOf(leaveQueue.get(i));
            int fill = 3 - num.length();
            for (int j = 0; j < fill; j++) {
                sb.append(" ");
            }
            sb.append(num);
        }
        System.out.println(sb.toString());
    }
}
```
