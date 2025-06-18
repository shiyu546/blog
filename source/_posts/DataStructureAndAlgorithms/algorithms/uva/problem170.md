---
title: problem170 Clock Patience
date: 2025-06-06 10:34:30
mathjax: true
categories:
- [数据结构与算法,uva]
tags:
- 算法
- uva
description: "UVA problem170.Clock Patience <br>
- 模拟扑克牌游戏"
---

## 题目

Card sharp Albert (Foxy) Smith is writing a book on patience games. To double check the examples in the book, he is writing programs to find the optimal play of a given deal. The description of Clock Patience reads as follows: “The cards are dealt out (face down) in a circle, representing a clock, with a pile in each hour position and an extra pile in the centre of the clock. The first card goes face down on one o’clock, the next on two, and so on clockwise from there, with each thirteenth card going to the center of the clock. This results in thirteen piles, with four cards face down in each pile.

{% asset_img 170_pic1.jpg 图示%}

The game then starts. The top card of the ‘king’ pile (the last card dealt) is exposed to become the current card. Each move thereafter consists of placing the current card face up beneath the pile corresponding to its value and exposing the top card of that pile as the new current card. Thus if
the current card is an Ace it is placed under the ‘one’ pile and the top card of that pile becomes the current card. The game ends when the pile indicated by the current card has no face down cards in it.You win if the entire deck is played out, i.e. exposed.”

Write a program that will read in a number of shuffled decks, and play the game.

### Input

The input will consist of decks of cards arranged in four lines of 13 cards, cards separated by a single blank. Each card is represented by two characters, the first is the rank (A, 2, 3, 4, 5, 6, 7, 8, 9, T, J, Q, K) followed by the suit (H, D, C, S). The input will be terminated by a line consisting of a single ‘#’.The deck is listed from bottom to top, so the first card dealt is the last card listed.

### Output

The output will consist of one line per deck. Each line will contain the number of cards exposed during the game (2 digits, with a leading zero if necessary), a comma, and the last card exposed (in the format used in the input).

### SampleInput

TS QC 8S 8D QH 2D 3H KH 9H 2H TH KS KC
9D JH 7H JD 2S QS TD 2C 4H 5H AD 4D 5D
6D 4S 9S 5S 7S JS 8H 3D 8C 3S 4C 6S 9C
AS 7C AH 6H KD JC 7D AC 5C TC QD 6C 3C
\#

### SampleOutput

44,KD

### 题意

按照钟表的摆放共有13个牌堆，将一副扑克牌背面朝上依次发放到13个牌堆上(第一个牌堆1张，第二个牌堆1张，依此类推，到第13个牌堆之后又从第一个牌堆开始)，则每个牌堆4张牌。游戏开始：翻开第13个牌堆(K)上的第一张牌，牌的大小是几，就将该牌正面朝上放入第几个牌堆的底部，再翻开该牌堆的顶部一张牌，接着将该牌放入对应大小的牌堆，并翻开该牌堆顶部的一张牌，如此循环，直到待翻开牌的牌堆的牌全部已经翻开，游戏结束。

请输出翻开的牌的张数和最后一张翻开的牌。

## 思路

每个牌堆设置一个指示还剩几张未翻开牌的指示器，按规则更新指示器即可。

### 问题

#### 实现上的问题

注意输入的顺序，输入是4行为一组，牌按照从下至上，从右至左的顺序叠放。以例子为例：3C是牌底部的牌，TS是最上一张牌。所以发的时候，A牌堆发3C，2牌堆发6C，3牌堆发QD，从右至左，从下至上。下面的输入在牌堆底部。

#### 性能上的问题

## 实现

```JAVA {.line-numbers}
import java.io.*;
import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;

public class Main {

    public static char[] lastCard = new char[2];

    public static void main(String[] args) throws IOException {
        BufferedReader reader;
        reader = new BufferedReader(new InputStreamReader(System.in));
        int[] tops = new int[13];
        while (true) {
            List<String> cardArys = new ArrayList<>();
            String lineCards = reader.readLine();
            if (lineCards.equals("#")) {
                break;
            }
            cardArys.add(lineCards);
            for (int i = 0; i < 3; i++) {
                lineCards = reader.readLine();
                cardArys.add(lineCards);
            }
            Arrays.fill(tops, 0);

            int num = play(cardArys, tops);
            String res = formatResult(num);
            System.out.println(res);
        }
    }

    private static String formatResult(int num) {
        StringBuilder sb = new StringBuilder();
        if (num < 10) {
            sb.append("0");
            sb.append(num);
        } else {
            sb.append(num);
        }
        sb.append(",");
        sb.append(lastCard);
        return sb.toString();
    }

    /**
     * tops包含指示器，指示牌堆翻到哪里了
     */
    private static int play(List<String> cards, int[] tops) {
        int count = 0;
        int top = tops[0];
        char rank = cards.get(top).charAt(0);

        //全局数组用于存储最后一次翻的牌
        lastCard[0] = rank;
        lastCard[1] = cards.get(top).charAt(1);

        int index = getIndex(rank);
        tops[0]++;
        count++;
        int under = 13 - index;
        top = tops[under];
        //因为是从上往下翻，所以指示器是递增的，超过3说明该牌堆翻完了，游戏结束
        while (top <= 3) {
            tops[under]++;
            count++;

            rank = cards.get(top).charAt(under * 3);
            lastCard[0] = rank;
            lastCard[1] = cards.get(top).charAt(under * 3 + 1);
            index = getIndex(rank);
            under = 13 - index;
            top = tops[under];
        }
        return count;
    }

    public static int getIndex(char c) {
        switch (c) {
            case 'A':
                return 1;
            case '2':
                return 2;
            case '3':
                return 3;
            case '4':
                return 4;
            case '5':
                return 5;
            case '6':
                return 6;
            case '7':
                return 7;
            case '8':
                return 8;
            case '9':
                return 9;
            case 'T':
                return 10;
            case 'J':
                return 11;
            case 'Q':
                return 12;
            case 'K':
                return 13;
        }
        return 0;
    }
}
```
