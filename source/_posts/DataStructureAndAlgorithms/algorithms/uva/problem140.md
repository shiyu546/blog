---
title: problem140 Bandwidth
date: 2024-10-17 21:04:51
mathjax: true
categories:
- [数据结构与算法,uva]
tags:
- 算法
- uva
description: "UVA 140.Bandwidth <br>
- 寻找图中节点按顺序摆放时最近的摆放方式"
---

## 题目

Given a graph (V,E) where V is a set of nodes and E is a set of arcs in $V\times V$ , and an ordering on the elements in V , then the bandwidth of a node v is defined as the maximum distance in the ordering between v and any node to which it is connected in the graph. The bandwidth of the ordering is then defined as the maximum of the individual bandwidths.For example, consider the graph on the right:
{% asset_img 140_pic1.png 图 %}

This can be ordered in many ways, two of which are illustrated below:
{% asset_img 140_pic2.png 节点顺序 %}
For these orderings, the bandwidths of the nodes (in order) are 6, 6, 1, 4, 1, 1, 6, 6 giving an ordering bandwidth of 6, and 5, 3, 1, 4, 3, 5, 1, 4 giving an ordering bandwidth of 5.

Write a program that will find the ordering of a graph that minimises the bandwidth.

### Input

Input will consist of a series of graphs. Each graph will appear on a line by itself. The entire file will be terminated by a line consisting of a single ‘#’. For each graph, the input will consist of a series of records separated by ‘;’. Each record will consist of a node name (a single upper case character in the the range ‘A’ to ‘Z’), followed by a ‘:’ and at least one of its neighbours. The graph will contain no more than 8 nodes.

### Output

Output will consist of one line for each graph, listing the ordering of the nodes followed by an arrow(->) and the bandwidth for that ordering. All items must be separated from their neighbours by exactly one space. If more than one ordering produces the same bandwidth, then choose the smallest in lexicographic ordering, that is the one that would appear first in an alphabetic listing.

### SampleInput

A:FB;B:GC;D:GC;F:AGH;E:HD
\#

### SampleOutput

A B C F G D H E -> 3

### 题意

题目给出了一个图(图由图上的顶点集合V和边集合E表示)，把图的顶点按顺序排列得到一个数组，记为$(a_1,a_2,...,a_n)$,定义任意顶点$a_k$的bandwidth为$a_k$到与其相邻的顶点的距离的最大值。

以示例图为例，共有A,B,C,D,E,F,G,H共8个顶点，8个顶点任意排列可以得到$8!=40320$种排列，取其中一种排列(A,B,C,D,E,F,G,H),每一个顶点与其在图中直接相连的点都有一个distance值，例如A与B，F直接相连，则distance(A,B)为A，B在数组中的距离(顶点的距离由顶点在数组中相隔几个元素表示，这里A与B相邻，则distance(A,B)=1).同理distance(A,F)=5(A与F相隔B,C,D,E四个顶点)，distance(A,C)，distance(A,D)，distance(A,E)，distance(A,G)，distance(A,H)则不存在。顶点A所有存在的distance值的最大值称为顶点A的bandwidth值。
$$ bandwidth(A)=max(distance(A,x)) \quad\quad x是顶点且distance(A,x)存在 $$
在这个排列中，每一个顶点都有一个bandwidth值，所有顶点bandwidth值得最大值就称为这个排列得bandwidth值。

现在在所有的排列中，找出bandwidth值最小得排列，如果两个排列得bandwidth值相同，则输出字典序较小得排列。

## 思路

图中的顶点都由一个大写字母表示，假设有8个点，不妨将这8个点对应到1，2，3，4，5，6，7，8这8个数字上，按照从小到大的顺序遍历8个数字的全排列，并计算排列的bandwidth值，即可求出bandwidth值最小的排列。

现在问题转换为找到一种方式按照从小到大排列这8个数字，这里的规则是从右往左找到第一组两个相邻的数字ab，有a < b,从a右侧找到比a大但是小于其他数字的数字(也就是a往右的所有数字比a大的最小值)c，交换a和c，并对交换之后c后面的数字按从小到大排序.这里给出5位数字的排列顺序：

```plaintext

12345
12354   (从右往左数，35是第一组满足3<5的数，3后比3大的最小值是4，所以3和4交换，交换之后4后面的数字按从小到大排序)
12435
12453
12534
12543
...
54321
```

### 问题

#### 实现上的问题

#### 性能上的问题

本题采用的是暴力破解的方法，根据节点的位置以及遍历方式，是可以消除掉部分不必要的遍历(所谓的剪枝)，但实现上面就会复杂很多。

## 实现

