---
layout: post
title: Travelling Saleman problem with local search
date: '2024-06-26 02:02:00'
tags:
- local-search
- or-tool
- technical
- optimization
---

Another approach to combinatorial optimization problems is to use local search (LS). With constraint programming (CP), we slowly expanding the search frontier by making one choice after another, and prune the search space along the way by removing choices that do not meet constraints. Once a full solution is reached, we are certain that is a valid solution. 

On the other hand, LS starts with a complete configuration which may not be valid. From there, making edits to get closer to a valid solution. 

# Techniques 

## Heuristic 

Given the initial solution configuration, make greedy changes to it in order to reach a more optimal state. For example, in the traveling saleman problem (TSP), for 2-Opt change, swapping any 2 edges that makes the total distance decrease. 

Greedy heuristic, though can improve the objective function, risks getting stucked in local optimal state. If an intermidiate state between local and global optimum has a lower objective value, the greedy heuristic will not consider it. Meta-heuristic methods are helping with that. 

## Meta-heuristics

These are techniques that improves local search efficency. 

- Tabu Search: Keep track of which opts ve been made to avoid trying an opt twice. 
- Simulated Annealing: Have a temperature variable that is decreasing overtime. The lower it is, the higher chance for the solver to accept a move to a state with less optimal objective value. Takes a step back in order to move forward. 

# When to use local search

When the search space is large even pruning it with constraints does not help