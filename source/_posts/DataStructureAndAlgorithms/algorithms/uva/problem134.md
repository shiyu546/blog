---
title: problem134 Loglan—A Logical Language
date: 2024-10-01 10:04:10
mathjax: true
categories:
- [数据结构与算法,uva]
tags:
- 算法
- uva
description: "UVA 134.Loglan—A Logical Language <br>
- 正则表达式匹配的简化版本"
---

## 题目

Loglan is a synthetic speakable language designed to test some of the fundamental problems of linguistics,such as the Sapir Whorf hypothesis. It is syntactically unambiguous, culturally neutral and metaphysically parsimonious. What follows is a gross over-simplification of an already very small grammar of some 200 rules.

Loglan sentences consist of a series of words and names, separated by spaces, and are terminated by a period (.=>. Loglan words all end with a vowel; names, which are derived extra-linguistically, end with a consonant. Loglan words are divided into two classes — little words which specify the structure of a sentence, and predicates which have the form CCVCV or CVCCV where C represents a consonant and V represents a vowel (see examples later).

The subset of Loglan that we are considering uses the following grammar:

```plaintext
//file
A                 =>  a | e | i | o | u
MOD               =>  ga | ge | gi | go | gu
BA                =>  ba | be | bi | bo | bu
DA                =>  da | de | di | do | du
LA                =>  la | le | li | lo | lu
NAM               =>  {all names}
PREDA             =>  {all predicates}
< sentence >      =>  < statement > | < predclaim >
< predclaim >     =>  < predname > BA < preds > | DA < preds >
< preds >         =>  < predstring > | < preds > A < predstring >
< predname >      =>  LA < predstring > | NAM
< predstring >    =>  PREDA | < predstring > PREDA
< statement >     =>  < predname > < verbpred > < predname > | < predname > < verbpred >
< verbpred >      =>  MOD < predstring >
```

Write a program that will read a succession of strings and determine whether or not they are correctly formed Loglan sentences.

### Input

Each Loglan sentence will start on a new line and will be terminated by a period (.). The sentence may occupy more than one line and words may be separated by more than one space. The input will be terminated by a line containing a single ‘#’. You can assume that all words will be correctly formed.

### Output

Output will consist of one line for each sentence containing either ‘Good’ or ‘Bad!’.

### SampleInput

la mutce bunbo mrenu bi ditca.
la fumna bi le mrenu.
djan ga vedma le negro ketpi.
\#

### SampleOutput

Good
Bad!
Good

### 题意

题目给出了一些规则，指出A由B构成，例如

```plaintext

A                 =>  a | e | i | o | u
```

表示A可由a或e或i或o或u表示，其中一些规则含有递归规则，例如

```plaintext

< predstring >    =>  PREDA | < predstring > PREDA
```

表示 < predstring > 由PREDA或者< predstring > PREDA表示 ,递归< predstring >,< predstring >由1个或多个PREDA表示。
最后给定一个字符串句子，判断该句子能否表示为< sentence >.

## 思路

判断一个句子是否是< sentence >,可以采用类似正则表达式的判断方式，使用有限状态自动机判断句子是否能表示< sentence >.表达式分为两种：一种需要依赖其他的表达式才能解释，一种是最基本的表达式无需其他表达式能解释。例如PREDA表示所有的predicates，则不需要其他表达式解释；< preds >依赖< predstring >才能解释。

```plaintext

                              +----<predname> <verbpred> <predname>
                              |
            +----<statement>--+
            |                 |
            |                 +----<predname> <verbpred>
<sentence>--|
            |                 +----<predname> BA <preds>
            |                 |
            +----<predclaim>--+
                              |
                              +----DA <preds>

满足展开后的四种句型都是< sentence >,我们得到状态转移图:

    +--------+     <predname>     +-----+   <verbpred>        +-----+
--->| start  |------------------->|     |-------------------->|     |
    +--------+                    +-----+                     +-----+
      |                              |                           |                 
      |                              |-------------+             |                 
      |                              |BA       any |             |<predname>                  
      |                              |             |             |
      |                             \|/            +----------> \|/
      |      DA                    +-----+      <preds>        +-----+
      +--------------------------->|     |-------------------->| END |
                                   +-----+                     +-----+

其中部分状态转移依赖于其他表达式，我们可以完全展开，得到一个最基本的表达式状态转移图(any表示不需要任何条件即可转换到下一状态).                             
```

该依赖集合不存在二义性，所以不存在递归匹配的问题，问题得到了很大的简化。再采用手工的方式构建整个有限状态自动机。

### 问题

#### 实现上的问题

主要是输入的处理上，需要一个一个单词的读。

#### 性能上的问题

## 实现

```JAVA .{line-numbers}

import java.util.*;

public class Main {
    public static Set<Character> vowel = new HashSet<>();

    static {
        vowel.add('a');
        vowel.add('e');
        vowel.add('i');
        vowel.add('o');
        vowel.add('u');
    }

    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        Node[] nodes = new Node[13];
        for (int i = 1; i < nodes.length; i++) {
            nodes[i] = new Node();
        }
        createGraph(nodes);
        Node startNode = nodes[1];
        Node endNode = nodes[12];
        while (true) {
            List<String> words = new ArrayList<>();
            String word = scanner.next();
            if ("#".equals(word)) {
                break;
            }
            while (true) {
                if (word.contains(".")) {
                    word = word.substring(0, word.length() - 1);
                    if (!word.isEmpty()) {
                        words.add(word);
                    }
                    break;
                } else {
                    words.add(word);
                }
                word = scanner.next();
            }

            if (isMatch(startNode, words, endNode)) {
                System.out.println("Good");
            } else {
                System.out.println("Bad!");
            }
        }
    }

    public static enum LineType {
        A,
        MOD,
        BA,
        DA,
        LA,
        NAM,
        PREDA,
        ANY,
        NONE
    }

    public static class Node {
        public List<LinePair> next;

        public Node() {
        }
    }

    public static class LinePair {
        public Node node;
        public LineType lineType;

        public LinePair(Node node, LineType lineType) {
            this.node = node;
            this.lineType = lineType;
        }
    }

    public static boolean isMatch(Node startNode, List<String> words, Node endNode) {
        Node curNode = startNode;
        label1:
        for (String word : words) {
            LineType wordLineType = getLineType(word);
            if (curNode.next == null) {
                return false;
            }
            for (LinePair pair : curNode.next) {
                if (pair.lineType.equals(wordLineType)) {
                    curNode = pair.node;
                    continue label1;
                }
            }
            return false;
        }
        if (curNode == endNode) {
            return true;
        }
        for (LinePair pair : curNode.next) {
            if (pair.lineType.equals(LineType.ANY) && pair.node == endNode) {
                return true;
            }
        }
        return false;
    }

    public static LineType getLineType(String word) {
        if (word.equals("a") || word.equals("e") || word.equals("i")
                || word.equals("o") || word.equals("u")) {
            return LineType.A;
        } else if (word.equals("ga") || word.equals("ge") || word.equals("gi")
                || word.equals("go") || word.equals("gu")) {
            return LineType.MOD;
        } else if (word.equals("ba") || word.equals("be") || word.equals("bi")
                || word.equals("bo") || word.equals("bu")) {
            return LineType.BA;
        } else if (word.equals("da") || word.equals("de") || word.equals("di")
                || word.equals("do") || word.equals("du")) {
            return LineType.DA;
        } else if (word.equals("la") || word.equals("le") || word.equals("li")
                || word.equals("lo") || word.equals("lu")) {
            return LineType.LA;
        } else if (isName(word)) {
            return LineType.NAM;
        } else if (isPredicate(word)) {
            return LineType.PREDA;
        }
        return LineType.NONE;
    }

    public static void createGraph(Node[] nodes) {
        nodes[1].next = new ArrayList<>();
        nodes[1].next.add(new LinePair(nodes[2], LineType.LA));
        nodes[1].next.add(new LinePair(nodes[4], LineType.NAM));
        nodes[1].next.add(new LinePair(nodes[9], LineType.DA));

        nodes[2].next = new ArrayList<>();
        nodes[2].next.add(new LinePair(nodes[3], LineType.PREDA));

        nodes[3].next = new ArrayList<>();
        nodes[3].next.add(new LinePair(nodes[3], LineType.PREDA));
        nodes[3].next.add(new LinePair(nodes[5], LineType.MOD));
        nodes[3].next.add(new LinePair(nodes[9], LineType.BA));

        nodes[4].next = new ArrayList<>();
        nodes[4].next.add(new LinePair(nodes[5], LineType.MOD));
        nodes[4].next.add(new LinePair(nodes[9], LineType.BA));

        nodes[5].next = new ArrayList<>();
        nodes[5].next.add(new LinePair(nodes[6], LineType.PREDA));

        nodes[6].next = new ArrayList<>();
        nodes[6].next.add(new LinePair(nodes[6], LineType.PREDA));
        nodes[6].next.add(new LinePair(nodes[7], LineType.LA));
        nodes[6].next.add(new LinePair(nodes[12], LineType.NAM));
        nodes[6].next.add(new LinePair(nodes[12], LineType.ANY));

        nodes[7].next = new ArrayList<>();
        nodes[7].next.add(new LinePair(nodes[8], LineType.PREDA));

        nodes[8].next = new ArrayList<>();
        nodes[8].next.add(new LinePair(nodes[8], LineType.PREDA));
        nodes[8].next.add(new LinePair(nodes[12], LineType.ANY));

        nodes[9].next = new ArrayList<>();
        nodes[9].next.add(new LinePair(nodes[10], LineType.PREDA));

        nodes[10].next = new ArrayList<>();
        nodes[10].next.add(new LinePair(nodes[10], LineType.PREDA));
        nodes[10].next.add(new LinePair(nodes[11], LineType.A));
        nodes[10].next.add(new LinePair(nodes[12], LineType.ANY));

        nodes[11].next = new ArrayList<>();
        nodes[11].next.add(new LinePair(nodes[10], LineType.PREDA));
    }

    public static boolean isPredicate(String word) {
        if (word.length() == 5) {
            if ((!vowel.contains(word.charAt(0)) &&
                    !vowel.contains(word.charAt(1)) &&
                    vowel.contains(word.charAt(2)) &&
                    !vowel.contains(word.charAt(3)) &&
                    vowel.contains(word.charAt(4)))
                    || (!vowel.contains(word.charAt(0)) &&
                    vowel.contains(word.charAt(1)) &&
                    !vowel.contains(word.charAt(2)) &&
                    !vowel.contains(word.charAt(3)) &&
                    vowel.contains(word.charAt(4)))) {
                return true;
            }
        }
        return false;
    }

    public static boolean isName(String word) {
        if (word == null || word.length() == 0) {
            return false;
        }
        if (!vowel.contains(word.charAt(word.length() - 1))) {
            return true;
        }
        return false;
    }
}
```
