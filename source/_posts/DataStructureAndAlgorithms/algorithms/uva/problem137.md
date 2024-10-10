---
title: problem137 Polygons
date: 2024-10-10 21:02:56
mathjax: true
categories:
- [数据结构与算法,uva]
tags:
- 算法
- uva
description: "UVA 137.Polygons <br>
- 计算几何问题:求两个多边形非相交部分的面积和"
---

## 题目

Given two convex polygons, they may or may not overlap. If they do overlap, they will do so to differing degrees and in different ways. Write a program that will read in the coordinates of the corners of two convex polygons and calculate the ‘exclusive or’ of the two areas, that is the area that is bounded by exactly one of the polygons. The desired area is shaded in the following diagram:
{% asset_img 137_pic1.png polyons %}

### Input

Input will consist of pairs of lines each containing the number of vertices of the polygon, followed by that many pairs of integers representing the x,y coordinates of the corners in a clockwise direction. All the coordinates will be positive integers less than 100. For each pair of polygons (pair of lines in the data file), your program should print out the desired area correct to two decimal places. The input will end with a line containing a zero (0).

### Output

Output will consist of a single line containing the desired area written as a succession of eight (8) digit fields with two (2) digits after the decimal point. There will not be enough cases to need more than one line.

Note: The symbol ⊔ in the Sample Output below represents a space.

### SampleInput

3 5 5 8 1 2 3
3 5 5 8 1 2 3
4 1 2 1 4 5 4 5 2
6 6 3 8 2 8 1 4 1 4 2 5 3
0

### SampleOutput

⊔⊔⊔⊔0.00⊔⊔⊔13.50

### 题意

给定两个多边形S1和S2，求两个多边形异或的面积，所谓异或面积是指属于S1或S2但不能S1和S2同时拥有的区域面积。例如S1和S2的面积分别为S1和S2，多边形相交部分面积为S3，则异或面积为S1-S3+S2-S3.如图中灰色阴影部分面积。

## 思路

题目非常直观，解法涉及到计算几何的知识，过程如下：求出两个多边形相交的区域，所以需要将相交部分的所有顶点给找出来，两个多边形相交，交点要么落在两条边相交的交点上，要么一个多边形的点在另一个多边形的内部，求出所有的点后在求出这些点所包围的区域，最后求出区域面积。

上面这些过程涉及到几个知识点：

1. 求出两条线段的交点。判断线段相交采用了向量叉乘，如果乘积大小为0表示平行或共线，针对共线特殊分析，找出共线时候的端点即为线段交点。非平行情况下参考下面的方式求出交点。
    {% asset_img 137_pic2.png 线段求交点 %}
2. 判断一个点是否在多边形内部。针对凸多边形，采用向量叉积的方式，叉积可以用来判断两个向量的相对位置，例如 $\vec{a} \times \vec{b}$ 大于0表示 $\vec{b}$ 在 $\vec{a}$ 的左边，当一个点在多边形每一条边的同一侧时则点在多边形内部。
3. 根据所有点求出所围成的区域，采用Graham scan扫描法。
4. 求出多边形的面积。采用叉乘计算。

### 问题

#### 实现上的问题

#### 性能上的问题

## 实现

