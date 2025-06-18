---
title: problem171 Car Trialling
date: 2025-06-10 16:34:10
mathjax: true
categories:
- [数据结构与算法,uva]
tags:
- 算法
- uva
description: "UVA problem171.Car Trialling <br>
- 字符串模式匹配问题"
---

## 题目

Car trialling requires the following of carefully worded instructions. When setting a trial, the organiser places traps in the instructions to catch out the unwary.

Write a program to determine whether an instruction obeys the following rules, which are loosely based on real car trialling instructions. **BOLD-TEXT** indicates text as it appears in the instruction(case sensitive), '|' separates options of which exactly one must be chosen, and '..' expands, so A..D is equivalent to A|B|C|D.

instruction = navigational | time-keeping | navigational **AND** time-keeping
navigational = directional | navigational **AND THEN** directional
directional = how direction | how direction where
how = **GO** | **GO** when | **KEEP**
direction = **RIGHT** | **LEFT**
when = **FIRST** | **SECOND** | **THIRD**
where = **AT** sign
sign = "signwords"
signwords = s-word | signwords s-word
s-word = letter | s-word letter
letter = A..Z | .
time-keeping = record | change
record = **RECORD TIME**
change = cas **TO** nnn **KMH**
cas = **CHANGE AVERAGE SPEED** | **CAS**
nnn = digit | nnn digit
digit = 0..9

Note that s-word and nnn are sequences of letters and digits respectively, with no intervening spaces. There will be one or more spaces between items except before a period (.), after the opening speech marks or before the closing speech marks.

### Input

Each input line will consist of not more than 75 characters. The input will be terminated by a line consisting of a single ‘#’.

### Output

The output will consist of a series of sequentially numbered lines, either containing the valid instruction, or the text ‘Trap!’ if the line did not obey the rules. The line number will be right justified in a field of 3 characters, followed by a full-stop, a single space, and the instruction, with sequences of more than one space reduced to single spaces.

### SampleInput

KEEP LEFT AND THEN GO RIGHT
CAS TO 20 KMH
GO FIRST RIGHT AT "SMITH ST." AND CAS TO 20 KMH
GO 2nd RIGHT
GO LEFT AT "SMITH STREET AND RECORD TIME
KEEP RIGHT AND THEN RECORD TIME
\#

### SampleOutput

1. KEEP LEFT AND THEN GO RIGHT
2. CAS TO 20 KMH
3. GO FIRST RIGHT AT "SMITH ST." AND CAS TO 20 KMH
4. Trap!
5. Trap!
6. Trap!

### 题意

题目给出了一系列的规则，这些规则共同解释了什么是指令。其中几种符号的解释：

* '|' : 或的意思，表示至少由一种组成。例如``letter = A..Z | .``表示letter要么是A到Z中的任意一个，要么是.。
* 粗体字符：表示文本。

现在给出一串字符串，判断该字符串是否是指令。

## 思路

该题和134是同一类型的题，都是通过规则构造出状态机，然后根据状态转移判断字符串能否到达状态机的终点。

状态机如下：

{% asset_img 171_pic1.jpg 图示%}

### 问题

#### 实现上的问题

#### 性能上的问题

## 实现

