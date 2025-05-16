---
title: problem166 Making Change
date: 2025-05-15 23:00:00
mathjax: true
categories:
- [数据结构与算法,uva]
tags:
- 算法
- uva
description: "UVA problem166.Making Change <br>
- 寻找买东西最少的硬币数"
---

## 题目

Given an amount of money and unlimited (almost) numbers of coins (we will ignore notes for this problem) we know that an amount of money may be made up in a variety of ways. A more interesting problem arises when goods are bought and need to be paid for, with the possibility that change may
need to be given. Given the finite resources of most wallets nowadays, we are constrained in the number of ways in which we can make up an amount to pay for our purchases — assuming that we can make up the amount in the first place, but that is another story.

The problem we will be concerned with will be to minimise the number of coins that change hands at such a transaction, given that the shopkeeper has an adequate supply of all coins. (The set of New Zealand coins comprises 5c, 10c, 20c, 50c, $1 and $2.) Thus if we need to pay 55c, and we do not hold a 50c coin, we could pay this as $2 \times 20c + 10c + 5c$  to make a total of 4 coins. If we tender $1 we will receive 45c in change which also involves 4 coins, but if we tender $1.05 ($1 + 5c), we get 50c change and the total number of coins that changes hands is only 3.

Write a program that will read in the resources available to you and the amount of the purchase and will determine the minimum number of coins that change hands.

### Input

Input will consist of a series of lines, each line defining a different situation. Each line will consist of 6 integers representing the numbers of coins available to you in the order given above, followed by a real number representing the value of the transaction, which will always be less than $5.00. The file will be terminated by six zeroes (0 0 0 0 0 0). The total value of the coins will always be sufficient to make up the amount and the amount will always be achievable, that is it will always be a multiple of 5c.

### Output

Output will consist of a series of lines, one for each situation defined in the input. Each line will consist of the minimum number of coins that change hands right justified in a field 3 characters wide.

### SampleInput

2 4 2 2 1 0 0.95
2 4 2 0 1 0 0.55
0 0 0 0 0 0

### SampleOutput

  2
  3

### 题意

硬币的面值有6种，分别为5分、10分、20分、50分、1元、2元，现在钱包里面有一定数目的钱，以每种面值的硬币有多少个给出。现在要买一个指定金额的商品，买东西既可以从钱包中凑出商品的价格，也可以给超出商品价格的钱让店家找零，拿出去的硬币数和找零的硬币数之和构成了这次交易的硬币数量，问以什么样的方式给钱使得交易的硬币数量总数最少。这里有两个假设：

1. 钱包里的钱数一定是超过商品金额；
2. 店家每一种面值的硬币数量都是无限的。

## 思路

如果不考虑找零，该问题可以归类为多重背包问题，通过动态规划递推出到达商品金额S最少需要多少个硬币.现在加上找零，则给出去的钱会超过物品金额S，设为T。则问题可以分为两部分：一部分是给出去的钱T的组成，设由b个硬币组成，另一部分是找零的钱，设由c个硬币组成。由于T不是固定的，我们可以遍历所有的T值，找到最小的b+c即可。

下一步需要确定T的上下界，T显然大于等于S，由于钱包的钱数量有限，钱的总数就构成了T的上界，即我们最多能给店家钱包里的钱的总金额。

现在问题分为了两部分：从钱包中找出使钱T由最少硬币组成的组合，找零部分由最少硬币组成的组合。前一个问题是多重背包问题，即给定几类硬币，每一类数量有限，问构成指定金额的最少硬币数。这类问题比较适合使用动态规划，动态规划的关键是找到递推关系，这里的关系是硬币的种类和构成的金额，令f(i,j)表示使用i类硬币构成j金额的最少硬币数。则
$$ f(i,j)=min \{ f(i-1,j-k*V(i))+k \: | \: 0\leq k \leq C(i) \} $$

其中V(i)表示第i类硬币的面值，C(i)表示第i类硬币的数量。

这里的递推关系含义如下：第i类硬币共有C(i)个，我们可以使用0到C(i)个，假设个数为k。要构成金额j，则必须减去我们使用的i类硬币的个数面值，即k*V(i),剩下的金额j-k*V(i)由i-1类硬币组成，我们已经定义其最少的硬币组成为f(i-1,j-k*V(i)),所以使用k个i类硬币，最少的硬币数就是f(i-1,j-k*V(i))+k。从所有k值中找出使得f最小的数即找到了使用i类硬币构成j金额的最小硬币数。

