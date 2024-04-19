# Tree Summing

## 1. 题目

LISP was one of the earliest high-level programming languages and, with FORTRAN, is one of the oldest languages currently being used. Lists, which are the fundamental data structures in LISP, can easily be adapted to represent other important data structures such as trees.

This problem deals with determining whether binary trees represented as LISP S-expressions possess a certain property.
Given a binary tree of integers, you are to write a program that determines whether there exists a root-to-leaf path whose nodes sum to a specified integer.

For example, in the tree shown on the right there are exactly four root-to-leaf paths. The sums of the paths are 27, 22, 26, and 18. Binary trees are represented in the input file as LISP S-expressions having the following form.

$
empty\ tree ::= ()
$

$
tree ::= empty \ tree\ |\ (integer\ tree\ tree)
$

The tree diagrammed above is represented by the expression
(5 (4 (11 (7 () ()) (2 () ()) ) ()) (8 (13 () ()) (4 () (1 () ()) ) ) )

Note that with this formulation all leaves of a tree are of the form

$(integer\ ()\ () )$

Since an empty tree has no root-to-leaf paths, any query as to whether a path exists whose sum is a specified integer in an empty tree must be answered negatively.

### Input

The input consists of a sequence of test cases in the form of integer/tree pairs. Each test case consists of an integer followed by one or more spaces followed by a binary tree formatted as an S-expression as described above. All binary tree S-expressions will be valid, but expressions may be spread over
several lines and may contain spaces. There will be one or more test cases in an input file, and input is terminated by end-of-file.

### Output

There should be one line of output for each test case (integer/tree pair) in the input file. For eachpair I; T (I represents the integer, T represents the tree) the output is the string ‘yes’ if there is aroot-to-leaf path in T whose sum is I and ‘no’ if there is no path in T whose sum is I.

### Sample Input

22 (5(4(11(7()())(2()()))()) (8(13()())(4()(1()()))))
20 (5(4(11(7()())(2()()))()) (8(13()())(4()(1()()))))
10 (3
(2 (4 () () )
(8 () () ) )
(1 (6 () () )
(4 () () ) ) )
5 ()

### Sample Output

yes
no
yes
no

### 1.1 题意

题意是给定用lisp语言的列表语法，用来表示二叉树这种数据结构，要求判断树是否有到叶子节点的路径节点和等于给定值。

## 2. 思路

一种方法是枚举所有的到叶子节点的路径，所以需要有某种方法遍历树的路径。在树的几种遍历方法中，当我们采用栈临时存储中间节点时，只需要遍历到叶子节点，判断当前栈内元素和，即可得到路径节点和。通过遍历树的节点，我们得到了所有的路径和，再和给定值比较，即可得到结果。

### 2.1 问题

* 第一个问题是输入的处理，给定一个比较值后，后面的输入就是树的列表表示，但输入不是分布在一行，而是有多行，所以需要有某种方法来判断列表是否输入完毕。
    >这里我们采用括号匹配法来解决这个问题，lisp列表的语法决定了其括号个数一定匹配。由于列表分布在多行，所以我们依次读取一行并判断'(' 和')'数量匹配是否相等，不匹配说明列表尚未读取完毕，接着读取；如果数量匹配相等则说明列表读物完毕.
    >
    >要注意一种特殊情况，即列表开头不在比较值那一行，这时由于括号数都是0，所以也会匹配，通过判断列表长度即可排除这种情况。

* 第二个问题是叶子节点的判定。
    >叶子节点有一个很特殊的性质就是接下来连续两个"()",所以在元素入栈时通过判断是否已经有连续两个"()"入栈，即可判断是否是叶子节点，进而获得当前路径上元素值的和。

## 3. 实现

代码实现由两部分组成:

1. 从输入中判断出一个case：即待比较值和列表；
2. 第二部分是通过对列表的遍历得到所有路径。

```JAVA
import java.io.File;
import java.io.FileNotFoundException;
import java.util.Scanner;
import java.util.Stack;

public class Main {
    public static void main(String[] args) {
        File file = new File("number.txt");
        Scanner scanner = null;
        try {
            scanner = new Scanner(file);
        } catch (FileNotFoundException e) {
            throw new RuntimeException(e);
        }
//        Scanner scanner=new Scanner(System.in);
        label:
        while (scanner.hasNextInt()) {
            int compareNumber = scanner.nextInt();
            int leftParenthese = 0, rightParenthese = 0;
            StringBuilder sb = new StringBuilder();
            //1.通过左右括号数量和列表长度判断列表是否读取完毕
            while (scanner.hasNextLine()) {
                String s = scanner.nextLine();
                sb.append(s);
                leftParenthese += numberOfChar(s, '(');
                rightParenthese += numberOfChar(s, ')');
                if (sb.length() > 0 && leftParenthese == rightParenthese) {
                    break;
                }
            }

            //2.将列表中所有空白字符去掉，对于下面入栈操作来说这里是关键
            String tree = sb.toString().replaceAll(" ", "").trim();
            if (tree.equals("()")) {
                System.out.printf("no\n");
                continue;
            }

            Stack<String> parenthese = new Stack<>();
            //totalSum用来标记当前栈内元素，也就是路径节点和
            int totalSum = 0;
            //count用来记录是否是连续的两个()，从而判断是否是叶子节点
            int count = 0;
            for (int i = 0; i < tree.length(); i++) {
                if (tree.charAt(i) == '(') {
                    parenthese.push("(");
                } else if (tree.charAt(i) == ')') {
                    //3.这里是出栈过程，根据栈顶元素不同，出栈规则也不一样
                    if (parenthese.peek().equals("(")) {
                        count++;
                    } else {
                        count = 0;
                        //3.1栈顶是数字则需要出栈直到栈顶是'('
                        while (!parenthese.peek().equals("(")) {
                            totalSum -= Integer.valueOf(parenthese.pop());
                        }
                    }
                    parenthese.pop();

                    if (count == 2) {
                        if (totalSum == compareNumber) {
                            System.out.printf("yes\n");
                            continue label;
                        }
                    }

                } else {
                    int index = nextIndexOf(tree, i);
                    String numberStr = tree.substring(i, index);
                    parenthese.push(numberStr);
                    totalSum += Integer.valueOf(numberStr);
                    count = 0;
                    i = index - 1;
                }
            }

            System.out.printf("no\n");

        }
    }

    private static int nextIndexOf(String s, int fromIndex) {
        int nextLeft = s.indexOf('(', fromIndex);
        int nextRight = s.indexOf(')', fromIndex);
        return nextLeft > nextRight ? nextRight : nextLeft;
    }

    private static int numberOfChar(String s, char sig) {
        int result = 0;
        if (s == null || s.length() == 0) {
            return result;
        }
        for (int i = 0; i < s.length(); i++) {
            if (s.charAt(i) == sig) {
                result++;
            }
        }
        return result;
    }
}
```
