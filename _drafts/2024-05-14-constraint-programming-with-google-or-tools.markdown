---
layout: post
title: Contraint programming with Google OR Tool
date: '2014-05-14 04:43:00'
tags:
- constraint-programming
- or-tool
---

Constraint programming (CP) is a technique employed in solving combinatorial problems. Combinatorial problems are set of problems that are quick to verify but take exponential time to try out all possible assignments. Brute force approaches often fail to terminate or take very long CPU time to be of any practical use. 

Instead of naiively trying out all combinations, CP works by picking a domain value for a decision variable one by one, and at each step, remove unsatisfying branches (prunning) and choose next assignments effectively (searching). When an assignment leads to an infeasible outcome, it backtrack on previous assignments and try next ones. 

## Pruning 

CP model the problem by definings a set of decision variables (e.g `colors[i]` a node `i` in graph coloring), each with its own domain of possible values. Contraints are the conditions these variables need to satisfy. i.e if a node's color should be different from ones' that are connected to it:

```python
for u, v in edges:
    model.Add(colors[node_u] != colors[node_v])
```



## Searching 


In constraint programming, a contraint store acts as both feasibility checker and a pruner. 
Each variable represents a decision to make on assigning value.
Contraint programming (CP) apply these 2 steps: prunning and backtracking. 
To help prunning faster, we choose "harder" variable that is likely to weed out more constraints 
To help backtracking, we breaks symmetry. 
For example, when coloring a graph, color is symmetrical in the sense that it does not matter which color a node is given. As long as, it is different from all nodes that are connected to it, then it is ok. 
And we can explore the node in the direction that are not 