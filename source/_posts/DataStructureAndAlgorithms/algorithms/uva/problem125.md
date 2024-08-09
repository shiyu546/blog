---
title: 125.Numbering Paths
date: 2024-08-08 17:46:10
mathjax: true
categories:
- [数据结构与算法,uva]
tags:
- 算法
- uva
description: "UVA 125.Numbering Paths <br>
- 计算有向图两点之间的路径条数"
---

## 题目

Problems that process input and generate a simple “yes” or “no” answer are called decision problems.

One class of decision problems, the NP-complete problems, are not amenable to general efficient solutions.Other problems may be simple as decision problems, but enumerating all possible “yes” answers may be very difficult (or at least time-consuming).

This problem involves determining the number of routes available to an emergency vehicle operating in a city of one-way streets.

Given the intersections connected by one-way streets in a city, you are to write a program that determines the number of different routes between each intersection. A route is a sequence of one-way streets connecting two intersections.

Intersections are identified by non-negative integers. A one-way street is specified by a pair of intersections. For example, j k indicates that there is a one-way street from intersection j to intersection k. Note that two-way streets can be modeled by specifying two one-way streets: j k and k j.

Consider a city of four intersections connected by the following one-way streets:
0 1
0 2
1 2
2 3
There is one route from intersection 0 to 1, two routes from 0 to 2 (the routes are 0 -> 1 ! 2 and 0 -> 2), one route from 0 to 3, one route from 1 to 2, one route from 1 to 3, one route from 2 to 3, and no other routes.

It is possible for an infinite number of different routes to exist. For example if the intersections above are augmented by the street 3 2, there is still only one route from 0 to 1, but there are infinitely many different routes from 0 to 2. This is because the street from 2 to 3 and back to 2 can be repeated yielding a different sequence of streets and hence a different route. Thus the route 0 -> 2 -> 3 -> 2 -> 3 -> 2 is a different route than 0 ->  2 ->  3 ->  2.

### Input

The input is a sequence of city specifications. Each specification begins with the number of one-way streets in the city followed by that many one-way streets given as pairs of intersections. Each pair 'j k' represents a one-way street from intersection j to intersection k. In all cities, intersections are numbered sequentially from 0 to the “largest” intersection. All integers in the input are separated by
whitespace. The input is terminated by end-of-file.

There will never be a one-way street from an intersection to itself. No city will have more than 30 intersections.

### Output

For each city specification, a square matrix of the number of different routes from intersection j to intersection k is printed. If the matrix is denoted M, then $M[j][k]$ is the number of different routes from intersection j to intersection k. The matrix M should be printed in row-major order, one row per line. Each matrix should be preceded by the string 'matrix for city k' (with k appropriately instantiated, beginning with 0).

If there are an infinite number of different paths between two intersections a '-1' should be printed. DO NOT worry about justifying and aligning the output of each matrix. All entries in a row should be separated by whitespace.

### SampleInput

7 0 1 0 2 0 4 2 4 2 3 3 1 4 3
5
0 2
0 1 1 5 2 5 2 1
9
0 1 0 2 0 3
0 4 1 4 2 1
2 0
3 0
3 1

### SampleOutput

matrix for city 0
0 4 1 3 2
0 0 0 0 0
0 2 0 2 1
0 1 0 0 0
0 1 0 1 0
matrix for city 1
0 2 1 0 0 3
0 0 0 0 0 1
0 1 0 0 0 2
0 0 0 0 0 0
0 0 0 0 0 0
0 0 0 0 0 0
matrix for city 2
-1 -1 -1 -1 -1
0 0 0 0 1
-1 -1 -1 -1 -1
-1 -1 -1 -1 -1
0 0 0 0 0

### 题意

题目会给定一组数字对$<i,j>$,i,j表示节点值，非负整数表示。一组数字对中所有节点最大的值记为max，则该组包含从0到max的所有节点$\{i|0\leq i \leq max,i是整数\}$,数字对<i,j>表示从i到j是连通的，现给定一组数字对$\{<i,j>|\text{group of }<i,j>\}$,要求求出0到max任意两个节点是否连通，并判断节点之间有多少条路径可达。

