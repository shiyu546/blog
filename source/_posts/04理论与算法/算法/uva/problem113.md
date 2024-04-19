# Power of Cryptography

## 1. 题目

Current work in cryptography involves (among other things) large prime numbers and computing powers of numbers modulo functions of these primes. Work in this area has resulted in the practical use of results from number theory and other branches of mathematics once considered to be of only theoretical interest.

This problem involves the efficient computation of integer roots of numbers.

Given an integer $n\geq 1$ and an integer $p\geq 1$ you are to write a program that determines $\sqrt[n]{p}$, the positive n-th root of p. In this problem, given such integers n and p, p will always be of the form $k^n$ for an integer k (this integer is what your program must find).

### Input

The input consists of a sequence of integer pairs n and p with each integer on a line by itself. For all such pairs $1 \leq n \leq 200, 1 \leq p \leq 10^{101}$ and there exists an integer k, $ 1 \leq k \leq 10^{9} $such that $k^n = p$.

### Output

For each integer pair n and p the value $\sqrt[n]{p}$ should be printed, i.e., the number k such that $k^n = p$.

### Sample Input

2
16
3
27
7
4357186184021382204544

### Sample Output

43
1234

### 1.1题意

题意比较简单，给定两个数字n和p，找出数字k，使得$k^n=p$.

## 2. 思路

从题目出发，找出$k^n=p$。因p的范围比较大，已经超出了整数所能表示的范围，64位整型能表示$2^{64}$, 是小于$10^{101}$的，所以只能用字符串来表示p。

开方是比较困难的运算，且现在数字是字符串表示的，所以没法直接使用现有的工具函数。经过判断，先把k缩小到一个比较小的范围，然后采用二分法逐步递进到正确值。过程如下:

1. k的n次方为p，我们可以从p的位数和n的大小推算出k大概多少位。以10为底计算，10的n次方n+1位，100的n次方有2n+1位，依此类推,10的m次方的n次方位数为m*n+1.假设p的位数为b，且$(10^m)^n\leq p\leq (10^{m+1})^n $,则  
$$ m*n+1\leq b \leq (m+1)*n+1 $$
b和n都是已知，所以可以推算出m的值，进而确定k的位数：
$$ m=\lfloor (b-1)/n \rfloor $$

2. 采用二分法，k的位数介于m和m+1位之间，我们取$10^m$和$10^{m+1}-1$作为二分查找的上下界，取中间值c,并计算$c^k$的值并与p的大小相比较。根据比较结果调整上下界，逐步逼近到k。

### 2.1问题

整数的乘方在这里不适用，所以需要实现字符串的乘方运算。

效率方面，字符串实现乘法和加法时，保留中间结果最开始采用StringBuilder的insert()方法, 通过比较发现，append()方法结合reverse()方法效率更高。

## 3. 实现

```JAVA
import java.util.Scanner;

public class Main {
    private static final char[] numbers = {'0', '1', '2', '3', '4', '5', '6', '7', '8', '9'};

    public static void main(String[] args) {
//        File file = new File("number.txt");
//        Scanner scanner = null;
//        try {
//            scanner = new Scanner(file);
//        } catch (FileNotFoundException e) {
//            throw new RuntimeException(e);
//        }
        Scanner scanner = new Scanner(System.in);
        label:
        while (scanner.hasNextInt()) {
            String expStr = scanner.nextLine();
            int exp = Integer.valueOf(expStr);
            String number = scanner.nextLine();

            //1.place时计算k的位数范围
            int place = (number.length() - 1) / exp;
            int min = intPower(10, place);
            int max = intPower(10, place + 1);

            //2.二分查找
            while (min < max) {
                int middle = (min + max) / 2;

                //3.计算k的n次方
                String powStr = multiplePower(String.valueOf(middle), exp);

                //比较字符串的数字大小，
                int compare = compareStr(powStr, number);
                if (compare == 1) {
                    max = middle;
                } else if (compare == -1) {
                    min = middle;
                } else {
                    System.out.printf("%s\n", middle);
                    continue label;
                }
            }
        }
    }

    private static int intPower(int base, int exp) {
        int result = 1;
        for (int i = 0; i < exp; i++) {
            result *= base;
        }
        return result;
    }

    //乘方计算
    private static String multiplePower(String s1, int power) {
        String result = "1";
        for (int i = 0; i < power; i++) {
            result = multipleStr(result, s1);
        }
        return result;
    }

    //字符串乘法
    private static String multipleStr(String s1, String s2) {
        int temp = 0;
        StringBuilder result = new StringBuilder();
        result.append('0');
        for (int i = s2.length() - 1; i >= 0; i--) {
            StringBuilder sb = new StringBuilder();
            for (int j = s1.length() - 1; j >= 0; j--) {
                int bitNums1 = s1.charAt(j) - '0';
                int bitNums2 = s2.charAt(i) - '0';
                int mult = (bitNums1 * bitNums2 + temp) % 10;
                temp = (bitNums1 * bitNums2 + temp) / 10;
                sb.append(numbers[mult]);
            }
            if (temp != 0) {
                sb.append(numbers[temp]);
                temp = 0;   //reset temp=0 for next cycle
            }
            sb = sb.reverse();
            //fill number of 0:s2.length() - i - 1;
            for (int k = 0; k < s2.length() - i - 1; k++) {
                sb.append('0');
            }

            result = addStr(result, sb);
        }
        return result.toString();
    }

    //字符串加法
    private static StringBuilder addStr(StringBuilder s1, StringBuilder s2) {
        int temp = 0;
        int bitNums1, bitNums2, add;
        int k1 = s1.length() - 1, k2 = s2.length() - 1;

        int reslen = k1 > k2 ? k1 + 2 : k2 + 2;
        StringBuilder result = new StringBuilder(reslen);

        while (k1 >= 0 && k2 >= 0) {
            bitNums1 = s1.charAt(k1) - '0';
            bitNums2 = s2.charAt(k2) - '0';
            add = (bitNums1 + bitNums2 + temp) % 10;
            temp = (bitNums1 + bitNums2 + temp) / 10;
            result.append(numbers[add]);
            k1--;
            k2--;
        }
        if (k1 >= 0) {
            while (k1 >= 0) {
                bitNums1 = s1.charAt(k1) - '0';
                add = (bitNums1 + temp) % 10;
                temp = (bitNums1 + temp) / 10;
                result.append(numbers[add]);
                k1--;
            }
        } else if (k2 >= 0) {
            while (k2 >= 0) {
                bitNums2 = s2.charAt(k2) - '0';
                add = (bitNums2 + temp) % 10;
                temp = (bitNums2 + temp) / 10;
                result.append(numbers[add]);
                k2--;
            }
        }
        if (temp != 0) {
            result.append(numbers[temp]);
        }

        return result.reverse();
    }

    //比较字符串，本质上比较字符串表示的数字大小
    private static int compareStr(String s1, String s2) {
        if (s1.length() > s2.length()) {
            return 1;
        } else if (s1.length() < s2.length()) {
            return -1;
        }
        for (int i = 0; i < s1.length(); i++) {
            int bitNums1 = s1.charAt(i) - '0';
            int bitNums2 = s2.charAt(i) - '0';
            if (bitNums1 > bitNums2) {
                return 1;
            } else if (bitNums1 < bitNums2) {
                return -1;
            }
        }
        return 0;
    }
}
```
