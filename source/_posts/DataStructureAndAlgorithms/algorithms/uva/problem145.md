---
title: problem145 Gondwanaland Telecom
date: 2024-11-25 23:20:28
mathjax: true
categories:
- [数据结构与算法,uva]
tags:
- 算法
- uva
description: "UVA problem145.Gondwanaland Telecom <br>
- 计算话费"
---

## 题目

Gondwanaland Telecom makes charges for calls according to distance and time of day. The basis of the charging is contained in the following schedule, where the charging step is related to the distance:

| Charging Step <br>(distance)  |Day Rate <br> 8am to 6pm      | Evening Rate <br>6pm to 10pm  |Night Rate <br>10pm to 8am|
| :-------------:           | :-------------:          |             :-----:       |           :------:    |
|         A                 |         0.10             |         0.06              |          0.02         |
|         B                 |         0.25             |         0.15              |          0.05         |
|         C                 |         0.53             |         0.33              |          0.13         |
|         D                 |         0.87             |         0.47              |          0.17         |
|         E                 |         1.44             |         0.80              |          0.30         |

All charges are in dollars per minute of the call. Calls which straddle a rate boundary are charged according to the time spent in each section. Thus a call starting at 5:58 pm and terminating at 6:04 pm will be charged for 2 minutes at the day rate and for 4 minutes at the evening rate. Calls less than a minute are not recorded and no call may last more than 24 hours.

Write a program that reads call details and calculates the corresponding charges.

### Input

Input lines will consist of the charging step (upper case letter ‘A’..‘E’), the number called (a string of 7 digits and a hyphen in the approved format) and the start and end times of the call, all separated by exactly one blank. Times are recorded as hours and minutes in the 24 hour clock, separated by one blank and with two digits for each number. Input will be terminated by a line consisting of a single ‘#’.

### Output

Output will consist of the called number, the time in minutes the call spent in each of the charge categories, the charging step and the total cost in the format shown below.

### SampleInput

A 183-5724 17 58 18 04
\#

### SampleOutput

```markdown

      10   16   22   28   31    39
183-5724    2   4     0    A  0.44
```

### 题意

题目给出了一张话费计算表，根据类型和时间段不同，每分钟话费收费不同。现给出通话类型和通话起止时间，起止时间不超过24小时，要求出该次通话在每个时间段的时长，以及通话总费用。

## 思路

计算话费的关键是算出起止时间经过了多少个时间段，每个时间段时长是多少。以起始时间为基准，依次判断下一个时间点是什么，直到终止时间。由于通话时间不超过24小时，我们取两天的时间段即可。

```markdown

 0:00     8am         18pm      22pm   24:00(next day)    8am        18pm      22pm        24:00
 o---------o-----------o---------o------o------------------o----------o---------o------------o
```

 起始时间和结束时间一定落在这条轴上，且起始时间小于结束时间。

### 问题

#### 实现上的问题

起始时间是严格小于结束时间的，如果两者相等，说明通话时间有24小时。

#### 性能上的问题

## 实现