```JAVA {.line-numbers}
import java.io.*;
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

public class Main {

    public static int length = 24;

    public static int end = 23;
    public static int[][] graphs = new int[length][length];
    public static String[] nodes = {"START", "GO", "KEEP", "FIRST", "SECOND",
            "THIRD", "RECORD", "TIME", "CHANGE", "AVERAGE",
            "SPEED", "CAS", "RIGHT", "LEFT", "AND",
            "THEN", "AT", "left\"", "s-words", "right\"",
            "TO", "nnn", "KMH", "END"};
    public static Map<String, Integer> map = new HashMap<>();

    static {
        map.put("START", 0);
        map.put("GO", 1);
        map.put("KEEP", 2);
        map.put("FIRST", 3);
        map.put("SECOND", 4);
        map.put("THIRD", 5);
        map.put("RECORD", 6);
        map.put("TIME", 7);
        map.put("CHANGE", 8);
        map.put("AVERAGE", 9);
        map.put("SPEED", 10);
        map.put("CAS", 11);
        map.put("RIGHT", 12);
        map.put("LEFT", 13);
        map.put("AND", 14);
        map.put("THEN", 15);
        map.put("AT", 16);
        map.put("left\"", 17);
        map.put("s-words", 18);
        map.put("right\"", 19);
        map.put("TO", 20);
        map.put("nnn", 21);
        map.put("KMH", 22);
        map.put("END", 23);
    }

    public static void main(String[] args) throws IOException {
        BufferedReader reader;
        reader = new BufferedReader(new InputStreamReader(System.in));
        initGraphs(graphs);
        int count = 1;
        while (true) {
            String instructions = reader.readLine();
            if (instructions.equals("#")) {
                break;
            }
            List<CommandNode> words = parseInstructions(instructions);
            boolean res = validWords(words);

            StringBuilder sb = new StringBuilder();
            for (int i = 0; i < words.size(); i++) {
                String command = words.get(i).command;
                if (i == 0) {
                    sb.append(command);
                } else if (words.get(i - 1).command.equals("left\"")) {
                    sb.append(command);
                } else {
                    if (command.equals("right\"")) {
                        sb.append("\"");
                    } else {
                        sb.append(" ");
                        if (command.equals("left\"")) {
                            sb.append("\"");
                        } else {
                            sb.append(command);
                        }
                    }
                }
            }
            String instructionRes = sb.toString();

            if (res) {
                System.out.printf("%3d. %s\n", count, instructionRes);
            } else {
                System.out.printf("%3d. Trap!\n", count);
            }
            count++;
        }
    }

    /**
     * 判断一句指令是否有效
     */
    private static boolean validWords(List<CommandNode> words) {
        int start = 0;
        for (int i = 0; i < words.size(); i++) {
            String word = words.get(i).command;
            int type = words.get(i).type;
            Integer index;
            if (type == 1 && wordIsSword(word)) {
                index = 18;
            } else {
                index = map.get(word);
                if (index == null) {
                    if (wordIsNumber(word)) {
                        index = 21;
                    }
                }
            }

            if (index == null) {
                return false;
            }
            if (graphs[start][index] == 1) {
                start = index;
            } else {
                return false;
            }
        }

        if (graphs[start][end] == 1) {
            return true;
        }
        return false;
    }

    private static boolean wordIsSword(String word) {
        for (int i = 0; i < word.length(); i++) {
            if ((word.charAt(i) >= 'A' && word.charAt(i) <= 'Z') || word.charAt(i) == '.') {
                continue;
            } else {
                return false;
            }
        }
        return true;
    }

    private static boolean wordIsNumber(String word) {
        for (int i = 0; i < word.length(); i++) {
            if (word.charAt(i) >= '0' && word.charAt(i) <= '9') {
                continue;
            } else {
                return false;
            }
        }
        return true;
    }

    /**
     * 指令以字符串形式存在，解析成数组
     */
    private static List<CommandNode> parseInstructions(String instructions) {
        List<CommandNode> words = new ArrayList<>();
        int pos = 0, index = 0;
        boolean leftQuote = false, rightQuote = false;
        while (index < instructions.length()) {
            if (instructions.charAt(index) == ' ') {
                if (pos == index) {
                    index++;
                    pos++;
                } else {
                    String word = instructions.substring(pos, index);
                    if (leftQuote) {
                        CommandNode node = new CommandNode();
                        node.command = word;
                        node.type = 1;
                        words.add(node);
                    } else {
                        CommandNode node = new CommandNode();
                        node.command = word;
                        node.type = 0;
                        words.add(node);
                    }

                    pos = index + 1;
                    index = pos;
                }
            } else if (instructions.charAt(index) == '"') {
                if (!leftQuote && !rightQuote) {
                    leftQuote = true;
                } else if (leftQuote && !rightQuote) {
                    leftQuote = false;
                    rightQuote = true;
                }

                if (leftQuote) {
                    CommandNode node = new CommandNode();
                    node.command = "left\"";
                    node.type = 0;
                    words.add(node);
                    pos = index + 1;
                    index = pos;
                } else if (rightQuote) {
                    if (pos < index) {
                        String lastWord = instructions.substring(pos, index);
                        CommandNode node = new CommandNode();
                        node.command = lastWord;
                        node.type = 1;
                        words.add(node);
                    }

                    CommandNode quoteNode = new CommandNode();
                    quoteNode.command = "right\"";
                    quoteNode.type = 0;
                    words.add(quoteNode);
                    pos = index + 1;
                    index = pos;
                    leftQuote = false;
                    rightQuote = false;
                }
            } else {
                index++;
            }
        }

        if (pos < index) {
            String lastWord = instructions.substring(pos, index);
            if (leftQuote) {
                CommandNode node = new CommandNode();
                node.command = lastWord;
                node.type = 1;
                words.add(node);
            } else {
                CommandNode node = new CommandNode();
                node.command = lastWord;
                node.type = 0;
                words.add(node);
            }
        }
        return words;
    }

    /**
     * 初始化状态机
     */
    private static void initGraphs(int[][] graphs) {
        for (int i = 0; i < length; i++) {
            for (int j = 0; j < length; j++) {
                graphs[i][j] = 0;
            }
        }
        graphs[0][6] = 1;
        graphs[0][8] = 1;
        graphs[0][11] = 1;
        graphs[0][1] = 1;
        graphs[0][2] = 1;

        graphs[1][3] = 1;
        graphs[1][4] = 1;
        graphs[1][5] = 1;
        graphs[1][12] = 1;
        graphs[1][13] = 1;

        graphs[2][12] = graphs[2][13] = 1;
        graphs[3][12] = graphs[3][13] = 1;
        graphs[4][12] = graphs[4][13] = 1;
        graphs[5][12] = graphs[5][13] = 1;

        graphs[6][7] = 1;
        graphs[7][23] = 1;

        graphs[8][9] = 1;
        graphs[9][10] = 1;
        graphs[10][20] = 1;
        graphs[11][20] = 1;

        graphs[12][14] = graphs[12][16] = graphs[12][23] = 1;
        graphs[13][14] = graphs[13][16] = graphs[13][23] = 1;

        graphs[14][6] = graphs[14][8] = graphs[14][11] = graphs[14][15] = 1;
        graphs[15][1] = graphs[15][2] = 1;

        graphs[16][17] = 1;
        graphs[17][18] = 1;

        graphs[18][18] = graphs[18][19] = 1;
        graphs[19][14] = graphs[19][23] = 1;

        graphs[20][21] = 1;
        graphs[21][22] = 1;
        graphs[22][23] = 1;
    }

    public static class CommandNode {
        public String command;
        public int type;
    }
}
```
