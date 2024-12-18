---
title: problem146 ID Codes
date: 2024-12-04 20:11:05
mathjax: true
categories:
- [数据结构与算法,uva]
tags:
- 算法
- uva
description: "UVA problem146.ID Codes <br>
- 计算字符串字典序的下一个字符串"
---

## 题目

It is 2084 and the year of Big Brother has finally arrived, albeit a century late. In order to exercise greater control over its citizens and thereby to counter a chronic breakdown in law and order, the Government decides on a radical measure — all citizens are to have a tiny microcomputer surgically implanted in their left wrists. This computer will contains all sorts of personal information as well as a transmitter which will allow people’s movements to be logged and monitored by a central computer.(A desirable side effect of this process is that it will shorten the dole queue for plastic surgeons.)

An essential component of each computer will be a unique identification code, consisting of up to 50 characters drawn from the 26 lower case letters. The set of characters for any given code is chosen somewhat haphazardly. The complicated way in which the code is imprinted into the chip makes it much easier for the manufacturer to produce codes which are rearrangements of other codes than to produce new codes with a different selection of letters. Thus, once a set of letters has been chosen all possible codes derivable from it are used before changing the set.

For example, suppose it is decided that a code will contain exactly 3 occurrences of ‘a’, 2 of ‘b’ and 1 of ‘c’, then three of the allowable 60 codes under these conditions are:
abaabc
abaacb
ababac

These three codes are listed from top to bottom in alphabetic order. Among all codes generated with this set of characters, these codes appear consecutively in this order.

Write a program to assist in the issuing of these identification codes. Your program will accept a sequence of no more than 50 lower case letters (which may contain repeated characters) and print the successor code if one exists or the message ‘No Successor’ if the given code is the last in the sequence for that set of characters.

### Input

Input will consist of a series of lines each containing a string representing a code. The entire file will be terminated by a line consisting of a single ‘#’.

### Output

Output will consist of one line for each code read containing the successor code or the words ‘No Successor’.

### SampleInput

abaacb
cbbaa
\#

### SampleOutput

ababac
No Successor

### 题意

给定一个小写字母的集合C，以及由集合中所有字母组成的一个字符串s，请求出由集合C中所有字母构成的字符串集合中，按字典序排序下s的下一个字符串s1.

## 思路

字典序限制了字符必须由给定的集合中的字母组成，再来考虑什么样的字符串比当前字符串大。假设字符串由a,b两种字母构成，当字符串S中出现ab的形式时，则S一定有下一个串，因为我们可以通过交换ab得到一个更大的串。我们可以把这个结论推广到多个字母组成的字符串中，从右往左数，如果右边存在比当前字母字典序更大的字母，那一定有下一个字符串。

第二个问题是哪一个是字典序的下一个字符串。如果当前位置index的字母存在右边的字母比它大，找到最小的一个，然后交换两者位置，index右边的再按字典序排序，得到的就是字典序下一个字符串。

### 问题

#### 实现上的问题

#### 性能上的问题

## 实现

```JAVA .{line-numbers}
//source code
import java.util.Arrays;
import java.util.Scanner;

public class Main {
    private static int[] charIndex = new int[26];

    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        while (true) {
            String text = scanner.nextLine();
            if ("#".equals(text)) {
                break;
            }

            //用来判断当前索引右边是否有字母比当前位置的大，有则找到最小的一个
            for (int i = 0; i < 26; i++) {
                charIndex[i] = 0;
            }

            int length = text.length();
            boolean hasSuccessor = false;
            label:
            for (int i = length - 1; i >= 0; i--) {
                int index = text.charAt(i) - 'a';
                char last = text.charAt(i);
                //从下一个字母找起
                for (int j = index + 1; j < 26; j++) {
                    if (charIndex[j] > 0) {
                        //找到了，先交换，再排序
                        int replaceIndex = 0;
                        char[] arys = new char[length - i - 1];
                        for (int k = 0; k < length - i - 1; k++) {
                            arys[k] = text.charAt(i + k + 1);
                            if (arys[k] - 'a' == j) {
                                replaceIndex = k;
                            }
                        }
                        char temp = arys[replaceIndex];
                        arys[replaceIndex] = last;
                        last = temp;
                        Arrays.sort(arys);

                        //拼接得到下一个字符串
                        StringBuilder sb = new StringBuilder();
                        sb.append(text, 0, i);
                        sb.append(last);
                        sb.append(arys);
                        System.out.println(sb.toString());
                        hasSuccessor = true;
                        break label;
                    }
                }
                charIndex[index] = 1;
            }
            if (!hasSuccessor) {
                System.out.println("No Successor");
            }
        }
    }
}
```
