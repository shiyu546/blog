---
title: 124.Following Orders
date: 2024-08-02 09:45:34
mathjax: true
categories:
- [数据结构与算法,uva]
tags:
- 算法
- uva
description: "UVA 124.Following Orders <br>
- 给定一组变量和变量的大小关系，按字典序输出所有满足关系的变量序列."
---

## 题目

Order is an important concept in mathematics and in computer science. For example, Zorn’s Lemma states: "a partially ordered set in which every chain has an upper bound contains a maximal element."Order is also important in reasoning about the fix-point semantics of programs.

This problem involves neither Zorn’s Lemma nor fix-point semantics, but does involve order.

Given a list of variable constraints of the form x < y, you are to write a program that prints all orderings of the variables that are consistent with the constraints.

For example, given the constraints x < y and x < z there are two orderings of the variables x, y,and z that are consistent with these constraints: xyz and xzy.

### Input

The input consists of a sequence of constraint specifications. A specification consists of two lines: a list of variables on one line followed by a list of constraints on the next line. A constraint is given by a pair of variables, where ‘x y’ indicates that x < y.

All variables are single character, lower-case letters. There will be at least two variables, and no more than 20 variables in a specification.There will be at least one constraint, and no more than 50 constraints in a specification. There will be at least one, and no more than 300 orderings consistent with the constraints in a specification.

Input is terminated by end-of-file.

### Output

For each constraint specification, all orderings consistent with the constraints should be printed. Orderings are printed in lexicographical (alphabetical) order, one per line.

Output for different constraint specifications is separated by a blank line.

### SampleInput

a b f g
a b b f
v w x y z
v y x v z v w v

### SampleOutput

abfg
abgf
agbf
gabf

wxzvy
wzxvy
xwzvy
xzwvy
zwxvy
zxwvy

### 题意

对于每一组数据给出两行数据，第一行是多个变量，变量之间用空格分隔，第二行是变量的大小关系，每两个一组，表示前者小于后者，例如a b表示a < b.以示例为例

>a b f g
 a b b f

第一行*a b f g*表示共有4个变量a,b,f,g。第二行*a b b f*表示两对大小关系：a < b 和b < f。现要求输出所有满足大约束条件的字符串(一个字符串满足大小约束条件表示字符串中所有字符的顺序符合大小约束，例如a < b,那么abfg就满足条件，bafg就不满足，因为后者b在a的前面)，示例的所有输出如下：abfg,abgf,agbf,gabf,每一个字符串显示一行，且按字典序排序。

## 思路

输出所有的可能就是一个排列的过程，现在需找到一种方法排列所有可能的字符串，并过滤掉不满足约束条件的部分。这里我们采用插入排序类似的排列过程，通过初始的一个变量，不断加入变量，构造所有满足条件的序列，直至所有变量都加入序列。以*a b f g h*变量为例，过程如下：

1. 选定一个初始变量，这里选择a；
2. 将b加入a序列，共有两种可能：在a之前或之后，由于a < b,所以在a之前不满足约束条件，只能加入a之后构成ab序列；
3. 将f加入序列，同b类似，f只能加入最后构成abf序列；
4. 将g加入序列，由于g可以出现在任何位置，所以输出4个序列：abfg,abgf,agbf,gabf;
5. 将h加入序列，h需要加入4个序列，每一个生成5个序列，所以输出有20个序列。

有两条需要注意：

1. 将变量加入字符时可能有多种选择，每一种选择我们都需要保存；
2. 一个变量加入的序列可能不止一个，如5中所示，h需要加入的序列有4个。

我们采用了一个队列来存储序列，每加入一个变量时，队列中的序列都需要拿出来，并将变量加入序列之后的所有序列都投入到队列中去。需要注意的一是将变量加入序列时有多种少可能，我们需要知道序列中比变量大的最小的下标lessMin和比变量小的最大的下标biggerMax,序列中在lessMin左边和biggerMax右边的都可以插入；二是有多少个序列需要插入当前变量，很明显的一点是序列的长度如果与变量是第几个插入序列的值相同，表示序列已经插入过变量，这是就需要选取下一个变凉了。

