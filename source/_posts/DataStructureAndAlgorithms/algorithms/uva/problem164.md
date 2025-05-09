---
title: problem164 String Computer
date: 2025-05-06 16:30:04
mathjax: true
categories:
- [数据结构与算法,uva]
tags:
- 算法
- uva
description: "UVA problem164.String Computer <br>
- 寻找将A字符串变为B字符串的最小操作步骤"
---

## 题目

Extel have just brought out their newest computer, a string processing computer dubbed the X9091.It is hoped that it will have some value in cryptography and related fields. (It is rumoured that the Taiwanese are working on a clone that will correct Stage 1 essays, but we will ignore such vapourware).This computer will accept input strings and produce output strings from them, depending on the programs loaded into them at the time. The chip is the ultimate in RISC technology — it has only three transformation instructions:

* Delete a character at a particular position.
* Insert a character at a particular position.
* Change the character at a particular position to a different character.

Programs for this machine are written in a form of machine code where each instruction has the format ‘ZXdd’ — Z represents the code for the instruction (D, I or C), X is a character and dd represents a two digit number. A program is terminated by a special halt instruction consisting of the letter ‘E’.Note that each instruction works on the string in memory at the time the instruction is executed.

To see how this all works consider the following example. It is desired to transform the string ‘abcde’ to the string ‘bcgfe’. This could be achieved by a series of Change commands, but is not minimal. The following program is better.

```plaintext

     abcde
Da01 bcde % note the ‘a’ is necessary because it is checked by the hardware
Cg03 bcge
If04 bcgfe
E    bcgfe % Terminates the program
```

Write a program that will read in two strings (the input string and the target string) and will produce a minimal X9091 program necessary to transform the input string into the target string. Since there may be multiple solutions, only one should be produced. Any solution that satisfies these criteria will be accepted.

### Input

Input will consist of a series of lines, each line containing two strings separated by exactly one space.The strings will consist of no more than 20 lower case characters. The file will be terminated by a line consisting of a single ‘#’.

### Output

Output will consist of a series of lines, one for each line of the input. Each will consist of a program in X9091 language.

### SampleInput

abcde bcgfe
\#

### SampleOutput

Da01Cg03If04E

### 题意

有一个机器能执行三种指令，D,I和C操作，分别代表删除指定字符，插入字符，改变字符。现在给定一对字符串，要求将字符串a变成字符串b，最少需要多少条指令，并输出指令序列。

## 思路

