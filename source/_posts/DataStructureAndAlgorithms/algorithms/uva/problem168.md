---
title: problem168 Theseus and the Minotaur
date: 2025-05-17 23:08:34
mathjax: true
categories:
- [数据结构与算法,uva]
tags:
- 算法
- uva
description: "UVA problem168.Theseus and the Minotaur <br>
- 路径追踪问题"
---

## 题目

Those of you with a classical education may remember the legend of Theseus and the Minotaur. This is an unlikely tale involving a bull headed monster, an underground maze full of twisty little passages all alike, love-lorn damsels and balls of silk. In line with the educational nature of this contest, we will now reveal the true story.

The maze was actually a series of caverns connected by reasonably straight passages, some of which could only be traversed in one direction. In order to trap the Minotaur, Theseus smuggled a large supply of candles into the Labyrinth, as he had discovered that the Minotaur was afraid of light.Theseus wandered around somewhat aimlessly until he heard the Minotaur approaching along a tunnel.At this point he lit a candle and set off in pursuit. The Minotaur retreated into the cavern it had just left and fled by another passage. Theseus followed, slowly gaining, until he reached the k’th cavern since lighting the candle. Here he had enough time to place the lighted candle in the middle of the cavern, light another one from it, and continue the chase. As the chase progressed, a candle was left in each k’th cavern passed through, thereby limiting the movement of the Minotaur. Whenever the Minotaur entered a cavern, it would check the exits in a particular order, fleeing down the first that did not lead
directly to a lit cavern. (Remember that as Theseus was carrying a lit candle, the Minotaur never exited a cavern by the tunnel used to enter it.) Eventually the Minotaur became trapped, enabling Theseus to defeat it.

Consider the following Labyrinth as an example, where in this case the Minotaur checks the exits from a cavern in alphabetical order:

{% asset_img 168_pic1.jpg 图示%}

Assume that Theseus is in cavern C when he hears the Minotaur approaching from A, and that for this scenario, the value of k is 3. He lights a candle and gives chase, pursuing it through A, B, D (leaves a candle), G, E, F (another candle), H, E, G (another), H, E (trapped).

Write a program that will simulate Theseus’s pursuit of the Minotaur. The description of a labyrinth will identify each cavern by an upper case character and will list the caverns reachable from that cavern in the order that the Minotaur will attempt them, followed by the identifiers for the caverns which the Minotaur and Theseus were in when contact was first made, followed by the value of k.

### Input

Input will consist of a series of lines. Each line will describe a scenario in the format shown below (which describes the above example). No line will contain more than 255 characters. The file will be terminated by a line consisting of a single ‘#’.

### Output

Output will consist of one line for each Labyrinth. Each line will identify the lit caverns, in the order in which the candles were left, and the cavern in which the Minotaur was trapped, following the format shown in the example below.

### SampleInput

A:BCD;B:AD;D:BG;F:H;G:DEH;E:FGH;H:EG;C:AD. A C 3
\#

### SampleOutput

D F G /E

### 题意

题目引申自一个传说故事：忒修斯和弥诺陶洛斯。讲诉的是在山洞中忒修斯追捕弥诺陶洛斯，山洞由一个个的洞穴组成，洞穴之间由通道连接，通道可能是单向也可能是双向。初始时忒修斯在洞穴中乱串直到察觉到弥诺陶洛斯，此时弥诺陶洛斯一定在临近的通道中，然后弥诺陶洛斯会退回到来时的洞穴，故事开始。弥诺陶洛斯每逃往下一个洞穴，忒修斯就会出现在弥诺陶洛斯上一次出现的洞穴，由于弥诺陶洛斯怕光，每当追捕经过了k个洞穴，忒修斯就会插一根蜡烛，这样弥诺陶洛斯就不会再从这个洞穴经过。且弥诺陶洛斯不会往回走，即它从A逃到B洞穴时，此时A会被忒修斯占住，下一个逃亡时弥诺陶洛斯不会再从B逃到A。当弥诺陶洛斯无路可逃时，就会被忒修斯抓住，故事结束。

