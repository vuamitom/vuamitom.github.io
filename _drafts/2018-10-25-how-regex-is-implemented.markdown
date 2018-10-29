---
layout: post
title:  How regex is implemented
date: '2018-10-25 11:04:00'
tags:
- engineer
---

Examined 2 regex libraries: OpenJDK, and google Re2

Common:
- both created an acyclic directed graph representing regex expr
Diff:
- no of optimization
- google has an additional step of compiling into Prog.

##### OpenJDK

- Peephole optimization
- Boyer-Moore search 
- create a syntax tree (state machine ADG): A normal sequence without or logic looks more like a linked list. With an '|', it adds a branch tree node
- BMP match 
- inspect optimization step

##### Google Re2
1- minor optimization
-- single char class to a literal e.g [.]
-- [Aa] to literal A with ASCII case folding
2- optimization when collapsing the final expression 
-- Factors common prefixes from alternation.
// For example,
//     ABC|ABD|AEF|BCX|BCY
// simplifies to
//     A(B(C|D)|EF)|BC(X|Y)
// and thence to
//     A(B[CD]|EF)|BC[XY]
My own example: ab+cd|abc+d -> a(?:b+cd|bc+d)

3- Simplify call on Regex
- Coalesce
E.g ab+bc --> ab{2,}cd
E.g ab*bc --> ab{1,}cd
- Simplify
-- simplify repeat
ab{2,}cd --> abb+c
ab{1,}cd --> ab+c
General case: x{n,m} means n copies of x and m copies of x?.
  // The machine will do less work if we nest the final m copies,
  // so that x{2,5} = xx(x(x(x)?)?)?
-- simpify char class

4- After compilation 
- optimization
- flatten
- build byte map


- different from OpenJDK in that after creating a statemachine, it compile that statemachine into a program (object)
- DFA Deterministic finite automation 
- peep-hole optimization 
- coloring algorithm, mark and merge
- Prog:Flatten graph rewritting algorithm

##### Some additional concept in handling Unicode

- Surrogate pairs: pair of code points that make up a single character
- canonical equivalence: an unicode character can be represented by different set of codepoint(s). Codepoints are said to be canonically equivalent when they have the same appearance in print or on display (i.e canonical decomposition)