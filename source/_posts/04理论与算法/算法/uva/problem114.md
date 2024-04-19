# Simulation Wizardry

## 1. 题目

Simulation is an important application area in computer science involving the development of computer models to provide insight into real-world events. There are many kinds of simulation including (and certainly not limited to) discrete event simulation and clock-driven simulation. Simulation often involves approximating observed behavior in order to develop a practical approach.

This problem involves the simulation of a simplistic pinball machine. In a pinball machine, a steel ball rolls around a surface, hitting various objects (bumpers) and accruing points until the ball “disappears” from the surface.

You are to write a program that simulates an idealized pinball machine. This machine has a flat surface that has some obstacles (bumpers and walls). The surface is modeled as an m $\times$ n grid with the origin in the lower-left corner. Each bumper occupies a grid point. The grid positions on the edge of the surface are walls. Balls are shot (appear) one at a time on the grid, with an initial position, direction, and lifetime.

In this simulation, all positions are integral, and the ball’s direction is one of: up, down, left, or right. The ball bounces around the grid, hitting bumpers (which accumulates points) and walls (which does not add any points). The number of points accumulated by hitting a given bumper is the value of that bumper. The speed of all balls is one grid space per timestep. A ball “hits” an obstacle during a timestep when it would otherwise move on top of the bumper or wall grid point. A hit causes the ball to “rebound” by turning right (clockwise) 90 degrees, without ever moving on top of the obstacle and without changing position (only the direction changes as a result of a rebound). Note that by this definition sliding along a wall does not constitute “hitting” that wall.

A ball’s lifetime indicates how many time units the ball will live before disappearing from the surface. The ball uses one unit of lifetime for each grid step it moves. It also uses some units of lifetime for each bumper or wall that it hits. The lifetime used by a hit is the cost of that bumper or wall. As long as the ball has a positive lifetime when it hits a bumper, it obtains the full score for that bumper. Note that a ball with lifetime one will “die” during its next move and thus cannot obtain points for hitting a bumper during this last move. Once the lifetime is non-positive (less than or equal to zero), the ball disappears and the game continues with the next ball.

### Input

Your program should simulate one game of pinball. There are several input lines that describe the game. The first line gives integers m and n, separated by a space. This describes a cartesian grid where 1 $\leq$ x $\leq$ m and 1 $\leq$ y $\leq$ n on which the game is “played”. It will be the case that 2 < m < 51 and 2 < n < 51. The next line gives the integer cost for hitting a wall. The next line gives the number of bumpers, an integer p $\geq$ 0.

The next p lines give the x position, y position, value, and cost, of each bumper, as four integers per line separated by space(s). The x and y positions of all bumpers will be in the range of the grid. The value and cost may be any integer (i.e., they may be negative; a negative cost adds lifetime to a ball that hits the bumper).

The remaining lines of the file represent the balls. Each line represents one ball, and contains four integers separated by space(s): the initial x and y position of the ball, the direction of movement, and its lifetime. The position will be in range (and not on top of any bumper or wall). The direction will be one of four values: 0 for increasing x (right), 1 for increasing y (up), 2 for decreasing x (left), and 3 for decreasing y (down). The lifetime will be some positive integer.

### Output

There should be one line of output for each ball giving an integer number of points accumulated by that ball in the same order as the balls appear in the input. After all of these lines, the total points for all balls should be printed.

### Sample Input

4 4
0
2
2 2 1 0
3 3 1 0
2 3 1 1
2 3 1 2
2 3 1 3
2 3 1 4
2 3 1 5

### Sample Output

0
0
1
2
2
5

### 1.1 题意

题目比较长，主要是讲述了这样一个场景：在一个棋盘上，有一个小球，小球会朝指定方向移动，每次移动一个并消耗一点生命值，如果碰到棋盘边缘或者棋盘中的障碍物，则小球不移动并右转90°。碰撞墙壁或者障碍物会损失生命值(损失的可能为负数，即加生命值)，当小球生命值小于等于0时，球消失。小球碰到障碍物时，会获取障碍物的分数。题目给定了棋盘大小，障碍物的个数和位置，以及多个小球，需要输出每个小球获得的分数。

