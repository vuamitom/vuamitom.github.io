---
layout: post
title: How not to create 2D arrays in Python
date: '2024-08-02 22:02:00'
tags:
- python
- technical
---

Python has many idioms that make writing the language a joy. 
One of them is, to quickly initialize an array of n elements you can do this 

```bash
>>> a = [None] * 4 
>>> a
[None, None, None, None]
```

Then what about creating a 2D 4x4 array, should it be this:
```bash 
>>> a = [[None] * 4] * 4
```

The answer is no. Doing that gives you an array where each row has the same underlying memory. 

```bash
>>> a[0][1] = 1
>>> a
[[None, 1, None, None], [None, 1, None, None], [None, 1, None, None], [None, 1, None, None]]
```

Then what is the proper way to initialize a 2D array with each element an independent memory location:

```bash
>>> b = [[None] * 4 for i in range(0, 4)]
>>> b[0][1] = 1
>>> b
[[None, 1, None, None], [None, None, None, None], [None, None, None, None], [None, None, None, None]]
>>> 
```

Voila!