现在给出了洞穴的结构，使用邻接矩阵的形式，给出了每一个洞穴节点的相邻节点，即可通往的洞穴，注意这些节点的顺序：当弥诺陶洛斯从当前节点有多个可选路径时，按照下一个节点给出的顺序找到第一个可通行的节点并前往。然后给出了忒修斯和弥诺陶洛斯初始洞穴节点和k值，求出点燃蜡烛的洞穴顺序和最终在哪个洞穴抓住了弥诺陶洛斯。

## 思路

题目本质上就是图上的路径问题，注意题意的限制，一个是洞穴会被点亮，二是来时的路下一步不能往回走，逐步跟踪忒修斯和弥诺陶洛斯所在节点，并判断弥诺陶洛斯所在节点是否有下一步节点可去，没有则结束。

### 问题

#### 实现上的问题

#### 性能上的问题

该题很容易超时，主要出现在处理输入上。最开始使用split解析每一行输入，使用map存储邻接矩阵，导致超时。通过修改输入的处理方式，才降低了时间复杂度。

## 实现

```JAVA {.line-numbers}
//source code
import java.util.*;

public class Main {

    private static char[] alperbet = {'A', 'B', 'C', 'D', 'E', 'F', 'G', 'H', 'I', 'J', 'K', 'L', 'M', 'N', 'O', 'P', 'Q', 'R', 'S', 'T', 'U', 'V', 'W', 'X', 'Y', 'Z'};

    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);

        //用来存储邻接矩阵
        LinkedList<Integer>[] cavernsLine = new LinkedList[26];

        while (true) {
            for (int i = 0; i < 26; i++) {
                cavernsLine[i] = new LinkedList<Integer>();
            }
            String line = scanner.nextLine();
            if (line.equals("#")) {
                break;
            }

            int i = 0;
            while (i != line.length()) {
                int index = line.charAt(i++) - 'A';
                if (line.charAt(i) == ':') {
                    i++;
                }
                while (line.charAt(i) != ';' && line.charAt(i) != '.') {
                    cavernsLine[index].add(line.charAt(i) - 'A');
                    i++;
                }
                if (line.charAt(i) == '.') {
                    i += 2;
                    break;
                }
                i++;
            }
            int minotaurStart = line.charAt(i) - 'A';
            i += 2;
            int theseusStart = line.charAt(i) - 'A';
            i += 2;
            int k = Integer.parseInt(line.substring(i));

            //这里用来记录点燃蜡烛的洞穴
            Set<Integer> candlesSet = new HashSet<>();
            int count = 0;
            StringBuilder sb = new StringBuilder();
            while (true) {
                //寻找弥诺陶洛斯逃往的下一个洞穴，为空则表示没有路径可逃
                Integer nextCavern = findNextCavernNew(theseusStart, minotaurStart, cavernsLine, candlesSet);
                if (nextCavern == null) {
                    sb.append("/");
                    sb.append(alperbet[minotaurStart]);
                    break;
                } else {
                    theseusStart = minotaurStart;
                    minotaurStart = nextCavern;
                    count++;

                    //这里用来记录点燃蜡烛的洞穴
                    if (count == k) {
                        sb.append(alperbet[theseusStart]);
                        sb.append(" ");
                        candlesSet.add(theseusStart);
                        count = 0;
                    }
                }
            }
            System.out.println(sb.toString());
        }
    }

    private static Integer findNextCavernNew(int theseusStart, int minotaurStart,
                                             LinkedList[] caverns, Set<Integer> candlesSet) {
        LinkedList<Integer> nextCaverns = caverns[minotaurStart];
        if (nextCaverns == null) {
            return null;
        }
        for (int i = 0; i < nextCaverns.size(); i++) {
            int nextCavern = nextCaverns.get(i);

            //下一个洞穴能否去主要是看是否是来时的路和是否有蜡烛，两者有一则不能去
            if (nextCavern == theseusStart || candlesSet.contains(nextCavern)) {
                continue;
            } else {
                return nextCavern;
            }

        }
        return null;
    }
}
```
