---
title: problem161 Traffic Lights
date: 2025-04-11 09:03:43
mathjax: true
categories:
- [数据结构与算法,uva]
tags:
- 算法
- uva
description: "UVA problem161.Traffic Lights <br>
- 计算多个红绿灯同一时刻为绿灯的最早时刻"
---

## 题目

One way of achieving a smooth and economical drive to work is to ‘catch’ every traffic light, that is have every signal change to green as you approach it. One day you notice as you come over the brow of a hill that every traffic light you can see has just changed to green and that therefore your chances of catching every signal is slight. As you wait at a red light you begin to wonder how long it will be before all the lights again show green, not necessarily all turn green, merely all show green simultaneously,even if it is only for a second.

Write a program that will determine whether this event occurs within a reasonable time. Time is measured from the instant when they all turned green simultaneously, although the initial portion while they are all still green is excluded from the reckoning.

### Input

Input will consist of a series of scenarios. Data for each scenario will consist of a series of integers representing the cycle times of the traffic lights, possibly spread over many lines, with no line being longer than 100 characters. Each number represents the cycle time of a single signal. The cycle time is the time that traffic may move in one direction; note that the last 5 seconds of a green cycle is actually orange.

Thus the number 25 means a signal that (for a particular direction) will spend 20 seconds green,5 seconds orange and 25 seconds red. Cycle times will not be less than 10 seconds, nor more than 90 seconds. There will always be at least two signals in a scenario and never more than 100. Each scenario will be terminated by a zero (0).

The file will be terminated by a line consisting of three zeroes (0 0 0).

### Output

Output will consist of a series of lines, one for each scenario in the input. Each line will consist of the time in hours, minutes and seconds that it takes for all the signals to show green again after at least one of them changes to orange. Follow the format shown in the examples. Time is measured from the instant they all turn green simultaneously. If it takes more than five hours before they all show green simultaneously, the message ‘Signals fail to synchronise in 5 hours’ should be written instead.

### SampleInput

19 20 0
30
25 35 0
0 0 0

### SampleOutput

00:00:40
00:05:00

### 题意

有一系列的交通信号灯，灯按照绿、黄、红的颜色交替变化。每一个灯都有一个时间t，其中t-5秒是绿灯，5秒的黄灯，接着t秒的红灯，然后t-5秒绿灯依此循环。现在有一系列这样的灯，在t=0时刻都是刚刚绿灯，问至少有一个灯变黄之后所有的灯再次都是绿色的最近的时刻，如果5个小时内没有这样的时刻，输出:"Signals fail to synchronise in 5 hours".

## 思路

假设存在这样的时刻，那这一刻一定发生在某一个灯变绿的时刻，因为只有这样的时刻才能称为最近的时刻。从最近的时刻出发，将所有的灯变绿的时刻按时间轴排序，依此判断该时刻是否所有的灯都是绿色，如找到即为输出。

这里涉及到两个点：

1. 某一个绿灯之后最近的一个绿灯时刻是多少。
2. 如何判断该时刻所有的灯是否是绿灯。

对某一个信号灯，其时间为t，则信号灯绿色的时间区间为
$$[2nt,2nt+t-5),n=0,1,2,3...$$
假设当前绿灯的时刻为$t_1$,则对该信号灯而言下一个绿灯时刻满足
$$ 2nt>t_1 ，满足条件的n取最小值$$
即
$$n=\lceil t_1/2t \rceil $$

获取所有信号灯变为绿灯距当前时间最近的时刻，取最小值即为下一个绿灯时刻。

要判断所有灯是否是绿灯，取一个时间为t的信号灯，假设当前时间为$t_1$,则$t_1$满足
$$ \exists n,使得 2nt <= t_1 < 2nt+t-5 成立 $$
式子时，$t_1$时刻该信号灯是绿灯。
=>
$$ (t_1+5-t)/2t < n <= t_1/2t 且 n\in Z $$
=>
$$ \lceil (t_1+5-t)/2t \rceil < n <= \lfloor t_1/2t \rfloor $$
通过取整然后判断不等式两边的式子的值，判断n是否存在，从而得到时刻$t_1$该信号灯是否是绿灯。同样的方式判断所有的信号灯，即可判断该时刻$t_1$是否所有的灯都是绿灯。

### 问题

#### 实现上的问题

判断一个信号灯是否是绿灯时，需要取整，但此时分子有可能是负数，需要额外处理一下。

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
        while (true) {
            List<Integer> series = new ArrayList<>();
            int num = scanner.nextInt();
            if (num == 0 && scanner.nextInt() == 0 && scanner.nextInt() == 0) {
                break;
            }
            while (num != 0) {
                series.add(num);
                num = scanner.nextInt();
            }
            int time = 0;
            while (true) {
                //time表示当前时刻的绿灯，该方法用来查找离该时刻最近的一个绿灯
                time = findGreenLightTime(series, time);
                if (time > 5 * 60 * 60) {
                    System.out.println("Signals fail to synchronise in 5 hours");
                    break;
                }
                boolean res = allGreen(series, time);
                if (res) {
                    int hour = time / (60 * 60);
                    int minute = (time - hour * 3600) / 60;
                    int second = time - hour * 3600 - minute * 60;
                    String outStr = genTimeStr(hour, minute, second);
                    System.out.println(outStr);
                    break;
                }
            }


        }
    }

    private static String genTimeStr(int hour, int minute, int second) {
        StringBuilder sb = new StringBuilder();
        sb.append(formatInt(hour));
        sb.append(":");
        sb.append(formatInt(minute));
        sb.append(":");
        sb.append(formatInt(second));
        return sb.toString();
    }

    private static String formatInt(int num) {
        String numStr = String.valueOf(num);
        int blankNum = 2 - numStr.length();
        StringBuilder sb = new StringBuilder();
        for (int i = 0; i < blankNum; i++) {
            sb.append("0");
        }
        sb.append(numStr);
        return sb.toString();
    }

    /**
     * decide if at the time all trafic light is green
     *
     * @param series
     * @param time
     * @return
     */
    private static boolean allGreen(List<Integer> series, int time) {
        for (int i = 0; i < series.size(); i++) {
            int num = series.get(i);
            int up = Math.floorDiv(time, 2 * num);
            int remain = (time + 5 - num) % num;
            int down;
            if (time + 5 - num > 0) {
                down = remain == 0 ? (time + 5 - num) / (2 * num) : (time + 5 - num) / (2 * num) + 1;
            } else {
                down = (time + 5 - num) / (2 * num);
            }

            if ((remain == 0 && up > down)
                    || (remain != 0 && up >= down)) {
                continue;
            } else {
                return false;
            }
        }
        return true;
    }

    /**
     * find the next green light time from currentTime
     *
     * @param series
     * @param currentTime
     * @return
     */
    private static int findGreenLightTime(List<Integer> series, int currentTime) {
        if (currentTime == 0) {
            int min = Integer.MAX_VALUE;
            for (int i = 0; i < series.size(); i++) {
                if (series.get(i) < min) {
                    min = series.get(i);
                }
            }
            return min * 2;
        } else {
            int min = Integer.MAX_VALUE;
            for (int i = 0; i < series.size(); i++) {
                int interval = series.get(i);
                int tmp = currentTime % (2 * interval) == 0 ? currentTime / (2 * interval) :
                        currentTime / (2 * interval) + 1;
                int nextTime = tmp * 2 * interval;
                if (nextTime < min && nextTime > currentTime) {
                    min = nextTime;
                }
            }
            return min;
        }
    }
}
```
