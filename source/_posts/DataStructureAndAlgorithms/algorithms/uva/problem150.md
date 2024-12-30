---
title: problem150 Double Time
date: 2024-12-30 09:03:33
mathjax: true
categories:
- [数据结构与算法,uva]
tags:
- 算法
- uva
description: "UVA problem150.Double Time <br>
- 两种日历下判断当天在另一种日历下的日期"
---

## 题目

In 45 BC a standard calendar was adopted by Julius Caesar — each year would have 365 days, and every fourth year have an extra day — the 29th of February. However this calendar was not quite accurate enough to track the true solar year, and it became noticeable that the onset of the seasons
was shifting steadily through the year. In 1582 Pope Gregory XIII ruled that a new style calendar should take effect. From then on, century years would only be leap years if they were divisible by 400.Furthermore the current year needed an adjustment to realign the calendar with the seasons. This new calendar, and the correction required, were adopted immediately by Roman Catholic countries, where the day following Thursday 4 October 1582 was Friday 15 October 1582. The British and Americans(among others) did not follow suit until 1752, when Wednesday 2 September was followed by Thursday 14 September. (Russia did not change until 1918, and Greece waited until 1923.) Thus there was a long period of time when history was recorded in two different styles.

Write a program that will read in a date, determine which style it is in, and then convert it to the other style.

### Input

Input will consist of a series of lines, each line containing a day and date (such as Friday 25 December 1992). Dates will be in the range 1 January 1600 to 31 December 2099, although converted dates may lie outside this range. Note that all names of days and months will be in the style shown, that is the first letter will be capitalised with the rest lower case. The file will be terminated by a line containing a single ‘#’.

### Output

Output will consist of a series of lines, one for each line of the input. Each line will consist of a date in the other style. Use the format and spacing shown in the example and described above. Note that there must be exactly one space between each pair of fields. To distinguish between the styles, dates in the old style must have an asterisk (‘*’) immediately after the day of the month (with no intervening space). Note that this will not apply to the input.

### SampleInput

Saturday 29 August 1992
Saturday 16 August 1992
Wednesday 19 December 1991
Monday 1 January 1900
\#

### SampleOutput

Saturday 16* August 1992
Saturday 29 August 1992
Wednesday 1 January 1992
Monday 20* December 1899

### 题意

历史上历法曾进行过变更，在1582年以前采用的旧日历记法一年有365天，每四年润一天，但这种日历和地球公转周期不完全一致，导致季节会逐渐偏移。1582年创建的公历规定了每四年润一天，但如果是整百的年份(年份能被100整除)则只有年份能被400整除才润一天。同时公里约定了1582年10月4日星期四的下一天是1582年10月15日星期五。

现在给定一个包含星期的日期，判断该日期是旧历还是新历，同时给出该天在另外一种日历下的日期。

## 思路

现判断给出的日期是哪种日历，通过计算该日期在新旧两种日历下的星期与给出的星期匹配，由于计算过程中会计算日期与新旧两种日历下指定日期(例如1582年10月4日，1582年10月15日)的天数差异，则可算出过去的天数的日期。

给定日期d,星期t，计算d与新旧历d1,d2(d1,d2是物理意义上的同一天，但两种日历下日期不一样)的天数差异day1，day2，再计算d在两种日历下的星期t1，t2，并与t比较。如果t与t1相等表明d是旧历，我们要求的是当天的另一种历法日期，即从d2开始，过了day1的日期。d表示从d2开始，过了day2天的日期，通过计算day1与day2的差值diff，然后把d往前或往后推diff天，得到d2过了day1天的日期。t与t2相等同理。

### 问题

#### 实现上的问题

计算某个日期往前或往后多少天时，由于这里的天数不会超过1个月，所以计算方法只考虑了前或后在一个月之内的情况，对于更远时间范围的情况则不适用。

#### 性能上的问题

## 实现

