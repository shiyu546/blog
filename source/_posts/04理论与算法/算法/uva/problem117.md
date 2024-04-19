# The Postal Worker Rings Once

## 1. 题目

Graph algorithms form a very important part of computer science and have a lineage that goes back at least to Euler and the famous Seven Bridges of Königsberg problem. Many optimization problems involve determining efficient methods for reasoning about graphs.

This problem involves determining a route for a postal worker so that all mail is delivered while the postal worker walks a minimal distance, so as to rest weary legs.

Given a sequence of streets (connecting given intersections) you are to write a program that determines the minimal cost tour that traverses every street at least once. The tour must begin and end at the same intersection.

The “real-life” analogy concerns a postal worker who parks a truck at an intersection and then walks all streets on the postal delivery route (delivering mail) and returns to the truck to continue with the next route.

The cost of traversing a street is a function of the length of the street (there is a cost associated with delivering mail to houses and with walking even if no delivery occurs).

In this problem the number of streets that meet at a given intersection is called the degree of the intersection. There will be at most two intersections with odd degree. All other intersections will have even degree, i.e., an even number of streets meeting at that intersection.

### Input

The input consists of a sequence of one or more postal routes. A route is composed of a sequence of street names (strings), one per line, and is terminated by the string ‘deadend’ which is NOT part of the route. The first and last letters of each street name specify the two intersections for that street, the length of the street name indicates the cost of traversing the street. All street names will consist
of lowercase alphabetic characters.

For example, the name foo indicates a street with intersections f and o of length 3, and the name computer indicates a street with intersections c and r of length 8. No street name will have the same first and last letter and there will be at most one street directly connecting any two intersections. As specified, the number of intersections with odd degree in a postal route will be at most two. In each postal route there will be a path between all intersections, i.e., the intersections are connected.

### Output

For each postal route the output should consist of the cost of the minimal tour that visits all streets at least once. The minimal tour costs should be output in the order corresponding to the input postal routes.

### Sample Input

one
two
three
deadend
mit
dartmouth
linkoping
tasmania
york
emory
cornell
duke
kaunas
hildesheim
concord
arkansas
williams
glasgow
deadend

### Sample Output

11
114

### 1.1题意

题目给出了一连串的字符，每个字符的首尾是两个节点，字符长度是节点的边。要求出从某个点出发遍历图中每一条边并回到该点的最短路径。

## 2. 思路

初步看起来并没有什么什么头绪，但是结合题目给出的条件：图中节点度为奇数的个数最多为两个，结合欧拉路径和欧拉环路定理，可以做进一步分析：

> 欧拉路径：欧拉路是指从图中任意一个点开始到图中任意一个点结束的路径，并且图中每条边通过的且只通过一次。
> 欧拉环路：欧拉回路是指起点和终点相同的欧拉路径。
>
> 无向图存在欧拉回路的充要条件：
> 一个无向图存在欧拉回路，当且仅当该图所有顶点度数都为偶数, 且该图是连通图。
>
> 无向图存在欧拉路径的充要条件：
> 当且仅当该图顶点度数为奇数的点的个数为0或者2。

因为无向图不可能存在1个度为奇数的节点，所以度为奇数的节点个数只有两种情况：0和2.

1. 当度为奇数的节点个数为0时，图满足形成欧拉回路的充要条件，这时存在某个顶点，从该顶点出发每条边只经过一次就回到该顶点，所以此时的最短路径就是图的边长度之和。
2. 当度为奇数的节点个数为2时，图满足欧拉路径，此时从一个奇数度节点a出发，遍历图一定到另一个奇数度节点b终止，此时已经遍历了图。但是按题目要求，起点和终点一定一样，所以我们还需要从b出发回到a节点。
    此时我们可以把求最短路径分为两部分：一部分是图的总边长度，第二部分是两个奇数度节点之间的最短距离，这部分最短距离可以用dijkstra算法求。

### 2.1问题

#### 2.1.1 实现上的问题

