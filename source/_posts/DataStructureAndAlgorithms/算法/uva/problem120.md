---
title: 120.Stacks of Flapjacks
date: 2024-04-21 21:59:42
categories:
- [数据结构与算法,uva]
tags:
- 算法
- uva
---

## 题目

Stacks and Queues are often considered the bread and butter of data structures and find use in architecture parsing, operating systems, and discrete event simulation. Stacks are also important in the theory of formal languages.

This problem involves both butter and sustenance in the form of pancakes rather than bread in addition to a finicky server who flips pancakes according to a unique, but complete set of rules.

Given a stack of pancakes, you are to write a program that indicates how the stack can be sorted so that the largest pancake is on the bottom and the smallest pancake is on the top. The size of a pancake is given by the pancake’s diameter. All pancakes in a stack have different diameters.

Sorting a stack is done by a sequence of pancake “flips”. A flip consists of inserting a spatula between two pancakes in a stack and flipping (reversing) all the pancakes on the spatula (reversing the sub-stack). A flip is specified by giving the position of the pancake on the bottom of the sub-stack to be flipped (relative to the whole stack). The pancake on the bottom of the whole stack has position 1 and the pancake on the top of a stack of n pancakes has position n.

A stack is specified by giving the diameter of each pancake in the stack in the order in which the pancakes appear.

For example, consider the three stacks of pancakes below (in which pancake 8 is the top-most pancake of the left stack):

```plaintext
8           7           2
4           6           5
6           4           8
7           8           4
5           5           6
2           2           7
------------>------------>
    flip(3)      flip(1)
```

The stack on the left can be transformed to the stack in the middle via flip(3). The middle stack can be transformed into the right stack via the command flip(1).

### Input

The input consists of a sequence of stacks of pancakes. Each stack will consist of between 1 and 30 pancakes and each pancake will have an integer diameter between 1 and 100. The input is terminated by end-of-file. Each stack is given as a single line of input with the top pancake on a stack appearing first on a line, the bottom pancake appearing last, and all pancakes separated by a space.

### Output

For each stack of pancakes, the output should echo the original stack on one line, followed by some sequence of flips that results in the stack of pancakes being sorted so that the largest diameter pancake is on the bottom and the smallest on top. For each stack the sequence of flips should be terminated by a ‘0’ (indicating no more flips necessary). Once a stack is sorted, no more flips should be made.

### Sample Input

1 2 3 4 5
5 4 3 2 1
5 1 2 3 4

### Sample Output

1 2 3 4 5
0
5 4 3 2 1
1 0
5 1 2 3 4
1 2 0

### 题意

给定一个stack，栈里面元素大小不同，并且规定了栈只有一种操作：flip。flip操作是指定栈中某个位置的元素，从该元素到栈顶的所有元素做一次翻转。以上面例子为例，栈从顶至底的元素为：8，4，6，7，5，2。flip(i)指定的是栈中第i个元素，元素从栈底开始计算，栈底是第一个元素。例如flip(3)指定栈底往上第三个元素，这里是7，flip操作将8,4,6,7的栈顺序翻转过来，变成7,6,4,8。所以栈变为7,6,4,8,5,2。

```plaintext
8  <--|          7        /|\ 栈顶，序号为6
4     |翻转      6         |
6     |          4         |
7  <--|          8         |
5                5         |  序号2
2                2         | 栈底，序号为1  
----------------->
    flip(3)  
```

现在的问题是如何通过flip操作将栈中元素调整为有序，即从栈底到栈顶元素按照从大到小排列，栈底元素最大，往上逐渐变小，直至栈顶最小。

## 思路

考虑flip操作的特性，我们可以将栈顶元素翻转到任意位置。如果栈顶是最大元素，那么直接翻转到栈底，栈底即确定了元素。由栈底往上，逐步确定每个位置的元素，则栈有序。

整个过程可以概括为由栈底往上，查找未确认的最大值，将其flip到指定位置。

1. 由栈底往上，定义当前待确认的位置为index；
2. 由于所有未确认的元素都在index及其上面，从未确认的元素中找出最大值indexMax;
3. 通过flip(indexMax)将未确认的最大元素翻转到栈顶;
4. flip(index)即可将未确认的最大元素翻转到指定位置;
5. 迭代，直至index到栈顶。

```plaintext
8                            2                      7                4   
4                            5                      5                6
6                            7                      2                2 
7                            6                      6                5
5                            4                      4                7
2                            8                      8                8
----------------------------->---------------------->---------------->....
    index=1,                          index=2           flip(index=2)    index++
    indexMax=6                        indexMax=4
    indexMax在栈顶，不需要翻转，        flip(indexMax=4)
    直接flip(index=1)
```

该过程很类似选择排序，每一个选择未排序的元素中的最大值，迭代直至整个数组有序。

### 问题

#### 实现上的问题

#### 性能上的问题

## 实现

```JAVA
import java.util.Scanner;

public class Main {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        while (scanner.hasNextLine()) {
            String line = scanner.nextLine();
            System.out.println(line);
            String[] pancakes = line.split(" ");
            int[] pancakeNums = new int[pancakes.length];
            for (int i = 0; i < pancakes.length; i++) {
                pancakeNums[i] = Integer.valueOf(pancakes[i]);
            }
            int sortedIndex = pancakes.length - 1;  //record the top sorted index,under the index all pancakes is sorted
            for (; sortedIndex > 0; --sortedIndex) {
                int curMaxIndex = findMaxValIndex(sortedIndex, pancakeNums);
                if (curMaxIndex == sortedIndex) {
                    //如果最大元素已经在指定位置，则不需要翻转
                    continue;
                }
                if (curMaxIndex != 0) {
                    //如果最大元素不在栈顶，则需要翻转
                    flip(curMaxIndex, pancakeNums);
                    System.out.printf("%d ", pancakes.length - curMaxIndex);
                }

                flip(sortedIndex, pancakeNums);
                System.out.printf("%d ", pancakes.length - sortedIndex);
            }
            System.out.printf("%d\n", 0);
        }
    }

    /**
     * flip操作
     */
    private static void flip(int curMaxIndex, int[] pancakeNums) {
        int start = 0, end = curMaxIndex;
        while (start < end) {
            int temp = pancakeNums[start];
            pancakeNums[start] = pancakeNums[end];
            pancakeNums[end] = temp;
            start++;
            end--;
        }
    }

    /**
     * 查找当前未排序的元素中的最大值
     */
    private static int findMaxValIndex(int sortedIndex, int[] pancakeNums) {
        int num = pancakeNums[sortedIndex];
        int index = sortedIndex;
        for (int i = sortedIndex - 1; i >= 0; i--) {
            if (pancakeNums[i] > num) {
                index = i;
                num = pancakeNums[i];
            }
        }
        return index;
    }
}
```

