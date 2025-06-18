---
title: problem172 Calculator Language
date: 2025-06-15 10:50:25
mathjax: true
categories:
- [数据结构与算法,uva]
tags:
- 算法
- uva
description: "UVA problem172.Calculator Language <br>
- 表达式计算"
---

## 题目

Calculator Language (CL) supports assignment, positive and negative integers and simple arithmetic.The allowable characters in a CL statement are thus:

A..Z variable names
0..9 digits
\+ addition operator
\- subtraction operator
\* multiplication operator
/ integer division operator
= assignment operator
() brackets
_ negative sign

All operators have the same precedence and are right associative, thus 15-8-3 = 15-(8-3) = 10.As one would expect, brackets will force the expression within them to be evaluated first. Brackets may be nested arbitrarily deeply. An expression never has two operators next to each other (even if separated by a bracket), an assignment operator is always immediately preceded by a variable and the leftmost operator on a line is always an assignment. For readability, spaces may be freely inserted into an expression, except between a negative sign and a number. A negative sign will not appear before a variable. All variables are initialised to zero (0) and retain their values until changed explicitly.

Write a program that will accept and evaluate expressions written in this language. Each expression occupies one line and contains at least one assignment operator, and maybe more.

### Input

Input will consist of a series of lines, each line containing a correct CL expression. No line will be longer than 100 characters. The file will be terminated by a line consisting of a single ‘#’.

### Output

Output will consist of a series of lines, one for each line of the input. Each line will consist of a list of the final values of all variables whose value changes as a result of the evaluation of that expression. If more than one variable changes value, they should be listed in alphabetical order, separated by commas. If a variable changes value more than once in an expression, only the final value is output. A variable is
said to change value if its value after the expression has been evaluated is different from its value before the expression was evaluated. If no variables change value, then print the message ‘No Change’.Follow the format shown below exactly.

### SampleInput

A = B = 4
C = (D = 2)\*_2
C = D = 2 \* _2
F = C - D
E = D \* _10
Z = 10 / 3
\#

### SampleOutput

A = 4, B = 4
C = -4, D = 2
D = -4
No Change
E = 40
Z = 3

### 题意

题目给出了一个表达式，表达式由变量，数字和运算符和括号组成。每个变量的初始值为0，一个表达式中可能存在给多个变量赋值，如果一个表达式中变量的值改变了，就输出该变量。

## 思路

表达式以中缀表达式的格式给出，先将表达式转换成后缀表达式，然后计算表达式，遇到赋值则保存下给变量赋得值，然后和初始值比较，不同则输出。

### 问题

#### 实现上的问题

#### 性能上的问题

## 实现