**[原文](https://www.cnblogs.com/scau20110726/archive/2013/02/25/2932155.html "标题")**

要想找到最少的指令序列，需要找到两个字符串的最长公共子序列，保持该子序列不变，操作剩余字符得到指令序列。这种问题称为LCS，一般采用动态规划方法，但找到最长序列之后，如何执行指令序列仍然是个问题，所以从动态规划的角度出发，将dp定义为最长操作距离，并记录上一步是从哪里操作过来的，通过回溯来得到指令序列。

具体思路如下：

先定义dp[i][j]，dp[i][j]表示a串前i个字符和b串前j个字符的最短编辑距离,dp[i][j]可以由下列三个步骤中的一个得到：
1. dp[i][j]=dp[i-1][j]+1，dp[i-1][j]表示把a[0:i-1]转化为b[0:j]需要多少步骤，对于a[0:i]字符串，删除最后一个字符得到a[0:i-1],且通过dp[i-1][j]步转换为b[j],所以dp[i][j]=dp[i-1][j]+1。
2. dp[i][j]=dp[i][j-1]+1，dp[i][j-1]表示把a[0:i]转化为b[0:j-1]需要多少步骤，通过在a[0:i]后添加b[j]得到a[0:i]到b[0:j]的距离，所以dp[i][j]=dp[i][j-1]+1。
3. dp[i][j]=dp[i-1][j-1]+(a[i]==b[j]?0:1)，如果a[i]==b[j]，dp[i][j]转化为dp[i-1][j-1],不等，则通过改变a[i]为b[j],此时编辑距离+1.

最小编辑距离dp[i][j]=min{1,2,3}
最后来解决一下如何保存路径的问题,p[i][j]={1,2,3}，表示得到dp[i][j]的时候是用了第几个策略
p[i][j]=1，说明用了策略1，是删除了a[i]，所以输出删除对应的语句，注意此时对应的操作位置是j+1，并接着去到p[i-1][j]
p[i][j]=2，说明用了策略2，是插入了b[j]，所以输出插入对应的语句，注意此时对应的操作位置是j，并接着去到p[i][j-1]
p[i][j]=0，说明用了策略3，但没有更换，不用输出，并接着去到p[i-1][j-1]
p[i][j]=3，说明用了策略3，是更换了a[i]，所以输出更换对应的语句，注意此时对应的位置是j，并接着去到p[i-1][j-1]
令imax=len(a),jmax=len(b),从p[imax][jmax]开始，当`(imax==0 && jmax==0)`表示迭代到了尽头，表示a串和b串都已经递归处理完。

### 问题

#### 实现上的问题

#### 性能上的问题

## 实现

```JAVA .{line-numbers}
//source code
import java.util.Scanner;

public class Main {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        while (true) {
            String inputLine = scanner.nextLine();
            if (inputLine.equals("#")) {
                break;
            }
            String[] strs = inputLine.split(" ");
            String stra = " " + strs[0];
            String strb = " " + strs[1];
            int lena = stra.length() - 1;
            int lenb = strb.length() - 1;
            getCommonStrs(stra, strb, lena, lenb);
        }

    }

    private static void getCommonStrs(String stra, String strb, int lena, int lenb) {
        int[][] dp = new int[stra.length()][strb.length()];
        int[][] p = new int[stra.length()][strb.length()];

        for (int i = 0; i < stra.length(); i++) {
            dp[i][0] = i;
            p[i][0] = 1;
        }

        for (int i = 0; i < strb.length(); i++) {
            dp[0][i] = i;
            p[0][i] = 2;
        }

        for (int i = 1; i <= lena; i++) {
            for (int j = 1; j <= lenb; j++) {
                int[] s = new int[3];
                s[0] = dp[i - 1][j] + 1;
                s[1] = dp[i][j - 1] + 1;
                s[2] = dp[i - 1][j - 1] + (stra.charAt(i) == strb.charAt(j) ? 0 : 1);
                p[i][j] = min(s);
                dp[i][j] = s[p[i][j] - 1];
                if (p[i][j] == 3 && stra.charAt(i) == strb.charAt(j)) {
                    p[i][j] = 0;
                }
            }
        }

        print(p, stra, strb, lena, lenb);
    }

    private static int min(int[] s) {
        int index = 0, num = s[0];
        for (int k = 1; k < s.length; k++) {
            if (s[k] < num) {
                num = s[k];
                index = k;
            }
        }
        return index + 1;
    }

    private static void print(int[][] p, String stra, String strb, int i, int j) {
        StringBuilder sb = new StringBuilder();
        while (true) {
            if (i == 0 && j == 0) break;
            //当p二维数组确定好之后，每一个字符在a串中做什么操作位置都是固定的，所以这里可以从末尾开始处理
            else if (p[i][j] == 1) //删除a串第i个字符
            {
                sb.insert(0, "D" + stra.charAt(i) + formatNum(j + 1));
                i = i - 1;
            } else if (p[i][j] == 2) //在a串最后（第i个字符后）插入字符b[j]
            {
                sb.insert(0, "I" + strb.charAt(j) + formatNum(j));
                j = j - 1;
            } else if (p[i][j] == 0) //什么都不做
            {
                i = i - 1;
                j = j - 1;
            } else //把a[i]变为b[j]
            {
                sb.insert(0, "C" + strb.charAt(j) + formatNum(j));
                i = i - 1;
                j = j - 1;
            }
        }
        sb.append("E");
        System.out.println(sb.toString());
    }

    public static String formatNum(int num) {
        if (num >= 10) {
            return String.valueOf(num);
        }
        return "0" + String.valueOf(num);
    }
}
```
