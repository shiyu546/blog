# History Grading

## 1. 题目

Many problems in Computer Science involve maximizing some measure according to constraints.

Consider a history exam in which students are asked to put several historical events into chronological order. Students who order all the events correctly will receive full credit, but how should partial credit be awarded to students who incorrectly rank one or more of the historical events?

Some possibilities for partial credit include:

  1. 1 point for each event whose rank matches its correct rank。

  2. 1 point for each event in the longest (not necessarily contiguous) sequence of events which are in the correct order relative to each other.

For example, if four events are correctly ordered 1 2 3 4 then the order 1 3 2 4 would receive a score of 2 using the first method (events 1 and 4 are correctly ranked) and a score of 3 using the second method (event sequences 1 2 4 and 1 3 4 are both in the correct order relative to each other).

In this problem you are asked to write a program to score such questions using the second method.

Given the correct chronological order of n events 1，2，..., n as c~1~，c~2~，..., c~n~ where 1 <=ci <=n denotes the ranking of event i in the correct chronological order and a sequence of student responses r~1~, r~2~, ..., r~n~ where 1 <= ri <=n denotes the chronological rank given by the student to event i; determine the length
of the longest (not necessarily contiguous) sequence of events in the student responses that are in the correct chronological order relative to each other.

### Input

The input file contains one or more test cases, each of them as described below.

The first line of the input will consist of one integer n indicating the number of events with 2 <= n <= 20. The second line will contain n integers, indicating the correct chronological order of n events.

The remaining lines will each consist of n integers with each line representing a student’s chronological ordering of the n events. All lines will contain n numbers in the range [1 ... n], with each number appearing exactly once per line, and with each number separated from other numbers on the same line by one or more spaces.

### Output

For each test case, the output must follow the description below
For each student ranking of events your program should print the score for that ranking. There should be one line of output for each student ranking.

Warning: Read carefully the description and consider the difference between ’ordering’ and ’ranking’.

### Sample Input

4
4 2 3 1
1 3 2 4
3 2 1 4
2 3 4 1
10
3 1 2 4 9 5 10 6 8 7
1 2 3 4 5 6 7 8 9 10
4 7 2 3 10 6 9 1 5 8
3 1 2 4 9 5 10 6 8 7
2 10 1 3 8 4 9 5 7 6

### Sample Output

1
2
3
6
5
10
9

## 2. 思路

题目本质上就是求两个序列的最长公共子序列，很自然而然的想到dp(动态规划)，通过将子序列的最长公共序列记录下来，不断拓展序列直至序列本身。以正确序列4231和匹配序列3241为例，表格每一个数字表示所在行列子序列的最长公共子序列。

|   | 4 | 2 | 3 | 1 |
|:-:|:-:|:-:|:-:|:-:|
| 3 |   |   |   |   |
| 2 |   |   |   |   |
| 1 |   |   |   |   |
| 4 |   |   |   |   |

第一行表示子序列3分别与4，42，423，4231子序列的最长子序列，得到：

|   | 4 | 2 | 3 | 1 |
|:-:|:-:|:-:|:-:|:-:|
| 3 | 0 | 0 | 1 | 1 |
| 2 |   |   |   |   |
| 1 |   |   |   |   |
| 4 |   |   |   |   |

第二行表示32与4，42，423，4231的最长子序列，得到：

|   | 4 | 2 | 3 | 1 |
|:-:|:-:|:-:|:-:|:-:|
| 3 | 0 | 0 | 1 | 1 |
| 2 | 0 | 1 | 1 | 1 |
| 1 |   |   |   |   |
| 4 |   |   |   |   |

通过判断，我们得到表格i行j列的元素值dp\[i][j]的值为：

$$ dp_{ij}=\begin{cases}
max(dp_{(i-1)j}, dp_{i(j-1)})+1 & row[i]==col[j]\\
max(dp_{(i-1)j}, dp_{i(j-1)}) & row[i]!=col[j]
\end{cases}$$

即横子序列第i个元素和竖子序列第j个元素判断是否相等，相等则最长子序列值为当前元素所在表格位置的左边第一个元素和上面第一个元素(不存在就为0)的较大值加1；否则就等于两者较大值。例如2行2列，row[2]=2, col[2]=2, 相等，且左上元素都是0，所以dp\[2][2]=1. 最终得到的表格右下角元素即为最长子序列长度。所以本例中最长公共子序列长度为2.

|   | 4 | 2 | 3 |    1    |
|:-:|:-:|:-:|:-:|:-------:|
| 3 | 0 | 0 | 1 |    1    |
| 2 | 0 | 1 | 1 |    1    |
| 1 | 0 | 1 | 1 |    2    |
| 4 | 1 | 1 | 1 | **_2_** |

### 2.1 问题

题目比较坑的是输入，第一行给定的是序列元素个数，第二行是正确的序列，但序列不是按输入的顺序，而是按照输入的第i个元素在第几个位置来算的。以序列[3 1 2 4 9 5 10 6 8 7]为例，3表示第一个元素出现在第三个位置，1表示第二个元素出现在第一个位置，依此类推，所以输入序列要转换一下：

| 原始输入 | 3 | 1 | 2 | 4 | 9 | 5 | 10 | 6 | 8 | 7 |
|---|---|---|---|---|---|---|----|---|---|---|
| 正确的序列 | 2 | 3 | 1 | 4 | 6 | 8 | 10 | 9 | 5 | 7 |

## 3. 实现

```JAVA
import java.io.File;
import java.io.FileNotFoundException;
import java.util.Scanner;

public class Main {
    public static void main(String[] args) {
        File file = new File("number.txt");
        Scanner scanner = null;
        try {
            scanner = new Scanner(file);
        } catch (FileNotFoundException e) {
            throw new RuntimeException(e);
        }
//        Scanner scanner = new Scanner(System.in);
        int number = 0;
        boolean correct = true;
        int[] correctSeq = new int[0], compareSeq;
        while (scanner.hasNextLine()) {
            String line = scanner.nextLine();
            String arr[] = line.split("\\s+");
            if (arr.length == 1) {
                number = Integer.valueOf(arr[0]);
                correct = true;
            } else {
                if (correct) {
                    correctSeq = new int[number];
                    for (int i = 0; i < number; i++) {
                        correctSeq[Integer.valueOf(arr[i]) - 1] = i + 1;
                    }
                    correct = false;
                } else {
                    compareSeq = new int[number];
                    for (int i = 0; i < number; i++) {
                        compareSeq[Integer.valueOf(arr[i]) - 1] = i + 1;
                    }
                    int maxLength = longestSeq(correctSeq, compareSeq);
                    System.out.printf("%d\n", maxLength);
                }
            }
        }
    }

    /**
     * 寻找最长公共子序列
     */
    private static int longestSeq(int[] correctSeq, int[] compareSeq) {
        int length = correctSeq.length;
        int[][] dq = new int[length + 1][length + 1];
        for (int i = 0; i < length + 1; i++) {
            dq[0][i] = 0;
            dq[i][0] = 0;
        }
        for (int i = 1; i <= length; i++) {
            for (int j = 1; j <= length; j++) {
                if (compareSeq[i - 1] == correctSeq[j - 1]) {
                    dq[i][j] = dq[i - 1][j] > dq[i][j - 1] ? dq[i - 1][j] + 1 : dq[i][j - 1] + 1;
                } else {
                    dq[i][j] = dq[i - 1][j] > dq[i][j - 1] ? dq[i - 1][j] : dq[i][j - 1];
                }
            }
        }
        return dq[length][length];
    }
}
```
