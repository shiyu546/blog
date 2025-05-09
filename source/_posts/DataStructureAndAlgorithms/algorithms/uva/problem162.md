---
title: problem162 Beggar My Neighbour
date: 2025-04-16 08:39:25
mathjax: true
categories:
- [数据结构与算法,uva]
tags:
- 算法
- uva
description: "UVA problem162.Beggar My Neighbour <br>
- 按规则计算游戏的胜负"
---

## 题目

“Beggar My Neighbour” (sometimes known as “Strip Jack Naked”) is a traditional card game, designed to help teach beginners something about cards and their values. A standard deck is shuffled and dealt face down to the two players, the first card to the non-dealer, the second to the dealer, and so on until each player has 26 cards. The dealer receives the last card. The non-dealer starts the game by playing the top card of her deck (the second last card dealt) face up on the table. The dealer then covers it by playing her top card face up. Play continues in this fashion until a “face” card (Ace, King, Queen or Jack) is played. The next player must then “cover” that card, by playing one card for a Jack, two for a Queen, three for a King and four for an Ace. If a face card is played at any stage during this sequence,play switches and the other player must cover that card. When this sequence has ended, the player who exposed the last face card takes the entire heap, placing it face down under her existing deck. She then starts the next round by playing one card face up as before, and play continues until one player cannot play when called upon to do so, because they have no more cards.

Write a program that will simulate playing this game. Remember that a standard deck (or pack) of cards contains 52 cards. These are divided into 4 suits — Spades (♠), Hearts (♡), Diamonds (♢) and Clubs (♣). Within each suit there are 13 cards — Ace (A), 2–9, Ten (T), Jack (J), Queen (Q) and
King (K).

### Input

Input will consist of a series of decks of cards. Each deck will give the cards in order as they would be dealt (that is in the example deck below, the non-dealer would start the game by playing the ‘H2’).Decks will occupy 4 lines with 13 cards on each. The designation of each card will be the suit (S, H, D,C) followed by the rank (A, 2–9, T, J, Q, K). There will be exactly one space between cards. The file will be terminated by a line consisting of a single ‘#’.

### Output

Output will consist of a series of lines, one for each deck in the input. Each line will consist of the number of the winning player (1 is the dealer, 2 is the first to play) and the number of cards in the winner’s hand (ignoring any on the stack), right justified in a field of width 3.

### SampleInput

HA H3 H4 CA SK S5 C5 S6 C4 D5 H7 HJ HQ
D4 D7 SJ DT H6 S9 CT HK C8 C9 D6 CJ C6
S8 D8 C2 S2 S3 C7 H5 DJ S4 DQ DK D9 D3
H9 DA SA CK CQ C3 HT SQ H8 S7 ST H2 D2
\#

### SampleOutput

1 44

### 题意

使用一副扑克牌玩游戏，牌共有52张，大小为2-10、J、Q、K、A,花色有黑红梅方，每种花色13张共52张牌。游戏有两名玩家参加：庄和闲，52张牌轮流发牌，先闲后庄，各得26张，牌皆背面朝上。玩法如下：

闲先从分到得牌顶翻一张放到桌面，牌面朝上，然后轮到庄，庄也从牌顶抽一张盖住闲得牌，然后轮到闲，闲再从牌顶抽一张牌面朝上盖住庄得牌，然后轮到庄，依此类推。即双方轮流抽一张牌盖住对方的牌。当碰到face card时，玩法稍微有点不一样，face card是大小为J、Q、K、A得牌，当玩家将face card放到牌桌上盖住牌时，另一方必须抽n张牌盖住(J时n=1,Q时n=2，K时n=3，A时n=4),如果在抽牌过程中抽到了face card，则角色互换，即另一方要抽n张牌盖住。放完n张后，最后一个放face card的玩家赢得牌桌上的所有牌，将牌桌上的牌一把翻转过来，放到自己牌堆的底下。

直到有一方无牌可出时，另一方胜利。现给出牌的顺序，计算胜者和胜者手上有多少牌。

## 思路

主要是模拟玩牌的一个回合的过程，循环该过程直到一方无牌。

### 问题

#### 实现上的问题

#### 性能上的问题

## 实现

```JAVA .{line-numbers}
//source code
import java.util.*;

public class Main {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        while (true) {
            List<String> cardList = new ArrayList<>();
            String lineCards = scanner.nextLine();
            if (lineCards.equals("#")) {
                break;
            }
            String[] cards = lineCards.split(" ");
            cardList.addAll(Arrays.asList(cards));
            for (int i = 0; i < 3; i++) {
                lineCards = scanner.nextLine();
                cards = lineCards.split(" ");
                cardList.addAll(Arrays.asList(cards));
            }

            //split to two player
            LinkedList<String> nodealer = new LinkedList();
            LinkedList<String> dealer = new LinkedList();
            for (int i = 0; i < cardList.size(); i++) {
                if (i % 2 == 0) {
                    nodealer.push(cardList.get(i));
                } else {
                    dealer.push(cardList.get(i));
                }
            }

            playGame(nodealer, dealer);

            if (nodealer.size() > 0) {
                System.out.printf("%d%3d\n", 2, nodealer.size());
            } else {
                System.out.printf("%d%3d\n", 1, dealer.size());
            }
        }
    }

    private static void playGame(Queue<String> nodealer, Queue<String> dealer) {
        int user = 1;
        List<String> deskCards = new ArrayList<>();
        String topCard = nodealer.poll();
        deskCards.add(topCard);
        //循环玩牌，直到有一方无牌为止
        while (user != 0) {
            topCard = deskCards.get(deskCards.size() - 1);
            user = dealCards(topCard, user, dealer, nodealer, deskCards);
        }
    }

    /**
     * 玩牌过程
     * @param topCard 桌面牌堆顶上的牌
     * @param user 回合玩家，轮到谁出牌
     * @param dealers 庄家的牌堆
     * @param nondealers 闲家的牌堆
     * @param deskCards 桌面牌堆
     * @return
     */
    private static int dealCards(String topCard, int user, Queue<String> dealers, Queue<String> nondealers,
                                 List<String> deskCards) {
        Queue<String> handleList, topHandler;
        if (user == 2) {
            handleList = nondealers;
            topHandler = dealers;
        } else {
            handleList = dealers;
            topHandler = nondealers;
        }

        int loop = 0;
        if (topCard.charAt(1) == 'J') {
            loop = 1;
        } else if (topCard.charAt(1) == 'Q') {
            loop = 2;
        } else if (topCard.charAt(1) == 'K') {
            loop = 3;
        } else if (topCard.charAt(1) == 'A') {
            loop = 4;
        }

        if (loop == 0) {
            String dealCard = handleList.poll();
            if (dealCard == null) {
                return 0;
            }
            deskCards.add(dealCard);
            user = user == 2 ? 1 : 2;
            return user;
        }

        for (int i = 0; i < loop; i++) {
            String dealCard = handleList.poll();
            if (dealCard == null) {
                return 0;
            }
            if (dealCard.charAt(1) == 'J' || dealCard.charAt(1) == 'Q'
                    || dealCard.charAt(1) == 'K' || dealCard.charAt(1) == 'A') {
                user = user == 2 ? 1 : 2;
                deskCards.add(dealCard);
                return user;
            } else {
                deskCards.add(dealCard);
            }
        }
        //all cards in desktop blongs to the other one
        for (int i = 0; i < deskCards.size(); i++) {
            topHandler.offer(deskCards.get(i));
        }
        deskCards.clear();

        topCard = topHandler.poll();
        deskCards.add(topCard);
        return user;
    }
}
```