该题是模拟题，按照给定的题意编程即可. 需要注意题意中给定的细节:

* 小球每动一下生命值都减1，只有减完1后生命值仍大于0，才能获取障碍物的分数。
* 障碍物或墙可能有负的生命值，即小球碰撞后会加生命值，但前提条件是小球碰撞后(生命值减1)后仍然有正生命值。如果小球生命值为1，墙壁消耗-4，那么小球在碰撞墙壁后会消失。

## 2. 思路

按照规则即可，注意细节。

### 2.1 问题

#### 2.1.1 实现上的问题

#### 2.1.2 性能上的问题

本题的问题不在于算法和思路，而在于输入的处理。通过使用java中的scanner类，发现处理超时。通过搜索相关资料，了解到可以采用带缓冲区的输入来处理。通过使用BufferedReader缓存输入，达到加快处理输入的目的。BufferedReader使用框架如下：

```JAVA
// JAVA
static BufferedReader reader;
static StringTokenizer tokenizer;

/**
 * call this method to initialize reader for InputStream
 */
static void init(InputStream input) {
    reader = new BufferedReader(
            new InputStreamReader(input));
    tokenizer = new StringTokenizer("");
}
```

## 3. 实现

```JAVA
import java.util.*;
import java.io.*;

public class Main {
    public static void main(String[] args) throws IOException {
//        File file = new File("number.txt");
//
//        Reader.initFile(file);
        Reader.init(System.in);
        Grid grid = new Grid();

        grid.setM(Reader.nextInt());
        grid.setN(Reader.nextInt());
        grid.setWallCost(Reader.nextInt());

        int bumperNumbers = Reader.nextInt();

        Map<Integer, Bumper> bumperMap = new HashMap<>();
        for (int i = 0; i < bumperNumbers; i++) {
            Bumper bumper = new Bumper();
            bumper.setPointx(Reader.nextInt());
            bumper.setPointy(Reader.nextInt());
            bumper.setValue(Reader.nextInt());
            bumper.setCost(Reader.nextInt());
            int key = bumper.getPointx() * 100 + bumper.getPointy();
            bumperMap.put(key, bumper);
        }

        int totalScore = 0;
        while (Reader.hasNext()) {
            Ball ball = new Ball();
            ball.setPointx(Reader.nextInt());
            ball.setPointy(Reader.nextInt());
            ball.setDirection(Reader.nextInt());
            ball.setLifetime(Reader.nextInt());

            //input is over,handler ball move
            int ballScore = 0;
            //1.生命值大于1才能在下一次移动或碰撞中不消失
            while (ball.getLifetime() > 1) {
                //2.判断下一步的位置并按照不同的方式处理(墙，bumper，空位置)
                NextInfo nextInfo = nextPostion(ball, grid, bumperMap);
                if (nextInfo.getType() == 1) {
                    //upgrade lifetime,continue with ball with one move
                    ball.setLifetime(ball.getLifetime() - 1);
                } else if (nextInfo.getType() == 2) {
                    int lifetime = ball.getLifetime();
                    lifetime = lifetime - 1;
                    lifetime = lifetime - grid.getWallCost();
                    ball.setLifetime(lifetime);
                } else {
                    Bumper bumper = nextInfo.getBumper();
                    int lifetime = ball.getLifetime();
                    lifetime = lifetime - 1;
                    if (lifetime > 0) {
                        ballScore += bumper.getValue();
                    }
                    lifetime = lifetime - bumper.getCost();
                    ball.setLifetime(lifetime);
                }
            }
            totalScore += ballScore;
            System.out.println(ballScore);
        }
        System.out.println(totalScore);
    }

    /**
     * the next position is bumpers or wall or nothing
     *
     * @param ball
     * @return type of next position:
     * 1: empty position
     * 2: wall
     * 3: bumpers
     */
    private static NextInfo nextPostion(Ball ball, Grid grid, Map<Integer, Bumper> bumperMap) {
        NextInfo nextInfo = new NextInfo();
        int direction = ball.getDirection();
        int pointx = ball.getPointx();
        int pointy = ball.getPointy();
        switch (direction) {
            case 0:
                pointx++;
                break;
            case 1:
                pointy++;
                break;
            case 2:
                pointx--;
                break;
            case 3:
                pointy--;
                break;
        }
        if (pointx == 1 || pointx == grid.getM()
                || pointy == 1 || pointy == grid.getN()) {
            nextInfo.setType(2);
            ball.setDirection((direction + 3) % 4);
            nextInfo.setBall(ball);
            return nextInfo;
        }

        int key = pointx * 100 + pointy;
        if (bumperMap.containsKey(key)) {
            Bumper bumper = bumperMap.get(key);
            ball.setDirection((direction + 3) % 4);
            nextInfo.setType(3);
            nextInfo.setBumper(bumper);
            return nextInfo;
        }

        ball.setPointx(pointx);
        ball.setPointy(pointy);
        nextInfo.setType(1);
        return nextInfo;
    }

    private static class Grid {
        int m;
        int n;

        int wallCost;

        public int getM() {
            return m;
        }

        public void setM(int m) {
            this.m = m;
        }

        public int getN() {
            return n;
        }

        public void setN(int n) {
            this.n = n;
        }

        public int getWallCost() {
            return wallCost;
        }

        public void setWallCost(int wallCost) {
            this.wallCost = wallCost;
        }
    }

    private static class Bumper {
        int pointx;
        int pointy;
        int value;
        int cost;

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

        public int getValue() {
            return value;
        }

        public void setValue(int value) {
            this.value = value;
        }

        public int getCost() {
            return cost;
        }

        public void setCost(int cost) {
            this.cost = cost;
        }
    }

    private static class Ball {
        int pointx;
        int pointy;
        int direction;
        int lifetime;

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

        public int getDirection() {
            return direction;
        }

        public void setDirection(int direction) {
            this.direction = direction;
        }

        public int getLifetime() {
            return lifetime;
        }

        public void setLifetime(int lifetime) {
            this.lifetime = lifetime;
        }
    }

    private static class NextInfo {
        private Ball ball;
        private Bumper bumper;

        int type;

        public Ball getBall() {
            return ball;
        }

        public void setBall(Ball ball) {
            this.ball = ball;
        }

        public Bumper getBumper() {
            return bumper;
        }

        public void setBumper(Bumper bumper) {
            this.bumper = bumper;
        }

        public int getType() {
            return type;
        }

        public void setType(int type) {
            this.type = type;
        }
    }

    private static class Reader {
        static BufferedReader reader;
        static StringTokenizer tokenizer;

        /**
         * call this method to initialize reader for InputStream
         */
        static void init(InputStream input) {
            reader = new BufferedReader(
                    new InputStreamReader(input));
            tokenizer = new StringTokenizer("");
        }

        static void initFile(File input) {
            try {
                reader = new BufferedReader(
                        new FileReader(input));
            } catch (FileNotFoundException e) {
                throw new RuntimeException(e);
            }
            tokenizer = new StringTokenizer("");
        }

        /**
         * get next word
         */
        static String next() throws IOException {
            while (!tokenizer.hasMoreTokens()) {
                //TODO add check for eof if necessary
                tokenizer = new StringTokenizer(
                        reader.readLine());
            }
            return tokenizer.nextToken();
        }

        static Boolean hasNext() throws IOException {
            if (!tokenizer.hasMoreTokens()) {
                String str = reader.readLine();
                if (str == null) {
                    return false;
                }
                tokenizer = new StringTokenizer(str);
            }
            return true;
        }

        static Integer nextInt() throws IOException {
            String str = next();
            if (str == null) {
                return null;
            }
            return Integer.parseInt(str);
        }
    }
}

```