```JAVA {.line-numbers}
import java.io.*;
import java.util.*;

public class Main {
    public static Map<String, Integer> variablesMap = new HashMap<>();

    public static void main(String[] args) throws IOException {
        BufferedReader reader = new BufferedReader(new InputStreamReader(System.in));
        while (true) {
            String line = reader.readLine();
            if (line.equals("#")) {
                break;
            }

            //将表达式转换成一个个的节点，包括变量、数字、运算符和括号
            List<Node> expressionNodes = parseLine(line);

            //将中缀表达式转换成后缀表达式，后缀表达式中没有括号
            List<Node> postFixExpressions = change2Postfix(expressionNodes);

            //计算表达式，返回的值中包括所有赋过值得变量
            Map<String, Integer> pairs = calculateVariables(postFixExpressions);
            List<MyPair> pairsList = new ArrayList<>();

            //过滤掉赋过值的变量但值没变化的变量
            for (Map.Entry<String, Integer> pair : pairs.entrySet()) {
                Integer val = variablesMap.get(pair.getKey());
                int calVal = pair.getValue();
                if ((val == null && calVal != 0) || (val != null && calVal != val)) {
                    MyPair tpPair = new MyPair();
                    tpPair.variable = pair.getKey();
                    tpPair.value = pair.getValue();

                    pairsList.add(tpPair);
                    variablesMap.put(pair.getKey(), pair.getValue());
                }
            }

            Collections.sort(pairsList, new Comparator<MyPair>() {
                @Override
                public int compare(MyPair o1, MyPair o2) {
                    return o1.variable.compareTo(o2.variable);
                }
            });
            if (pairsList.size() == 0) {
                System.out.println("No Change");
            } else {
                StringBuilder sb = new StringBuilder();
                for (int i = 0; i < pairsList.size(); i++) {
                    MyPair pair = pairsList.get(i);
                    if (i > 0) {
                        sb.append(", ");
                    }
                    sb.append(pair.variable);
                    sb.append(" = ");
                    sb.append(pair.value);
                }
                System.out.println(sb.toString());
            }
        }
    }

    /**
     * 后缀表达式计算：建立栈，如果是变量和数字，直接入栈，如果是运算符，则从栈中弹出两个数字，计算结果，将结果入栈。
     * 如果运算符是等号，则意味着有变量赋值，需要记录。
     */
    private static Map<String, Integer> calculateVariables(List<Node> postFixExpressions) {
        Map<String, Integer> changes = new HashMap<>();
        Stack<Node> stack = new Stack<>();
        for (int i = 0; i < postFixExpressions.size(); i++) {
            Node curNode = postFixExpressions.get(i);
            if (curNode.type == 1 || curNode.type == 2) {
                stack.push(curNode);
            } else if (curNode.type == 3) {
                Node secondNode = stack.pop();
                Node firstNode = stack.pop();
                if (curNode.value.equals("+")) {
                    int val = getVal(secondNode, changes) + getVal(firstNode, changes);
                    Node node = changeValue2Node(val);
                    stack.push(node);
                } else if (curNode.value.equals("-")) {
                    int val = getVal(firstNode, changes) - getVal(secondNode, changes);
                    Node node = changeValue2Node(val);
                    stack.push(node);
                } else if (curNode.value.equals("*")) {
                    int val = getVal(firstNode, changes) * getVal(secondNode, changes);
                    Node node = changeValue2Node(val);
                    stack.push(node);
                } else if (curNode.value.equals("/")) {
                    int val = getVal(firstNode, changes) / getVal(secondNode, changes);
                    Node node = changeValue2Node(val);
                    stack.push(node);
                } else if (curNode.value.equals("=")) {
                    int val = getVal(secondNode, changes);
                    Node node = changeValue2Node(val);
                    stack.push(node);

                    changes.put(firstNode.value, val);
                }
            }
        }
        return changes;
    }

    private static int getVal(Node node, Map<String, Integer> changes) {
        if (node.type == 1) {
            Integer tmpVal = changes.get(node.value);
            if (tmpVal != null) {
                return tmpVal;
            }
            Integer val = variablesMap.get(node.value);
            if (val == null) {
                return 0;
            } else {
                return val;
            }
        } else if (node.type == 2) {
            if (node.value.charAt(0) == '_') {
                String val = node.value.substring(1);
                return -Integer.parseInt(val);
            } else {
                return Integer.parseInt(node.value);
            }
        }
        return 0;
    }

    private static Node changeValue2Node(int value) {
        Node node = new Node();
        if (value < 0) {
            value = -value;
            node.value = "_" + value;
            node.type = 2;
        } else {
            node.value = String.valueOf(value);
            node.type = 2;
        }
        return node;
    }

    /**
     * 改变表达式为后缀表达式,主要规则为：建立栈和队列，如果是变量和数字，直接加入队列，如果是运算符和左括号，则直接入栈。如果是右括号，则连续出栈并加入队列直到栈顶是左括号。
     */
    private static List<Node> change2Postfix(List<Node> expressionNodes) {
        Stack<Node> stack = new Stack<Node>();
        List<Node> postExpression = new ArrayList<>();
        for (int i = 0; i < expressionNodes.size(); i++) {
            Node curNode = expressionNodes.get(i);
            if (curNode.type == 1 || curNode.type == 2) {
                postExpression.add(curNode);
            } else if (curNode.type == 3) {
                stack.push(curNode);
            } else if (curNode.type == 4) {
                if (curNode.value.equals("(")) {
                    stack.push(curNode);
                } else {
                    while (true) {
                        Node node = stack.pop();
                        if (node.value.equals("(")) {
                            break;
                        }
                        postExpression.add(node);
                    }
                }
            }
        }
        while (!stack.isEmpty()) {
            Node node = stack.pop();
            postExpression.add(node);
        }
        return postExpression;
    }

    private static List<Node> parseLine(String line) {
        int startPos = 0;

        List<Node> expressions = new ArrayList<>();

        for (int i = 0; i < line.length(); i++) {
            if (line.charAt(i) >= 'A' && line.charAt(i) <= 'Z') {
                startPos = i + 1;
                while (startPos < line.length()) {
                    if (line.charAt(startPos) >= 'A' && line.charAt(startPos) <= 'Z') {
                        startPos++;
                    } else {
                        break;
                    }
                }
                String variable = line.substring(i, startPos);
                Node node = new Node();
                node.value = variable;
                node.type = 1;
                expressions.add(node);
                i = startPos - 1;
            } else if (line.charAt(i) == '_' || line.charAt(i) >= '0' && line.charAt(i) <= '9') {
                startPos = i + 1;
                while (startPos < line.length()) {
                    if (line.charAt(startPos) == '_' || line.charAt(startPos) >= '0' && line.charAt(startPos) <= '9') {
                        startPos++;
                    } else {
                        break;
                    }
                }
                String digits = line.substring(i, startPos);
                Node node = new Node();
                node.value = digits;
                node.type = 2;
                expressions.add(node);
                i = startPos - 1;
            } else if (line.charAt(i) == '+' || line.charAt(i) == '-' || line.charAt(i) == '*' || line.charAt(i) == '/'
                    || line.charAt(i) == '=') {
                Node node = new Node();
                node.value = String.valueOf(line.charAt(i));
                node.type = 3;
                expressions.add(node);
            } else if (line.charAt(i) == '(' || line.charAt(i) == ')') {
                Node node = new Node();
                node.value = String.valueOf(line.charAt(i));
                node.type = 4;
                expressions.add(node);
            }
        }
        return expressions;
    }

    /**
     * type: 1:variable;2:digits;3:operator,4:bracket
     */
    public static class Node {
        public String value;
        public int type;
    }

    public static class MyPair {
        public String variable;
        public int value;
    }

}
```
