---
layout: post
title: Contraint programming with Google ORTool
date: '2024-05-21 02:02:00'
tags:
- constraint-programming
- or-tool
- technical
- optimization
---

Constraint programming (CP) is a technique employed in solving combinatorial problems. Combinatorial problems are set of problems that are quick to verify but take exponential time to try out all possible assignments. Brute force approaches often fail to terminate or take very long CPU time to be of any practical use. 

Instead of naiively trying out all combinations, CP works by picking a domain value for a decision variable one by one, and at each step, remove unsatisfying branches (prunning) and choose next assignments effectively (searching). When an assignment leads to an infeasible outcome, it backtrack on previous assignments and try next ones. 

## Pruning 

CP model the problem by definings a set of decision variables (e.g `colors[i]` a node `i` in graph coloring), each with its own domain of possible values. Contraints are the conditions these variables need to satisfy. i.e if a node's color should be different from ones' that are connected to it:

```python
for u, v in edges:
    model.Add(colors[node_u] != colors[node_v])
```

So if `colors[0]` is assigned to color `0`, `0` is removed from the value domains of nodes that are connected to `colors[0]`, which saves CPU time on infeasible combinations. The earlier in the assignment process a value is removed, the more (exponential) downstream choices are saved. 

Besides basic constraints that are required to satisfy the problem goal, other non-primary constraints are often added to help removing unlikely or symmetrical choices faster. **Symmetrical choices** are changes in variable assignments that are relatively identical. In the case of graph coloring, given connected `colors[0]` and `colors[1]`, color assignment of `0-1` and `1-0` are the same as actual color values does not matter as much as whether they differ. So to avoid iterating over symmetrical choices, additional constraints can be imposed to further reduce combinations that need to be explored. For example:

```python 
# Break the symmetry of colors by ordering them
    for i in range(num_nodes):
        model.Add(colors[i] <= (i + 1))
```

Other constraints can be inferred from domain knowledge of the problem. Given N nodes in the graph, the baseline solution is to given each of them a different color, right. Still, the variable domain can be reduced further based on the fact that if a node in the graph has max degree of M, then we won't need more than M + 1 different colors to color the graph. The smaller the domain, the less combinations to explore. 

```
colors = [model.NewIntVarFromDomain(cp_model.Domain.FromValues(range(max_degree)), f'node{i}') for i in range(num_nodes)]
```

## Searching 

At each assignment step, after pruning if there are still more choices, CP model needs to pick a not-yet assigned variable and assign to it a value from the possible domain. A few techniques can help. 

- **Fail fast** remember that during prunning, a domain value removal earlier in the assignment process worth more since it elimiates more downstream choices. So it can help a CP model to terminate faster if we examine "harder" variable first, i.e variable with more constraints. In graph programming, a higher degree node should be prioritized, since once we assign a value to it, we can cross out more options for other variables. With Google OR, we can do that by sorting the node by degree first, and add decision strategy:
```python
model.AddDecisionStrategy(colors, cp_model.CHOOSE_FIRST, cp_model.SELECT_MIN_VALUE)  
```
- **Greedy** approach makes choices based on heuristical rules that are devised to optimize some easily calculable metrics. In optimizing locally for that metrics, the model is likely to reach a global optimal faster. As a example, in graph coloring, a simple rule can be to explore branch with which the domains of remaining variables are the largest, thus, more likely to be able to select existing colors. 
- **Branch and bounds** approach: say node `i` has a domain of `[1,3, 4]` possible colors. Each assignment is a possible color that the CP model can assign to variable `colors[i]`. The model can choose wich branch to explore first by coming up with an optimistic estimation of the cost of each branch and try out the lower cost first. 

*(Example code uses [Google OR tool](https://developers.google.com/optimization))*