```JAVA .{line-numbers}
//source code
import java.util.Arrays;
import java.util.HashMap;
import java.util.Map;
import java.util.Scanner;

public class Main {
    private static int[] node = new int[26];

    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        while (true) {
            String graph = scanner.nextLine();
            if ("#".equals(graph)) {
                break;
            }
            Arrays.fill(node, 0);
            for (int i = 0; i < graph.length(); i++) {
                if (graph.charAt(i) >= 'A' && graph.charAt(i) <= 'Z') {
                    node[graph.charAt(i) - 'A'] = 1;
                }
            }
            int nodeNumber = 0;
            Map<Character, Integer> node2OrderMap = new HashMap<>();
            Map<Integer, Character> order2NodeMap = new HashMap<>();

            for (int i = 0; i < node.length; i++) {
                if (node[i] == 1) {
                    nodeNumber++;
                    order2NodeMap.put(nodeNumber, (char) ('A' + i));
                    node2OrderMap.put((char) ('A' + i), nodeNumber);
                }
            }
            int[][] graphArys = new int[nodeNumber + 1][nodeNumber + 1];
            for (int i = 0; i < graphArys.length; i++) {
                for (int j = 0; j < graphArys.length; j++) {
                    graphArys[i][j] = 0;
                }
            }
            String[] edges = graph.split(";");
            for (String edge : edges) {
                String[] connect = edge.split(":");
                char first = connect[0].charAt(0);
                for (int i = 0; i < connect[1].length(); i++) {
                    char second = connect[1].charAt(i);
                    graphArys[node2OrderMap.get(first)][node2OrderMap.get(second)] = 1;
                    graphArys[node2OrderMap.get(second)][node2OrderMap.get(first)] = 1;
                }
            }

            int[] numbers = new int[nodeNumber + 1];
            for (int i = 1; i < numbers.length; i++) {
                numbers[i] = i;
            }
            int minBandwidth = Integer.MAX_VALUE;
            int[] minAry = new int[nodeNumber + 1];
            Distance distance = calculateBandwidth(numbers, graphArys, order2NodeMap);
            if (distance.bandwidth < minBandwidth) {
                minBandwidth = distance.bandwidth;
                System.arraycopy(numbers, 0, minAry, 0, numbers.length);
            }
            while (getNextNumbers(numbers, distance)) {
                distance = calculateBandwidth(numbers, graphArys, order2NodeMap);
                if (distance.bandwidth < minBandwidth) {
                    minBandwidth = distance.bandwidth;
                    System.arraycopy(numbers, 0, minAry, 0, numbers.length);
                }
            }
            StringBuilder sb = new StringBuilder();
            for (int i = 1; i < minAry.length; i++) {
                sb.append(order2NodeMap.get(minAry[i]));
                sb.append(" ");
            }
            sb.append("-> ");
            sb.append(minBandwidth);
            sb.append("\n");
            System.out.printf(sb.toString());
        }

    }

    /**
     * 寻找排列的下一个排列
     */
    private static boolean getNextNumbers(int[] numbers, Distance distance) {
        for (int i = numbers.length - 1; i > 1; i--) {
            if (numbers[i] > numbers[i - 1]) {
                int minIndexVal = Integer.MAX_VALUE, minIndex = i;
                for (int j = i; j < numbers.length; j++) {
                    if (numbers[j] > numbers[i - 1] && numbers[j] < minIndexVal) {
                        minIndexVal = numbers[j];
                        minIndex = j;
                    }
                }
                int temp = numbers[minIndex];
                numbers[minIndex] = numbers[i - 1];
                numbers[i - 1] = temp;
                Arrays.sort(numbers, i, numbers.length);
                return true;
            }
        }
        //the numbers is biggest
        return false;
    }

    /**
     * 计算排列bandwidth
     */
    private static Distance calculateBandwidth(int[] numbers, int[][] graphArys, Map<Integer, Character> order2NodeMap) {
        int max = -1;
        int firstIndex = 1, secondIndex = 1;
        for (int i = 1; i < numbers.length; i++) {
            int first = numbers[i];
            for (int j = 1; j < graphArys.length; j++) {
                if (graphArys[first][j] == 1) {
                    int second = j;
                    for (int k = 1; k < numbers.length; k++) {
                        if (numbers[k] == second) {
                            int distance = Math.abs(k - i);
                            if (distance > max) {
                                max = distance;
                                firstIndex = i;
                                secondIndex = k;
                            }
                        }
                    }
                }
            }
        }
        Distance distance = new Distance(max, firstIndex, secondIndex);
        return distance;
    }

    public static class Distance {
        public Distance(int bandwidth, int startIndex, int endIndex) {
            this.bandwidth = bandwidth;
            this.startIndex = startIndex;
            this.endIndex = endIndex;
        }

        private int bandwidth;
        private int startIndex;
        private int endIndex;
    }
}
```
