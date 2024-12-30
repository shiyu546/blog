---
title: problem149 Forests
date: 2024-12-29 09:25:03
mathjax: true
categories:
- [数据结构与算法,uva]
tags:
- 算法
- uva
description: "UVA problem149.Forests <br>
- 判断能看见多少颗树"
---

## 题目

The saying “You can't see the wood for the trees” is not only a cliche, but is also incorrect. The real problem is that you can't see the trees for the wood. If you stand in the middle of a “wood” (in NZ terms, a patch of bush), the trees tend to obscure each other and the number of distinct trees you can actually see is quite small. This is especially true if the trees are planted in rows and columns (as in a pine plantation), because they tend to line up. The purpose of this problem is to find how many distinct trees you can see from an arbitrary point in a pine plantation (assumed to stretch “for ever”).
{% asset_img 149_pic1.png %}

You can only see a distinct tree if no part of its trunk is obscured by a nearer tree — that is if both sides of the trunk can be seen, with a discernible gap between them and the trunks of all trees closer to you. Also, you can’t see a tree if it is apparently “too small”. For definiteness, “not too small” and “discernible gap” will mean that the angle subtended at your eye is greater than 0.01 degrees (you are assumed to use one eye for observing). Thus the two trees marked O(white) obscure at least the trees marked O(grey) from the given view point.

Write a program that will determine the number of trees visible under these assumptions, given the diameter of the trees, and the coordinates of a viewing position. Because the grid is infinite, the origin is unimportant, and the coordinates will be numbers between 0 and 1.

### Input

Input will consist of a series of lines, each line containing three real numbers of the form ‘0.nn’. The first number will be the trunk diameter — all trees will be assumed to be cylinders of exactly this diameter, with their centres placed exactly on the points of a rectangular grid with a spacing of one unit. The next two numbers will be the x and y coordinates of the observer. To avoid potential problems, say by being too close to a tree, we will guarantee that $diameter \leq x,y \leq (1 - diameter)$. To avoid problems with trees being too small you may assume that diameter$\geq 0.1$. The file will be terminated by a line consisting of three zeroes.

### Output

Output will consist of a series of lines, one for each line of the input. Each line will consist of the number of trees of the given size, visible from the given position.

### SampleInput

0.10 0.46 0.38
0 0 0

### SampleOutput

128

### 题意

给定直角坐标系，坐标系上的整数坐标点都是树的中心，树的直径都是固定值diameter.现给定一个观察点，判断从该位置能看到多少颗树。树能看到的含义是该树不被其他树遮挡，且至少与近距离的树从观察角度保持0.01度的角度，同时树在观察点看到的宽度不能低于0.01度。

## 思路

由近到远遍历，判断每一颗树是否与近处的所有树有遮挡。遍历的问题在于什么时候终止判断，如果树观察到的角度过小则停止遍历，这个距离的树距离观察点将近286左右，但只需要取10左右的树即可(原理？).

### 问题

#### 实现上的问题

#### 性能上的问题

## 实现

```JAVA .{line-numbers}
//source code
import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;
import java.util.Scanner;

public class Main {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        while (true) {
            String diamterStr = scanner.next();
            String xStr = scanner.next();
            String yStr = scanner.next();

            if ("0".equals(diamterStr) && "0".equals(xStr) && "0".equals(yStr)) {
                break;
            }
            double diameter = Double.valueOf(diamterStr);
            double x = Double.valueOf(xStr);
            double y = Double.valueOf(yStr);


            int totalTree = 0;
            List<Region> regions = new ArrayList<>();
            for (int index = 1; index <= 20; index++) {
                //get the corner of around
                int topX = getLeftTopCornerX(index);
                int topY = getLeftTopCornerY(index);
                int bottomX = getLeftBottomCornerX(index);
                int bottomY = getLeftBottomCornerY(index);
                List<Integer> rows = Arrays.asList(topY, bottomY);
                List<Integer> cols = Arrays.asList(topX, bottomX);
                int roundNum = 0, tooFar = 0;


                for (int j = 0; j < rows.size(); j++) {
                    for (int i = topX; i <= bottomX; i += 1) {
                        //calculate region
                        Region region = calculateRange(i, rows.get(j), x, y, diameter);
                        if (region != null) {
                            regions.add(region);
                        }
                    }
                }
                for (int i = bottomY + 1; i < topY; i += 1) {
                    for (int j = 0; j < cols.size(); j++) {
                        //calculate region
                        Region region = calculateRange(cols.get(j), i, x, y, diameter);
                        if (region != null) {
                            regions.add(region);
                        }
                    }
                }
            }
            for (int i = 0; i < regions.size(); i++) {
                Region cur = regions.get(i);
//                System.out.println("pos:(" + cur.startAngle + "," + cur.endAngle + ")");
                int j = 0;
                for (; j < i; j++) {
                    if (obsecured(regions.get(j), cur)) {
                        break;
                    }
                }
                if (j >= i) {
                    totalTree++;
                }
            }
            System.out.println(totalTree);
        }
    }

    private static Region calculateRange(int centerX, Integer centerY, double x, double y, double diameter) {
        double angle = 0;
        if (centerX == x) {
            if (centerY > y) {
                angle = 90.00;
            } else {
                angle = 270.00;
            }
        } else {
            angle = Math.toDegrees(Math.atan(((double) (centerY - y)) / (centerX - x)));
            if (centerX > x && centerY >= y) {
                angle += 0;
            } else if (centerX < x && centerY >= y) {
                angle += 180;
            } else if (centerX < x && centerY < y) {
                angle += 180;
            } else {
                angle += 360;
            }
        }

        double angleA = Math.toDegrees(Math.asin(diameter / (2.0 * Math.sqrt((x - centerX) * (x - centerX) + (y - centerY) * (y - centerY)))));
        if (angleA * 2 <= 0.01) {
            return null;
        }
        Region region = new Region();
        region.startAngle = angle - angleA;
        region.endAngle = angle + angleA;
        return region;
    }

    private static int getLeftBottomCornerY(int index) {
        return -(index - 1);
    }

    private static int getLeftBottomCornerX(int index) {
        return index;
    }

    private static int getLeftTopCornerY(int index) {
        return index;
    }

    private static int getLeftTopCornerX(int index) {
        return -(index - 1);
    }

    public static boolean obsecured(Region region, Region current) {
        if (region.startAngle - current.endAngle >= 0.01 || current.startAngle - region.endAngle >= 0.01) {
            return false;
        }
        return true;
    }

    public static class Region {
        double startAngle;
        double endAngle;
    }
}
```
