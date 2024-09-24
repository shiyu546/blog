---
title: problem132 Bumpy Objects
date: 2024-09-24 20:14:46
mathjax: true
categories:
- [数据结构与算法,uva]
tags:
- 算法
- uva
description: "UVA 132.Bumpy Objects <br>
- 计算几何凸包问题"
---

## 题目

Consider objects such as these. They are polygons, specified by the coordinates of a centre of mass and their vertices. In the figure, centres of mass are shown as black squares. The vertices will be numbered consecutively anti-clockwise as shown.
{% asset_img 132_pic1.png 多边形 %}

An object can be rotated to stand stably if two vertices can be found that can be joined by a straight line that does not intersect the object, and, when this line is horizontal, the centre of mass lies above the line and strictly between its endpoints. There are typically many stable positions and each is defined by one of these lines known as its base line. A base line, and its associated stable position,is identified by the highest numbered vertex touched by that line.

Write a program that will determine the stable position that has the lowest numbered base line.Thus for the above objects, the desired base lines would be 6 for object 1, 6 for object 2 and 2 for the square. You may assume that the objects are possible, that is they will be represented as non
self-intersecting polygons, although they may well be concave.

### Input

Successive lines of a data set will contain: a string of less than 20 characters identifying the object; the coordinates of the centre of mass; and the coordinates of successive points terminated by two zeroes (0 0), on one or more lines as necessary. There may be successive data sets (objects). The end of data will be defined by the string ’#’.

### Output

Output will consist of the identification string followed by the number of the relevant base line.

### SampleInput

Object2
4 3
3 2 5 2 6 1 7 1 6 3 4 7 1 1 2 1 0 0
Square
2 2
1 1 3 1 3 3 1 3 0 0
\#

### SampleOutput

Object2 6
Square 2

### 题意

题目给了一个多边形和多边形的质心，如果多边形上任意两点满足以下条件：

1. 两点构成的直线不与多边形相交；
2. 绕质心旋转使得两点构成的线段水平，如果质心在线段之上且处于两点之间。

这种线段就叫做基线，基线上点数字较大者就是这条基线的大小，现在求出多边形的最小基线。

## 思路

这道题题意不是很清晰，至少在描述上缺少很多确定性的信息，例如：line与object相交的判定，多边形多点共线如何判定。

通过查看他人解决方案，问题求解可分为：求解多边形的凸包多边形，再判断该凸包多边形与质心的位置求出所有的baseline，再取baseline的最小值。

### 问题

