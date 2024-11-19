---
title: problem142 Mouse Clicks
date: 2024-11-04 10:47:17
mathjax: true
categories:
- [数据结构与算法,uva]
tags:
- 算法
- uva
description: "UVA problem142.Mouse Clicks <br>
- 判断鼠标点击的点属于哪个区域或找出与点最近的图标"
---

## 题目

A typical windowing system on a computer will provide a number of icons on the screen as well as some defined regions. When the mouse button is clicked, the system has to determine where the cursor is and what is being selected. For this problem we assume that a mouse click in (or on the border of) a region selects that region, otherwise it selects the closest visible icon (or icons in the case of a tie).

Consider the following screen:

{% asset_img 142_pic1.png 屏幕示意图 %}

A mouse click at ‘a’ will select region A. A mouse click at ‘b’ will select icon 1. A mouse click at ‘c’ will select icons 6 and 7. A mouse click at ‘d’ is ambiguous. The ambiguity is resolved by assuming that one region is in front of another. In the data files, later regions can be assumed to be in front of earlier regions. Since regions are labelled in order of appearance (see later) ‘d’ will select C. Note that regions always overlap icons so that obscured icons need not be considered and that the origin (0,0) is at the top left corner.

Write a program that will read in a series of region and icon definitions followed by a series of mouse clicks and return the selected items. Coordinates will be given as pairs of integers in the range 0..499 and you can assume that all icons and regions lie wholly within the screen. Your program must number all icons (even invisible ones) in the order of arrival starting from 1 and label regions alphabetically in the order of arrival starting from ‘A’.

### Input

Input will consist of a series of lines. Each line will identify the type of data: ‘I’ for icon, ‘R’ for region and ‘M’ for mouse click. There will be no separation between the specification part and the event part, however no icon or region specifications will follow the first mouse click. An ‘I’ will be followed by the coordinates of the centre of the icon, ‘R’ will be followed by the coordinates of the top left and bottom right corners respectively and ‘M’ will be followed by the coordinates of the cursor at the time of the click. There will always be at least one visible icon and never more than 25 regions and 50 icons. The entire file will be terminated by a line consisting of a single ‘#’.

### Output

Output will consist of one line for each mouse click, containing the selection(s) for that click. Regions will be identified by their single character identifier, icon numbers will be written out right justified in a field of width 3, and where there is more than one icon number they will appear in increasing numerical order.

### SampleInput

I 216 28
R 22 19 170 102
I 40 150
I 96 138
I 36 193
R 305 13 425 103
I 191 184
I 387 200
R 266 63 370 140
I 419 134
I 170 102
M 50 50
M 236 30
M 403 167
M 330 83
\#

### SampleOutput

A
  1
  6  7
C

### 题意

给定一块屏幕，屏幕上共有两种图案：区域和图标。区域是一个矩形区域，由区域的左上角和右下角两个坐标点确定，图标是屏幕上的一个点。现在用鼠标点击屏幕(本质上是确定了屏幕上的一个点)，现判断点击的点是否属于某个区域，如果属于，输出区域编号；如果不属于，判断离点最近的图标，如果有多个图标，按顺序输出。

输入部分由多个区域，图标绘制以及鼠标点击构成，按顺序执行。

tips：

1. 区域所围成的矩形边界也属于区域；
2. 如果图标属于某个区域，则图标被区域覆盖掉导致不可见；
3. 多个区域重叠时，重叠部分后绘制的区域覆盖掉先绘制的区域;
4. 坐标原点在屏幕左上，横向向右是x的正轴，向下是y的正轴。

## 思路

由于区域的优先级很高，所以先判断鼠标点击的范围是否属于某个区域，而且由于区域的覆盖属性，所以先判断后绘制的区域，再判断先绘制的区域。判断点是否属于区域只需判断点的横纵坐标是否在区域的范围即可。

如果鼠标点击的点不属于区域，则需要比较点与所有可见图标的距离，取最小值的集合。假设点击的点为$A(x_1,y_1)$,图标点为$B(x_2,y_2)$这里直接采用点距离的坐标公式:
$$ Len(AB)=\sqrt{(x_1-x_2)^2+(y_1-y_2)^2} $$
由于我们不需要具体的值而只需要比较Len的大小，所以不需要开根号。

