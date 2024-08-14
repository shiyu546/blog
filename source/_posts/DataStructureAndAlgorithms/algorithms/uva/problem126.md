---
title: 126.The Errant Physicist	
date: 2024-08-14 22:35:33
mathjax: true
categories:
- [数据结构与算法,uva]
tags:
- 算法
- uva
description: "UVA 126.The Errant Physicist	 <br>
- 按指定格式输出两个多项式的乘积"
---

## 题目

The well-known physicist Alfred E Neuman is working on problems that involve multiplying polynomials of x and y. For example, he may need to calculate $(-x^8y + 9x^3 - 1 + y)\times(x^5 y + 1 + x^3)$ getting the answer
$$-x^{13}y^2 - x^{11}y + 8x^8y + 9x^6 - x^5y + x^5y^2 + 8x^3 + x^3y - 1 + y$$

Unfortunately, such problems are so trivial that the great man’s mind keeps drifting off the job,and he gets the wrong answers. As a consequence, several nuclear warheads that he has designed have detonated prematurely, wiping out five major cities and a couple of rain forests.

You are to write a program to perform such multiplications and save the world.

### Input

The file of input data will contain pairs of lines, with each line containing no more than 80 characters.The final line of the input file contains a ‘#’ as its first character. Each input line contains a polynomial written without spaces and without any explicit exponentiation operator. Exponents are positive non-zero unsigned integers. Coefficients are also integers, but may be negative. Both exponents and coefficients are less than or equal to 100 in magnitude. Each term contains at most one factor in x and one in y.

### Output

Your program must multiply each pair of polynomials in the input, and print each product on a pair of lines, the first line containing all the exponents, suitably positioned with respect to the rest of theinformation, which is in the line below.

The following rules control the output format:

1. Terms in the output line must be sorted in decreasing order of powers of x and, for a given power of x, in increasing order of powers of y.
2. Like terms must be combined into a single term. For example, $40x^2y^3 -  38x^2y^3$ is replaced by $2x^2y^3$.
3. Terms with a zero coefficient must not be displayed.
4. Coefficients of 1 are omitted, except for the case of a constant term of 1.
5. Exponents of 1 are omitted.
6. Factors of x0 and y0 are omitted.
7. Binary pluses and minuses (that is the pluses and minuses connecting terms in the output) have a single blank column both before and after.
8. If the coefficient of the first term is negative, it is preceded by a unary minus in the first column,with no intervening blank column. Otherwise, the coefficient itself begins in the first output column.
9. The output can be assumed to fit into a single line of at most 80 characters in length.
10. There should be no blank lines printed between each pair of output lines.
11. The pair of lines that contain a product should be the same length — trailing blanks should appear after the last non-blank character of the shorter line to achieve this.

### SampleInput

-yx8+9x3-1+y
x5y+1+x3
1
1
\#

### SampleOutput

```plaintext
//output
  13 2    11       8      6    5     5 2     3    3
-x  y  - x  y +  8x y + 9x  - x y + x y  + 8x  + x y - 1 + y

1
```

### 题意

每两行为一组给定了两个多项式，多项式含有两个变量x,y,多项式没有空格和指数操作运算。要求出两个多项式的乘积并按指定的格式输出：指数占第一排，其余的占第二排，且指数在指定的位置上。

## 思路

针对两个多项式相乘，将第一个多项式的每一项与第二个多项式的每一项相乘，乘积相加。例如
$$(a+b)*(c+d)=a*c+a*d+b*c+b*d$$
实现上面每一个多项式都形如 $ax^by^c$,所以可以采用含有a,b,c三个变量的对象表示多项式中的一项。相乘过程如下：

1. 两个多项式每一项两两项相乘，得到乘积，例如第一个多项式有u项，第二个有v项，乘之后积共有u*v项；
2. 排序。按照x的次幂降次排序，如果x的次幂相同，按照y的次幂升序排序。
3. 合并同类项，如果两个项x的次幂和y的次幂相同，则系数相加，合并为一项。

### 问题

#### 实现上的问题

多项式乘法思路上没有什么问题，主要考虑的问题是处理输入和按指定格式输出。输入主要考虑如何把一行的多项式分割成一项项的项，几个关键节点包括：

1. 如果出现'+','-'，表示上一个项结束，下一个项开始，注意'-'可能出现在多项式的开头，需要排除这种情况；
2. 出现'x','y'表示前一个数字结束，后面的是x,y的系数，注意标记前一个位置数属于什么类别，是系数，x的指数或者是y的指数。

#### 性能上的问题

## 实现