#### 实现上的问题

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
        while (true) {
            String objectName = scanner.next();
            if (objectName.equals("#")) {
                break;
            }
            int massPointx = scanner.nextInt();
            int massPointy = scanner.nextInt();
            Point massPoint = new Point(massPointx, massPointy, 0);
            int number = 1;
            List<Point> points = new ArrayList<>();
            while (true) {
                int pointx = scanner.nextInt();
                int pointy = scanner.nextInt();
                if (pointx == 0 && pointy == 0) {
                    break;
                }
                Point point = new Point(pointx, pointy, number);
                points.add(point);
                number++;
            }
            //caculate poly
            int baseIndex = findLeftBottomMostPoint(points);
            Point basePoint = points.get(baseIndex);
            points.remove(baseIndex);
            points.sort((a, b) -> {
                //counterclock
                double lenA = Math.sqrt((a.getPointy() - basePoint.getPointy()) * (a.getPointy() - basePoint.getPointy())
                        + (a.getPointx() - basePoint.getPointx()) * (a.getPointx() - basePoint.getPointx()));
                double lenB = Math.sqrt((b.getPointy() - basePoint.getPointy()) * (b.getPointy() - basePoint.getPointy())
                        + (b.getPointx() - basePoint.getPointx()) * (b.getPointx() - basePoint.getPointx()));
                double ancA = (a.getPointx() - basePoint.getPointx()) / lenA;
                double ancB = (b.getPointx() - basePoint.getPointx()) / lenB;
                if (ancA == ancB && lenA > lenB) {
                    return 1;
                } else if (ancA == ancB && lenA < lenB) {
                    return -1;
                }
                return ancA > ancB ? -1 : 1;
            });
            points.add(0, basePoint);

            List<Point> boundaryPoints = findBoundary(points);
            boundaryPoints = removeLinePoints(boundaryPoints);

            int minBaseLine = Integer.MAX_VALUE;
            for (int i = 0; i < boundaryPoints.size(); i++) {
                int j = (i + 1) % boundaryPoints.size();
                int crossMulti =
                        (boundaryPoints.get(j).pointx - boundaryPoints.get(i).pointx)
                                * (massPoint.pointy - boundaryPoints.get(i).pointy) -
                                (massPoint.pointx - boundaryPoints.get(i).pointx)
                                        * (boundaryPoints.get(j).pointy - boundaryPoints.get(i).pointy);
                int betweenPointA = (massPoint.pointx - boundaryPoints.get(i).pointx)
                        * (boundaryPoints.get(j).pointx - boundaryPoints.get(i).pointx) +
                        (massPoint.pointy - boundaryPoints.get(i).pointy)
                                * (boundaryPoints.get(j).pointy - boundaryPoints.get(i).pointy);
                int betweenPointB = (boundaryPoints.get(i).pointx - boundaryPoints.get(j).pointx)
                        * (massPoint.pointx - boundaryPoints.get(j).pointx) +
                        (boundaryPoints.get(i).pointy - boundaryPoints.get(j).pointy)
                                * (massPoint.pointy - boundaryPoints.get(j).pointy);
                if (crossMulti > 0 && betweenPointA > 0 && betweenPointB > 0) {
                    int baseLine = Math.max(boundaryPoints.get(i).number, boundaryPoints.get(j).number);
                    if (minBaseLine > baseLine) {
                        minBaseLine = baseLine;
                    }
                }
            }
            System.out.println(objectName + " " + minBaseLine);
        }
    }

    private static List<Point> removeLinePoints(List<Point> points) {
        int top = 1, index = 2;
        for (; index < points.size(); index++) {
            int crossMulti = getMulti(points.get(top - 1), points.get(top), points.get(index));
            if (crossMulti == 0) {
                points.set(top, points.get(index));
            } else {
                top++;
                points.set(top, points.get(index));
            }
        }
        List<Point> result = new ArrayList<>(points.subList(0, top + 1));
        return result;
    }

    private static List<Point> findBoundary(List<Point> points) {
        int top = 1, index = 2;
        for (; index < points.size(); index++) {
            int crossMulti = getMulti(points.get(top - 1), points.get(top), points.get(index));
            while (top > 0 && crossMulti < 0) {
                top--;
                crossMulti = getMulti(points.get(top - 1), points.get(top), points.get(index));
            }
            points.set(top + 1, points.get(index));
            top++;
        }
        List<Point> result = new ArrayList<>(points.subList(0, top + 1));
        return result;
    }

    public static int getMulti(Point x, Point y, Point z) {
        return (y.pointx - x.pointx) * (z.pointy - y.pointy) - (y.pointy - x.pointy) * (z.pointx - y.pointx);
    }


    private static int findLeftBottomMostPoint(List<Point> points) {
        Point basePoint = points.get(0);
        int result = 0;
        for (int i = 1; i < points.size(); i++) {
            Point point = points.get(i);
            if (point.getPointy() < basePoint.getPointy()) {
                basePoint = point;
                result = i;
            } else if (point.getPointy() == basePoint.getPointy()) {
                if (point.getPointx() < basePoint.getPointx()) {
                    basePoint = point;
                    result = i;
                }
            }
        }
        return result;
    }

    public static class Point {
        private int pointx;
        private int pointy;

        int number;

        public Point(int pointx, int pointy, int number) {
            this.pointx = pointx;
            this.pointy = pointy;
            this.number = number;
        }

        public int getNumber() {
            return number;
        }

        public void setNumber(int number) {
            this.number = number;
        }

        public int getPointx() {
            return pointx;
        }

        public void setPointx(int pointx) {
            this.pointx = pointx;
        }

        public int getPointy() {
            return pointy;
        }

        public void setPointy(int pointy) {
            this.pointy = pointy;
        }

        @Override
        public int hashCode() {
            return super.hashCode();
        }

        @Override
        public boolean equals(Object obj) {
            Point p = (Point) obj;
            return p.getPointx() == pointx && p.getPointy() == pointy;
        }
    }
}
```
