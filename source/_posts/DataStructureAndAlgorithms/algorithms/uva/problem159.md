---
title: problem159 Word Crosses
date: 2025-04-08 09:52:35
mathjax: true
categories:
- [数据结构与算法,uva]
tags:
- 算法
- uva
description: "UVA problem159.Word Crosses <br>
- 按格式输出字符串"
---

## 题目

A word cross is formed by printing a pair of words, the first horizontally and the second vertically,so that they share a common letter. A leading word cross is one where the common letter is as near as possible to the beginning of the horizontal word, and, for this letter, as close as possible to the beginning of the vertical word. Thus DEFER and PREFECT would cross on the first ‘E’ in each word,PREFECT and DEFER would cross on the ‘R’. Double leading word crosses use two pairs of words arranged so that the two horizontal words are on the same line and each pair forms a leading word cross.

Write a program that will read in sets of four words and form them (if possible) into double leading word crosses.

### Input

Input will consist of a series of lines, each line containing four words (two pairs). A word consists of 1 to 10 upper case letters, and will be separated from its neighbours by at least one space. The file will be terminated by a line consisting of a single ‘#’.

### Output

Output will consist of a series of double leading word crosses as defined above. Leave exactly three spaces between the horizontal words. If it is not possible to form both crosses, write the message ‘Unable to make two crosses’.

Leave 1 blank line between output sets.

### SampleInput

MATCHES CHEESECAKE PICNIC EXCUSES
PEANUT BANANA VACUUM GREEDY
\#

### SampleOutput

```plaintext

 C
 H
 E
 E
 S
 E          E
 C          X
MATCHES   PICNIC
 K          U
 E          S
            E
            S

Unable to make two crosses
```

### 题意

给出两个单词，一个水平放置，一个竖直放置。然后定义了两个单词共同拥有的一个字母称为word cross:该字母首先离水平放置的单词首字母尽可能的近，在该基础上要离垂直放置的单词首字母尽可能的近。单词输出形式如例子输出所示。现在给定两组单词，两组单词的水平放置单词要在一行，请输出这两组单词。

## 思路

按照word cross的定义：共同的字母离水平放置单词的首字母尽可能近，然后再是离垂直放置单词首字母尽可能近，这样可以唯一确定相交的字母。确定之后然后按指定格式输出。

### 问题

#### 实现上的问题

注意输入的处理，由于一组单词之间的空格可能不止一个，所以使用split切分时需要过滤一下空白字符串。

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
        boolean firstLine = true;
        while (true) {
            String line = scanner.nextLine();
            if (line.equals("#")) {
                break;
            }
            String[] words = line.split(" ");
            List<String> result = new ArrayList<>();

            //过滤掉空白字符
            for (String s : words) {
                if (s != null && !s.trim().isEmpty()) {
                    result.add(s);
                }
            }
            String firstHorizon = result.get(0);
            String firstVertical = result.get(1);
            String secondHorizon = result.get(2);
            String secondVertical = result.get(3);

            //找到相交的字母在两个单词中的位置
            Pair firstPair = findPairs(firstHorizon, firstVertical);
            Pair secondPair = findPairs(secondHorizon, secondVertical);

            if (firstLine) {
                firstLine = false;
            } else {
                System.out.println();
            }

            if (firstPair.horizonIndex == -1 || secondPair.horizonIndex == -1) {
                System.out.println("Unable to make two crosses");
            } else {
                printCross(firstHorizon, firstVertical, firstPair, secondHorizon, secondVertical, secondPair);
            }
        }
    }

    /**
     * 输出，将整个输出内容分为几部分：水平放置的单词上半部分，下半部分
     */
    private static void printCross(String word, String word1, Pair firstPair, String word2, String word3, Pair secondPair) {
        StringBuilder sb = new StringBuilder();
        int firstStart = firstPair.horizonIndex;
        int secondStart = secondPair.horizonIndex;
        int spaceNum = word.length() - (firstPair.horizonIndex + 1) + 3 + secondPair.horizonIndex;
        if (firstPair.verticallyIndex >= secondPair.verticallyIndex) {
            //第一组先输出，此时右边没有字符输出
            int secondRow = firstPair.verticallyIndex - secondPair.verticallyIndex;
            for (int i = 0; i < secondRow; i++) {
                sb.append(genSpace(firstStart));
                sb.append(word1.charAt(i));
                sb.append("\n");
            }
            //共同输出
            for (int i = secondRow; i < firstPair.verticallyIndex; i++) {
                sb.append(genSpace(firstStart));
                sb.append(word1.charAt(i));
                sb.append(genSpace(spaceNum));
                sb.append(word3.charAt(i - secondRow));
                sb.append("\n");
            }
            sb.append(word);
            sb.append("   ");
            sb.append(word2);
            sb.append("\n");

        } else {
            int firstRow = secondPair.verticallyIndex - firstPair.verticallyIndex;
            for (int i = 0; i < firstRow; i++) {
                int spaceNum1 = word.length() + 3 + secondPair.horizonIndex;
                sb.append(genSpace(spaceNum1));
                sb.append(word3.charAt(i));
                sb.append("\n");
            }

            for (int i = firstRow; i < secondPair.verticallyIndex; i++) {
                sb.append(genSpace(firstStart));
                sb.append(word1.charAt(i - firstRow));
                sb.append(genSpace(spaceNum));
                sb.append(word3.charAt(i));
                sb.append("\n");
            }
            sb.append(word);
            sb.append("   ");
            sb.append(word2);
            sb.append("\n");
        }

        //输出水平放置的下半部分
        int max = Math.max(word1.length() - firstPair.verticallyIndex - 1,
                word3.length() - secondPair.verticallyIndex - 1);
        for (int i = 0; i < max; i++) {
            int vertIndex1 = firstPair.verticallyIndex + i + 1;
            int vertIndex2 = secondPair.verticallyIndex + i + 1;
            if (vertIndex1 < word1.length() && vertIndex2 < word3.length()) {
                sb.append(genSpace(firstStart));
                sb.append(word1.charAt(vertIndex1));
                sb.append(genSpace(spaceNum));
                sb.append(word3.charAt(vertIndex2));
                sb.append("\n");
            } else if (vertIndex1 < word1.length() && vertIndex2 >= word3.length()) {
                sb.append(genSpace(firstStart));
                sb.append(word1.charAt(vertIndex1));
                sb.append("\n");
            } else if (vertIndex1 >= word1.length() && vertIndex2 < word3.length()) {
                sb.append(genSpace(firstStart + 1));
                sb.append(genSpace(spaceNum));
                sb.append(word3.charAt(vertIndex2));
                sb.append("\n");
            } else {
                //can't reach here
            }
        }
        System.out.printf(sb.toString());
    }

    private static String genSpace(int num) {
        StringBuilder sb = new StringBuilder();
        for (int i = 0; i < num; i++) {
            sb.append(" ");
        }
        return sb.toString();
    }

    public static Pair findPairs(String horizonWord, String verticalWord) {
        Pair res = new Pair();
        for (int i = 0; i < horizonWord.length(); i++) {
            char index = horizonWord.charAt(i);
            int vertIndex = verticalWord.indexOf(index);
            if (vertIndex >= 0) {
                res.horizonIndex = i;
                res.verticallyIndex = vertIndex;
                return res;
            }
        }
        res.horizonIndex = -1;
        return res;
    }

    public static class Pair {
        public int horizonIndex;
        public int verticallyIndex;
    }
}
```