```JAVA .{line-numbers}
//source file
import java.math.BigDecimal;
import java.math.RoundingMode;
import java.util.*;

public class Main {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        StringBuilder sb = new StringBuilder();
        while (scanner.hasNext()) {
            int num1 = scanner.nextInt();
            if (num1 == 0) {
                break;
            }
            List<Point> num1Points = new ArrayList<>();
            for (int i = 0; i < num1; i++) {
                Point point = new Point(new BigDecimal(scanner.nextInt()),
                        new BigDecimal(scanner.nextInt()));
                num1Points.add(point);
            }
            int num2 = scanner.nextInt();
            List<Point> num2Points = new ArrayList<>();
            for (int i = 0; i < num2; i++) {
                Point point = new Point(new BigDecimal(scanner.nextInt()),
                        new BigDecimal(scanner.nextInt()));
                num2Points.add(point);
            }

            //获取两个多边形的交点
            //1.分别判断点是否在另一个多边形内部
            List<Point> lineIntersects = new ArrayList<>();
            for (int i = 0; i < num1Points.size(); i++) {
                Point cur = num1Points.get(i);
                if (pointIsInPolygon(cur, num2Points)) {
                    lineIntersects.add(cur);
                }
            }

            for (int i = 0; i < num2Points.size(); i++) {
                Point cur = num2Points.get(i);
                if (pointIsInPolygon(cur, num1Points)) {
                    lineIntersects.add(cur);
                }
            }

            //两个多边形两两线段判断是否相交，相交则求出交点
            for (int i = 0; i < num1Points.size(); i++) {
                Point cur = num1Points.get(i);
                Point next = num1Points.get((i + 1) % num1Points.size());
                for (int j = 0; j < num2Points.size(); j++) {
                    Point curb = num2Points.get(j);
                    Point nextb = num2Points.get((j + 1) % num2Points.size());
                    getLineIntersect(cur, next, curb, nextb, lineIntersects);
                }
            }

            if (!lineIntersects.isEmpty()) {
                lineIntersects.sort((p1, p2) -> {
                    if (p1.px.compareTo(p2.px) < 0) {
                        return -1;
                    } else if (p1.px.compareTo(p2.px) > 0) {
                        return 1;
                    } else {
                        if (p1.py.compareTo(p2.py) < 0) {
                            return -1;
                        } else if (p1.py.compareTo(p2.py) > 0) {
                            return 1;
                        } else {
                            return 0;
                        }
                    }
                });
                boolean first = true;
                Point before = null;
                Iterator<Point> it = lineIntersects.iterator();
                while (it.hasNext()) {
                    Point cur = it.next();
                    if (first) {
                        before = cur;
                        first = false;
                    } else {
                        if (cur.px.compareTo(before.px) == 0 &&
                                cur.py.compareTo(before.py) == 0) {
                            it.remove();
                        } else {
                            before = cur;
                        }
                    }
                }

                //Graham scan扫描法求出相交的区域
                int baseIndex = findLeftBottomMostPoint(lineIntersects);
                Point basePoint = lineIntersects.get(baseIndex);
                lineIntersects.remove(baseIndex);
                lineIntersects.sort((a, b) -> {
                    //counterclock
                    double lenA = Math.sqrt(a.py.subtract(basePoint.py).multiply(a.py.subtract(basePoint.py))
                            .add(a.px.subtract(basePoint.px).multiply((a.px.subtract(basePoint.px)))).doubleValue());
                    double lenB = Math.sqrt((b.py.subtract(basePoint.py)).multiply(b.py.subtract(basePoint.py))
                            .add(b.px.subtract(basePoint.px).multiply(b.px.subtract(basePoint.px))).doubleValue());
                    double ancA = (a.px.subtract(basePoint.px)).doubleValue() / lenA;
                    double ancB = (b.px.subtract(basePoint.px)).doubleValue() / lenB;
//                System.out.printf("%f %f\n", ancA, ancB);
                    if (ancA == ancB && lenA > lenB) {
                        return 1;
                    } else if (ancA == ancB && lenA < lenB) {
                        return -1;
                    }
                    return ancA > ancB ? -1 : 1;
                });
                lineIntersects.add(0, basePoint);

                List<Point> boundary = findBoundary(lineIntersects);

                //计算面积
                BigDecimal area1 = calculateArea(num1Points);
                BigDecimal area2 = calculateArea(num2Points);
                BigDecimal area3 = calculateArea(boundary);
                BigDecimal finalArea = area1.subtract(area3).add(area2.subtract(area3)).setScale(2, RoundingMode.HALF_UP);
                String areaStr = finalArea.toPlainString();
                int blankCnt = 8 - areaStr.length();
                for (int i = 0; i < blankCnt; i++) {
                    sb.append(" ");
                }
                sb.append(areaStr);
            } else {
                BigDecimal area1 = calculateArea(num1Points);
                BigDecimal area2 = calculateArea(num2Points);
                BigDecimal finalArea = area1.add(area2).setScale(2, RoundingMode.HALF_UP);
                String areaStr = finalArea.toPlainString();
                int blankCnt = 8 - areaStr.length();
                for (int i = 0; i < blankCnt; i++) {
                    sb.append(" ");
                }
                sb.append(areaStr);
            }
        }
        System.out.println(sb.toString());
    }

    /**
     * Graham scan扫描法计算凸包
     */
    private static List<Point> findBoundary(List<Point> points) {
        if (points.size() <= 3) {
            return points;
        }
        int top = 1, index = 2;
        for (; index < points.size(); index++) {
            Point x = points.get(top - 1);
            Point y = points.get(top);
            Point z = points.get(index);
            BigDecimal crossMulti = getMulti(x, y, z);
            while (top > 0 && crossMulti.compareTo(BigDecimal.ZERO) < 0) {
                top--;
//                System.out.printf("%d,%d,%d", top, index, points.size());
                crossMulti = getMulti(points.get(top - 1), points.get(top), points.get(index));
            }
            points.set(top + 1, points.get(index));
            top++;
        }
        List<Point> result = new ArrayList<>(points.subList(0, top + 1));
        return result;
    }

    public static BigDecimal getMulti(Point p1, Point p2, Point p3) {
        Point vec1 = new Point(p2.px.subtract(p1.px), p2.py.subtract(p1.py));
        Point vec2 = new Point(p3.px.subtract(p1.px), p3.py.subtract(p1.py));
        return vec1.px.multiply(vec2.py).subtract(vec2.px.multiply(vec1.py));
    }

    /**
     * 计算线段的交点
     *
     * @param p1
     * @param p2
     * @param p3
     * @param p4
     */
    public static void getLineIntersect(Point p1, Point p2, Point p3, Point p4, List<Point> linePoints) {
        Point vec1 = new Point(p2.px.subtract(p1.px), p2.py.subtract(p1.py));
        Point vec2 = new Point(p4.px.subtract(p3.px), p4.py.subtract(p3.py));
        BigDecimal multi = crossMulti(vec1, vec2);
        if (multi.compareTo(BigDecimal.ZERO) == 0) {
            //parallel
            Point vec3 = new Point(p3.px.subtract(p1.px), p3.py.subtract(p1.py));
            if (crossMulti(vec1, vec3).compareTo(BigDecimal.ZERO) == 0) {
                //overlap
                Point min = p1.px.compareTo(p2.px) > 0 ? p2 : p1;
                Point max = p1.px.compareTo(p2.px) > 0 ? p1 : p2;
                if (p3.px.compareTo(min.px) < 0) {
                    if (p4.px.compareTo(min.px) == 0) {
                        linePoints.add(p4);
                    } else if (p4.px.compareTo(min.px) > 0 && p4.px.compareTo(max.px) <= 0) {
                        linePoints.add(min);
                        linePoints.add(p4);
                    } else if (p4.px.compareTo(max.px) > 0) {
                        linePoints.add(min);
                        linePoints.add(max);
                    }
                } else if (p3.px.compareTo(min.px) == 0) {
                    if (p4.px.compareTo(min.px) < 0) {
                        linePoints.add(min);
                    } else if (p4.px.compareTo(min.px) > 0 && p4.px.compareTo(max.px) <= 0) {
                        linePoints.add(min);
                        linePoints.add(p4);
                    } else if (p4.px.compareTo(max.px) > 0) {
                        linePoints.add(min);
                        linePoints.add(max);
                    }
                } else if (p3.px.compareTo(min.px) > 0 && p3.px.compareTo(max.px) < 0) {
                    if (p4.px.compareTo(min.px) < 0) {
                        linePoints.add(min);
                        linePoints.add(p3);
                    } else if (p4.px.compareTo(min.px) >= 0 && p4.px.compareTo(max.px) <= 0) {
                        linePoints.add(p3);
                        linePoints.add(p4);
                    } else if (p4.px.compareTo(max.px) > 0) {
                        linePoints.add(p3);
                        linePoints.add(max);
                    }
                } else if (p3.px.compareTo(max.px) == 0) {
                    if (p4.px.compareTo(min.px) < 0) {
                        linePoints.add(min);
                        linePoints.add(max);
                    } else if (p4.px.compareTo(min.px) >= 0 && p4.px.compareTo(max.px) <= 0) {
                        linePoints.add(p3);
                        linePoints.add(p4);
                    } else if (p4.px.compareTo(max.px) > 0) {
                        linePoints.add(p3);
                    }
                } else {
                    if (p4.px.compareTo(min.px) < 0) {
                        linePoints.add(min);
                        linePoints.add(max);
                    } else if (p4.px.compareTo(min.px) >= 0 && p4.px.compareTo(max.px) <= 0) {
                        linePoints.add(max);
                        linePoints.add(p4);
                    }
                }
            }
        } else {
            BigDecimal tup = p3.px.subtract(p1.px).multiply(p4.py.subtract(p3.py))
                    .subtract(p3.py.subtract(p1.py).multiply(p4.px.subtract(p3.px)));
            BigDecimal sup = p3.px.subtract(p1.px).multiply(p2.py.subtract(p1.py))
                    .subtract(p3.py.subtract(p1.py).multiply(p2.px.subtract(p1.px)));
            BigDecimal t = tup.divide(multi, 2, RoundingMode.HALF_UP);
            BigDecimal s = sup.divide(multi, 2, RoundingMode.HALF_UP);
            if (t.compareTo(BigDecimal.ZERO) >= 0 && t.compareTo(BigDecimal.ONE) <= 0
                    && s.compareTo(BigDecimal.ZERO) >= 0 && s.compareTo(BigDecimal.ONE) <= 0) {
                BigDecimal x = p1.px.add(tup.multiply(p2.px.subtract(p1.px)).divide(multi, 10, RoundingMode.HALF_UP));
                BigDecimal y = p1.py.add(tup.multiply(p2.py.subtract(p1.py)).divide(multi, 10, RoundingMode.HALF_UP));
                Point sect = new Point(x, y);
                linePoints.add(sect);
            }
        }
    }

    public static BigDecimal crossMulti(Point vec1, Point vec2) {
        return vec1.px.multiply(vec2.py).subtract(vec2.px.multiply(vec1.py));
    }


    /**
     * 判断点是否在多边形内部
     *
     * @param point
     * @param polygon
     * @return
     */
    public static boolean pointIsInPolygon(Point point, List<Point> polygon) {
        for (int i = 0; i < polygon.size(); i++) {
            Point cur = polygon.get(i);
            Point next = polygon.get((i + 1) % polygon.size());
            Point vec1 = new Point(next.px.subtract(cur.px), next.py.subtract(cur.py));
            Point vec2 = new Point(point.px.subtract(cur.px), point.py.subtract(cur.py));
            BigDecimal direct = crossMulti(vec1, vec2);
            if (direct.compareTo(BigDecimal.ZERO) > 0) {
                return false;
            }
        }
        return true;
    }

    /**
     * 计算多边形面积
     */
    public static BigDecimal calculateArea(List<Point> polygon) {
        BigDecimal area = BigDecimal.ZERO;
        for (int i = 0; i < polygon.size(); i++) {
            Point x1 = polygon.get(i);
            Point x2 = polygon.get((i + 1) % polygon.size());
            BigDecimal partArea = new BigDecimal("0.5");
            BigDecimal temp = x1.px.multiply(x2.py).subtract(x1.py.multiply(x2.px));
            partArea = partArea.multiply(temp);
            area = area.add(partArea);
        }
        return area.abs();
    }

    private static int findLeftBottomMostPoint(List<Point> points) {
        Point basePoint = points.get(0);
        int result = 0;
        for (int i = 1; i < points.size(); i++) {
            Point p = points.get(i);
            if (p.py.compareTo(basePoint.py) < 0) {
                basePoint = p;
                result = i;
            } else if (p.py.compareTo(basePoint.py) == 0) {
                if (p.px.compareTo(basePoint.px) < 0) {
                    basePoint = p;
                    result = i;
                }
            }
        }
        return result;
    }

    public static class Point {
        private BigDecimal px;
        private BigDecimal py;

        public Point(BigDecimal px, BigDecimal py) {
            this.px = px;
            this.py = py;
        }

        public BigDecimal getPx() {
            return px;
        }

        public void setPx(BigDecimal px) {
            this.px = px;
        }

        public BigDecimal getPy() {
            return py;
        }

        public void setPy(BigDecimal py) {
            this.py = py;
        }
    }
}
```