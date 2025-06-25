---
title: problem174 Strategy
date: 2025-06-24 10:44:52
mathjax: true
categories:
- [数据结构与算法,uva]
tags:
- 算法
- uva
description: "UVA problem174.Strategy <br>
- 模拟对策"
---

## 题目

A well known psychology experiment involves people playing a game in which they can either trade with each other or attempt to cheat the other player. If both players TRADE then each gains one point. If one TRADEs and the other CHEATs then the TRADEr loses 2 points and the CHEATer wins 2. If both CHEAT then each loses 1 point.

There are a variety of different strategies for playing this game, although most people are either unable to find a winning strategy, or, having decided on a strategy, do not stick to it. Thus it is fairer to attempt to evaluate these strategies by simulation on a computer. Each strategy is simulated by an automaton. An automaton is characterised by a program incorporating the strategy, a memory for previous encounters and a count reflecting the score of that automaton. The count starts at zero and is altered according to the above rules after each encounter. The memory is able to determine what happened on up to the last two encounters with each other contender.

Write a program that will read in details of up to 10 different strategies, play each strategy against each other strategy 10 times and then print out the final scores. Strategies will be in the form of simple programs obeying the following grammar:

```plaintext
< program > ::= < statement >.
< statement > ::= < command > | < if stat >
< if stat > ::= IF < condition > THEN < statement > ELSE < statement >
< condition > ::= < cond > | < cond > < op > < condition >
< op > ::= AND | OR
< cond > ::= < memory > {= | #} {< command > | NULL}
< memory > ::= {MY | YOUR} LAST {1 | 2}
< command > ::= TRADE | CHEAT
```

Note that LAST1 refers to the previous encounter between these two automata, LAST2 to the encounter before that and that ‘MY’ and ‘YOUR’ have the obvious meanings. Spaces and line breaks may appear anywhere in the program and are for legibility only. The symbol ‘#’ means ‘is not equal to’. NULL indicates that an encounter has not ocurred. The following are valid programs:

CHEAT.
IF MY LAST1 = CHEAT THEN TRADE ELSE CHEAT.
IFYOURLAST2=NULLTHENTRADEELSEIFYOURLAST1=TRADETHENTRADE
ELSECHEAT.

### Input

Input will consist of a series of programs. Each program will be no longer than 255 characters and may be split over several lines for convenience. There will be no more than 10 programs. The file will be terminated by a line containing only a single ‘#’.

### Output

Output will consist of one line for each line of input. Each line will consist of the final score of the relevant program right justified in a field of width 3.

### SampleInput

CHEAT.
IF MY LAST1 = CHEAT THEN TRADE ELSE CHEAT.
IFYOURLAST2=NULLTHENTRADEELSEIFYOURLAST1=TRADETHENTRADE
ELSECHEAT.
\#

### SampleOutput

  1
-12
-13

### 题意

两个用户玩游戏，如果都出TRADE则都得1分，一方出CHEAT一方出TRADE则出CHEAT的一方得2分，出TRADE得一方减2分；如果都出CHEAT则都减1分。

现在给出不超过10个程序，每一个程序都是一个表达式，最终可以计算出的是TRADE还是CHEAT。每一个程序都和其他的程序玩10次，最终可以得到一个分数S，把某个程序和其他程序玩的最终分数S加起来，得到该程序的分数。请输出每一个程序的分数。

表达式中cond格式为:

```plaintext
< cond > ::= < memory > {= | #} {< command > | NULL}
< memory > ::= {MY | YOUR} LAST {1 | 2}
```

MY表示我出的，YOUR表示对方出的，LAST1表示上一轮，LAST2表示上上一轮，=表示相等，#表示不相等。

## 思路

将一个程序与剩余的程序都玩10次，得到分数，并累加即可。

