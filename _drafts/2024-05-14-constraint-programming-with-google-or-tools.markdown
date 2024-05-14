---
layout: post
title: Contraint programming with Google OR Tool
date: '2014-05-14 04:43:00'
tags:
- constraint-programming
- or-tool
---

In constraint programming, a contraint store acts as both feasibility checker and a pruner. 
Each variable represents a decision to make on assigning value.
Contraint programming (CP) apply these 2 steps: prunning and backtracking. 
To help prunning faster, we choose "harder" variable that is likely to weed out more constraints 
To help backtracking, we breaks symmetry. 