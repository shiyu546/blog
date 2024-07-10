---
title: 122.Trees on the level
date: 2024-07-10 14:43:01
mathjax: true
categories:
- [数据结构与算法,uva]
tags:
- 算法
- uva
description: "UVA 122"
---

## 题目

Trees are fundamental in many branches of computer science (Pun definitely intended). Current stateof-the art parallel computers such as Thinking Machines’ CM-5 are based on fat trees. Quad- and octal-trees are fundamental to many algorithms in computer graphics.

This problem involves building and traversing binary trees.

Given a sequence of binary trees, you are to write a program that prints a level-order traversal of each tree. In this problem each node of a binary tree contains a positive integer and all binary trees have have fewer than 256 nodes.

In a level-order traversal of a tree, the data in all nodes at a given level are printed in left-to-right order and all nodes at level k are printed before all nodes at level k + 1.

For example, a level order traversal of the tree on the right is:5, 4, 8, 11, 13, 4, 7, 2, 1.
{% asset_img 122_pic1.png 树的例子 %}

In this problem a binary tree is specified by a sequence of pairs ‘(n,s)’ where n is the value at the node whose path from the root is given by the string s. A path is given be a sequence of ‘L’s and ‘R’s where ‘L’ indicates a left branch and ‘R’ indicates a right branch. In the tree diagrammed above, the node containing 13 is specified by (13,RL), and the node containing 2 is specified by (2,LLR). The root node is specified by (5,) where the empty string indicates the path from the root to itself. A binary tree is considered to be completely specified if every node on all root-to-node paths in the tree is given a value exactly once.

### Input

The input is a sequence of binary trees specified as described above. Each tree in a sequence consists of several pairs ‘(n,s)’ as described above separated by whitespace. The last entry in each tree is ‘()’.No whitespace appears between left and right parentheses.

All nodes contain a positive integer. Every tree in the input will consist of at least one node and no more than 256 nodes. Input is terminated by end-of-file.

### Output

For each completely specified binary tree in the input file, the level order traversal of that tree should be printed. If a tree is not completely specified, i.e., some node in the tree is NOT given a value or a node is given a value more than once, then the string ‘not complete’ should be printed.

### Sample Input

(11,LL) (7,LLL) (8,R)
(5,) (4,L) (13,RL) (2,LLR) (1,RRR) (4,RR) ()
(3,L) (4,R) ()

### Sample Output

5 4 8 11 13 4 7 2 1
not complete

### 题意

树是一种很重要的数据结构，现给定一系列的节点，每个节点包含了值和位置信息(位置信息指当这一系列的节点构成一棵树时，该节点在树种的位置)，位置信息由一串'L'和'R'构成，表示从根节点开始到当前节点的路径，以案例中的树为例，节点值为4是根节点的直属左子节点，所以从根节点到当前节点的路径是'L'，节点值为2从根节点按左，左，右的顺序到达，所以位置信息为"LLR"，依此类推。

现给定很多个树的系列节点，判断给定的系列节点是否构成一棵树，如果构成，就按层级遍历的方式打印节点值；否则打印"not complete".

## 思路

从题意得知问题求解落在判断一系列的点是否构成树，构成树的一个充分条件就是判断每个节点是否有父节点(根节点除外)，再根据题意不能有重复节点，得到以下的判断条件：

1. 任何节点(除根节点)的父节点必须存在；
2. 节点不允许占用同一个位置，也就是系列节点出现两个位置相同的节点，也不构成树。

为了打印节点方便，我们按照树的层级关系先将节点排序，即先打印的节点在前面。然后判断每一个节点是否有父节点，如果有，然后打印树。例如以sample input第一个系列节点为例，先将节点系列
> (11,LL) (7,LLL) (8,R) (5,) (4,L) (13,RL) (2,LLR) (1,RRR) (4,RR)

按层级关系排序，得到
> (5,) (4,L) (8,R) (11,LL) (13,RL) (4,RR) (7,LLL) (2,LLR) (1,RRR)

然后依此判断各节点是否有父节点，都有，则按顺序遍历输出节点值。

### 问题

#### 实现上的问题

#### 性能上的问题

## 实现

```JAVA {.line-numbers}
//file
import java.util.*;
import java.util.stream.Collectors;

public class Main {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        while (scanner.hasNext()) {
            List<Node> nodes = new ArrayList<>();
            while (true) {
                //该循环读到系列节点结尾则跳出，每次读一个节点
                String line = scanner.next();
                if (line.equals("()")) {
                    break;
                }
                line = line.trim().replaceAll("\\(", "").replaceAll("\\)", "");
                String[] lines = line.split(",");
                int val = Integer.parseInt(lines[0]);
                String str = "";
                if (lines.length > 1) {
                    str = lines[1];
                }
                Node node = new Node(val, str);
                nodes.add(node);
            }
            //handler：排序
            List<Node> list = nodes.stream().sorted((s1, s2) -> {
                String s1Val = s1.str;
                String s2Val = s2.str;
                if (s1Val.length() > s2Val.length()) {
                    return 1;
                } else if (s1Val.length() < s2Val.length()) {
                    return -1;
                } else {
                    for (int i = 0; i < s1Val.length(); i++) {
                        if (s1Val.charAt(i) == 'L' && s2Val.charAt(i) == 'R') {
                            return -1;
                        } else if (s1Val.charAt(i) == 'R' && s2Val.charAt(i) == 'L') {
                            return 1;
                        }
                    }
                    return 0;
                }
            }).collect(Collectors.toList());

            Set<String> valSet = nodes.stream().map(s -> s.str).collect(Collectors.toSet());
            if (list.size() != valSet.size()) {
                System.out.println("not complete");
            } else {
                List<Integer> cols = new ArrayList<>();
                int i = 0;
                for (; i < list.size(); i++) {
                    String s1Val = list.get(i).str;
                    String parent = "";
                    if (s1Val.length() >= 1) {
                        parent = s1Val.substring(0, s1Val.length() - 1);
                    }
                    //判断是否有父节点
                    if (s1Val.equals("") || valSet.contains(parent)) {
                        cols.add(list.get(i).val);
                    } else {
                        System.out.println("not complete");
                        i = list.size() + 1;
                    }
                }
                if (i == list.size()) {
                    //print number
                    for (int j = 0; j < cols.size(); j++) {
                        if (j < cols.size() - 1) {
                            System.out.printf("%d ", cols.get(j));
                        } else {
                            System.out.printf("%d\n", cols.get(j));
                        }

                    }
                }
            }
        }

    }

    static class Node {
        int val;
        String str;

        public Node(int val, String str) {
            this.val = val;
            this.str = str;
        }
    }
}
```