示例给了4组数字，其中节点最大值是3，所以共有$\{0,1,2,3\}$4个节点。
0 1
0 2
1 2
2 3

各节点路径如下：
0->0: {}
0->1: {0->1}
0->2: {0->2,0->1->2}
0->3: {0->2->3,0->1->2->3}
1->0: {}
1->1: {}
1->2: {1->2}
1->3: {1->2->3}
...

所以0->2有两条，0->1有一条等等。特别的如果3->2是通的，0->2就有无限条:$\{0->2, 0->1->2, 0->2->3->2, 0->2->3->2->3->2,...\}$.

## 思路

这是一个图的问题，即判断有向图中任意两点之间有多少条路径，一种方法是采用DFS(深度优先遍历)遍历两个节点的所有路径，求出路径数。采用DFS面临的一个问题是如果路径构成环，需要采用一定的方式判断避免路径成环。同时如果两点之间的一条路径中如果有节点在环上，则该两点间的路径就有无限多条。例如a->b->c->d,如果b在环上，假如b的环为b->e->f->b,则a->d的路径可以加入任意个b环，所以路径有无限多条。

### 问题

#### 实现上的问题

1. 初始考虑DFS时没有考虑到环的问题，导致DFS陷入了无限循环；
2. 对两个指定节点做DFS时需要考虑的边界条件过多导致引入了很多额外的辅助工具，例如先求出所有在环上的点，再求出任意两点之间是否连通的矩阵来辅助判断两点之间做DFS时的路径数。导致实现比较复杂，运行超时。

#### 性能上的问题

初始时通过双重循环对任意两点做DFS导致超时，通过查看他人的解决方案发现不需要任意两点都做DFS，通过从当前点出发做DFS计算当前点a与所有可达的点的路径数，当所有点都遍历一遍，也就算出了任意两点的路径数(不可达则路径数为0,同时这里也隐式解决了上面方案引入的连通矩阵问题，因为DFS遍历过程就包含了两点是否连通的判断).

当做DFS时，我们需要避免环(同理DFS本身就能判断环，不需要额外算出所有的环上节点)，所以遍历节点做DFS之后还需要修正路径是否有节点在环上，从而调整路径数，调整分为两步：

1. 在DFS中发现环时标记出环上的所有节点，
2. 采用Floyd算法判断路径上是否有节点在环上(在扩展了任意两点之间的连通性后，通过遍历环节点，判断环节点是否在两点的路径上)。

## 实现