#### 2.1.2 性能上的问题

## 3. 实现

```JAVA
import java.util.*;

public class Main {
    public static void main(String[] args) {
//        File file = new File("number.txt");
//        Scanner scanner = null;
//        try {
//            scanner = new Scanner(file);
//        } catch (FileNotFoundException e) {
//            throw new RuntimeException(e);
//        }
        Scanner scanner = new Scanner(System.in);

        int[][] graph = new int[26][26];
        Set<Character> points = new HashSet<>();
        initGraph(graph);
        int graphLength = 0;
        while (scanner.hasNext()) {
            String street = scanner.nextLine();
            if (street.equals("deadend")) {
                //1.每一次的输入会有多个图，这里是单个图输入终止，下面的用来计算最短路径
                int length = computeLength(graph, graphLength, points);
                System.out.println(length);

                //2.这里初始化一些参数，每个图初始需要初始化这些参数
                initGraph(graph);
                graphLength = 0;
                points = new HashSet<>();
                continue;
            }
            char startInter = street.charAt(0);
            char endInter = street.charAt(street.length() - 1);
            graph[startInter - 'a'][endInter - 'a'] = street.length();
            graph[endInter - 'a'][startInter - 'a'] = street.length();
            graphLength += street.length();
            points.add(startInter);
            points.add(endInter);
        }
    }

    private static void initGraph(int[][] graph) {
        int length = graph.length;
        for (int i = 0; i < length; i++) {
            for (int j = 0; j < length; j++) {
                graph[i][j] = -1;
            }
        }
    }

    /**
     * compute minlength of route
     *
     * @param graph
     * @return
     */
    private static int computeLength(int[][] graph, int graphLength, Set<Character> points) {
        int length = graph.length;
        //计算奇数度节点个数
        int oddDimension = 0;
        List<Integer> oddIndex = new ArrayList<>();
        for (Character ch : points) {
            int dimension = 0;
            for (int i = 0; i < length; i++) {
                if (graph[ch - 'a'][i] != -1) {
                    dimension++;
                }
            }
            if (dimension % 2 != 0) {
                oddDimension++;
                oddIndex.add(ch - 'a');
            }

        }

        if (oddDimension == 0) {
            //euler circuit
            return graphLength;
        } else {
            //euler path
            int startPoint = oddIndex.get(0);
            int endPoint = oddIndex.get(1);
            //采用Dijkstra算法算出两个奇数度节点的最短路径
            Set<Integer> alreadyShortIndex = new HashSet<>();
            alreadyShortIndex.add(startPoint);
            int point = findshortestIndex(graph, alreadyShortIndex, startPoint);
            while (point != endPoint) {
                alreadyShortIndex.add(point);
                updatePath(graph, alreadyShortIndex, startPoint, point);
                point = findshortestIndex(graph, alreadyShortIndex, startPoint);
            }
            return graphLength + graph[startPoint][endPoint];
        }
    }

    private static void updatePath(int[][] graph, Set<Integer> alreadyShortIndex, int startPoint, int point) {
        for (int i = 0; i < graph.length; i++) {
            if (!alreadyShortIndex.contains(i)) {
                if (graph[point][i] != -1) {
                    if (graph[startPoint][i] == -1 ||
                            (graph[startPoint][i] > graph[startPoint][point] + graph[point][i])) {
                        graph[startPoint][i] = graph[startPoint][point] + graph[point][i];
                    }
                }
            }
        }
    }

    private static int findshortestIndex(int[][] graph, Set<Integer> alreadyShortIndex, int startPoint) {
        int shortest = Integer.MAX_VALUE;
        int shortIndex = -1;
        for (int i = 0; i < graph.length; i++) {
            if (graph[startPoint][i] != -1 && !alreadyShortIndex.contains(i)
                    && graph[startPoint][i] < shortest) {
                shortest = graph[startPoint][i];
                shortIndex = i;
            }
        }
        return shortIndex;
    }
}
```