```JAVA .{line-numbers}
//file
import java.util.ArrayList;
import java.util.List;
import java.util.Scanner;

public class Main {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        String line = scanner.nextLine();
        while (!"#".equals(line)) {
            String otherLine = scanner.nextLine();
            List<Term> terms = parse(line);
            List<Term> termTwo = parse(otherLine);

            //多项式相乘
            List<Term> multResult = new ArrayList<>();
            for (int i = 0; i < terms.size(); i++) {
                for (int j = 0; j < termTwo.size(); j++) {
                    Term multTerm = multTerm(terms.get(i), termTwo.get(j));
                    multResult.add(multTerm);
                }
            }

            //排序
            multResult.sort((terma, termb) -> {
                if (terma.expx < termb.expx) {
                    return 1;
                } else if (terma.expx > termb.expx) {
                    return -1;
                } else {
                    if (terma.expy < termb.expy) {
                        return -1;
                    } else if (terma.expy > termb.expy) {
                        return 1;
                    }
                }
                return 0;
            });

            //合并同类项
            int index = 0, scanIndex = 1;
            while (scanIndex < multResult.size()) {
                if (multResult.get(index).expx == multResult.get(scanIndex).expx
                        && multResult.get(index).expy == multResult.get(scanIndex).expy) {
                    multResult.get(index).coefficient += multResult.get(scanIndex).coefficient;
                    scanIndex++;
                } else {
                    if (index + 1 == scanIndex) {
                        index++;
                        scanIndex++;
                    } else {
                        index++;
                        multResult.set(index, multResult.get(scanIndex));
                        scanIndex++;
                    }
                }
            }

            //输出
            StringBuilder sbtop = new StringBuilder();
            StringBuilder sbSub = new StringBuilder();
            for (int i = 0; i <= index; i++) {
                if (multResult.get(i).coefficient == 0) {
                    continue;
                }
                if (multResult.get(i).coefficient > 0 && i > 0) {
                    sbtop.append("   ");
                    sbSub.append(" + ");
                } else if (multResult.get(i).coefficient < 0 && i > 0) {
                    sbtop.append("   ");
                    sbSub.append(" - ");
                } else if (multResult.get(i).coefficient < 0 && i == 0) {
                    sbtop.append(" ");
                    sbSub.append("-");
                }
                if ((multResult.get(i).coefficient > 1 || multResult.get(i).coefficient < -1)
                        || (multResult.get(i).expx == 0 && multResult.get(i).expy == 0)) {
                    int coffNum = multResult.get(i).coefficient > 0 ? multResult.get(i).coefficient : -multResult.get(i).coefficient;
                    String coff = String.valueOf(coffNum);
                    for (int j = 0; j < coff.length(); j++) {
                        sbtop.append(" ");
                    }
                    sbSub.append(coff);
                }

                if (multResult.get(i).expx != 0) {
                    sbtop.append(" ");
                    sbSub.append('x');
                    if (multResult.get(i).expx != 1) {
                        String expxStr = String.valueOf(multResult.get(i).expx);
                        for (int j = 0; j < expxStr.length(); j++) {
                            sbSub.append(" ");
                        }
                        sbtop.append(expxStr);
                    }
                }

                if (multResult.get(i).expy != 0) {
                    sbtop.append(" ");
                    sbSub.append('y');
                    if (multResult.get(i).expy != 1) {
                        String expyStr = String.valueOf(multResult.get(i).expy);
                        for (int j = 0; j < expyStr.length(); j++) {
                            sbSub.append(" ");
                        }
                        sbtop.append(expyStr);
                    }
                }
            }
            System.out.println(sbtop.toString());
            if (sbSub.length() == 0) {
                System.out.println("0");
            } else {
                System.out.println(sbSub.toString());
            }

            line = scanner.nextLine();
        }
    }

    /**
     * 处理输入多项式，分割成一个个的项
     */
    private static List<Term> parse(String line) {
        List<Term> result = new ArrayList<>();
        int index = 0;
        int coefficient = 1, expx = 0, expy = 0;
        boolean isCoefficient = true, isExpx = false, isExpy = false;
        String num = "";
        while (index < line.length()) {
            if (line.charAt(index) == '+' || line.charAt(index) == '-'
                    || line.charAt(index) == 'x' || line.charAt(index) == 'y') {
                int number;
                if (!num.equals("")) {
                    number = Integer.valueOf(num);
                } else {
                    number = 1;
                }
                if (isCoefficient) {
                    coefficient = number * coefficient;
                } else if (isExpx) {
                    expx = number;
                } else if (isExpy) {
                    expy = number;
                }
                if ((line.charAt(index) == '+' || line.charAt(index) == '-')) {
                    if (index > 0) {
                        Term term = new Term();
                        term.setCoefficient(coefficient);
                        term.setExpx(expx);
                        term.setExpy(expy);
                        result.add(term);
                    }
                    expx = 0;
                    expy = 0;
                    num = "";
                    coefficient = line.charAt(index) == '-' ? -1 : 1;
                    isCoefficient = true;
                    isExpx = false;
                    isExpy = false;
                } else if (line.charAt(index) == 'x') {
                    num = "";
                    isCoefficient = false;
                    isExpx = true;
                    isExpy = false;
                } else {
                    num = "";
                    isCoefficient = false;
                    isExpx = false;
                    isExpy = true;
                }

            } else {
                num += line.charAt(index);
            }
            index++;
        }
        int numbery;
        if (!num.equals("")) {
            numbery = Integer.valueOf(num);
        } else {
            numbery = 1;
        }
        if (isCoefficient) {
            coefficient = coefficient * numbery;
        } else if (isExpx) {
            expx = numbery;
        } else if (isExpy) {
            expy = numbery;
        }
        Term term3 = new Term();
        term3.setCoefficient(coefficient);
        term3.setExpx(expx);
        term3.setExpy(expy);
        result.add(term3);
        return result;
    }

    /**
     * 项相乘
     */
    private static Term multTerm(Term terma, Term termb) {
        Term mult = new Term();
        mult.setCoefficient(terma.coefficient * termb.coefficient);
        mult.setExpx(terma.expx + termb.expx);
        mult.setExpy(terma.expy + termb.expy);
        return mult;
    }


    /**
     * 每一项都是形如ax^by^c的形式，分别用coefficient，expx，expy表示a,b,c
     */
    private static class Term {
        private int coefficient;
        private int expx;
        private int expy;

        public int getCoefficient() {
            return coefficient;
        }

        public void setCoefficient(int coefficient) {
            this.coefficient = coefficient;
        }

        public int getExpx() {
            return expx;
        }

        public void setExpx(int expx) {
            this.expx = expx;
        }

        public int getExpy() {
            return expy;
        }

        public void setExpy(int expy) {
            this.expy = expy;
        }
    }
}
```
