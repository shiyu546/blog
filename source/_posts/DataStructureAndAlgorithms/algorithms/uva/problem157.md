---
title: problem157 Route Finding
date: 2025-03-27 09:32:12
mathjax: true
categories:
- [数据结构与算法,uva]
tags:
- 算法
- uva
description: "UVA problem157.Route Finding <br>
- 计算公交线路网中两点的最短路径"
---

## 题目

Many cities provide a comprehensive public transport system, often integrating bus routes, suburban commuter train services and underground railways. Routes on such systems can be categorised according to the stations or stops along them. We conventionally think of them as forming lines (where the vehicle shuttles from one end of the route to the other and returns), loops (where the two ends of the “branch” are the same and vehicles circle the system in both directions) and connections, where each end of the route connects with another route. Obviously all of these can be thought of as very similar,and can connect with each other at various points along their routes. Note that vehicles can travel in both directions along all routes, and that it is only possible to change between routes at connecting stations.

To simplify matters, each route is given a designation letter from the set ‘A’ to ‘Z’, and each station along a route will be designated by another letter from the set ‘a’ to ‘z’. Connecting stations will have more than one designation. Thus an example could be:

{% asset_img 157_pic1.jpg 交通图示%}

A common problem in such systems is finding a route between two stations. Once this has been done we wish to find the “best” route, where “best” means “shortest time”.

Write a program that will read in details of such a system and then will find the fastest routes between given pairs of stations. You can assume that the trip between stations always takes 1 unit of time and that changing between routes at a connecting station takes 3 units of time.

### Input

Input will consist of two parts. The first will consist of a description of a system, the second will consist of pairs of stations. The description will start with a number between 1 and 26 indicating how many routes there are in the system. This will be followed by that many lines, each describing a single route.Each line will start with the route identifier followed by a ‘:’ followed by the stations along that route,in order.Connections will be indicated by an ‘=’ sign followed by the complete alternative designation.All connections will be identified at least once, and if there are more than two lines meeting at a connection, some or of all the alternative designations may be identified together. That is, there may
be sequences such as ‘. . . hc=Bg=Cc=Abd. . . ’. If the route forms a loop then the last station will be the same as the first. This is the only situation in which station letters will be repeated.

The next portion of the input file will consist of a sequence of lines each containing two stations written contiguously. The file will be terminated by a line consisting of a single ‘#’.

### Output

Output will consist of a series of lines, one for each pair of stations in the input. Each line will consist of the time for the fastest route joining the two stations, right justified in a field of width 3, followed by a colon and a space and the sequence of stations representing the shortest journey. Follow the example shown below. Note that there will always be only one fastest route for any given pair of stations and that the route must start and finish at the named stations (not at any synonyms thereof), hence the time for the route must include the time for any inter-station transfers.

Note: The example input below refers to the diagram given above.

### SampleInput

4
A:fgmpnxzabjd=Dbf
D:b=Adac=Ccf
B:acd=Azefg=Cbh
C:bac
AgAa
AbBh
BhDf
\#

### SampleOutput

5: Agfdjba
9: Abaz=Bdefgh
10: Bhg=Cbac=Dcf

### 题意

题目给出了一个交通线路网，线路网由多条公交线路组成，线路可能组成环线。不同线路会有交叉站点，这种称为换乘站。现在给出以下的简化模型：假设同一线路下前往相邻站耗时1个时间单位，不同线路换乘时换乘需耗费3个时间单位。现在任意给出两个站点，要求出两个站点往返的最少耗时。

## 思路

将公交线路网抽象成图，公交站点当作图中的节点，注意换乘站比较特殊，如果换乘站当作一个节点，那么边的耗时就无法衡量，因为同线路通过时间和换乘通过时间不一致，所以不同线路的换乘当作不同节点，换乘站之间用换乘时间3当作边的权重连接。

然后采用迪杰斯特拉算法计算最短路径。

### 问题

#### 实现上的问题

