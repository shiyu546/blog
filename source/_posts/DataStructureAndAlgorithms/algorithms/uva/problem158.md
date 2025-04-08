---
title: problem158 Calendar
date: 2025-04-03 14:27:09
mathjax: true
categories:
- [数据结构与算法,uva]
tags:
- 算法
- uva
description: "UVA problem158.Calendar <br>
- 按今天日期显示日历中标记的年纪念日"
---

## 题目

Most of us have a calendar on which we scribble details of important events in our lives — visits to the dentist, the Regent 24 hour book sale, Programming Contests and so on. However there are also the fixed dates: partner’s birthdays, wedding anniversaries and the like; and we also need to keep track of these. Typically we need to be reminded of when these important dates are approaching — the more important the event, the further in advance we wish to have our memories jogged.

Write a program that will provide such a service. The input will specify the year for which the calendar is relevant (in the range 1901 to 1999). Bear in mind that, within the range specified, all years that are divisible by 4 are leap years and hence have an extra day (February 29th) added. The output will specify “today’s” date, a list of forthcoming events and an indication of their relative importance.

### Input

The first line of input will contain an integer representing the year (in the range 1901 to 1999). This will be followed by a series of lines representing anniversaries or days for which the service is requested.

An anniversary line will consist of the letter ‘A’; three integer numbers (D;M; P) representing the date, the month and the importance of the event; and a string describing the event, all separated by one or more spaces. P will be a number between 1 and 7 (both inclusive) and represents the number of days before the event that the reminder service should start. The string describing the event will always be present and will start at the first non-blank character after the priority.

A date line will consist of the letter ‘D’ and the date and month as above. All anniversary lines will precede any date lines. No line will be longer than 255 characters in total.The file will be terminated by a line consisting of a single #.

### Output

Output will consist of a series of blocks of lines, one for each date line in the input. Each block will consist of the requested date followed by the list of events for that day and as many following days as necessary.

The output should specify the date of the event (D and M), right justified in fields of width 3,and the relative importance of the event. Events that happen today should be flagged as shown below,events that happen tomorrow should have P stars, events that happen the day after tomorrow should
have P - 1 stars, and so on. If several events are scheduled for the same day, order them by relative importance (number of stars).

If there is still a conflict, order them by their appearance in the input stream. Follow the format used in the example below.

Leave 1 blank line between blocks.

### SampleInput

1993
A 23 12 5 Partner's birthday
A 25 12 7 Christmas
A 20 12 1 Unspecified Anniversary
D 20 12
\#

### SampleOutput

Today is: 20 12
20 12 *TODAY* Unspecified Anniversary
23 12 \*\*\* Partner's birthday
25 12 \*\*\* Christmas

### 题意

日历中会有一些周年纪念日，每个周年纪念日包含四个部分：日，月，P值，事件。现在给定一个日期，如果该天是周年纪念日，按指定格式输出。如果明天是周年纪念日，按格式<日,月,P个星星，事件>输出，后天按格式<日,月,(P-1)个星星，事件>输出，每过一天，如果该天是周年纪念日，输出的格式中星星数基于P值多减1.

如果周年纪念日的星星数大于0个，则输出。同一个日期的周年纪念日按照时间先后顺序输出，当同一天有多个纪念日时，按照星星的数量由多到少排序，如果星星数也相同，按照在输入中出现的顺序排序。

## 思路

由于P值范围为1-7，计算指定日期之后的7天，计算每一天是否是周年纪念日，然后排序。

### 问题

#### 实现上的问题

#### 性能上的问题

## 实现

