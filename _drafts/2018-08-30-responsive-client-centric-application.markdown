---
layout: post
title: On making a smooth client centric javascript application
date: '2018-08-30 17:43:00'
tags:
- rich client
- technical
---

Working on frontend's performance means focus of perceived performance vs actual performance. 

Some example includes
- pre-load above the fold. 
- load part of the data first. 
- breaking down processing into smaller chunk (like react fibre)


1. have your own event queue to better arbitrage different action

2. throttle request that happens too often while only the last one matters

3. localStorage can acts as an cache

4. indexeddb don't insert too large a record. it will get stored as blob

5. use virtualized list 

6. design with worker thread first.

7. it pays to have some form of arbiter at interfaces such as databases, network, callback executioner... such that you have control of priority and better awareness of what's going on. 

8. Some form of placeholder to reduce re-layout helps. E.g server can communicate total no of images first. Client display accordingly before actually getting the picture.

9. Loading spinner should not be there if wait time is too short. 

