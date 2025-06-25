---
title: problem175 Keywords
date: 2025-06-24 10:54:20
mathjax: true
categories:
- [数据结构与算法,uva]
tags:
- 算法
- uva
description: "UVA problem175.Keywords <br>
- 给出关键词对，判断标题是否满足关键词对"
---

## 题目

Many researchers are faced with an ever increasing number of journal articles to read and find it difficult to locate papers of relevance to their particular lines of research. However, it is possible to subscribe to various services which claim that they will find articles that fit an ‘interest profile’ that you supply, and pass them on to you. One simple way of performing such a search is to determine whether a pair of keywords occurs ‘sufficiently’ close to each other in the title of an article. The threshold is determined by the researchers themselves, and refers to the number of words that may occur between the pair of keywords. Thus an archeologist interested in cave paintings could specify her profile as “0 rock art”,meaning that she wants all titles in which the words “rock” and “art” appear with 0 words in between,that is next to each other. This would select not only “Rock Art of the Maori” but also “Pop Art,Rock, and the Art of Hang-glider Maintenance”.

Write a program that will read in a series of profiles followed by a series of titles and determine which of the titles (if any) are selected by each of the profiles. A title is selected by a profile if at least one pair of keywords from the profile is found in the title, separated by no more than the given threshold. For the purposes of this program, a word is a sequence of letters, preceded by one or more blanks and terminated by a blank or the end of line marker.

### Input

Input will consist of no more than 50 profiles followed by no more than 250 titles. Each profile and title will be numbered in the order of their appearance, starting from 1, although the numbers will not appear in the file.

* Each profile will start with the characters ‘P: ’, and will consist of a number representing a threshold, followed by two or more keywords in lower case.
* Each title will start with the characters ‘T: ’, and will consist of a string of characters terminated by ‘|’. The character ‘|’ will not occur anywhere in a title except at the end. No title will be longer than 255 characters, and if necessary it will flow on to more than one line. No line will be longer than eighty characters and each continuation line of a title will start with at least one blank. Line breaks will only occur between words.

All non-alphabetic characters are to be ignored, thus the title “Don't Rock --- the Boat as Metaphor in 1984” would be treated as “Dont Rock the Boat as Metaphor in” and “HP2100X” will be treated as “HPX”.

The file will be terminated by a line consisting of a single ‘#’.

### Output

Output will consist of a series of lines, one for each profile in the input. Each line will consist of the profile number (the number of its appearance in the input) followed by ‘:’, a blank space, and the numbers of the selected titles in numerical order, separated by commas and with no spaces.

### SampleInput

P: 0 rock art
P: 3 concepts conceptions
P: 1 art rock metaphor concepts
T: Rock Art of the Maori|
T: Jazz and Rock - Art Brubeck and Elvis Presley|
T: Don't Rock --- the Boat as Metaphor in 1984, Concepts
and (Mis)-Conceptions of an Art Historian.|
T: Carved in Rock, The Art and Craft of making promises
believable when your `phone bills have gone
through the roof|
\#

### SampleOutput

1: 1,2
2: 
3: 1,2,3,4

### 题意

题目给出了多个profile，每一个profile由一个threshold和多个关键词组成，如果一个标题中，profile中任意的关键词对出现在了标题中，且中间相隔的单词个数小于等于threshold的值，则称该标题满足该profile。

现在给出了多个profile和多个标题，计算每一个profile满足哪些标题。

## 思路

这里的关键是profile中任意一对关键词对在标题中满足要求，需要做两步操作：

1. 从标题中找出profile中的关键词，并记录每一个关键词出现的位置。
2. 遍历关键词对(双重for循环)，由于每一个关键词出现的位置信息已记录且为数组，所以关键词对的两个数组每一个元素都比较，判断距离是否小于等于threshold。

### 问题

#### 实现上的问题

输入中有很多字符是干扰字符，需要去除。

#### 性能上的问题

## 实现

```JAVA {.line-numbers}
import java.io.*;
import java.util.*;

public class Main {
    public static void main(String[] args) throws IOException {
        BufferedReader reader = new BufferedReader(new InputStreamReader(System.in));
        List<Profile> profiles = new ArrayList<>();
        List<List<String>> titles = new ArrayList<>();
        while (true) {
            String line = reader.readLine();
            if (line.equals("#")) {
                break;
            }
            if (line.startsWith("P: ")) {
                Profile profile = new Profile();
                String[] split = line.substring(3).trim().split("\\s+");
                int number = Integer.parseInt(split[0]);
                List<String> words = new ArrayList<>();
                for (int i = 1; i < split.length; i++) {
                    words.add(split[i]);
                }
                profile.number = number;
                profile.words = words;
                profiles.add(profile);
            } else if (line.startsWith("T: ")) {
                StringBuilder sb = new StringBuilder();
                sb.append(line.substring(3));
                while (!sb.toString().contains("|")) {
                    sb.append(" ");
                    sb.append(reader.readLine().trim());
                }
                // Remove everything after and including '|'
                String full = sb.toString();
                int barIndex = full.indexOf('|');
                full = full.substring(0, barIndex);
                // Replace non-letters with spaces and split
                full = full.replaceAll("[^a-zA-Z ]", "").toLowerCase();
                String[] words = full.trim().split("\\s+");
                titles.add(Arrays.asList(words));
            }
        }

        for (int i = 0; i < profiles.size(); i++) {
            Profile p = profiles.get(i);
            List<Integer> matchedTitles = new ArrayList<>();
            for (int j = 0; j < titles.size(); j++) {
                if (matches(titles.get(j), p)) {
                    matchedTitles.add(j + 1); // title numbers start from 1
                }
            }

            System.out.print((i + 1) + ":");
            if (!matchedTitles.isEmpty()) {
                for (int k = 0; k < matchedTitles.size(); k++) {
                    System.out.print((k == 0 ? " " : ",") + matchedTitles.get(k));
                }
            } else {
                System.out.print(" ");
            }
            System.out.println();
        }
    }

    public static class Profile {
        public int number;
        public List<String> words;
    }

    /**
     * titleWords是将title按空格切分后的单词数组
     * 这里用来判断title是否满足profile
     */
    private static boolean matches(List<String> titleWords, Profile p) {

        //positions记录了profile中的关键词在title中的位置，一个关键词在title中可能出现不止一次，所以用数组记录
        Map<String, List<Integer>> positions = new HashMap<>();
        for (int i = 0; i < titleWords.size(); i++) {
            String word = titleWords.get(i);
            if (p.words.contains(word)) {
                positions.computeIfAbsent(word, k -> new ArrayList<>()).add(i);
            }
        }

        List<String> kws = p.words;
        int n = kws.size();

        //双重循环遍历关键词对
        for (int i = 0; i < n; i++) {
            for (int j = i + 1; j < n; j++) {
                String w1 = kws.get(i);
                String w2 = kws.get(j);
                List<Integer> pos1 = positions.getOrDefault(w1, new ArrayList<>());
                List<Integer> pos2 = positions.getOrDefault(w2, new ArrayList<>());

                //遍历两个关键词的位置对
                for (int a : pos1) {
                    for (int b : pos2) {
                        if (w1.equals(w2) && a == b) continue;
                        if (Math.abs(a - b) - 1 <= p.number) return true;
                    }
                }
            }
        }
        return false;
    }
}
```
