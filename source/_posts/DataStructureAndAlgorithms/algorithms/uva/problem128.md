---
title: problem128. Software CRC
date: 2024-09-03 09:14:47
mathjax: true
categories:
- [数据结构与算法,uva]
tags:
- 算法
- uva
description: "UVA 128.Software CRC <br>
- 以二进制形式计算给定数字的余数"
---

## 题目

You work for a company which uses lots of personal computers. Your boss, Dr Penny Pincher, has wanted to link the computers together for some time but has been unwilling to spend any money on the Ethernet boards you have recommended. You, unwittingly, have pointed out that each of the
PCs has come from the vendor with an asynchronous serial port at no extra cost. Dr Pincher, of course, recognizes her opportunity and assigns you the task of writing the software necessary to allow communication between PCs.

You’ve read a bit about communications and know that every transmission is subject to error and that the typical solution to this problem is to append some error checking information to the end of each message. This information allows the receiving program to detect when a transmission error has occurred (in most cases). So, off you go to the library, borrow the biggest book on communications you can find and spend your weekend (unpaid overtime) reading about error checking.

Finally you decide that CRC (cyclic redundancy check) is the best error checking for your situation and write a note to Dr Pincher detailing the proposed error checking mechanism noted below.

> **CRC Generation**
The message to be transmitted is viewed as a long positive binary number. The first byte of the message is treated as the most significant byte of the binary number. The second byte is the next most significant, etc. This binary number will be called “m” (for message).Instead of transmitting “m” you will transmit a message, “m2”, consisting of “m” followed by a two-byte CRC value.
The CRC value is chosen so that “m2” when divided by a certain 16-bit value “g” leaves a remainder of 0. This makes it easy for the receiving program to determine whether the message has been corrupted by transmission errors. It simply divides any message received by “g”. If the remainder of the division is zero, it is assumed that no error has occurred.
You notice that most of the suggested values of “g” in the book are odd, but don’t see any other similarities, so you select the value 34943 for “g” (the generator value).You are to devise an algorithm for calculating the CRC value corresponding to any message that might be sent. To test this algorithm you will write a program which reads lines from standard input and writes to standard output.

### Input

Each input line will contain no more than 1024 ASCII characters (each line being all characters up to,but not including the end of line character) as input.

The input is terminated by a line that contains a ‘#’ in column 1.

### Output

For each input line calculates the CRC value for the message contained in the line, and writes the numeric value of the CRC bytes (in hexadecimal notation) on an output line.

Note that each CRC printed should be in the range 0 to 34942 (decimal).

### SampleInput

this is a test

A
\#

### SampleOutput

77 FD
00 00
0C 86

### 题意

题目给出了一个二进制串作为被除数(将其当作无符号数)，令该串为m，给定余数34943(设为t),通过在m之后拼接2字节的二进制串，使得拼接后的二进制串$m_2$能被34943整除，现求拼接的2字节串的值。

## 思路

从除法和求余出发，二进制串m拼接2字节的数字$m_1$表示为:
 $$ m_2=m*2^{16}+m_1 \tag{1}$$
根据题意，有$m_2$被t整除，得：
 $$ m_2\%t=0 \tag{2}$$
代入(1)式，得
$$ m*2^{16}+m_1=t{k_1} \tag{3}$$
假设$m*2^{16}$除以t得到商$k_2$和余数$r_1$,
$$ m*2^{16}=t{k_2}+r_1 \tag{4}$$
代入(3)式，得：
$$ m_1+r_1=t({k_1-k_2}) $$

由题意，$m_1$是小于t得数，$r_1$是余数也小于t,所以$m_1+r_1=t$,所以$m_1=t-r_1$，求$m_1$等价于求出$r_1$得值。

### 问题

#### 实现上的问题

主要是二进制除法得问题，需要采用除法规则求出余数。除法从最高位开始，拿到大于除数的数去除除数，得到商的一位以及余数，重复该过程直到不能除为止。

#### 性能上的问题

最开始的想法是取出串的最高4字节去除除数，余数再放回串的高位，然后重复取出4字节，直接串结束或无法再除。但这种方法有性能上的问题，主要是拼接动作和取出强转比较耗时。通过了解取高位除然后拼接并不需要取出和放回串的操作，通过遍历每一字节拿到当前被除数的值并计算商和余数。

第二个问题是输入要采用输入缓冲区buffer的形式，提高性能。

## 实现

```JAVA {.line-nubmers}
//file
import java.io.*;
import java.util.Scanner;

public class Main {
    public static char[] hexArray = {'0', '1', '2', '3', '4', '5', '6', '7', '8', '9', 'A', 'B', 'C', 'D', 'E', 'F'};

    public static void main(String[] args) throws IOException {
        int divideNum = 34943;
        BufferedReader reader = new BufferedReader(new InputStreamReader(System.in));
        String line = reader.readLine();
        while (!"#".equals(line)) {
            char[] lineBytes = line.toCharArray();

            long remainInt = 0;
            //遍历字节数组，本质上就是取高位去除除数，得到余数再和剩余被除数高位相加得到剩余的高位被除数，也就是除法的运算步骤
            for (int i = 0; i < lineBytes.length; i++) {
                remainInt = ((remainInt << 8) + lineBytes[i]) % divideNum;
            }
            remainInt = (remainInt << 16) % divideNum;

            if (remainInt == 0) {
                System.out.println("00 00");
            } else {
                //余数只是r1，我们要计算的是m1
                int offset = divideNum - (int) remainInt;
                StringBuilder sb = new StringBuilder();
                for (int i = 0; i < 4; i++) {
                    int indexH = offset >> (12 - i * 4) & 0xF;
                    sb.append(hexArray[indexH]);
                    if (i == 1) {
                        sb.append(" ");
                    }
                }
                System.out.println(sb.toString());
            }
            line = reader.readLine();
        }
    }
}
```
