---
layout: post
title: On making a smooth client centric javascript application
date: '2018-08-30 17:43:00'
tags:
- rich client
- technical
---


1. have your own event queue to better arbitrage different action

2. throttle request that happens too often while only the last one matters

3. localStorage can acts as an cache

4. indexeddb don't insert too large a record. it will get stored as blob

5. use virtualized list 

6. design with worker thread first.

7. it pays to have some form of arbiter at interfaces such as databases, network, callback executioner... such that you have control of priority and better awareness of what's going on. 

