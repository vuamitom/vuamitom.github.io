---
layout: post
title: Warehouse assignment with Google OR Tool
date: '2024-07-24 18:02:00'
tags:
- or-tool
- technical
- optimization
---

Google OR Tool is a powerful library with built-in support for certain classes of optimization problems. For example, assignment problem is a well-known problem, in which N number of customers should be assigned to M number of warehouses such that the total serving distance is minimized. Replacing warehouses with fire stations, we may want to minimize the longest distance instead. 

But solving an optimization problem is not as simple as plugging the data into the library. The process can be fasten if is assisted with domain knowledge to further prune the search space and to break symmetry. E.g observing the ditribution of warehouses and customers can lead to additional constraints like a customer is unlikely to be assigned to a warehouse that is too far away. 

![](/content/images/warehouse_assignment.png)