问题的关键在于解析表达式，由于表达式计算可能涉及到我方或对方前两轮出的值，所以建立一个数组记录出过的值。条件表达式的计算涉及到多个AND OR的优先级计算，解决方式是先将每一个条件计算成布尔值，然后先计算AND，根据AND更新左右两侧的布尔值，再依次计算OR，最终得到条件表达式的值。

第二个问题是根据条件表达式是true还是false，执行THEN 或者 ELSE 语句。由于嵌套关系，IF对应的ELSE要排除掉中间成对出现的IF ELSE。

### 问题

#### 实现上的问题

#### 性能上的问题

## 实现

```JAVA {.line-numbers}
import java.io.*;
import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;

public class Main {
    public static void main(String[] args) throws IOException {
        BufferedReader reader = new BufferedReader(new InputStreamReader(System.in));
        StringBuilder sb = new StringBuilder();
        while (true) {
            String line = reader.readLine();
            if (line.equals("#")) {
                break;
            }
            sb.append(line);
        }

        //移除空白字符
        String programs = sb.toString().replaceAll("\\s+", "");

        //按.进行分割成不同的程序
        String[] stragies = programs.split("\\.");
        int[] scores = new int[stragies.length];
        Arrays.fill(scores, 0);
        for (int i = 0; i < stragies.length; i++) {
            for (int j = i + 1; j < stragies.length; j++) {
                calculateScore(i, stragies[i], j, stragies[j], scores);
            }
        }
        for (int i = 0; i < scores.length; i++) {
            System.out.printf("%3d\n", scores[i]);
        }
    }

    private static void calculateScore(int idx, String stragyi, int jdx, String stragyj, int[] scores) {
        char[] typei = new char[10];
        int[] scorei = new int[10];
        char[] typej = new char[10];
        int[] scorej = new int[10];

        //每一个程序都和对方玩10次
        for (int i = 0; i < 10; i++) {
            //计算这一回双方出的值
            typei[i] = parseStragy(stragyi, i, typei, typej);
            typej[i] = parseStragy(stragyj, i, typej, typei);
        }

        for (int i = 0; i < 10; i++) {
            int lastScorei = i - 1 < 0 ? 0 : scorei[i - 1];
            int lastScorej = i - 1 < 0 ? 0 : scorej[i - 1];
            if (typei[i] == 'T' && typej[i] == 'T') {
                scorei[i] = lastScorei + 1;
                scorej[i] = lastScorej + 1;
            } else if (typei[i] == 'T' && typej[i] == 'C') {
                scorei[i] = lastScorei - 2;
                scorej[i] = lastScorej + 2;
            } else if (typei[i] == 'C' && typej[i] == 'T') {
                scorei[i] = lastScorei + 2;
                scorej[i] = lastScorej - 2;
            } else {
                scorei[i] = lastScorei - 1;
                scorej[i] = lastScorej - 1;
            }
        }
        scores[idx] += scorei[9];
        scores[jdx] += scorej[9];
    }

    private static char parseStragy(String stragyi, int idx, char[] typei, char[] typej) {
        char result = calStatement(stragyi, idx, typei, typej);
        return result;
    }

    private static char calStatement(String stragyi, int idx, char[] typei, char[] typej) {
        int index = 0;

        if (stragyi.charAt(index) == 'T') {
            return 'T';
        } else if (stragyi.charAt(index) == 'C') {
            return 'C';
        } else {
            int thenIdx = stragyi.indexOf("THEN", index);
            String condition = stragyi.substring(index + 2, thenIdx);
            List<String> conds = split2Conds(condition);
            Boolean val = calculateVal(conds, idx, typei, typej);

            int ifcnt = 0;
            int nextElse = 0;
            for (int i = thenIdx + 4; i < stragyi.length(); i++) {
                if (stragyi.startsWith("IF", i)) {
                    ifcnt++;
                } else if (stragyi.startsWith("ELSE", i)) {
                    if (ifcnt == 0) {
                        nextElse = i;
                        break;
                    } else {
                        ifcnt--;
                    }
                }
            }
            if (val) {
                stragyi = stragyi.substring(thenIdx + 4, nextElse);
            } else {
                stragyi = stragyi.substring(nextElse + 4);
            }
            return calStatement(stragyi, idx, typei, typej);
        }
    }

    /**
     * 计算条件表达式的值，形如aaa AND bbb OR ccc，最终会计算出表达式是true还是false
     */
    private static Boolean calculateVal(List<String> conds, int idx, char[] my, char[] you) {
        List<Boolean> results = new ArrayList<>();
        for (int i = 0; i < conds.size(); i++) {
            String cond = conds.get(i);
            if (cond.equals("AND") || cond.equals("OR")) {
                results.add(true);
            } else {
                int index = 0;
                int subIndex;
                char[] tmp;

                if (cond.startsWith("MY")) {
                    tmp = my;
                } else {
                    tmp = you;
                }
                index = cond.indexOf("LAST");
                if (cond.charAt(index + 4) == '1') {
                    subIndex = idx - 1;
                } else {
                    subIndex = idx - 2;
                }
                if (cond.charAt(index + 5) == '=') {
                    if (cond.charAt(index + 6) == 'T') {
                        if (subIndex < 0) {
                            results.add(false);
                        } else {
                            if (tmp[subIndex] == 'T') {
                                results.add(true);
                            } else {
                                results.add(false);
                            }
                        }
                    } else if (cond.charAt(index + 6) == 'C') {
                        if (subIndex < 0) {
                            results.add(false);
                        } else {
                            if (tmp[subIndex] == 'C') {
                                results.add(true);
                            } else {
                                results.add(false);
                            }
                        }
                    } else {
                        if (subIndex < 0) {
                            results.add(true);
                        } else {
                            results.add(false);
                        }
                    }
                } else {
                    if (cond.charAt(index + 6) == 'T') {
                        if (subIndex < 0) {
                            results.add(true);
                        } else {
                            if (tmp[subIndex] == 'T') {
                                results.add(false);
                            } else {
                                results.add(true);
                            }
                        }
                    } else if (cond.charAt(index + 6) == 'C') {
                        if (subIndex < 0) {
                            results.add(true);
                        } else {
                            if (tmp[subIndex] == 'C') {
                                results.add(false);
                            } else {
                                results.add(true);
                            }
                        }
                    } else {
                        if (subIndex < 0) {
                            results.add(false);
                        } else {
                            results.add(true);
                        }
                    }
                }
            }
        }

        for (int i = 0; i < conds.size(); i++) {
            String cond = conds.get(i);
            if (cond.equals("AND")) {
                Boolean first = results.get(i - 1);
                Boolean second = results.get(i + 1);
                if (!(first && second)) {
                    results.set(i - 1, false);
                    results.set(i + 1, false);
                }
            }
        }

        Boolean first = results.get(0);
        for (int i = 0; i < conds.size(); i++) {
            String cond = conds.get(i);
            if (cond.equals("OR")) {
                Boolean second = results.get(i + 1);
                if (first || second) {
                    first = true;
                } else {
                    first = false;
                }
            }
        }
        return first;
    }

    private static List<String> split2Conds(String condition) {
        List<String> conds = new ArrayList<>();
        int start = 0;
        for (int i = 0; i < condition.length() - 2; i++) {
            if ((condition.charAt(i) == 'A' && condition.charAt(i + 1) == 'N' && condition.charAt(i + 2) == 'D')
                    || (condition.charAt(i) == 'O' && condition.charAt(i + 1) == 'R')) {
                String cond = condition.substring(start, i);
                conds.add(cond);
                if (condition.charAt(i) == 'A') {
                    conds.add("AND");
                    start = i + 3;
                } else {
                    conds.add("OR");
                    start = i + 2;
                }
            }
        }
        conds.add(condition.substring(start));
        return conds;
    }
}
```