找零问题同理，不同的是找零中每一类硬币数量是无限的，所以k的数量上限由金额j和面值V(i)决定，即$k_{max}=j/V(i)$(由于f(i,j)表示使用i类硬币构成金额j，所以i类硬币个数面值总和超过j就没意义了)。

$$ f(i,j)=min \{ f(i-1,j-k*V(i))+k \: | \: 0\leq k \leq k_{max} \} $$

### 问题

#### 实现上的问题

#### 性能上的问题

## 实现

```JAVA .{line-numbers}
//source code
import java.math.BigDecimal;
import java.util.Scanner;

public class Main {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        while (true) {
            int fiveNum = scanner.nextInt();
            int tenNum = scanner.nextInt();
            int twentyNum = scanner.nextInt();
            int fiftyNum = scanner.nextInt();
            int hundredNum = scanner.nextInt();
            int twohundNum = scanner.nextInt();

            if (fiveNum == 0 && tenNum == 0 && twentyNum == 0 && fiftyNum == 0 && hundredNum == 0 && twohundNum == 0) {
                break;
            }

            String amountStr = scanner.next();

            int[] numAry = new int[7];
            numAry[1] = fiveNum;
            numAry[2] = tenNum;
            numAry[3] = twentyNum;
            numAry[4] = fiftyNum;
            numAry[5] = hundredNum;
            numAry[6] = twohundNum;

            int amount = new BigDecimal(amountStr).multiply(new BigDecimal(100)).intValue();

            int totalAmount = fiveNum * 5 + tenNum * 10 + twentyNum * 20 + fiftyNum * 50 + hundredNum * 100 + twohundNum * 200;
            int cols = totalAmount / 5;
            int[][] dp = new int[7][cols + 1];
            for (int i = 0; i < dp.length; i++) {
                for (int j = 0; j < dp[i].length; j++) {
                    if (j == 0) {
                        dp[i][j] = 0;
                    } else {
                        dp[i][j] = Integer.MAX_VALUE;
                    }
                }
            }

            //第一个dp是多重背包问题
            for (int i = 1; i < dp.length; i++) {
                for (int j = 1; j < dp[i].length; j++) {
                    int min = Integer.MAX_VALUE;
                    for (int k = 0; k <= numAry[i]; k++) {
                        int remainAmount = j * 5 - k * getAmountByIndex(i);
                        if (remainAmount >= 0 && dp[i - 1][remainAmount / 5] != Integer.MAX_VALUE) {
                            min = Math.min(min, dp[i - 1][remainAmount / 5] + k);
                        }
                    }
                    dp[i][j] = min;
                }
            }

            //charge
            int exceedAmount = totalAmount - amount;
            int[][] exceedDp = new int[7][exceedAmount / 5 + 1];
            for (int i = 0; i < exceedDp.length; i++) {
                for (int j = 0; j < exceedDp[i].length; j++) {
                    if (j == 0) {
                        exceedDp[i][j] = 0;
                    } else {
                        exceedDp[i][j] = Integer.MAX_VALUE;
                    }
                }
            }

            //第二哥dp是完全背包问题，exceedDp对硬币种类和找零金额做动态规划
            for (int i = 1; i < exceedDp.length; i++) {
                for (int j = 1; j < exceedDp[i].length; j++) {
                    int min = Integer.MAX_VALUE;
                    int val = j * 5;
                    int maxk = val / getAmountByIndex(i);
                    for (int k = 0; k <= maxk; k++) {
                        int remainAmount = j * 5 - k * getAmountByIndex(i);
                        if (remainAmount >= 0 && exceedDp[i - 1][remainAmount / 5] != Integer.MAX_VALUE) {
                            min = Math.min(min, exceedDp[i - 1][remainAmount / 5] + k);
                        }
                    }
                    exceedDp[i][j] = min;
                }
            }

            //gather
            int start = amount / 5;
            int totalMin = Integer.MAX_VALUE;
            for (int i = start; i <= cols; i++) {
                if (dp[6][i] != Integer.MAX_VALUE && exceedDp[6][i - start] != Integer.MAX_VALUE) {
                    if (totalMin > dp[6][i] + exceedDp[6][i - start]) {
                        totalMin = dp[6][i] + exceedDp[6][i - start];
                    }
                }
            }
            System.out.printf("%3d\n",totalMin);
        }

    }

    public static int getAmountByIndex(int index) {
        switch (index) {
            case 1:
                return 5;
            case 2:
                return 10;
            case 3:
                return 20;
            case 4:
                return 50;
            case 5:
                return 100;
            case 6:
                return 200;
            default:
                return 0;
        }
    }

}
```