```JAVA .{line-numbers}
//source code
import java.math.BigDecimal;
import java.math.RoundingMode;
import java.util.ArrayList;
import java.util.List;
import java.util.Scanner;

public class Main {
    private static int[][] charges = {{10, 6, 2}, {25, 15, 5}, {53, 33, 13}, {87, 47, 17}, {144, 80, 30}};
    private static Time nightRate = new Time(8, 0, 2);
    private static Time dayRate = new Time(18, 0, 0);
    private static Time eveningRate = new Time(22, 0, 1);
    private static Time middleNight = new Time(24, 0, 2);
    private static Time nextnightRate = new Time(32, 0, 2);
    private static Time nextdayRate = new Time(42, 0, 0);
    private static Time nexteveningRate = new Time(46, 0, 1);
    private static Time nextmiddleNight = new Time(48, 0, 2);

    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        while (true) {
            String line = scanner.nextLine();
            if ("#".equals(line)) {
                break;
            }
            String[] items = line.split(" ");
            int type = 0;
            if (items[0].equals("A")) {
                type = 0;
            } else if (items[0].equals("B")) {
                type = 1;
            } else if (items[0].equals("C")) {
                type = 2;
            } else if (items[0].equals("D")) {
                type = 3;
            } else if (items[0].equals("E")) {
                type = 4;
            }

            Time a = new Time(Integer.valueOf(items[2]), Integer.valueOf(items[3]));
            Time b = new Time(Integer.valueOf(items[4]), Integer.valueOf(items[5]));

            int price = 0;
            int typeMinuteNight = 0, typeMinuteDay = 0, typeMinuteEven = 0;
            //返回一个数组，每相邻的两个节点间的时间表示一个时间段
            List<Time> sortTimes = theNextTime(a, b);
            for (int i = 1; i < sortTimes.size(); i++) {
                int diffMinute = diffminute(sortTimes.get(i - 1), sortTimes.get(i));
                if (sortTimes.get(i).type == 0) {
                    typeMinuteDay += diffMinute;
                } else if (sortTimes.get(i).type == 1) {
                    typeMinuteEven += diffMinute;
                } else if (sortTimes.get(i).type == 2) {
                    typeMinuteNight += diffMinute;
                }
                int priceTemp = diffMinute * charges[type][sortTimes.get(i).type];
                price += priceTemp;
            }

            String priceStr = new BigDecimal(price).divide(new BigDecimal(100))
                    .setScale(2, RoundingMode.HALF_UP).toPlainString();

            formatPrint(items[1], typeMinuteDay, typeMinuteEven, typeMinuteNight, items[0],
                    priceStr);
        }
    }

    private static void formatPrint(String item, int typeMinuteDay, int typeMinuteEven,
                                    int typeMinuteNight, String item1, String priceStr) {
        StringBuilder sb = new StringBuilder();
        int itemLen = 10 - item.length();
        for (int i = 0; i < itemLen; i++) {
            sb.append(" ");
        }
        sb.append(item);
        int minuteLen = 6 - String.valueOf(typeMinuteDay).length();
        for (int i = 0; i < minuteLen; i++) {
            sb.append(" ");
        }
        sb.append(typeMinuteDay);
        minuteLen = 6 - String.valueOf(typeMinuteEven).length();
        for (int i = 0; i < minuteLen; i++) {
            sb.append(" ");
        }
        sb.append(typeMinuteEven);
        minuteLen = 6 - String.valueOf(typeMinuteNight).length();
        for (int i = 0; i < minuteLen; i++) {
            sb.append(" ");
        }
        sb.append(typeMinuteNight);

        itemLen = 3 - item1.length();
        for (int i = 0; i < itemLen; i++) {
            sb.append(" ");
        }
        sb.append(item1);

        itemLen = 8 - priceStr.length();
        for (int i = 0; i < itemLen; i++) {
            sb.append(" ");
        }
        sb.append(priceStr);
        System.out.println(sb.toString());
    }

    public static List<Time> theNextTime(Time a, Time b) {
        Time b1 = b;
        if (!compare(a, b)) {
            b1 = new Time(b.hour + 24, b.minute);
        }
        List<Time> sortTimes = new ArrayList<>();
        sortTimes.add(a);
        if (compareAndInsert(a, b1, nightRate, sortTimes)) {
            return sortTimes;
        }
        if (compareAndInsert(a, b1, dayRate, sortTimes)) {
            return sortTimes;
        }
        if (compareAndInsert(a, b1, eveningRate, sortTimes)) {
            return sortTimes;
        }
        if (compareAndInsert(a, b1, middleNight, sortTimes)) {
            return sortTimes;
        }
        if (compareAndInsert(a, b1, nextnightRate, sortTimes)) {
            return sortTimes;
        }
        if (compareAndInsert(a, b1, nextdayRate, sortTimes)) {
            return sortTimes;
        }
        if (compareAndInsert(a, b1, nexteveningRate, sortTimes)) {
            return sortTimes;
        }
        if (compareAndInsert(a, b1, nextmiddleNight, sortTimes)) {
            return sortTimes;
        }
        return sortTimes;
    }

    public static boolean compareAndInsert(Time a, Time b, Time c, List<Time> sortTimes) {
        if (compare(a, c)) {
            if (compare(b, c)) {
                b.type = c.type;
                sortTimes.add(b);
                return true;
            } else {
                sortTimes.add(c);
                return false;
            }
        }
        return false;
    }

    public static boolean compare(Time a, Time b) {
        if (a.hour < b.hour || (a.hour == b.hour && a.minute < b.minute)) {
            return true;
        }
        return false;
    }

    public static int diffminute(Time a, Time b) {
        int minuteDiff = (b.hour - a.hour) * 60 + b.minute - a.minute;
        return minuteDiff;
    }

    public static class Time {
        int hour;
        int minute;

        int type;

        public Time(int hour, int minute) {
            this.hour = hour;
            this.minute = minute;
        }

        public Time(int hour, int minute, int type) {
            this.hour = hour;
            this.minute = minute;
            this.type = type;
        }
    }
}
```
