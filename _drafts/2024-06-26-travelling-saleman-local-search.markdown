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

Another approach to combinatorial optimization problems is to use local search (LS). With constraint programming (CP), we slowly expand the search frontier by making one choice after another, and prune the search space along the way by removing choices that do not meet constraints. Once a full solution is reached, we are certain that is a valid solution. 

On the other hand, LS starts with a complete configuration which may not be valid. From there, making edits to get closer to a valid solution. 

# Techniques 

## Heuristic 

Given the initial solution configuration, make greedy changes to it in order to reach a more optimal state. Being greedy means making decision that optimizes for a local metric that may lead to an lower overall objective value. The local metric is something quick to compute. For example, in the traveling saleman problem (TSP), instead of asking which k edges we can swap to make the total distance decrease, we can look for which are the two longest edges. Now swapping k longest edges does not guarantee a lower overal objective value, but it is easier to calculate. 

Sometimes, a greedy optimizer can get stuck jumping back and forth between states. Because deterministic edge selection like the one mentioned above can get into a local optimal state. If an intermidiate state between local and global optimum has a lower objective value, the greedy heuristic will not consider it. We can introduce some randomness or rely on meta-heuristics to address that. 

It is fairly straightforward to make edge selection more random. For example, randomly choose an edge to swap with the longest edge. Or simply select 2 random edges. But simply picking random edges can be time consuming. Meta-heuristic methods can help escaping local optima more efficiently. 

## Meta-heuristics

Meta heuristics approaches like Tabu Search and Simulated Annealing do not only consider the current solution state to make decision. They keep track of the search process to avoid duplicate moves or when to move away from local optima. 
- Tabu Search: keep history of moves that have been tried which need not be explored again. 
- Simulated Annealing: Have a temperature variable that is decreasing overtime. The lower it is, the higher chance for the solver to accept a move to a state with less optimal objective value. Takes a step back in order to move forward. 

# When to use local search

When the search space is large even pruning it with constraints does not really help. 