---
title: problem131 The Psychic Poker Player
date: 2024-09-16 01:48:23
mathjax: true
categories:
- [数据结构与算法,uva]
tags:
- 算法
- uva
description: "UVA 131.The Psychic Poker Player <br>
- 五张抽，判断最大的牌型"
---

## 题目

In 5-card draw poker, a player is dealt a hand of five cards (which may be looked at). The player may then discard between zero and five of his or her cards and have them replaced by the same number of cards from the top of the deck (which is face down). The object is to maximize the value of the final hand. The different values of hands in poker are given at the end of this problem.

Normally the player cannot see the cards in the deck and so must use probability to decide which cards to discard. In this problem, we imagine that the poker player is psychic and knows which cards are on top of the deck. Write a program which advises the player which cards to discard so as to maximize the value of the resulting hand.

### Input

Input will consist of a series of lines, each containing the initial five cards in the hand then the first five cards on top of the deck. Each card is represented as a two-character code. The first character is the face-value (A=Ace, 2–9, T=10, J=Jack, Q=Queen, K=King) and the second character is the suit (C=Clubs, D=Diamonds, H=Hearts, S=Spades). Cards will be separated by single spaces. Each input line will be from a single valid deck, that is there will be no duplicate cards in each hand and deck.

### Output

Each line of input should produce one line of output, consisting of the initial hand, the top five cards on the deck, and the best value of hand that is possible. Input is terminated by end of file.

Note: Use the sample input and output as a guide.

Note that the order of the cards in the player’s hand is irrelevant, but the order of the cards in the deck is important because the discarded cards must be replaced from the top of the deck. Also note that examples of all types of hands appear in the sample output, with the hands shown in decreasing order of value.

### SampleInput

TH JH QC QD QS QH KH AH 2S 6S
2H 2S 3H 3S 3C 2D 3D 6C 9C TH
2H 2S 3H 3S 3C 2D 9C 3D 6C TH
2H AD 5H AC 7H AH 6H 9H 4H 3C
AC 2D 9C 3S KD 5S 4D KS AS 4C
KS AH 2H 3C 4H KC 2C TC 2D AS
AH 2C 9S AD 3C QH KS JS JD KD
6C 9C 8C 2D 7C 2H TC 4C 9S AH
3D 5S 2H QD TD 6S KH 9H AD QH

### SampleOutput

Hand: TH JH QC QD QS Deck: QH KH AH 2S 6S Best hand: straight-flush
Hand: 2H 2S 3H 3S 3C Deck: 2D 3D 6C 9C TH Best hand: four-of-a-kind
Hand: 2H 2S 3H 3S 3C Deck: 2D 9C 3D 6C TH Best hand: full-house
Hand: 2H AD 5H AC 7H Deck: AH 6H 9H 4H 3C Best hand: flush
Hand: AC 2D 9C 3S KD Deck: 5S 4D KS AS 4C Best hand: straight
Hand: KS AH 2H 3C 4H Deck: KC 2C TC 2D AS Best hand: three-of-a-kind
Hand: AH 2C 9S AD 3C Deck: QH KS JS JD KD Best hand: two-pairs
Hand: 6C 9C 8C 2D 7C Deck: 2H TC 4C 9S AH Best hand: one-pair
Hand: 3D 5S 2H QD TD Deck: 6S KH 9H AD QH Best hand: highest-card

### 题意

每个人起5张牌，牌桌上还有一个牌堆，然后可以从手牌中拿出k($0\leq k \leq 5$)张牌(5张牌中的任意k张)与牌堆顶端k张牌交换，使得手牌中的牌型最大。

题目涉及到两种扑克术语：花色和大小，扑克牌有黑红梅方四种花色；大小指扑克牌上的点数大小，共有 **[A,2,3,4,5,6,7,8,9,T,J,Q,K]** 共13种。如果手牌中5张牌是相连的，则称这种牌型为顺子。例如 **[2,3,4,5,6]** , **[5,6,7,8,9]** , **[9,T,J,Q,K]** 等,其中A既可以当作开头，也可以当作结尾，所以 **[A,2,3,4,5]** 和 **[T,J,Q,K,A]** 都是顺子。

牌型说明：

