---
title: 123.Searching Quickly
date: 2024-07-11 08:48:34
mathjax: true
categories:
- [数据结构与算法,uva]
tags:
- 算法
- uva
description: "UVA 123"
---

## 题目

Searching and sorting are part of the theory and practice of computer science. For example, binary search provides a good example of an easy-to-understand algorithm with sub-linear complexity. Quicksort is an efficient O(n log n) [average case] comparison based sort.

KWIC-indexing is an indexing method that permits efficient “human search” of, for example, a list of titles.

Given a list of titles and a list of “words to ignore”, you are to write a program that generates a KWIC (Key Word In Context) index of the titles. In a KWIC-index, a title is listed once for each keyword that occurs in the title. The KWIC-index is alphabetized by keyword.

Any word that is not one of the “words to ignore” is a potential keyword.

For example, if words to ignore are “the, of, and, as, a” and the list of titles is:

> Descent of Man
The Ascent of Man
The Old Man and The Sea
A Portrait of The Artist As a Young Man

A KWIC-index of these titles might be given by:

>
                      a portrait of the ARTIST as a young man
                                    the ASCENT of man
                                        DESCENT of man
                             descent of MAN
                          the ascent of MAN
                                the old MAN and the sea
    a portrait of the artist as a young MAN
                                    the OLD man and the sea
                                      a PORTRAIT of the artist as a young man
                    the old man and the SEA
          a portrait of the artist as a YOUNG man

### Input

The input is a sequence of lines, the string ‘::’ is used to separate the list of words to ignore from the list of titles. Each of the words to ignore appears in lower-case letters on a line by itself and is no more than 10 characters in length. Each title appears on a line by itself and may consist of mixed-case (upper and lower) letters. Words in a title are separated by whitespace. No title contains more than 15 words.

There will be no more than 50 words to ignore, no more than than 200 titles, and no more than 10,000 characters in the titles and words to ignore combined. No characters other than ‘a’–‘z’, ‘A’–‘Z’, and white space will appear in the input.

### Output

The output should be a KWIC-index of the titles, with each title appearing once for each keyword in the title, and with the KWIC-index alphabetized by keyword. If a word appears more than once in a title, each instance is a potential keyword.

The keyword should appear in all upper-case letters. All other words in a title should be in lowercase letters. Titles in the KWIC-index with the same keyword should appear in the same order as they appeared in the input file. In the case where multiple instances of a word are keywords in the same title, the keywords should be capitalized in left-to-right order.

Case (upper or lower) is irrelevant when determining if a word is to be ignored.The titles in the KWIC-index need NOT be justified or aligned by keyword, all titles may be listed left-justified.

### SampleInput

is
the
of
and
as
a
but
::
Descent of Man
The Ascent of Man
The Old Man and The Sea
A Portrait of The Artist As a Young Man
A Man is a Man but Bubblesort IS A DOG

### SampleOutput

a portrait of the ARTIST as a young man
the ASCENT of man
a man is a man but BUBBLESORT is a dog
DESCENT of man
a man is a man but bubblesort is a DOG
descent of MAN
the ascent of MAN
the old MAN and the sea
a portrait of the artist as a young MAN
a MAN is a man but bubblesort is a dog
a man is a MAN but bubblesort is a dog
the OLD man and the sea
a PORTRAIT of the artist as a young man
the old man and the SEA
a portrait of the artist as a YOUNG man

### 题意

题目给了两组字符串，一组表示需要忽略的单词，另一组是一些title，每个title都是一个句子，由多个单词组成，单词之间用空格分开。title中出现的不在忽略的单词组里面的单词称为keyword。现需要按字典序将所有出现的keyword的title都按行输出，也就是每个keyword输出一行，该行输出内容是keyword所在的title，并且除了keyword大写，其余的都是小写。如果是相同的keyword，则按keyword所在的title在输入中出现的顺序排列。

## 思路

题意已明确给出输出所有keyword所在title，并且按规定的顺序排列，所以想到现找出所有keyword，然后排序，按序输出。整个步骤如下：

