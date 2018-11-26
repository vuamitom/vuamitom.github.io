---
layout: post
title: How regex is implemented: brics library
date: '2018-11-22 10:16:00'
tags:
- engineer
---

This library is by far the fastest when comparing with openJDK `java.util.regex` and `com.google.re2j`. Let looks at what lie under the hood. `dk.brics.automaton` is a Finite automata library with application in Regex. The idea is similar to google re2j, which is to construct a DFA from regex string and matching an input string means advancing from one state to another. (google re2j is surprisingly the slowest in my test case. Which is probably due to my particular regex input, or some bug with Java port. I have not examined it yet)

There seem to be a number of pruning:



- remove dead state: state is considered dead if it is not reachable from any valid state.
- reduce: combine adjacent and overlapping edges with the same destination.


The idea of brics is simpler. It creates a dfa, states of which contains map from input characters to transition to another state. A similar optimization to Google Re2 is that it sought to group characters into non overlapping byte ranges, each of which lead to single transition from a given state. 