1. Straight Flush(同花顺): 同花顺是指五张手牌花色相同且组成顺子.
2. four-of-a-kind(四条): 四张同样的牌+任意一张牌.
3. Full House(葫芦): 三条带一对，即三张同样的牌带两张同样的牌.
4. Flush(五张同花): 用五张同一花色但不相连的牌型(不构成顺子)组成.
5. Straight(五张顺子): 由五张相连但不同花色的牌组成.
6. Three-of-a-Kind(三条): 即五张牌中由三张同样的牌，其余两张各不相同.
7. Two Pair(两对): 五张牌组成两对,剩余一张牌不相同.
8. One Pair(一对): 五张牌中两张组成一对，其余三张各不相同.
9. High Card(大牌): 无以上任何牌型时.

从1到9，牌型依此减小，即1牌型最大，9牌型最小。

## 思路

通过枚举所有的换牌方式(即换桌牌顶的一张牌、两张牌，三张牌、四张牌、五张牌)，得到所有的可能手牌，然后判断手牌的最大牌型。取所有可能的手牌牌型中的最大者，即为最大牌型。

### 问题

#### 实现上的问题

判断牌型只需按照9种牌型来判断，问题在于找到一种方式枚举所有的手牌交换方式。考虑交换一张手牌时，桌牌顶的一张牌可与手牌中任意一张交换，即$C_5^1$种可能；交换两张时可与手牌中任意两张交换，即$C_5^2$种可能，依此类推。现需找到一种方式遍历所有这些可能：

```plaintext
//enum
[1,2,3,0,0]
[1,2,0,3,0]
[1,2,0,0,3]
[1,0,2,3,0]
[1,0,2,0,3]
[1,0,0,2,3]
[0,1,2,3,0]
[0,1,2,0,3]
[0,1,0,2,3]
[0,0,1,2,3]
```

以上是k=3换3张牌时所有的可能(数组下标表示手牌的位置，元素不为0表示交换该位置的手牌，例如第二条[1,2,0,3,0]表示换手牌中的1，2，4张牌，其中下标从1开始)，遵循的规则如下：

1. 如果从尾往前数第一个出现不是0的位置k不在最末尾，则将k位所在数字往末尾移一格得到新的一种可能；
2. 如果最末尾有数字，则从末尾开始，找到第一个位置k数字不为0，且该数字右边第一格为0，将k所在数字往末尾移动一格，同时后面所有数字都紧挨着k+1逐个往后排。例如[1,2,0,0,3]找到的数字为2，将2往后移一格，得到[1,0,2,0,3],同时2后面的数字都要挨着2逐个往后排，即3要挨着2，得到[1,0,2,3,0].

当找不到这两种可能时，表示遍历已经结束。

#### 性能上的问题

## 实现

