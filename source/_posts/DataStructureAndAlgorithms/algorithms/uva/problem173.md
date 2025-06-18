---
title: problem173 Network Wars
date: 2025-06-17 16:16:48
mathjax: true
categories:
- [数据结构与算法,uva]
tags:
- 算法
- uva
description: "UVA problem173.Network Wars <br>
- 模拟游戏"
---

## 题目

It is the year 2126 and comet Swift-Tuttle has struck the earth as predicted. The resultant explosion emits a large cloud of high energy neutrons that eliminates all human life. The accompanying electromagnetic storm causes two unusual events: many of the links between various parts of the electronic network are severed, and some postgraduate AI projects begin to merge and mutate, in much the same way as animal life did several million years ago. In a very short time two programs emerge, Paskill and Lisper, which move through the network marking each node they visit: Paskill activates a modified Prolog interpreter and Lisper activates the ‘Hello World’ program. However ‘Hello World’ has mutated into an endless loop that so ties up the node that no other program, not even Lisper, can re-enter that node and the Prolog interpreter immediately reverse compiles (and destroys) any program that enters. However, Paskill knows which nodes it has visited and never tries to re-enter them. Thus if Lisper attempts to enter a node already visited by Paskill it will be annihilated; neither can enter a node already visited by Lisper, if either (or both) cannot move both will halt and if they ever arrive at a node simultaneously they annihilate each other. Both programs move through the network at the same speed.

Write a program to simulate these events. All nodes in the the network are labelled with a single uppercase letter as shown below. When moving to the next node, Paskill searches alphabetically forwards from the current node, whereas Lisper searches alphabetically backwards from the current node, both wrapping round if necessary. Thus, (in the absence of the other) if Paskill enters the network below at A, it would visit the nodes in the order A, B, C, D, G, H, E, F; if Lisper enters the network at H it would visit them in the order H, G, E, F. Simulation stops when one or more of the above events occurs. If more than one event occurs, mention Paskill first.

{% asset_img 173_pic1.jpg 图示%}

### Input

Input will consist of a series of lines. Each line will describe a network and indicate the starting nodes for the two programs. A network is described as a series of nodes separated by ‘;’ and terminated by a period (‘.’). Each node is described by its identifier, a ‘:’ and one or more of the nodes connected to it.Each link will be mentioned at least once, as will each node, although not all nodes will be ‘described’.After the period will appear the labels of the starting nodes — first Paskill and then Lisper. No line will contain more than 255 characters. The file will be terminated by a line consisting of a single ‘#’.

### Output

Output will consist of one line for each network. Each line will specify the terminating event and the
node where it occurs. The terminating event is one or two of the following:

* Lisper destroyed in node ?
* {Paskill/Lisper} trapped in node ?
* Both annihilated in node ?

### SampleInput

A:BD;C:BD;F:E;G:DEH;H:EG. A H
E:AB. A B
B:ACD. B D
A:B;B:C;D:E. A D
\#

### SampleOutput

Paskill trapped in node D Lisper trapped in node F
Both annihilated in node E
Lisper destroyed in node B
Lisper trapped in node E

### 题意

给定一个无向图，图上每一个节点都由一个字母表示。有两个程序Paskill和Lisper在图上行走，初始时，它们分别在节点$S_1$和$S_2$。Paskill从相邻边中，选择节点字母序比当前节点大的最小节点前进，而Lisper则从相邻边中选择比当前节点字母序小的最大节点移动。Paskill会记住走过的每一个节点，走过的节点不会再走，而Lisper走过的节点无法再走。

当两个程序至少有一个没法走动时，游戏结束。或者两个程序走到了同一个节点，或者Lisper走到了Paskill走过的节点，也会结束。不同的结束状态输出不同的结束信息。

## 思路

程序走动的过程是固定的，按照过程模拟即可。

### 问题

#### 实现上的问题

输出比较繁琐，注意处理。

#### 性能上的问题

## 实现