换乘站可能有多条线路相交，这时候这些换乘节点需要两两连接并设置边的权重为3.

#### 性能上的问题

## 实现

```JAVA .{line-numbers}
//source code
import java.util.*;

public class Main {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        int lineNum = Integer.valueOf(scanner.nextLine());
        List<String> lines = new ArrayList<>();
        for (int i = 0; i < lineNum; i++) {
            String line = scanner.nextLine();
            lines.add(line);
        }

        List<String> paths = new ArrayList<>();
        while (true) {
            String path = scanner.nextLine();
            if (path.equals("#")) {
                break;
            }
            paths.add(path);
        }

        Map<String, Set<String>> connectMap = new HashMap<>();

        //得到图中所有的节点
        List<String> points = genPtAndConnect(lines, connectMap);

        Map<String, Integer> ptIndexMap = new HashMap<>();
        Map<Integer, String> ptIndex2NodeMap = new HashMap<>();
        for (int i = 0; i < points.size(); i++) {
            ptIndexMap.put(points.get(i), i);
            ptIndex2NodeMap.put(i, points.get(i));
        }

        //create adjacency matrix
        int size = points.size();
        int[][] matrix = new int[size][size];
        for (int i = 0; i < size; i++) {
            for (int j = 0; j < size; j++) {
                matrix[i][j] = Integer.MAX_VALUE;
            }
        }

        //创建图的连接矩阵
        fillMatrix(matrix, ptIndexMap, lines, connectMap);

        for (int i = 0; i < paths.size(); i++) {
            String path = paths.get(i);
            String startNode = path.substring(0, 2);
            String endNode = path.substring(2);
            int startIndex = ptIndexMap.get(startNode);
            int endIndex = ptIndexMap.get(endNode);

            List<Integer> pathNodes = new ArrayList<>();

            //采用迪杰斯特拉算法计算
            int minPath = getMinalPath(matrix, startIndex, endIndex, pathNodes);

            String pathItem = getPathFrom(pathNodes, ptIndex2NodeMap);
            System.out.printf("%3d: %s\n", minPath, pathItem);
        }
    }

    private static String getPathFrom(List<Integer> pathNodes, Map<Integer, String> ptIndex2NodeMap) {
        Collections.reverse(pathNodes);
        List<String> nodes = new ArrayList<>();
        for (int i = 0; i < pathNodes.size(); i++) {
            nodes.add(ptIndex2NodeMap.get(pathNodes.get(i)));
        }
        StringBuilder sb = new StringBuilder();
        char currentLine = nodes.get(0).charAt(0);
        sb.append(currentLine);
        sb.append(nodes.get(0).charAt(1));
        for (int i = 1; i < nodes.size(); i++) {
            String current = nodes.get(i);
            if (current.charAt(0) == currentLine) {
                sb.append(current.charAt(1));
            } else {
                sb.append("=");
                sb.append(current);
                currentLine = current.charAt(0);
            }
        }
        return sb.toString();
    }

    private static int getMinalPath(int[][] matrix, int startIndex, int endIndex, List<Integer> pathNodes) {
        //use dijkstra to find the minimal path
        int size = matrix.length;

        /**
         * dijkstra算法将节点分为两个集合，起始点集合为C，剩余的点为D。从起始点s开始，找到最近的节点n，将n加入C，并找到n的相邻节点，更新这些节点到s的距离。
         * 重复从D中找到最近的点，加入C，更新相邻节点距离这个过程，直至全部点都在C中。
         */
        //flag用来记录哪些节点在C中，哪些在D中，true在C中，false在D中。
        boolean[] flags = new boolean[size];

        //用来记录节点放入C中时该节点的前驱节点。
        int[] before = new int[size];

        //记录每个节点到起始节点的距离
        int[] dist = new int[size];

        for (int i = 0; i < size; i++) {
            flags[i] = true;
            before[i] = startIndex;
            dist[i] = matrix[startIndex][i];
        }

        flags[startIndex] = false;

        for (int i = 0; i < size; i++) {
            int min = Integer.MAX_VALUE;
            int minIndex = 0;
            for (int j = 0; j < size; j++) {
                if (flags[j] && dist[j] < min) {
                    min = dist[j];
                    minIndex = j;
                }
            }
            flags[minIndex] = false;
            //update matrix
            for (int j = 0; j < size; j++) {
                int tmp = matrix[minIndex][j];
                tmp = (tmp == Integer.MAX_VALUE) ? Integer.MAX_VALUE : dist[minIndex] + tmp;
                if (flags[j] && tmp < dist[j]) {
                    before[j] = minIndex;
                    dist[j] = tmp;
                }
            }
        }

        int distNum = dist[endIndex];
        int index = endIndex;

        pathNodes.add(endIndex);
        while (index != startIndex) {
            index = before[index];
            pathNodes.add(index);
        }
        return distNum;
    }

    private static void fillMatrix(int[][] matrix, Map<String, Integer> ptIndexMap, List<String> lines, Map<String, Set<String>> connectMap) {
        for (String line : lines) {
            line = optLine(line);
            char lineNumber = line.charAt(0);
            for (int i = 2; i < line.length() - 1; i++) {
                String indexi = String.valueOf(lineNumber) + line.charAt(i);
                String indexj = String.valueOf(lineNumber) + line.charAt(i + 1);
                int indi = ptIndexMap.get(indexi);
                int indj = ptIndexMap.get(indexj);
                matrix[indi][indj] = 1;
                matrix[indj][indi] = 1;
            }
        }
        for (Set<String> keyNode : connectMap.values()) {
            List<String> vals = new ArrayList<>(keyNode);
            for (int i = 0; i < vals.size(); i++) {
                for (int j = i + 1; j < vals.size(); j++) {
                    int indi = ptIndexMap.get(vals.get(i));
                    int indj = ptIndexMap.get(vals.get(j));
                    matrix[indi][indj] = 3;
                    matrix[indj][indi] = 3;
                }
            }
        }
    }

    private static String optLine(String line) {
        int index = line.indexOf("=");
        while (index >= 0) {
            line = line.substring(0, index) + line.substring(index + 3);
            index = line.indexOf("=");
        }
        return line;
    }

    private static List<String> genPtAndConnect(List<String> lines, Map<String, Set<String>> connectMap) {
        List<String> nodes = new ArrayList<>();
        Set<String> dupNodes = new HashSet<>();
        for (int i = 0; i < lines.size(); i++) {
            String line = lines.get(i);
            char lineNumber = line.charAt(0);
            for (int j = 2; j < line.length(); j++) {
                if (line.charAt(j) != '=') {
                    String node = String.valueOf(lineNumber) + line.charAt(j);
                    if (dupNodes.contains(node)) {
                        continue;
                    } else {
                        dupNodes.add(node);
                    }
                    nodes.add(node);
                } else {
                    int tmp = j;
                    String connNode = String.valueOf(lineNumber) + line.charAt(j - 1);
                    while (line.charAt(tmp) == '=') {
                        String conn = line.substring(tmp + 1, tmp + 3);

                        if (connectMap.containsKey(connNode)) {
                            Set<String> conSet = connectMap.get(connNode);
                            conSet.add(conn);
                            connectMap.put(conn, conSet);
                        } else if (connectMap.containsKey(conn)) {
                            Set<String> conSet = connectMap.get(conn);
                            conSet.add(connNode);
                            connectMap.put(connNode, conSet);
                        } else {
                            Set<String> conSet = new HashSet<>();
                            conSet.add(connNode);
                            conSet.add(conn);
                            connectMap.put(connNode, conSet);
                            connectMap.put(conn, conSet);
                        }
                        tmp += 2;
                    }
                    j = tmp - 1;
                }
            }
        }
        return nodes;
    }
}
```