### 问题

#### 实现上的问题

#### 性能上的问题

## 实现

```JAVA .{line-numbers}
//source code
import java.util.*;

public class Main {
    private static char[] alphas = {'A', 'B', 'C', 'D', 'E', 'F', 'G', 'H', 'I', 'J', 'K', 'L', 'M', 'N', 'O', 'P', 'Q', 'R', 'S', 'T', 'U', 'V', 'W', 'X', 'Y', 'Z'};

    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        List<Icon> icons = new ArrayList<>();
        List<Region> regions = new ArrayList<>();
        String line = scanner.next();
        while (true) {
            if ("#".equals(line)) {
                break;
            }
            if (line.equals("I")) {
                int x = scanner.nextInt();
                int y = scanner.nextInt();
                Icon icon = new Icon();
                icon.x = x;
                icon.y = y;
                iconIsVisable(icon, regions);
                icons.add(icon);
            } else if (line.equals("R")) {
                int topx = scanner.nextInt();
                int topy = scanner.nextInt();
                int bottomx = scanner.nextInt();
                int bottomy = scanner.nextInt();
                Region region = new Region();
                region.topx = topx;
                region.bottomx = bottomx;
                region.topy = topy;
                region.bottomy = bottomy;
                regions.add(region);

                //for every new region,all icons which is visible should verify the visiblity again
                for (Icon icon : icons) {
                    if (icon.visible && iconIsVisable(icon, Collections.singletonList(region)) >= 0) {
                        icon.visible = false;
                    }
                }
            } else if (line.equals("M")) {
                int x = scanner.nextInt();
                int y = scanner.nextInt();
                Icon click = new Icon();
                click.x = x;
                click.y = y;

                int regionIndex = iconIsVisable(click, regions);
                if (regionIndex >= 0) {
                    System.out.println(alphas[regionIndex]);
                } else {
                    List<Integer> nearestIcon = findNearestIcons(click, icons);
                    StringBuilder sb = formatPrint(nearestIcon);
                    System.out.println(sb.toString());
                }
            }
            line = scanner.next();
        }

    }

    private static StringBuilder formatPrint(List<Integer> nearestIcon) {
        StringBuilder sb = new StringBuilder();
        for (Integer it : nearestIcon) {
            String is = it.toString();
            for (int i = 0; i < 3 - is.length(); i++) {
                sb.append(" ");
            }
            sb.append(it.toString());
        }
        return sb;
    }

    private static List<Integer> findNearestIcons(Icon click, List<Icon> icons) {
        List<Integer> result = new ArrayList<>();
        int length = Integer.MAX_VALUE;
        for (int i = 0; i < icons.size(); i++) {
            Icon cur = icons.get(i);
            if (cur.visible) {
                int lengthSquare = (click.x - cur.x) * (click.x - cur.x) + (click.y - cur.y) * (click.y - cur.y);
                if (lengthSquare < length) {
                    result = new ArrayList<>();
                    result.add(i + 1);
                    length = lengthSquare;
                } else if (lengthSquare == length) {
                    result.add(i + 1);
                }
            }
        }
        return result;
    }

    private static int iconIsVisable(Icon icon, List<Region> regions) {
        int size = regions.size();
        for (int i = size - 1; i >= 0; i--) {
            if (between(icon.x, regions.get(i).topx, regions.get(i).bottomx)
                    && between(icon.y, regions.get(i).topy, regions.get(i).bottomy)) {
                icon.visible = false;
                return i;
            }
        }
        icon.visible = true;
        return -1;
    }


    private static boolean between(int n, int start, int end) {
        return (n >= start && n <= end) || (n >= end && n <= start);
    }


    public static class Icon {
        private int x;
        private int y;
        private boolean visible;
    }

    public static class Region {
        private int topx;
        private int topy;
        private int bottomx;
        private int bottomy;

    }
}
```
