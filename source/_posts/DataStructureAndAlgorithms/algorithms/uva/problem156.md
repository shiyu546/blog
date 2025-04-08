---
title: problem156 Ananagrams
date: 2025-03-24 22:28:33
mathjax: true
categories:
- [数据结构与算法,uva]
tags:
- 算法
- uva
description: "UVA problem156.Ananagrams <br>
- 判断字典内不是ananagrams的单词"
---

## 题目

Most crossword puzzle fans are used to anagrams — groups of words with the same letters in different orders — for example OPTS, SPOT, STOP, POTS and POST. Some words however do not have this attribute, no matter how you rearrange their letters, you cannot form another word. Such words are
called ananagrams, an example is QUIZ.

Obviously such definitions depend on the domain within which we are working; you might think that ATHENE is an ananagram, whereas any chemist would quickly produce ETHANE. One possible domain would be the entire English language, but this could lead to some problems. One could restrict the domain to, say, Music, in which case SCALE becomes a relative ananagram (LACES is not in the same domain) but NOTE is not since it can produce TONE.

Write a program that will read in the dictionary of a restricted domain and determine the relative ananagrams. Note that single letter words are, ipso facto, relative ananagrams since they cannot be “rearranged” at all. The dictionary will contain no more than 1000 words.

### Input

Input will consist of a series of lines. No line will be more than 80 characters long, but may contain any number of words. Words consist of up to 20 upper and/or lower case letters, and will not be broken across lines. Spaces may appear freely around words, and at least one space separates multiple words on the same line. Note that words that contain the same letters but of differing case are considered to be anagrams of each other, thus ‘tIeD’ and ‘EdiT’ are anagrams. The file will be terminated by a line consisting of a single ‘#’.

### Output

Output will consist of a series of lines. Each line will consist of a single word that is a relative ananagram in the input dictionary. Words must be output in lexicographic (case-sensitive) order. There will always be at least one relative ananagram.

### SampleInput

ladder came tape soon leader acme RIDE lone Dreis peat
ScAlE orb eye Rides dealer NotE derail LaCeS drIed
noel dire Disk mace Rob dries
\#

### SampleOutput

Disk
NotE
derail
drIed
eye
ladder
soon

### 题意

一个单词的字母如果任意排列后组成的单词都无法形成在字典中的词，则称这个词为ananagrams.现在给出一个词典，找出所有的ananagrams,并按字典序排列输出，单词任意排列不区分大小写。

## 思路

两个单词$S_1,S_2$如果相同，则单词字母按字典序排列后得到的单词S是一样的。基于这个我们转换每个单词得到重排列的单词，并判断是否重复。按字典序输出不重复的单词。

### 问题

#### 实现上的问题

#### 性能上的问题

## 实现

```JAVA .{line-numbers}
//source code
import java.util.*;

public class Main {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        List<String> words = new ArrayList<>();
        while (true) {
            String line = scanner.nextLine();
            if (line.equals("#")) {
                break;
            }
            String[] lineWords = line.split(" ");
            for (int i = 0; i < lineWords.length; i++) {
                if (!lineWords[i].equals(" ")) {
                    words.add(lineWords[i]);
                }
            }
        }
        //tag用来记录每一个单词是否有重排列相同的词，为1表示对应下标的单词有
        int[] tag = new int[words.size()];
        for (int i = 0; i < tag.length; i++) {
            tag[i] = 0;
        }
        Map<String, Integer> wordMap = new HashMap<>();
        for (int i = 0; i < words.size(); i++) {
            String ftWord = covertWord(words.get(i));
            if (wordMap.containsKey(ftWord)) {
                tag[i] = 1;
                int index = wordMap.get(ftWord);
                tag[index] = 1;
            } else {
                wordMap.put(ftWord, i);
            }
        }

        //过滤掉有相同的重排列的单词，就得到了ananagrams
        List<String> anaWords = new ArrayList<>();
        for (int i = 0; i < words.size(); i++) {
            if (tag[i] == 0) {
                anaWords.add(words.get(i));
            }
        }

        //按字典序排序
        anaWords.sort((String a, String b) -> {
            int lena = a.length();
            int lenb = b.length();
            int len = Math.min(lena, lenb);
            for (int i = 0; i < len; i++) {
                if (a.charAt(i) < b.charAt(i)) {
                    return -1;
                } else if (a.charAt(i) > b.charAt(i)) {
                    return 1;
                }
            }
            if (lena < lenb) {
                return -1;
            } else if (lena > lenb) {
                return 1;
            } else {
                return 0;
            }
        });
        for (int i = 0; i < anaWords.size(); i++) {
            System.out.println(anaWords.get(i));
        }
    }

    private static String covertWord(String s) {
        s = s.toLowerCase();
        char[] albs = s.toCharArray();
        Arrays.sort(albs);
        return new String(albs);
    }
}
```
