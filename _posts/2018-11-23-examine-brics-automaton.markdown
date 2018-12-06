---
layout: post
title: How dk.brics.automaton regex library works
date: '2018-11-22 10:16:00'
mathjax: true
tags:
- engineer
---

This [brics regex library](http://www.brics.dk/automaton/) is by far the fastest when comparing with openJDK `java.util.regex` and `com.google.re2j`. Let looks at what lie under the hood. `dk.brics.automaton` is a Finite automata library with application in Regex. The idea is similar to google re2j, which is to construct a DFA from regex string and matching an input string means advancing from one state to another. (google re2j is surprisingly the slowest in my test case. Which is probably due to my particular regex input, or some bug with Java port. I have not examined it yet)

#### Minimize DFA

Consider the regex `ab+cd|abc+d` being parsed by brics. At first, two separate automaton are created for each side of the union operator. 

```
// preliminary processed state graph

initial state: 4
state 0 [reject]:
  b -> 0
  c -> 3
state 1 [accept]:
state 2 [reject]:
  b -> 0
state 3 [reject]:
  d -> 1
state 4 [reject]:
  a -> 2

UNION

initial state: 2
state 0 [reject]:
  d -> 1
  c -> 0
state 1 [accept]:
state 2 [reject]:
  a -> 3
state 3 [reject]:
  b -> 4
state 4 [reject]:
  c -> 0
```

Then these automata are combined into a single automaton simply by merging the initial state. Subsequent state remains pretty much separated. 

```
// basic union state graph 
initial state: 6
state 0 [reject]:
  b -> 0
  c -> 5
state 1 [reject]:
  b -> 0
state 2 [reject]:
  b -> 4
state 3 [accept]:
state 4 [reject]:
  c -> 7
state 5 [reject]:
  d -> 8
state 6 [reject]:
  a -> 1
  a -> 2
state 7 [reject]:
  d -> 3
  c -> 7
state 8 [accept]:
```

The above automaton is non-deterministic since there are two possible transitions from state 6 given character 'a'. The next step is to transform this non-deterministic automaton into deterministic one. 

```
initial state: 7
state 0 [accept]:
state 1 [reject]:
  b -> 1
  c -> 6
state 2 [accept]:
state 3 [reject]:
  b -> 1
  c -> 8
state 4 [accept]:
state 5 [reject]:
  b -> 3
state 6 [reject]:
  d -> 4
state 7 [reject]:
  a -> 5
state 8 [reject]:
  d -> 2
  c -> 9
state 9 [reject]:
  d -> 0
  c -> 9
```

Basic ideas are 

- reduce: combine adjacent and overlapping edges with the same destination.
- remove dead state: state is considered dead if it is not reachable from any valid state.

Different choices of algorithm for DFA minimization

- Using huffman
- Using hopcroft
- using valmari
- using brozozowski

```
// after DFA minimization

initial state: 5
state 0 [reject]:
 b -> 2
 c -> 3
state 1 [reject]:
 b -> 0
state 2 [reject]:
 b -> 2
 c -> 4
state 3 [reject]:
 c -> 3
 d -> 6
state 4 [reject]:
 d -> 6
state 5 [reject]:
 a -> 1
state 6 [accept]:
```

#### Create a non overlapping byte range map for all transitions.

The idea of brics lib is very similar to Google Re2j. It creates a dfa, states of which contains map from input characters to transition to another state. It also sought to group characters into non overlapping byte ranges, each of which lead to single transition from a given state, to reduce size of the transition map. 

Consider the regex `a[a-z]+cd|abc+d`

```
// initial transition after DFA minimization
initial state: 4
state 0 [reject]:
  d-z -> 0
  a-b -> 0
  c -> 2
state 1 [reject]:
  a-z -> 0
state 2 [reject]:
  d -> 3
  a-b -> 0
  e-z -> 0
  c -> 2
state 3 [accept]:
  d-z -> 0
  a-b -> 0
  c -> 2
state 4 [reject]:
  a -> 1

// state transition after splitting into non overlapping byte ranges
initial state: 4
state 0 [reject]:
 a -> 0
 b -> 0
 c -> 2
 d -> 0
 e-z -> 0
state 1 [reject]:
 a -> 0
 b -> 0
 c -> 0
 d -> 0
 e-z -> 0
state 2 [reject]:
 a -> 0
 b -> 0
 c -> 2
 d -> 3
 e-z -> 0
state 3 [accept]:
 a -> 0
 b -> 0
 c -> 2
 d -> 0
 e-z -> 0
state 4 [reject]:
 a -> 1

```

#### Review of Hopcroft DFA minimization

Most minimization algorithm works by paritioning the initial set of coarse states into smaller sets, within which, states are equivalent. Equivalent states are states that, given a sequence of input characters, would all eventually transit into either accepting or rejecting output (i.e either a match or non-match result). Implementation differs. Among the choices of minimization alogrithm, [Hopcroft](http://i.stanford.edu/pub/cstr/reports/cs/tr/71/190/CS-TR-71-190.pdf) with time complexity of NlogN is probably the fastest generic minimization algorithm. 

Here is a summary of how Hopcroft works, given a automaton \\(A\\{S, I, f, F\\}\\) with S is the set of initial states, I is the set of possible input characters, f(s, i) is the transition mapping function and F is the set of terminating states.

Step 1. For each state in S, it backtracks by one transition, to obtain a set of states:

\\[ f^-1(s,i) = \\{t \| f(t,i) = s\\} \\]

Step 2. Dividing set of states obtained from above step into accepting and rejecting subset. 

\\[ A_{1} = \\{t \| t \in F \\} \\]
\\[ A_{2} = X - A_{1} \\]

Now we have X as as set of partitions. \\( X = \\{ A_{1}, A_{2} \\} \\)

Step 3. Select a parition, and a input character i, say \\((A_{2}, i)\\). Then loop through existing partitions in X to see if each need to be split further. A partition \\(A_{j}\\) need to be split in this iteration if, there exists state s in A that \\( f(s, i) \notin A_{2} \\). If no, we don't need to consider \\((A_{2}, i)\\) again. If yes, \\(A_{j}\\) is then split into set of states that transit into \\(A_{j}\\) and states that don't. And then choose the smaller set in next iteration of step 3.

Step 4. Iterations in step 3 stop when no partition can be further divided. 

#### References

1. [http://www-igm.univ-mlv.fr/~berstel/Exposes/2009-06-08MinimisationLiege.pdf](http://www-igm.univ-mlv.fr/~berstel/Exposes/2009-06-08MinimisationLiege.pdf)

2. [Wikipedia DFA Minimization](https://en.wikipedia.org/wiki/DFA_minimization)

3. [A N log N algorithm for minimizing states in a finite automaton](http://i.stanford.edu/pub/cstr/reports/cs/tr/71/190/CS-TR-71-190.pdf)