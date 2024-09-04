---
title: problem129 Krypton Factor
date: 2024-09-04 11:30:26
mathjax: true
categories:
- [数据结构与算法,uva]
tags:
- 算法
- uva
description: "UVA 129.Krypton Factor <br>
- 计算按给定规则出现的第n个字符串，并按指定格式输出"
---

## 题目

You have been employed by the organisers of a Super Krypton Factor Contest in which contestants have very high mental and physical abilities. In one section of the contest the contestants are tested on their ability to recall a sequenace of characters which has been read to them by the Quiz Master. Many of the contestants are very good at recognising patterns. Therefore, in order to add some difficulty to this test, the organisers have decided that sequences containing certain types of repeated subsequences should not be used. However, they do not wish to remove all subsequences that are repeated, since in that case no single character could be repeated. This in itself would make the problem too easy for the contestants.Instead it is decided to eliminate all sequences containing an occurrence of two adjoining identical subsequences. Sequences containing such an occurrence will be called “easy”. Other sequences will be called “hard”.

For example, the sequence ABACBCBAD is easy, since it contains an adjoining repetition of the subsequence CB. Other examples of easy sequences are:

* BB
* ABCDACABCAB
* ABCDABCD
Some examples of hard sequences are:
* D
* DC
* ABDAB
* CBABCBA
In order to provide the Quiz Master with a potentially unlimited source of questions you are asked to write a program that will read input lines from standard input and will write to standard output.

### Input

Each input line contains integers n and L (in that order), where n > 0 and L is in the range$1 \leq L \leq 26$.

Input is terminated by a line containing two zeroes.

### Output

For each input line prints out the n-th hard sequence (composed of letters drawn from the first L letters in the alphabet), in increasing alphabetical order (Alphabetical ordering here corresponds to the normal ordering encountered in a dictionary), followed (on the next line) by the length of that sequence. The first sequence in this ordering is ‘A’. You may assume that for given n and L there do exist at least n hard sequences.

As such a sequence is potentially very long, split it into groups of four (4) characters separated by a space. If there are more than 16 such groups, please start a new line for the 17th group.

Your program may assume a maximum sequence length of 80.

For example, with L = 3, the first 7 hard sequences are:
A
AB
ABA
ABAC
ABACA
ABACAB
ABACABA

### SampleInput

7 3
30 3
0 0

### SampleOutput

ABAC ABA
7
ABAC ABCA CBAB CABA CABC ACBA CABA
28

### 题意

给定一个字符串，该字符串要么是'easy',要么是'hard'。所谓的'hard'就是字符串中不能出现相邻的重复子串，否则就是'easy'字符串.现给定字符串中字符类型个数L，L表示字符串中能出现A到第L个字符，例如L=2，则字符串能出现A,B;如果L=3，能出现A,B,C.现要求出给定L下按字典序排序的第n个'hard'字符串.

## 思路

按照字典序排序，对于hard字符串s，下一个一定是在其后添加一个字符c使得字符串仍是hard字符串，且c在字典序中较小。判断c的值基于hard串的定义：没有相邻的子串，因为s已经是hard串，所以只需比较添加的以c结尾的子串是不是相邻的。
>以'ABCECD'为例，添加C字符，则如果有相邻子串，子串一定以C结尾，所以找出所有C字符，C字符之后则为相邻子串，比较子串即可。从后往前，第一个C字符出现在4下标(从0开始)，则相邻子串为'DC'，而C结尾的相同长度子串为'EC'，所以下标4处的不构成相邻子串；第二个C出现在2下标，相邻子串为'ECDC',前面子串为'ABC'，长度不一致不构成相邻子串。所以添加C后'ABCECDC'仍是hard字符串.

所以对于s来说，下一个hard串是在添加字符之后仍然是hard串，且添加的字符字典序最小的一个。

还有一种情况是s最大长度是80，如果s长度达到80，下一个hard怎么找。这时不能在s后面添加字符了，因为长度已经超过了，这时把s的最后一个字符C给提取出来，并从s中删掉该字符，从C的字典序下一个字符开始判断。例如s最后一个字符是'D',则从s中删掉'D',从'E'开始拼接到s判断是不是hard串。

如果到L个字符都无法拼接，则从s再提取最后一个字符C，从C字典序下一个判断。重复该过程直到找到hard串。

### 问题

#### 实现上的问题

#### 性能上的问题

## 实现

```JAVA .{line-numbers}
//file
import java.io.File;
import java.io.FileNotFoundException;
import java.util.Scanner;

public class Main {
    private static char[] aplhabet = {'a', 'A', 'B', 'C', 'D', 'E', 'F', 'G', 'H', 'I', 'J', 'K', 'L',
            'M', 'N', 'O', 'P', 'Q', 'R', 'S', 'T', 'U', 'V', 'W', 'X', 'Y', 'Z'};

    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        while (true) {
            int n = scanner.nextInt();
            int l = scanner.nextInt();
            if (n == 0 && l == 0) {
                break;
            }
            String letters = "A";
            for (int i = 1; i < n; i++) {
                letters = findNextString(letters, l, 1);
            }
            StringBuilder sb = new StringBuilder();
            int index = 0;
            for (int i = 0; i < letters.length(); i++) {
                if (i % 4 == 0 && i > 0) {
                    index++;
                    if (index == 16) {
                        sb.append("\n");
                    } else {
                        sb.append(" ");
                    }
                }
                sb.append(letters.charAt(i));
            }
            System.out.println(sb.toString());
            System.out.println(letters.length());
        }
    }

    /**
    *找到letters的下一个hard串 
    */
    private static String findNextString(String letters, int l, int start) {
        if (letters.length() >= 80) {
            //当长度达到80时，从末尾字符下一个开始寻找hard串
            char lastChar = letters.charAt(letters.length() - 1);
            letters = letters.substring(0, letters.length() - 1);
            int next = (int) (lastChar - 'A') + 1;
            return findNextString(letters, l, next + 1);
        }
        label2:
        for (int i = start; i <= l; i++) {
            label1:
            for (int j = letters.length() - 1; j >= 0; j--) {
                //相邻串比较，子串末尾字符一定是添加的字符
                if (letters.charAt(j) == aplhabet[i]) {
                    int length = letters.length() - j;
                    if (j - length + 1 >= 0) {
                        for (int k = j + 1; k < letters.length(); k++) {
                            if (letters.charAt(k - length) != letters.charAt(k)) {
                                continue label1;
                            }
                        }
                        continue label2;
                    }
                }
            }
            //运行完j这个循环表示第i个字符能构成hard串，添加到字符串s的末尾
            letters += aplhabet[i];
            return letters;
        }
        //运行到这里说明末尾没有字符能添加，和处理80长度上限一样，敲掉末尾字符，从该字符下一个开始判断
        char lastChar = letters.charAt(letters.length() - 1);
        letters = letters.substring(0, letters.length() - 1);
        int next = (int) (lastChar - 'A') + 1;
        return findNextString(letters, l, next + 1);
    }
}

```
