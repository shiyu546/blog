---
title: problem148 Anagram checker
date: 2024-12-18 10:19:52
mathjax: true
categories:
- [数据结构与算法,uva]
tags:
- 算法
- uva
description: "UVA problem148.Anagram checker <br>
- 给定字典和字符串，判断字符串的字母，可以由字典中哪些单词组成，输出所有的组合方式"
---

## 题目

It is often fun to see if rearranging the letters of a name gives an amusing anagram. For example, the letters of ‘WILLIAM SHAKESPEARE’ rearrange to form ‘SPEAK REALISM AWHILE’.

Write a program that will read in a dictionary and a list of phrases and determine which words from the dictionary, if any, form anagrams of the given phrases. Your program must find all sets of words in the dictionary which can be formed from the letters in each phrase. Do not include the set consisting of the original words. If no anagram is present, do not write anything, not even a blank line.

### Input

Input will consist of two parts. The first part is the dictionary, the second part is the set of phrases for which you need to find anagrams. Each part of the file will be terminated by a line consisting of a single ‘#’. The dictionary will be in alphabetic order and will contain up to 2000 words, one word per line. The entire file will be in upper case, and no dictionary word or phrase will contain more than 20 letters. You cannot assume the language being used is English.

### Output

Output will consist of a series of lines. Each line will consist of the original phrase, a space, an equal sign (=), another space, and the list of words that together make up an anagram of the original phrase, separated by exactly one space. These words must appear in alphabetic sequence.

### SampleInput

ABC
AND
DEF
DXZ
K
KX
LJSRT
LT
PT
PTYYWQ
Y
YWJSRQ
ZD
ZZXY
\#
ZZXY ABC DEF
SXZYTWQP KLJ YRTD
ZZXY YWJSRQ PTYYWQ ZZXY
\#

### SampleOutput

SXZYTWQP KLJ YRTD = DXZ K LJSRT PTYYWQ
SXZYTWQP KLJ YRTD = DXZ K LT PT Y YWJSRQ
SXZYTWQP KLJ YRTD = KX LJSRT PTYYWQ ZD
SXZYTWQP KLJ YRTD = KX LT PT Y YWJSRQ ZD

### 题意

如果两个字符串所构成的字母是一样的，则称为'anagram',现在给定一个字典，字典由多个字符串组成，再给定一个字符串S，如果S与多个字典中的字符串集R组成的字母一样，就按指定格式输出，注意S和R中的单词不能有一样的。

## 思路

暴力破解，计算所有的可能性。可以采用栈的方式来处理，从字典从头往尾遍历，如果字符串S包含当前单词r，将r入栈，并从S中减去r(S中减去r的字母)，再从r后一个单词接着遍历，重复这样的操作直至遍历到字典末尾。

1. 当r入栈时，判断S是否为空串，如果S是空串，此时栈中的元素(单词)就是组合成S的单词集，输出；
2. 到达字典末尾，表明当前栈不足以构成S，则将栈顶元素p弹出，并补偿栈顶元素p的字母到S，接着从p的下一个开始遍历。

字符串的比较和减去操作转换为对应的字母数的比较和减去操作，即每一个字符串都转换为对应字母数量的数组。

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
        List<String> dicts = new ArrayList<>();
        Map<String, int[]> dictMap = new HashMap<>();
        while (true) {
            String dict = scanner.nextLine();
            if ("#".equals(dict)) {
                break;
            }
            if ("".equals(dict)) {
                continue;
            }
            int[] arys = convertWordToAry(dict);
            dictMap.put(dict, arys);
            dicts.add(dict);
        }

        while (true) {
            String words = scanner.nextLine();
            if ("#".equals(words)) {
                break;
            }
            int[] wordAry = convertWordToAry(words);
            //当作栈来用，方便word为空时计算单词
            LinkedList<Integer> tempIndex = new LinkedList<>();
            int[] tempAry = new int[26];
            for (int i = 0; i < dicts.size(); i++) {
                int[] dictAry = dictMap.get(dicts.get(i));
                if (contains(wordAry, dictAry)) {
                    tempIndex.add(i);
                    tempAry = subArys(wordAry, dictAry);
                    break;
                }
            }

            if (tempIndex.size() <= 0) {
                continue;
            }

            int index = tempIndex.getLast();
            while (true) {
                int i;
                for (i = index + 1; i < dicts.size(); i++) {
                    int[] dictAry = dictMap.get(dicts.get(i));
                    if (contains(tempAry, dictAry)) {
                        tempIndex.add(i);
                        tempAry = subArys(tempAry, dictAry);
                        if (aryisZero(tempAry)) {
                            //S为空，则判断是否有重复单词并输出
                            String record = record(words, tempIndex, dicts);
                            if (!record.equals("")) {
                                System.out.println(record);
                            }
                        }
                        index = tempIndex.getLast();
                        break;
                    }
                }

                //遍历到达了字典末尾，栈中元素无法构成S，则弹出栈顶，补偿S并从下一个开始遍历
                if (i >= dicts.size()) {
                    if (tempIndex.size() == 0) {
                        break;
                    }
                    int tempIndexNum = tempIndex.removeLast();
                    tempAry = addArys(tempAry, dictMap.get(dicts.get(tempIndexNum)));
                    index = tempIndexNum;
                }
            }
        }
    }

    public static int[] convertWordToAry(String word) {
        int[] result = new int[26];
        for (int i = 0; i < 26; i++) {
            result[i] = 0;
        }
        for (int i = 0; i < word.length(); i++) {
            if (word.charAt(i) == ' ') {
                continue;
            }
            int index = word.charAt(i) - 'A';
            result[index]++;
        }
        return result;
    }

    public static boolean contains(int[] word, int[] dict) {
        for (int i = 0; i < 26; i++) {
            if (word[i] < dict[i]) {
                return false;
            }
        }
        return true;
    }

    public static int[] subArys(int[] word, int[] dict) {
        int[] result = new int[26];
        for (int i = 0; i < 26; i++) {
            result[i] = word[i] - dict[i];
        }
        return result;
    }

    public static int[] addArys(int[] word, int[] dict) {
        int[] result = new int[26];
        for (int i = 0; i < 26; i++) {
            result[i] = word[i] + dict[i];
        }
        return result;
    }

    public static boolean aryisZero(int[] ary) {
        for (int i = 0; i < ary.length; i++) {
            if (ary[i] != 0) {
                return false;
            }
        }
        return true;
    }

    public static String record(String word, LinkedList<Integer> index, List<String> dicts) {
        String[] words = word.split(" ");
        Set<String> wordSet = new HashSet<>();
        wordSet.addAll(Arrays.asList(words));
        boolean isSame = false;
        for (int i = 0; i < index.size(); i++) {
            if (wordSet.contains(dicts.get(index.get(i)))) {
                isSame = true;
                break;
            }
        }

        if (!isSame) {
            StringBuilder sb = new StringBuilder();
            sb.append(word);
            sb.append(" =");
            for (int i = 0; i < index.size(); i++) {
                sb.append(" ");
                sb.append(dicts.get(index.get(i)));
            }
            return sb.toString();
        }
        return "";
    }

}
```
