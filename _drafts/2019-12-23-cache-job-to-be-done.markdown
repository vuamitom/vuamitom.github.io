---
layout: post
title: Caching of job being done
date: '2019-12-23 21:36:18'
tags:
- technical
---

The main idea behind in-memory cache (e.g Memcache, Redis ...) is to avoid carrying out expansive, time or CPU consuming operations by keeping result of prior requests in to answer subsequent identical ones. That works well most of the time. However, under high traffic, a missing key can lead to peak traffic hit to backend services if multiple requests ask for that same key. In this post, I would illustrate that it would be beneficial if in-memory caches can be made aware of how to retrieve missing keys. 

## Cache get and set should be atomic

```python
# a dead simple in-memory cache
cache = dict()
```


```python
def calculate(key):
	""" a calculation guarded with a cache to avoid CPU hoarding """
	if key in cache:
		return cache[key]
	else:
		r = do_heavy_calculation(key)
		cache[key] = r 
		return r
```

That is simple enough. However, in a concurrent world, (assuming `cache` is thread-safe), `do_heavy_calculation` may be invoked multiple times when key is missing. 

A ILLUSTrATION OF THREAD INTERLEAVING 

## How to fix it

### Naiive implementation with lock 

```python
def calculate_with_lock(key):
	with cache_lock:
		return calculate(key)
```
But since `do_heavy_calculation` is really slow as its name already infer, wrapping a lock around the whole cache would be punishingly slow. Requests for other keys would be forced to wait for the lock while `do_heavy_calculation` is running. 

### Make job to be done an explicit cache

A better method would be to have an explicit cache for job being done. 