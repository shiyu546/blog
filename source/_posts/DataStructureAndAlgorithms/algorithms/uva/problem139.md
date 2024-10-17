---
title: problem139 Telephone Tangles
date: 2024-10-16 19:43:13
mathjax: true
categories:
- [数据结构与算法,uva]
tags:
- 算法
- uva
description: "UVA 139.Telephone Tangles <br>
- 按格式输出"
---

## 题目

A large company wishes to monitor the cost of phone calls made by its personnel. To achieve this the PABX logs, for each call, the number called (a string of up to 15 digits) and the duration in minutes.Write a program to process this data and produce a report specifying each call and its cost, based on standard Telecom charges.

International (IDD) numbers start with two zeroes (00) followed by a country code (1–3 digits) followed by a subscriber’s number (4–10 digits). National (STD) calls start with one zero (0) followed by an area code (1–5 digits) followed by the subscriber’s number (4–7 digits). The price of a call is determined by its destination and its duration. Local calls start with any digit other than 0 and are free.

### Input

Input will be in two parts. The first part will be a table of IDD and STD codes, localities and prices as follows: 

`Code △Locality name$price in cents per minute`

where △ represents a space. Locality names are 25 characters or less. This section is terminated by a line containing 6 zeroes (000000).

The second part contains the log and will consist of a series of lines, one for each call, containing the number dialled and the duration. The file will be terminated a line containing a single #. The numbers will not necessarily be tabulated, although there will be at least one space between them. Telephone numbers will not be ambiguous.

### Output

Output will consist of the called number, the country or area called, the subscriber’s number, the duration, the cost per minute and the total cost of the call, as shown below. Local calls are costed at zero. If the number has an invalid code, list the area as ‘Unknown’ and the cost as -1.00.

Note: The first line of the Sample Output below in not a part of the output, but only to show the exact tabulation format it must follow.

### SampleInput

088925 Broadwood$81
03 Arrowtown$38
0061 Australia$140
000000
031526 22
0061853279 3
0889256287213 122
779760 1
002832769 5
\#

### SampleOutput

```plaintext

1             17             51  56     62     69
031526        Arrowtown    1526  22  0.38   8.36
0061853279    Australia  853279   3  1.40   4.20
0889256287213 Broadwood 6287213 122  0.81  98.82
779760        Local      779760   1  0.00   0.00
002832769     Unknown             5        -1.00
```

### 题意

文中共定义了三种号码IDD、STD和Local，每种号码有特定的格式，IDD和STD的号码中含有一个区域码以及subscriber number，区域码对应的是一个区域的编号。输入给定了两部分内容，第一部分是(区域号码，区域名称，单价)三元组的集合，当IDD和STD的区域码在这个集合中时，即可根据号码的拨打时长算出拨打费用；第二部分给出了一些拨打的号码以及拨打时长，请按指定的格式输出每个号码的拨打信息.
`拨打号码 区域名称 订阅者号码 拨打时长 单价 总价`

## 思路

先判断拨打号码的类型，如果是Local则单独计算，如果是IDD和STD，判断号码的区域码是否在区域码集合中，在则计算，不在则号码Unknown。按照题意号码不会有歧义，意思是同一个号码不会属于多个区域，所以在号码合法且满足某一个区域的前提下号码只会属于一个区域。

由于IDD和STD到区域码的长度(带上前面的00或0)分别为3-5和2-6，即区域的长度范围为2-6，只需要遍历号码前缀2-6位判断是否在区域码集合中，并判断号码是否合理。

### 问题

#### 实现上的问题

#### 性能上的问题

## 实现

```JAVA .{line-numbers}
//source code
import java.math.BigDecimal;
import java.math.RoundingMode;
import java.util.HashMap;
import java.util.Map;
import java.util.Scanner;

public class Main {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);

        Map<String, Country> countryMap = new HashMap<>();
        while (true) {
            String location = scanner.nextLine();
            if ("000000".equals(location)) {
                break;
            }
            int blankIndex = location.indexOf(" ");
            String code = location.substring(0, blankIndex);
            location = location.substring(blankIndex + 1);
            int money = location.indexOf("$");
            String countryName = location.substring(0, money);
            int price = Integer.valueOf(location.substring(money + 1));
            countryMap.put(code, new Country(code, countryName, price));
        }
        while (true) {
            String number = scanner.next();
            if ("#".equals(number)) {
                break;
            }
            int minute = scanner.nextInt();
            if (number.charAt(0) != '0') {
                //前缀不为0，表示是Local
                System.out.println(formatPrint(number, minute, 2, null));
            } else {
                boolean legeal = false;
                //遍历号码前缀，判断是否在区域码集合中
                for (int i = 2; i <= 6; i++) {
                    if (number.length() < i) {
                        break;
                    }
                    String prefix = number.substring(0, i);
                    if (countryMap.containsKey(prefix)) {
                        //判断号码是否满足规则
                        int subLength = number.length() - prefix.length();
                        if ((prefix.startsWith("00") && subLength >= 4 && subLength <= 10)
                                || (prefix.startsWith("0") && subLength >= 4 && subLength <= 7)) {
                            //legeal
                            System.out.println(formatPrint(number, minute, 1, countryMap.get(prefix)));
                            legeal = true;
                            break;
                        }
                    }

                }
                if (!legeal) {
                    System.out.println(formatPrint(number, minute, 3, null));
                }
            }

        }
    }

    public static String formatPrint(String number, int minute, int type, Country country) {
        StringBuilder sb = new StringBuilder();
        if (type == 2) {
            //local
            fillBlank(number, sb, 16, 1);
            int len = 35 - number.length();
            fillBlank("Local", sb, len, 1);
            sb.append(number);
            fillBlank(String.valueOf(minute), sb, 5, 2);
            fillBlank("0.00", sb, 6, 2);
            fillBlank("0.00", sb, 7, 2);
        } else if (type == 1) {
            //IDD and STD
            fillBlank(number, sb, 16, 1);
            int len = 35 - (number.length() - country.code.length());
            fillBlank(country.countryName, sb, len, 1);
            sb.append(number.substring(country.code.length()));
            fillBlank(String.valueOf(minute), sb, 5, 2);
            BigDecimal formatPrice = new BigDecimal(country.price)
                    .divide(new BigDecimal(100), 2, RoundingMode.HALF_UP);
            fillBlank(formatPrice.toPlainString(), sb, 6, 2);
            BigDecimal total = new BigDecimal(country.price * minute)
                    .divide(new BigDecimal(100), 2, RoundingMode.HALF_UP);
            fillBlank(total.toPlainString(), sb, 7, 2);

        } else {
            //unknown
            fillBlank(number, sb, 16, 1);
            int len = 40 - String.valueOf(minute).length();
            fillBlank("Unknown", sb, len, 1);
            sb.append(String.valueOf(minute));
            fillBlank("-1.00", sb, 13, 2);
        }
        return sb.toString();
    }

    private static void fillBlank(String number, StringBuilder sb, int length, int type) {
        int blank = length - number.length();
        if (type == 1) {
            //fill right
            sb.append(number);
        }
        for (int i = 0; i < blank; i++) {
            sb.append(" ");
        }
        if (type == 2) {
            //fill left
            sb.append(number);
        }
    }

    public static class Country {
        public Country(String code, String countryName, int price) {
            this.code = code;
            this.countryName = countryName;
            this.price = price;
        }
        private String code;
        private String countryName;
        private int price;
    }
}
```