```JAVA .{line-numbers}
//source code
import java.util.HashMap;
import java.util.Map;
import java.util.Scanner;

public class Main {
    private static String[] weeks = {"", "Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday", "Sunday"};

    private static String[] months = {"", "January", "February", "March", "April", "May", "June"
            , "July", "August", "September", "October", "November", "December"};
    private static Map<String, Integer> monthMap = new HashMap<>();

    static {
        monthMap.put("January", 1);
        monthMap.put("February", 2);
        monthMap.put("March", 3);
        monthMap.put("April", 4);
        monthMap.put("May", 5);
        monthMap.put("June", 6);
        monthMap.put("July", 7);
        monthMap.put("August", 8);
        monthMap.put("September", 9);
        monthMap.put("October", 10);
        monthMap.put("November", 11);
        monthMap.put("December", 12);
    }

    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);

        //这里的两个日期是新旧历下不同日期，但是是物理意义上的同一天，即同一天的两种日历不同表示。
        MyDate oldDay = new MyDate();
        oldDay.year = 1582;
        oldDay.month = 10;
        oldDay.day = 5;

        MyDate newDay = new MyDate();
        newDay.year = 1582;
        newDay.month = 10;
        newDay.day = 15;

        while (true) {
            String dateStr = scanner.nextLine();
            if ("#".equals(dateStr)) {
                break;
            }
            String[] dateDetail = dateStr.split(" ");
            int monthNum = monthMap.get(dateDetail[2]);
            int year = Integer.valueOf(dateDetail[3]);
            int day = Integer.valueOf(dateDetail[1]);
            MyDate curDate = new MyDate();
            curDate.year = year;
            curDate.month = monthNum;
            curDate.day = day;
            //计算日期与给定日期的天数差异，用来计算星期
            int diffDay = dayDiff(oldDay, curDate, 1);
            int diffDayNew = dayDiff(newDay, curDate, 2);
            int week = (5 + diffDay) % 7 == 0 ? 7 : (5 + diffDay) % 7;
            int weekNew = (5 + diffDayNew) % 7 == 0 ? 7 : (5 + diffDayNew) % 7;
//            System.out.println("day:---" + diffDay + "---,---" + diffDayNew);
//            System.out.println("week:---" + week + "---,---" + weekNew);
            MyDate printDate = null;
            boolean old = true;
            if (weeks[week].equals(dateDetail[0])) {
                //convert to new cal
                //与指定的日子相差天数是固定的，计算该相差天数下的另一种日历表示
                int diff = diffDay - diffDayNew;
                old = false;
                if (diff > 0) {
                    printDate = getDaysAfter(curDate, diff, 2);
                } else {
                    printDate = getDaysBefore(curDate, -diff, 2);
                }
            } else if (weeks[weekNew].equals(dateDetail[0])) {
                int diff = diffDayNew - diffDay;
                if (diff > 0) {
                    printDate = getDaysAfter(curDate, diff, 1);
                } else {
                    printDate = getDaysBefore(curDate, -diff, 1);
                }
            }
            StringBuilder sb = new StringBuilder();
            sb.append(dateDetail[0]);
            sb.append(" ");
            sb.append(printDate.day);
            if (old) {
                sb.append("*");
            }
            sb.append(" ");
            sb.append(months[printDate.month]);
            sb.append(" ");
            sb.append(printDate.year);
            System.out.println(sb.toString());
        }

    }

    public static int dayDiff(MyDate baseDay, MyDate day, int type) {
        int totalDay = 0;
        if (type == 1) {
            //old cal
            for (int i = baseDay.year + 1; i < day.year; i++) {
                if (i % 4 == 0) {
                    totalDay += 366;
                } else {
                    totalDay += 365;
                }
            }
        } else {
            for (int i = baseDay.year + 1; i < day.year; i++) {
                if ((i % 100 != 0 && i % 4 == 0) || i % 400 == 0) {
                    totalDay += 366;
                } else {
                    totalDay += 365;
                }
            }
        }
        totalDay += getRemainDay(baseDay, type);
        totalDay += getRemainOfYear(baseDay, type);
        totalDay += getUsedOfYear(day, type);
        totalDay += day.day;
        return totalDay;
    }

    public static int getRemainDay(MyDate baseDay, int type) {
        int result = 0;
        int baseMonth = baseDay.month;
        if (baseMonth == 2) {
            if (type == 1) {
                if (baseDay.year % 4 == 0) {
                    result += 29 - baseDay.day;
                } else {
                    result += 28 - baseDay.day;
                }
            } else {
                if ((baseDay.year % 100 != 0 && baseDay.year % 4 == 0) || baseDay.year % 400 == 0) {
                    result += 29 - baseDay.day;
                } else {
                    result += 28 - baseDay.day;
                }
            }

        } else if (baseMonth == 1 || baseMonth == 3 || baseMonth == 5 ||
                baseMonth == 7 || baseMonth == 8 || baseMonth == 10 || baseMonth == 12) {
            result += 31 - baseDay.day;
        } else {
            result += 30 - baseDay.day;
        }
        return result;
    }

    public static int getRemainOfYear(MyDate baseDay, int type) {
        int result = 0;
        result = getMonthInterval(baseDay.year, baseDay.month + 1, 12, type);
        return result;
    }

    public static int getUsedOfYear(MyDate baseDay, int type) {
        int result = 0;
        result = getMonthInterval(baseDay.year, 1, baseDay.month - 1, type);
        return result;
    }

    public static int getMonthInterval(int year, int startMonth, int endMonth, int type) {
        int result = 0;
        for (int i = startMonth; i <= endMonth; i++) {
            if (i == 2) {
                if (type == 1) {
                    if (year % 4 == 0) {
                        result += 29;
                    } else {
                        result += 28;
                    }
                } else {
                    if ((year % 100 != 0 && year % 4 == 0) || year % 400 == 0) {
                        result += 29;
                    } else {
                        result += 28;
                    }
                }
            } else if (i == 1 || i == 3 || i == 5 ||
                    i == 7 || i == 8 || i == 10 || i == 12) {
                result += 31;
            } else {
                result += 30;
            }
        }
        return result;
    }

    public static MyDate getDaysBefore(MyDate date, int beforeDay, int type) {
        MyDate result = new MyDate();

        int day = date.day;
        if (day > beforeDay) {
            result.day = day - beforeDay;
            result.year = date.year;
            result.month = date.month;
            return result;
        } else {
            int month = date.month - 1;
            if (month <= 0) {
                month = 12;
                result.month = 12;
                result.year = date.year - 1;
            } else {
                result.year = date.year;
                result.month = month;
            }
            int monthDay = 0;
            if (month == 2) {
                if (type == 1) {
                    if (date.year % 4 == 0) {
                        monthDay = 29;
                    } else {
                        monthDay = 28;
                    }
                } else {
                    if ((date.year % 100 != 0 && date.year % 4 == 0) || date.year % 400 == 0) {
                        monthDay = 29;
                    } else {
                        monthDay = 28;
                    }
                }
            } else if (month == 1 || month == 3 || month == 5 ||
                    month == 7 || month == 8 || month == 10 || month == 12) {
                monthDay = 31;
            } else {
                monthDay = 30;
            }
            day = monthDay + day - beforeDay;
            result.day = day;
            return result;
        }
    }

    public static MyDate getDaysAfter(MyDate date, int afterDay, int type) {
        MyDate result = new MyDate();

        int day = date.day + afterDay;
        int month = date.month;
        int monthDay = 0;
        if (month == 2) {
            if (type == 1) {
                if (date.year % 4 == 0) {
                    monthDay = 29;
                } else {
                    monthDay = 28;
                }
            } else {
                if ((date.year % 100 != 0 && date.year % 4 == 0) || date.year % 400 == 0) {
                    monthDay = 29;
                } else {
                    monthDay = 28;
                }
            }
        } else if (month == 1 || month == 3 || month == 5 ||
                month == 7 || month == 8 || month == 10 || month == 12) {
            monthDay = 31;
        } else {
            monthDay = 30;
        }
        if (day > monthDay) {
            int remain = day - monthDay;
            int resMonth = date.month + 1;
            if (resMonth > 12) {
                result.year = date.year + 1;
                result.month = 1;
            } else {
                result.year = date.year;
                result.month = resMonth;
            }
            result.day = remain;
        } else {
            result.year = date.year;
            result.month = date.month;
            result.day = day;
        }
        return result;
    }


    public static class MyDate {
        int year;
        int month;
        int day;
    }
}
```