```JAVA .{line-numbers}
//file
import java.util.Scanner;

public class Main {
    private static String[] cardRawType = {"straight-flush", "four-of-a-kind", "full-house", "flush",
            "straight", "three-of-a-kind", "two-pairs", "one-pair", "highest-card"};

    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        while (scanner.hasNextLine()) {
            String line = scanner.nextLine();
            String[] cards = line.split(" ");
            String[] handCards = new String[5];
            String[] deskCards = new String[5];
            StringBuilder sb = new StringBuilder();
            sb.append("Hand: ");

            for (int i = 0; i < 10; i++) {
                if (i < 5) {
                    handCards[i] = cards[i];
                }
                if (i >= 5) {
                    if (i == 5) {
                        sb.append("Deck: ");
                    }
                    deskCards[i - 5] = cards[i];
                }
                sb.append(cards[i] + " ");
            }

            //用来记录手牌
            String[] copyCards = new String[5];
            resetHandCards(handCards, copyCards);
            int cardType = findCardType(copyCards);
            for (int i = 1; i <= 5; i++) {
                //tags用来记录手牌中哪些牌与牌桌上的牌顶交换
                int[] tags = new int[5];
                for (int j = 0; j < i; j++) {
                    tags[j] = 1;
                }
                for (int j = i; j < tags.length; j++) {
                    tags[j] = 0;
                }
                do {
                    //交换手牌
                    exchange(copyCards, deskCards, tags);
                    //找到手牌的牌型
                    int tmpCardType = findCardType(copyCards);
                    if (tmpCardType < cardType) {
                        cardType = tmpCardType;
                    }
                    //重置手牌为最开始的手牌
                    resetHandCards(handCards, copyCards);
                    //设置tags，寻找下一个该交换的手牌
                } while (hasNextTags(tags));
            }
            sb.append("Best hand: ");
            sb.append(cardRawType[cardType]);
            System.out.println(sb.toString());
        }
    }

    private static void resetHandCards(String[] handCards, String[] copyCards) {
        for (int i = 0; i < 5; i++) {
            copyCards[i] = handCards[i];
        }
    }

    private static void exchange(String[] copyCards, String[] deskCards, int[] tags) {
        int index = 0;
        for (int i = 0; i < tags.length; i++) {
            if (tags[i] == 1) {
                copyCards[i] = deskCards[index];
                index++;
            }
        }
    }

    //找到下一个手牌，tags标志手牌中该换的牌
    public static boolean hasNextTags(int[] tags) {
        int lastTag = 0;
        for (int i = tags.length - 1; i >= 0; i--) {
            if (tags[i] == 1) {
                lastTag = i;
                break;
            }
        }
        if (lastTag < tags.length - 1) {
            tags[lastTag] = 0;
            tags[lastTag + 1] = 1;
            return true;
        }
        for (int i = tags.length - 2; i >= 0; i--) {
            if (tags[i] == 1 && tags[i + 1] == 0) {
                tags[i] = 0;
                tags[i + 1] = 1;
                int j = i + 2;
                int tagNum = 0;
                for (int k = j; k < tags.length; k++) {
                    if (tags[k] == 1) {
                        tagNum++;
                        tags[k] = 0;
                    }
                }
                int start = j;
                for (int k = 0; k < tagNum; k++) {
                    tags[start++] = 1;
                }
                return true;
            }
        }
        return false;
    }


    public static int findCardType(String[] handCards) {
        if (isSameSuit(handCards) && isStraight(handCards)) {
            return 0;
        }
        if (cardNumberType(handCards) == 1) {
            return 1;
        }
        if (cardNumberType(handCards) == 2) {
            return 2;
        }
        if (isSameSuit(handCards)) {
            return 3;
        }
        if (isStraight(handCards)) {
            return 4;
        }
        if (cardNumberType(handCards) == 3) {
            return 5;
        }
        if (cardNumberType(handCards) == 4) {
            return 6;
        }
        if (cardNumberType(handCards) == 5) {
            return 7;
        }
        return 8;
    }

    public static boolean isSameSuit(String[] cards) {
        char suit = cards[0].charAt(1);
        for (int i = 1; i < cards.length; i++) {
            if (cards[i].charAt(1) != suit) {
                return false;
            }
        }
        return true;
    }

    public static boolean isStraight(String[] cards) {
        int[] order = new int[14];
        for (int i = 0; i < order.length; i++) {
            order[i] = 0;
        }
        for (String s : cards) {
            if (s.charAt(0) == 'A') {
                order[0] = 1;
                order[13] = 1;
            } else {
                order[cardIndex(s.charAt(0))] = 1;
            }
        }
        for (int i = 0; i <= order.length - 5; i++) {
            if (order[i] == 1 && order[i + 1] == 1 && order[i + 2] == 1 && order[i + 3] == 1 && order[i + 4] == 1) {
                return true;
            }
        }
        return false;
    }

    private static int cardNumberType(String[] cards) {
        int[] numbers = new int[13];
        for (int i = 0; i < numbers.length; i++) {
            numbers[i] = 0;
        }
        for (String s : cards) {
            numbers[cardIndex(s.charAt(0))]++;
        }
        int cnt = 0;
        int[] ary = new int[5];
        for (int i = 0; i < ary.length; i++) {
            ary[i] = 0;
        }
        for (int i = 0; i < numbers.length; i++) {
            if (numbers[i] > 0) {
                ary[cnt] = numbers[i];
                cnt++;
            }
        }
        if (cnt == 2 && (ary[0] == 4 || ary[1] == 4)) {
            //four kinds
            return 1;
        } else if (cnt == 2 && (ary[1] == 3 || ary[0] == 3)) {
            //full house
            return 2;
        } else if (cnt == 3 && (ary[0] == 3 || ary[1] == 3 || ary[2] == 3)) {
            //three of a kind
            return 3;
        } else if (cnt == 3 && ((ary[0] == 2 && ary[1] == 2) || (ary[0] == 2 && ary[2] == 2) || (ary[1] == 2 && ary[2] == 2))) {
            //two pair
            return 4;
        } else if (cnt == 4) {
            //one pair
            return 5;
        }
        return 6;
    }

    private static int cardIndex(char c) {
        int index;
        if (c == 'A') {
            index = 0;
        } else if (c == 'T') {
            index = 9;
        } else if (c == 'J') {
            index = 10;
        } else if (c == 'Q') {
            index = 11;
        } else if (c == 'K') {
            index = 12;
        } else {
            index = c - '1';
        }
        return index;
    }
}
```
