---
title: problem130 Roman Roulette
date: 2024-09-15 10:37:47
mathjax: true
categories:
- [数据结构与算法,uva]
tags:
- 算法
- uva
description: "UVA 130.Roman Roulette <br>
- 约瑟夫环问题的变种"
---

## 题目

The historian Flavius Josephus relates how, in the Romano-Jewish conflict of 67 A.D., the Romans took the town of Jotapata which he was commanding. Escaping, Jospehus found himself trapped in a cave with 40 companions. The Romans discovered his whereabouts and invited him to surrender, but
his companions refused to allow him to do so. He therefore suggested that they kill each other, one by one, the order to be decided by lot. Tradition has it that the means for effecting the lot was to stand in a circle, and, beginning at some point, count round, every third person being killed in turn.The sole survivor of this process was Josephus, who then surrendered to the Romans. Which begs the question: had Josephus previously practised quietly with 41 stones in a dark corner, or had he calculated mathematically that he should adopt the 31st position in order to survive?

Having read an account of this gruesome event you become obsessed with the fear that you will find yourself in a similar situation at some time in the future. In order to prepare yourself for such an eventuality you decide to write a program to run on your hand-held PC which will determine the
position that the counting process should start in order to ensure that you will be the sole survivor.

In particular, your program should be able to handle the following variation of the processes described by Josephus. n > 0 people are initially arranged in a circle, facing inwards, and numbered from 1 to n. The numbering from 1 to n proceeds consecutively in a clockwise direction. Your allocated number is 1. Starting with person number i, counting starts in a clockwise direction, until we get to person number k (k > 0), who is promptly killed. We then proceed to count a further k people in a clockwise direction, starting with the person immediately to the left of the victim. The person number k so selected has the job of burying the victim, and then returning to the position in the circle that the victim had previously occupied. Counting then proceeeds from the person to his immediate left, with the k-th person being killed, and so on, until only one person remains.

For example, when n = 5, and k = 2, and i = 1, the order of execution is 2, 5, 3, and 1. The survivor is 4.

### Input

Your program must read input lines containing values for n and k (in that order. Input will be terminated by a line containing values of ‘0’ for n and k.

Your program may assume a maximum of 100 people taking part in this event.

### Output

For each input line output the number of the person with which the counting should begin in order to ensure that you are the sole survivor. For example, in the above case the safe starting position is 3.

### SampleInput

5 2
1 5
0 0

### SampleOutput

3
1

### 题意

题目类似约瑟夫环，但稍微有点不同。整个过程描述如下：n个人排成一个圆，大家面朝圆里，从1开始顺时针编号1到n，你是1号。圈里面的人排除过程如下：

1. 从编号为i的人开始，从i开始顺时针方向取数，i为第一个，i顺时针后面的为第二个，直到第k个，该人m出圈。
2. 如果从m往后第一个人n，n为第一个，n顺时针方向数到k，该人调整位置到原来m所在的位置。
3. 从n的下一个人p开始，重复第一步，即把p当作i，顺时针方向数数，淘汰人。
4. 重复上面的步骤，直到圈里面只剩一个人。

最后假设你是编号为1的人，请问从编号几的人开始，最后你将存活。

```plaintext
以题目n=5,k=2,i=1为例，从编号1开始数：
+---------------+
|   2  |     3  |
|---+-------+   |
| 1 |       |---|
|---+-------+   |
|   5    |   4  |
+---------------+
从1开始顺时针数到k，即数到编号2，2出圈。
+---------------+
|      |     3  |
|---+-------+   |
| 1 |       |---|
|---+-------+   |
|   5    |   4  |
+---------------+
从2的后一个开始，即编号3开始，数到k，即数到编号4，将4移到编号2所在的位置:
+---------------+
|  4    |     3 |
|---+-------+   |
| 1 |       |---|
|   +-------+   |
|   |     5     |
+---------------+
从4的下一个人即编号3开始数，重复该过程：
+---------------+       +---------------+       +---------------+                                              
|  4    |     3 |       |       3       |       |               |                                              
|---+-------+   |       |---+-------+   |       |---+-------+   |                                              
| 1 |       |---|  -->  | 1 |       |---|  -->  | 1 |       |---|  -->                                              
|   +-------+   |       |   +-------+   |       |   +-------+   |                                              
|   |           |       |   |     4     |       |   |     4     |                                              
+---------------+       +---------------+       +---------------+                                    
    5出圈                    4换到5的位置           3出圈

+---------------+        +---------------+                                                    
|      1        |        |               |                                                  
|---+-------+   |        |---+-------+   |                                                  
|   |       |---|  -->   |   |       |---|                                                  
|   +-------+   |        |   +-------+   |                                                  
|        4      |        |        4      |                                                  
+---------------+        +---------------+                                          
1换到3的位置                 1出圈           

所以出圈顺序为：2->5->3->1,最终4保留下来。
```

## 思路

创建两个数组，一个记录位置，一个记录是否出圈，按照上述步骤重复淘汰人。注意问题求解，考虑到人站成一个环，假设从编号1开始，最终编号为q的人存活，由于环上没有顺序，从编号为2的人开始，则编号q+1的人存活,依此类推，要求编号为1的人存活.假设有n个人,令
$$ q+x=n+1 \tag{1} $$
通过移位x个人之后最终存活的人为编号1，得
$$ x=n+1-q $$
则从$1+x$即$n+2-q$开始即可最终存活编号为1的人。注意该数的大小，根据是否超过n要减去相应的n使其值落在$[1,n]$区间。

### 问题

#### 实现上的问题

要注意步骤2，m是先出圈，然后从m后的第一个人开始数，数到第k个人。

#### 性能上的问题

## 实现

```JAVA {.line-numbers}
//file
import java.util.Scanner;

public class Main {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        while (true) {
            int number = scanner.nextInt();
            int k = scanner.nextInt();
            if (number == 0 && k == 0) {
                break;
            }

            if (number == 1) {
                System.out.println(1);
                continue;
            }

            int[] arys = new int[number];
            int[] tags = new int[number];
            for (int i = 0; i < number; i++) {
                arys[i] = i + 1;
                tags[i] = 0;
            }

            int survivorNum = number;
            int index = 0;
            while (survivorNum > 1) {
                index = findNextPerson(index, k, number, tags);
                //找到第m个人，先让其出圈
                tags[index] = 1;
                int nextStart = (index + 1) % number;
                int nextPos = findNextPerson(nextStart, k, number, tags);

                //找到m后的第k个人n，换到m原来所在的位置，记得出圈的标志要正确设置
                int temp = arys[index];
                arys[index] = arys[nextPos];
                arys[nextPos] = temp;
                tags[nextPos] = 1;
                tags[index] = 0;
                survivorNum--;

                //从n后的第一个人开始，重复出圈过程
                index = findNextPerson((index + 1) % number, 1, number, tags);
            }

            for (int i = 0; i < number; i++) {
                if (tags[i] == 0) {
                    int tmp = number + 2 - arys[i];
                    System.out.println(tmp > number ? tmp - number : tmp);
                }
            }

        }
    }

    private static int findNextPerson(int index, int k, int number, int[] tags) {
        int count = 0;
        while (true) {
            if (tags[index] == 0) {
                count++;
                if (count == k) {
                    break;
                }
            }
            index = (index + 1) % number;
        }
        return index;
    }
}
```
