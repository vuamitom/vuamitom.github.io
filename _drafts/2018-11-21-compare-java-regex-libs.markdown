---
layout: post
title:  Benchmark different 
date: '2018-10-25 11:04:00'
tags:
- engineer
- regex
---

This page documents benchmark results running different java regex libraries.

For existing regex libraries in Java, I refered to the list in this reference [benchmark](http://tusker.org/regex/regex_benchmark.html). However, many libraries are no longer around. Some I could find does not work with unicode. In the end, I settle with these libraries: `java.util.regex` of openJDK 1.8, `com.google.re2j`, `dk.brics.automaton`. Will add more if I find others. 


#### Input data

Two set of test data are obtained by extracting plain text from https://en.wikipedia.org/wiki/Vietnam. The first test set consists of split sentences from extracted text. The second test set of long sentences are obtained by joining sentences from first set at random order. 

Regex expressions used to test:

```java
String[] spams = {
    "vay",
    "kho[aả]n vay",
    "vay v[oố]n",
    "vay ti[eề]n",
    "vay (?:t[ií]n|th[eế]) ch[aấ]p",
    "cho vay",
    "[uư]u [dđ][aã]i",
    "hỗ tr[oợ] v[oố]n",
    "gi[aả]i ng[aâ]n",
    "(?:cty|c[oô]ng ty) (?:t[aà]i ch[ií]nh|tc)",
    "(?:vay|v[oố]n|cvay) ti[eê]u d[uù]ng",
    "b[aả]o hi[eể]m nh[aâ]n th[oọ]",
    "th[eế] ch[aấ]p t[aà]i s[aả]n",
    "th[eẻ] t[ií]n d[uụ]ng",
    "l[aã]i su[aâấ]t|ls",
    "nv|(?:nh[aâ]n|chuy[eê]n) vi[eê]n",
    "t[uư] v[aấ]n",
    "ng[aâ]n h[aà]ng|nhnn",
    "điện thoại|s?[đd]t|tel\\b|telephone|call|liên (?:lạc|hệ)|\\blh\\b|gọi|contact|nhắn tin|(?:tin|lời) nhắn|sms"
};

```

For each regex library, the above list of regex string are run either separately, or combined with union operator. To examine the effect of capturing group on performance, the unioned regex is tested with both capturing group and non-capturing group. 

#### Result

##### java.util.regex

| Time | Union with capturing group | Union no capture | Separate run |
| ------ | ------ | ------ | ------ |
| Regex compilation | 1ms | 1ms | 0ms |
| Match 1500+ short sentences set (100 iterations) | 6715ms | 6832ms | 5139ms | 
| Match 300 long sentences set (10 iterations) | 749ms | 780ms | 107636ms | 

##### com.google.re2

| Time | Union with capturing group | Union no capture | Separate run |
| ------ | ------ | ------ | ------ |
| Regex compilation | 2ms | 30ms | 1ms |
| Match 1500+ short sentences set (100 iterations) | 32577ms | 24084ms | 18278ms | 
| Match 300 long sentences set (10 iterations) | 3519ms | 2573ms | 153393ms | 

##### dk.brics.automaton

| Time | Union with capturing group | Union no capture | Separate run |
| ------ | ------ | ------ | ------ |
| Regex compilation | 43ms | 122ms | 47ms |
| Match 1500+ short sentences set (100 iterations) | 190ms | 183ms | 2059ms | 
| Match 300 long sentences set (10 iterations) | 18ms | 45ms | 53051ms | 

#### Conclusion

`dk.brics.automaton` is has the fastest matching time (at the expense of long compiling time). TODO: look at the trade-off and how they do it.

Matching multiple regex separately is not always slower than combining them into one big regex in one run. It depends on average input length and the implementation. As demonstrated by this benchmark, running separate regex on short inputs is faster in case of `java.util.regex` and `com.google.re2j`. I guess it is due to having to traverse a larger state tree is costly for short inputs. On the contrary, when input is really long, running separate regexes are crucifyingly slower, because non-matching patterns have to run till the end of the input string. Unioning regexes and run them once avoids having to run through the whole input string for non-match patterns in case of a match. Besides, implementation like `com.google.re2j` caches state transition. A combined regex make better use of cached state transition. 

Surprisingly, despite all the performance claim, `com.google.re2j` performs the worst. It can be probably due to edge cases, unicode regex ... I don't know. There is some [similar bug report](https://groups.google.com/forum/#!topic/re2j-discuss/8c3L06m6wbY) on worse performance than JDK standard regex around the internet.

