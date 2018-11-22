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

https://en.wikipedia.org/wiki/Vietnam

#### Result

TODO

#### Conclusion

`dk.brics.automaton` is has the fastest matching time (at the expense of long compiling time). TODO: look at the trade-off and how they do it.

Matching multiple regex separately is not always slower than combining them into one big regex in one run. It depends on average input length and the implementation. As demonstrated by this benchmark, running separate regex on short inputs is faster in case of `java.util.regex` and `com.google.re2j`. I guess it is due to having to traverse a larger state tree is costly for short inputs. When input is longer, implementation like `com.google.re2j` caches state transition. A combined regex make better use of cached state transition.


Surprisingly, despite all the performance claim, `com.google.re2j` performs the worst. It can be probably due to edge cases, unicode regex ... I don't know. There is some [similar bug report](https://groups.google.com/forum/#!topic/re2j-discuss/8c3L06m6wbY) on worse performance than JDK standard regex around the internet.