1. 先从title中提取keyword，提取需要过滤ignore word，所以先保存所有ignore word，然后提取keyword过滤；
2. 保存keyword还需要保存keyword所在的title和keyword在title中的位置，所以这里创建了一个对象Node来保存三者的关系；
3. 将Node排序,这里采用快排。

```JAVA {.line-numbers}
//Node
static class Node {
        String word;
        int index;
        int offset;

        public Node(String word, int index, int offset) {
            this.word = word;
            this.index = index;
            this.offset = offset;
        }
    }
```

### 问题

#### 实现上的问题

#### 性能上的问题

## 实现

```JAVA {.line-numbers}
//file
import java.util.*;

public class Main {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        Set<String> ignoreSet = new HashSet<>();
        List<String> titles = new ArrayList<>();
        boolean transform = false;
        while (scanner.hasNextLine()) {
            String line = scanner.nextLine();
            line = line.toLowerCase();
            if (line.equals("::")) {
                transform = true;
                continue;
            }
            if (transform) {
                titles.add(line);
            } else {
                ignoreSet.add(line);
            }
        }

        List<Node> wordNodes = new ArrayList<>();
        for (int i = 0; i < titles.size(); i++) {
            String title = titles.get(i);
            String[] words = title.split(" ");
            int offset = 0;
            for (int k = 0; k < words.length; k++) {
                if (words[k].isEmpty() || ignoreSet.contains(words[k])) {
                    offset += words[k].length() + 1;
                    continue;
                }
                Node wordNode = new Node(words[k], i, offset);
                offset += words[k].length() + 1;
                wordNodes.add(wordNode);
            }
        }
        sortNodes(wordNodes);
        for (int i = 0; i < wordNodes.size(); i++) {
            Node node = wordNodes.get(i);
            String title = titles.get(node.index);
            int offset = node.offset;
            StringBuilder sb = new StringBuilder();
            sb.append(title.substring(0, offset));
            sb.append(node.word.toUpperCase());
            sb.append(title.substring(offset + node.word.length()));
            System.out.println(sb.toString());
        }
    }

    private static void sortNodes(List<Node> wordNodes) {
        int start = 0, end = wordNodes.size() - 1;
        sortQuick(wordNodes, start, end);
    }

    private static void sortQuick(List<Node> wordNodes, int start, int end) {
        Node baseNode = wordNodes.get(start);
        int tempStart = start, tempEnd = end;
        while (tempStart < tempEnd) {
            while (tempStart < tempEnd && compareNode(baseNode, wordNodes.get(tempEnd)) < 0) {
                tempEnd--;
            }
            if (tempStart < tempEnd) {
                wordNodes.set(tempStart, wordNodes.get(tempEnd));
                tempStart++;
            }
            while (tempStart < tempEnd && compareNode(baseNode, wordNodes.get(tempStart)) > 0) {
                tempStart++;
            }
            if (tempStart < tempEnd) {
                wordNodes.set(tempEnd, wordNodes.get(tempStart));
                tempEnd--;
            }
        }
        wordNodes.set(tempStart, baseNode);
        if (tempStart - 1 > start) {
            sortQuick(wordNodes, start, tempStart - 1);
        }
        if (tempStart + 1 < end) {
            sortQuick(wordNodes, tempStart + 1, end);
        }

    }

    private static int compareNode(Node baseNode, Node node) {
        String word = baseNode.word;
        String compareWord = node.word;
        int index = 0;
        while (index < word.length() && index < compareWord.length()) {
            if (word.charAt(index) < compareWord.charAt(index)) {
                return -1;
            } else if (word.charAt(index) > compareWord.charAt(index)) {
                return 1;
            }
            index++;
        }
        if (index == word.length() && index == compareWord.length()) {
            if (baseNode.index < node.index) {
                return -1;
            } else if (baseNode.index > node.index) {
                return 1;
            } else {
                if (baseNode.offset < node.offset) {
                    return -1;
                } else {
                    return 1;
                }
            }
        } else if (index == word.length()) {
            return -1;
        } else {
            return 1;
        }
    }

    static class Node {
        String word;
        int index;
        int offset;

        public Node(String word, int index, int offset) {
            this.word = word;
            this.index = index;
            this.offset = offset;
        }
    }
}
```
