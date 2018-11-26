---
layout: post
title:  How regex is implemented
date: '2018-10-25 11:04:00'
tags:
- engineer
---

Recently at work, I need to take a deeper look at how optimization is done by different regex libraries and how they combine regex patterns. This document is examining 2 regex implementations: `java.util.regex` of [OpenJDK](https://github.com/openjdk-mirror/jdk7u-jdk/tree/master/src/share/classes/java/util/regex) and `re2` of [Google](https://github.com/google/re2). 

## 1. A look under the hood

Conceptually, both libraries start with parsing regex string into a concrete syntax tree and then use it to process input strings. However, Google's `re2` when further by compiling this syntax tree into more primitive byte range based instructions and then creates a DFA for the input string.

### OpenJDK

OpenJDK is straighforward in its internal tree(graph) based representation of regex. A logical component in the input regex is represented by a node e.g: LiteralNode for char literal 'a', CurlyNode for literal with quantifiers b+ ... Each node possesses a `next` reference to the one after it. Doing regex matching is as simple as running from top node down to last node. Pseudocodely, it's like:

```
is_match(str) = match(root, str[i]) && match(root.next, str[i+1:])
```

OpenJDK does not do much optimization other than using [Boyer-Moore search](https://en.wikipedia.org/wiki/Boyer%E2%80%93Moore_string-search_algorithm) on sequence of literals. (A colleague pointed out to me that [KMP algorithm](https://en.wikipedia.org/wiki/Knuth%E2%80%93Morris%E2%80%93Pratt_algorithm) is an alternative)

### Google Re2

Compared to OpenJDK, Google `re2` employs many more optimization techniques at different phases of process. Conceptually, `re2` engine works in 3 phases: firstly it parses regex string into a syntax tree which is roughly similar to OpenJDK, secondly it compiles this syntax tree into simple instructions, and lastly, at matching time, it generates a DFA.

#### a. Parse Regex String

Similar to OpenJDK, at this step, the engine reads a regex string and returns a tree of nodes with opcode and reference to adjacent node. Some minor optimization is done at this step including:

- Special cases for character class: single char class to a literal e.g [.] as this is commonly used to escape the '.' literal. Or [Aa] to literal A with ASCII case folding.

- Factors common prefixes from alternation. `ABC|ABD|AEF|BCX|BCY` simplifies to `A(B(C|D)|EF)|BC(X|Y)` and then to `A(B[CD]|EF)|BC[XY]`. E.g:

```
ab+cd|abc+d -> a(?:b+cd|bc+d)
```

- Coalesce adjacent quantifiers:
```
ab+bc --> ab{2,}cd
ab+b+c --> ab{2,}cd
ab*bc --> ab{1,}cd
```
- Simplify repeat quantifier. General case: x{n,m} means n copies of x and m copies of x. The final m copies are nested so that machine do less work. This step is probably to help converting this expression to DFA later, which is [not capable of counting](https://en.wikipedia.org/wiki/Deterministic_finite_automaton). 

```
a{2,6} --> aa(?:a(?:a(?:aa?)?)?)?
ab{2,}cd --> abb+c
ab{1,}cd --> ab+c
```

#### b. Compile to instructions

This step converts the tree representation in the above step into an ordered list of instructions. Instructions are primitive, mostly byte range matching. Consider a simple regex `ab|cd`, running through this step will become:

```
5. alt -> 1 | 3
1. byte [61-61] -> 2
3. byte [62-62] -> 4
2. byte [63-63] -> 6
4. byte [7a-7a] -> 6
6. match! 0
```

After flattening to remove `alt` instruction:
```
1+ nop -> 3
2. byte [00-ff] -> 1
3+ byte [61-61] -> 5
4. byte [63-63] -> 7
5. byte [62-62] -> 6
6. match! 0
7. byte [64-64] -> 6
```

Last but not least, a byte map from char range to instruction id is created using coloring algorithm. Later on, when an input character is process, appropriate instruction id is looked up using this map. 
```
[00-60] -> 0
[61-61] -> 1
[62-62] -> 2
[63-63] -> 3
[64-64] -> 4
[65-ff] -> 0
```
#### c. Create DFA to search input string

This last step happens when the compiled regex obtained from previous step is used to match an input string. A DFA can be thought of as a form of caching. The matching execution starts with a state s, and for each byte c in input string, do `s = s->next[c]`, and then check if `s` represents a matching state. `s->next` is constructed lazily (incrementally). When a input byte `c` is processed and `c` does not exist in `s->next`, the next state n will be computed based on instructions stored in `s`. Then n will be stored by `s->next[c] = n`. 

DFA makes for a very simple runtime. Unlike `OpenJDK` which have most of its if-else logic happen at matching time, `re2` 's matching logic is reduced to simple map lookups. 

## 2. Implication for combining regular expression

To combine any random regular expression, the crude thing to do is to combine them using alternate operator `|`. In that case, `java.util.regex` simply using a `BranchNode` to connect the two expression tree, which results in a performance similar to running those expressions separately. 

On the other hand, Google `re2` can optimize for simple cases such as common prefix. And internally, it creates a single bytemap lookup for all branches.