```java {.line-numbers}
//Numbering Paths
import java.io.*;
import java.util.*;

public class Main {
    public static void main(String[] args) throws IOException {
        BufferedReader reader = new BufferedReader(new InputStreamReader(System.in));
        int index = 0;
        List<String> lines = new ArrayList<>();
        String line = null;
        while ((line = reader.readLine()) != null) {
            lines.add(line);
        }
        String allLine = String.join(" ", lines);
        StringTokenizer idata = new StringTokenizer(allLine);
        while (idata.hasMoreTokens()) {
            int number;
            if (idata.hasMoreTokens()) {
                number = Integer.parseInt(idata.nextToken());
            } else {
                idata = new StringTokenizer(reader.readLine());
                number = Integer.parseInt(idata.nextToken());
            }

            if (number == 0) {
                System.out.printf("matrix for city %d\n", index);
                index++;
                continue;
            }
            List<Route> routes = new ArrayList<>();
            int maxNum = 0;
            maxNum = initEdges(number, reader, idata, maxNum, routes);

            int[][] routeMatrix = initRouteMatrix(maxNum, routes);

            //init matrix:初始化结果矩阵
            int[][] matrix = new int[maxNum + 1][maxNum + 1];
            for (int i = 0; i < maxNum + 1; i++) {
                for (int j = 0; j < maxNum + 1; j++) {
                    matrix[i][j] = 0;
                }
            }

            for (int i = 0; i < maxNum + 1; i++) {
                LinkedList<Integer> path = new LinkedList<>();
                path.add(i);
                otherDfs(path, matrix, routeMatrix);
            }

            //遍历完节点需要处理路径上有环节点存在的情况
            for (int i = 0; i < maxNum + 1; i++) {
                //节点本身就在环上，所有可达的路径都变成无限条
                if (matrix[i][i] == -1) {
                    for (int j = 0; j < maxNum + 1; j++) {
                        matrix[i][j] = matrix[i][j] > 0 ? -1 : matrix[i][j];
                    }
                    continue;
                }

                for (int j = 0; j < maxNum + 1; j++) {
                    //节点i连通的环节点j，如果环节点连通其余的节点k，则i->k也有无数条路径
                    if (matrix[i][j] > 0 && matrix[j][j] == -1) {
                        for (int k = 0; k < maxNum + 1; k++) {
                            matrix[i][k] = matrix[j][k] != 0 ? -1 : matrix[i][k];
                        }
                    }
                }
            }

            System.out.printf("matrix for city %d\n", index);
            StringBuilder sb = new StringBuilder();
            for (int i = 0; i < maxNum + 1; i++) {
                sb.append(matrix[i][0]);
//                System.out.printf("%d", matrix[i][0]);
                for (int j = 1; j < maxNum + 1; j++) {
//                    System.out.printf(" %d", matrix[i][j]);
                    sb.append(" ");
                    sb.append(matrix[i][j]);
                }
                sb.append("\n");

//                System.out.printf("\n");
            }
            System.out.printf(sb.toString());
            index++;
        }
    }

    /**
     * 判断从path的最上层元素出发，可以抵达的路径，并增加两点之间的路径数
     */
    private static void otherDfs(LinkedList<Integer> path, int[][] matrix, int[][] routeMatrix) {
        int last = path.getLast();
        for (int i = 0; i < matrix.length; i++) {
            if (routeMatrix[last][i] > 0) {
                int j = 0;
                for (; j < path.size() && path.get(j) != i; j++) ;  //loop for detect ring and jump out of loop
                //检测到环，标记环上所有的节点，并接着往下遍历，不进入环
                if (j < path.size()) {
                    for (int k = j; k < path.size(); k++) {
                        matrix[path.get(k)][path.get(k)] = -1;
                    }
                    continue;
                }
                matrix[path.getFirst()][i] += routeMatrix[path.getFirst()][i] == 0 ? 1 : routeMatrix[path.getFirst()][i];
                path.add(i);
                otherDfs(path, matrix, routeMatrix);
            }
        }
        path.pollLast();
    }


    private static int[][] initRouteMatrix(int maxNum, List<Route> routes) {
        int[][] routeMatrix = new int[maxNum + 1][maxNum + 1];
        for (int i = 0; i < maxNum + 1; i++) {
            for (int j = 0; j < maxNum + 1; j++) {
                routeMatrix[i][j] = 0;
            }
        }
        for (Route route : routes) {
            routeMatrix[route.getStart()][route.getEnd()] += 1;
        }
        return routeMatrix;
    }

    //save the edge of graph
    private static int initEdges(int number, BufferedReader reader, StringTokenizer idata,
                                 int maxNum, List<Route> routes) throws IOException {
        for (int i = 0; i < number; i++) {
            int start, end;
            if (idata.hasMoreElements()) {
                start = Integer.parseInt(idata.nextToken());
            } else {
                idata = new StringTokenizer(reader.readLine());
                start = Integer.parseInt(idata.nextToken());
            }
            if (idata.hasMoreElements()) {
                end = Integer.parseInt(idata.nextToken());
            } else {
                idata = new StringTokenizer(reader.readLine());
                end = Integer.parseInt(idata.nextToken());
            }

            if (start > maxNum) {
                maxNum = start;
            }
            if (end > maxNum) {
                maxNum = end;
            }

            Route route = new Route();
            route.setStart(start);
            route.setEnd(end);
            routes.add(route);
        }
        return maxNum;
    }

    public static class Route {
        private int start;
        private int end;

        public int getStart() {
            return start;
        }

        public void setStart(int start) {
            this.start = start;
        }

        public int getEnd() {
            return end;
        }

        public void setEnd(int end) {
            this.end = end;
        }
    }
}
```