```plaintext
//
//a加入队列
+-----+  
|  a  |----->poll(a)----->insert(ab)  
+-----+                     |
   +------------------------+
  \|/
+-----+
| ab  |----->poll(ab)----->insert(abf)
+-----+                       |
   +--------------------------+
  \|/
+-----+
| abf |----->poll(abf)------>insert(abfg,abgf,agbf,gabf)
+-----+                        |
   +---------------------------+
  \|/
+------+
| abfg | 
+------+
| abgf | 
+------+
| agbf | 
+------+
| gabf | 
+------+
```

### 问题

#### 实现上的问题

#### 性能上的问题

## 实现

```JAVA {.line-numbers}
//
import java.util.*;

public class Main {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        boolean first = true;
        while (scanner.hasNextLine()) {
            String characters = scanner.nextLine();
            String constraints = scanner.nextLine();
            String[] chars = characters.split(" ");
            String[] constrantsAry = constraints.split(" ");
            //construct DAG
            Map<String, Integer> charMap = new HashMap<>();
            for (int i = 0; i < chars.length; i++) {
                charMap.put(chars[i], i);
            }

            //graph是用来判断两个变量的关系，如果a小于b，则graph[a][b]==1
            int[][] graph = new int[chars.length][chars.length];
            for (int i = 0; i < chars.length; i++) {
                for (int j = 0; j < chars.length; j++) {
                    graph[i][j] = 0;
                }
            }

            for (int i = 0; i < constrantsAry.length; i += 2) {
                graph[charMap.get(constrantsAry[i])][charMap.get(constrantsAry[i + 1])] = 1;
            }

            //由于有一些大小关系是隐藏的，因此需要遍历直接关系找出所有的间接关系，例如a<b,b<f，则a<f是隐藏的关系，需要找出来。
            for (int i = 0; i < chars.length; i++) {
                LinkedList<Integer> queue = new LinkedList<>();
                for (int j = 0; j < chars.length; j++) {
                    if (graph[j][charMap.get(chars[i])] != 0) {
                        queue.offer(j);
                    }
                }
                while (!queue.isEmpty()) {
                    Integer index = queue.peek();
                    graph[index][charMap.get(chars[i])] = 1;
                    queue.poll();
                    for (int j = 0; j < chars.length; j++) {
                        if (graph[j][index] != 0) {
                            queue.offer(j);
                        }
                    }
                }
            }


            LinkedList<String> result = new LinkedList<>();
            result.offer(chars[0]);
            String peek = result.peek();
            result.poll();
            for (int i = 1; i < chars.length; i++) {
                String current = chars[i];
                while (peek != null && peek.length() <= i) {
                    //判断变量可以插入哪些位置
                    int lessMin = -1, biggerMax = -1;
                    for (int k = 0; k < peek.length(); k++) {
                        String sub = peek.substring(k, k + 1);
                        if (graph[charMap.get(sub)][charMap.get(current)] == 1) {
                            biggerMax = k;
                        }
                        if (graph[charMap.get(current)][charMap.get(sub)] == 1 && lessMin == -1) {
                            lessMin = k;
                        }
                    }

                    int startIndex = 0, endIndex = peek.length();
                    if (lessMin > -1) {
                        endIndex = lessMin;
                    }
                    if (biggerMax > -1) {
                        startIndex = biggerMax + 1;
                    }

                    for (int j = startIndex; j <= endIndex; j++) {
                        String str = peek.substring(0, j) + current + peek.substring(j);
                        result.offer(str);
                    }

                    peek = result.peek();
                    if (peek != null && peek.length() == chars.length) {
                        break;
                    }
                    result.poll();
                }
            }

            result.sort((str1, str2) -> {
                int length = str1.length();
                for (int i = 0; i < length; i++) {
                    if (str1.charAt(i) < str2.charAt(i)) {
                        return -1;
                    } else if (str1.charAt(i) > str2.charAt(i)) {
                        return 1;
                    }
                }
                return 0;
            });

            if (first) {
                first = false;
            } else {
                System.out.println();
            }
            for (int i = 0; i < result.size(); i++) {
                System.out.println(result.get(i));
            }

        }

    }
}
```