```JAVA {.line-numbers}
import java.io.*;
import java.util.HashSet;
import java.util.Set;

public class Main {

    public static int len = 26;
    public static int[][] graphs = new int[len][len];

    public static char[] alphabet = "ABCDEFGHIJKLMNOPQRSTUVWXYZ".toCharArray();

    public static void main(String[] args) throws IOException {
        BufferedReader reader = new BufferedReader(new InputStreamReader(System.in));
        while (true) {
            for (int i = 0; i < len; i++) {
                for (int j = 0; j < len; j++) {
                    graphs[i][j] = 0;
                }
            }
            String line = reader.readLine();

            if (line.equals("#")) {
                break;
            }

            String graphsLine = "";
            char start = 'A', start2 = 'A';
            for (int i = 0; i < line.length(); i++) {
                if (line.charAt(i) == '.') {
                    graphsLine = line.substring(0, i);
                    start = line.charAt(i + 2);
                    start2 = line.charAt(i + 4);
                    break;
                }
            }

            //初始化图
            parseLine(graphsLine);
            play0Game(start, start2);
        }
    }

    private static void play0Game(char start, char start2) {
        //walk at simultaneously
        if (start == start2) {
            System.out.printf("Both annihilated in node %c\n", start);
            return;
        }
        Set<Character> walked = new HashSet<>();
        Set<Character> unreachable = new HashSet<>();
        char paskill = start, lisper = start2;

        char prev = start, prev2 = start2;
        boolean paskCanWalk = true, lisperCanWalk = true;
        char curPaskPos, curLisperPos;
        while (true) {
            prev = paskill;
            prev2 = lisper;
            walked.add(prev);
            unreachable.add(prev2);

            //找到Paskill程序的下一个位置
            paskill = findStartNext(paskill, walked, unreachable);

            //找到Lisper程序的下一个位置
            lisper = findStart2Next(lisper, unreachable);

            if (paskill == '*') {
                paskCanWalk = false;
                curPaskPos = prev;
            } else {
                paskCanWalk = true;
                curPaskPos = paskill;
            }
            if (lisper == '*') {
                lisperCanWalk = false;
                curLisperPos = prev2;
            } else {
                lisperCanWalk = true;
                curLisperPos = lisper;
            }

            StringBuilder sb = new StringBuilder();
            if (!paskCanWalk) {
                sb.append(" ");
                sb.append("Paskill trapped in node ");
                sb.append(curPaskPos);
            }
            if (!lisperCanWalk) {
                sb.append(" ");
                sb.append("Lisper trapped in node ");
                sb.append(curLisperPos);
            } else if (curPaskPos == curLisperPos) {
                sb.append(" ");
                sb.append("Both annihilated in node ");
                sb.append(lisper);
            } else if (walked.contains(curLisperPos)) {
                sb.append(" ");
                sb.append("Lisper destroyed in node ");
                sb.append(curLisperPos);
            }

            if (!sb.toString().isEmpty()) {
                System.out.println(sb.toString().trim());
                break;
            }
        }
    }

    private static char findStart2Next(char lisper, Set<Character> unreachable) {
        int index = lisper - 'A';
        int tmp = (index - 1) < 0 ? 25 : index - 1;
        while (true) {
            if (tmp == index) {
                break;
            }
            if (graphs[index][tmp] == 1 && !unreachable.contains(alphabet[tmp])) {
                return alphabet[tmp];
            } else {
                tmp = (tmp - 1) < 0 ? 25 : tmp - 1;
            }
        }
        return '*';
    }

    private static char findStartNext(char start, Set<Character> walked, Set<Character> unreachable) {
        int index = start - 'A';
        int tmp = (index + 1) % 26;
        while (true) {
            if (tmp == index) {
                break;
            }
            if (graphs[index][tmp] == 1 && !walked.contains(alphabet[tmp]) && !unreachable.contains(alphabet[tmp])) {
                return alphabet[tmp];
            } else {
                tmp = (tmp + 1) % 26;
            }
        }
        return '*';
    }

    private static void parseLine(String graphsLine) {
        String[] lines = graphsLine.split(";");
        for (int i = 0; i < lines.length; i++) {
            String[] tmps = lines[i].split(":");
            char start = tmps[0].charAt(0);
            String ends = tmps[1];
            for (int j = 0; j < ends.length(); j++) {
                char end = ends.charAt(j);
                graphs[start - 'A'][end - 'A'] = 1;
                graphs[end - 'A'][start - 'A'] = 1;
            }
        }
    }
}
```
