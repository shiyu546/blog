---
title: 127."Accordian" Patience
date: 2024-08-15 16:52:09
mathjax: true
categories:
- [数据结构与算法,uva]
tags:
- 算法
- uva
description: "UVA 127.\"Accordian\" Patience <br>
- 按照一定的规则将扑克牌叠起来，最终能叠多少堆"
---

## 题目

You are to simulate the playing of games of “Accordian” patience, the rules for which are as follows:

> Deal cards one by one in a row from left to right, not overlapping. Whenever the card matches its immediate neighbour on the left, or matches the third card to the left, it may be moved onto that card. Cards match if they are of the same suit or same rank. After making a move, look to see if it has made additional moves possible. Only the top card of each pile may be moved at any given time. Gaps between piles should be closed up as soon as they appear by moving all piles on the right of the gap one position to the left. Deal out the whole pack, combining cards towards the left whenever possible. The game is won if the pack is reduced to a single pile.

Situations can arise where more than one play is possible. Where two cards may be moved, you should adopt the strategy of always moving the leftmost card possible. Where a card may be moved either one position to the left or three positions to the left, move it three positions.

### Input

Input data to the program specifies the order in which cards are dealt from the pack. The input contains pairs of lines, each line containing 26 cards separated by single space characters. The final line of the input file contains a ‘#’ as its first character. Cards are represented as a two character code. The first character is the face-value (A=Ace, 2–9, T=10, J=Jack, Q=Queen, K=King) and the second character is the suit (C=Clubs, D=Diamonds, H=Hearts, S=Spades).

### Output

One line of output must be produced for each pair of lines (that between them describe a pack of 52 cards) in the input. Each line of output shows the number of cards in each of the piles remaining after playing “Accordian patience” with the pack of cards as described by the corresponding pairs of input lines.

### SampleInput

QD AD 8H 5S 3H 5H TC 4D JH KS 6H 8S JS AC AS 8D 2H QS TS 3S AH 4H TH TD 3C 6S
8C 7D 4C 4S 7S 9H 7C 5D 2S KD 2D QH JD 6D 9D JC 2C KH 3D QC 6C 9S KC 7H 9C 5C
AC 2C 3C 4C 5C 6C 7C 8C 9C TC JC QC KC AD 2D 3D 4D 5D 6D 7D 8D TD 9D JD QD KD
AH 2H 3H 4H 5H 6H 7H 8H 9H KH 6S QH TH AS 2S 3S 4S 5S JH 7S 8S 9S TS JS QS KS
\#

### SampleOutput

6 piles remaining: 40 8 1 1 1 1
1 pile remaining: 52

### 题意

题目是玩一个游戏，游戏规则如下：把一副扑克牌52张按从左到右平铺，共52摞,每摞一张。然后从左往右比较，一张牌如果和该牌左边的牌或者左边第三张牌匹配，则将该牌放到匹配的牌上。具体规则翻译如下：

>Deal cards one by one in a row from left to right, not overlapping. Whenever the card matches its immediate neighbour on the left, or matches the third card to the left, it may be moved onto that card. Cards match if they are of the same suit or same rank. After making a move, look to see if it has made additional moves possible. Only the top card of each pile may be moved at any given time. Gaps between piles should be closed up as soon as they appear by moving all piles on the right of the gap one position to the left. Deal out the whole pack, combining cards towards the left whenever possible. The game is won if the pack is reduced to a single pile.
按照从左到右的顺序处理牌。当一张牌匹配到该牌左边第一张牌或者左边第三张牌时，将该牌移动到匹配的牌上。牌匹配的意思是花色相同或者大小相同。当牌匹配移动后，继续判断该牌是否还能匹配移动，直至无法匹配为止。只有一摞牌上的第一张牌能移动，当一摞牌全移动完时，该摞直接消失，后一摞牌的左边第一摞就是消失的该摞牌的左边第一摞。处理所有的牌直到没有牌能继续移动为止。

当牌摞都不能移动时，判断还有多少摞。这里说了两点注意事项：

1. 当有多张牌能移动时，移动最左边的那张牌；
2. 当一张牌有多种移动选择时，移动牌到左手边第三张牌处。

## 思路

从牌的注意事项给出了移动规则：选择牌摞最左边的能移动的一摞牌，并将牌摞最上面一张牌移动到不能移动为止，循环该过程，直到整个牌堆不能移动为止。

所以选择从哪一摞开始判断牌堆是否能移动是比较关键的问题，考虑到上一次移动结束后，移动到的牌摞左边都没有变化，所以可以从移动到的位置的下一摞开始判断。第二个是如何判断牌能移动到哪里，按照先左三后左一循环比较直至左三左一都不能匹配，就是牌能移动到的终点。

### 问题

#### 实现上的问题

#### 性能上的问题

## 实现

```JAVA .{line-numbers}
//file
import java.util.LinkedList;
import java.util.Scanner;

public class Main {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        String line = scanner.nextLine();
        while (!"#".equals(line)) {
            String secondLine = scanner.nextLine();

            //使用一个链表初始化52摞牌，使用链表主要是删除节点方便
            LinkedList<LinkedList<String>> cardList = new LinkedList<>();
            initCards(line, cardList);
            initCards(secondLine, cardList);

            for (int i = 1; i < cardList.size(); ) {
                String topCard = cardList.get(i).peekLast();
                //temp用来记录当前牌所能匹配的终点位置下标，按照第二点注意事项，能匹配左三就匹配左三，不能就匹配左一，如果都不能，则结束，如此循环。
                int temp = i;
                while (true) {
                    //先匹配左三，匹配上了从当前位置接着匹配
                    if (temp - 3 >= 0) {
                        String topCardLeftThird = cardList.get(temp - 3).peekLast();
                        if (compareTwo(topCard, topCardLeftThird)) {
                            temp -= 3;
                            continue;
                        }
                    }
                    //再匹配左一，匹配上了从当前位置接着匹配
                    if (temp - 1 >= 0) {
                        String topCardNeighbour = cardList.get(temp - 1).peekLast();
                        if (compareTwo(topCard, topCardNeighbour)) {
                            temp = temp - 1;
                            continue;
                        }
                    }
                    break;
                }
                //都匹配不上，temp就记录了最终能匹配的位置
                if (temp != i) {
                    cardList.get(i).pollLast();
                    cardList.get(temp).add(topCard);
                    if (cardList.get(i).isEmpty()) {
                        cardList.remove(i);
                    }
                    //下一次判断匹配从上一次匹配到的终点位置的下一个位置开始，所以i赋值为temp+1
                    i = temp + 1;
                } else {
                    i++;
                }
            }

            StringBuilder sb = new StringBuilder();
            sb.append(cardList.size());
            if (cardList.size() == 1) {
                sb.append(" pile");
            } else {
                sb.append(" piles");
            }
            sb.append(" remaining:");
            for (int i = 0; i < cardList.size(); i++) {
                sb.append(" ");
                sb.append(cardList.get(i).size());
            }
            System.out.println(sb.toString());


            line = scanner.nextLine();
        }
    }

    public static boolean compareTwo(String stra, String strb) {
        if (stra.charAt(0) == strb.charAt(0) || stra.charAt(1) == strb.charAt(1)) {
            return true;
        }
        return false;
    }

    private static void initCards(String line, LinkedList<LinkedList<String>> cardList) {
        String[] cards = line.split(" ");
        for (String card : cards) {
            LinkedList<String> tmp = new LinkedList<>();
            tmp.add(card);
            cardList.add(tmp);
        }
    }
}
```