```JAVA .{line-numbers}
//source code
import java.util.ArrayList;
import java.util.Collections;
import java.util.List;
import java.util.Scanner;

public class Main {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        int year = scanner.nextInt();

        //用来存放周年纪念日列表
        List<AnniversaryEvent> events = new ArrayList<>();
        boolean first = true;
        while (true) {
            String type = scanner.next();

            if (type.equals("#")) {
                break;
            }
            int day = scanner.nextInt();
            int month = scanner.nextInt();

            if (type.equals("A")) {
                int p = scanner.nextInt();
                String event = scanner.nextLine();
                event = event.substring(event.indexOf(event.trim()));
                AnniversaryEvent ann = new AnniversaryEvent();
                ann.day = day;
                ann.month = month;
                ann.point = p;
                ann.event = event;
                events.add(ann);
            } else if (type.equals("D")) {
                if (first) {
                    first = false;
                } else {
                    System.out.printf("\n");
                }
                System.out.printf("Today is:%3d%3d\n", day, month);
                //starEvents用来存放星星数大于0的周年纪念日
                List<AnniversaryEvent> starEvents = new ArrayList<>();
                for (int i = 0; i < events.size(); i++) {
                    AnniversaryEvent event = events.get(i);
                    MyDate myDate = diff(year, month, day, event);
                    if (myDate != null) {
                        AnniversaryEvent starEv = new AnniversaryEvent();
                        starEv.day = event.day;
                        starEv.month = event.month;
                        starEv.point = myDate.stars;
                        starEv.event = event.event;
                        starEv.intervalDay = myDate.intervalDay;
                        starEvents.add(starEv);
                    }
                }

                //排序，现按照日期排序，日期相同按照星星数排序，都相同按list输入顺序排序
                Collections.sort(starEvents, (starEv1, starEv2) -> {
                    if (starEv1.intervalDay < starEv2.intervalDay) {
                        return -1;
                    } else if (starEv1.intervalDay > starEv2.intervalDay) {
                        return 1;
                    } else {
                        if (starEv1.point > starEv2.point) {
                            return -1;
                        } else if (starEv1.point == starEv2.point) {
                            return 0;
                        } else {
                            return 1;
                        }
                    }
                });
                for (int i = 0; i < starEvents.size(); i++) {
                    AnniversaryEvent st = starEvents.get(i);
                    System.out.printf("%3d%3d %-7s %s\n", st.day, st.month, getStars(st.point), st.event);
                }
            }

        }

    }

    //计算星星数
    public static String getStars(int point) {
        StringBuilder sb = new StringBuilder();
        if (point == 8) {
            return "*TODAY*";
        }
        for (int i = 1; i <= point; i++) {
            sb.append("*");
        }
        return sb.toString();
    }

    //判断给定日期之后的一周内是否有日期和指定的周年纪念日匹配，并计算匹配的星星数
    public static MyDate diff(int year, int month, int day, AnniversaryEvent event) {
        int annMonth = event.month;
        int annDay = event.day;
        MyDate curDate = new MyDate();
        curDate.year = year;
        curDate.month = month;
        curDate.day = day;
        if (month == annMonth && annDay == day) {
            curDate.intervalDay = 0;
            curDate.stars = 8;
            return curDate;
        }
        for (int i = 0; i < 7; i++) {
            curDate = getNextDay(curDate);
            if (curDate.month == annMonth && curDate.day == annDay) {
                int stars = event.point - i;
                if (stars > 0) {
                    curDate.intervalDay = i + 1;
                    curDate.stars = stars;
                    return curDate;
                }

            }
        }
        return null;
    }

    public static MyDate getNextDay(MyDate myDate) {
        MyDate result = new MyDate();
        int year = myDate.year;
        int month = myDate.month;
        int day = myDate.day;
        if (month == 1 || month == 3 || month == 5 || month == 7 || month == 8 || month == 10 || month == 12) {
            if (day <= 30) {
                day++;
            } else {
                month = month + 1;
                day = 1;
                if (month > 12) {
                    month = 1;
                    year++;
                }
            }

        } else if (month == 4 || month == 6 || month == 9 || month == 11) {
            if (day <= 29) {
                day++;
            } else {
                month = month + 1;
                day = 1;
            }
        } else {
            if (year % 4 == 0) {
                if (day <= 28) {
                    day++;
                } else {
                    month = month + 1;
                    day = 1;
                }
            } else {
                if (day <= 27) {
                    day++;
                } else {
                    month = month + 1;
                    day = 1;
                }
            }
        }
        result.year = year;
        result.month = month;
        result.day = day;
        return result;
    }

    public static class AnniversaryEvent {
        public int day;
        public int month;
        public int point;
        public String event;

        public int intervalDay;
    }

    public static class MyDate {
        public int year;
        public int month;
        public int day;
        public int intervalDay;

        public int stars;
    }
}

```
