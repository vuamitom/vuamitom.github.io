---
layout: post
title: Implement a simple thread-based MapReduce
date: '2019-11-15 14:36:18'
tags:
- technical
---

MapReduce is a programming model that distributes tasks across a cluster of processing units for parallel handling. The first face, is to map processing function to each data item. The second phase, reduce, is to aggregate the result. Apache Hadoop's MapReduce implementation has become a standin for MapReduce in developers' conversations. But what if we just want to make better use of all cores in a single machine. MapReduce as a model can still be useful.

Imagine having to process a video frame by frame. And processing function is CPU bound. 

```python

while datasource.has_more():
	item_id, item = datasource.get()
	processor.process((item_id, item)) 
```

Python multithreading interface comes in handy. We would need the queue to distribute tasks to workers. One decision point is that, what if we have a dedicated queue for each worker, and distribute them in round-robin maner? That would work too, but there may be cases when a worker's queue is pilled up. while others are done. 

```python 
import queue
class Processor:
	def __init__(self):
		self.queue = queue.Queue(maxsize=20)
```

`maxsize` is helpful. It blocks put method and prevent datasource from reading more. Imagine, we don't want to hold too many frames of a video in memory. 

The rest is about using lock judiciously. 

We make use of synchronized queue to communicate between master and workers. And use a condition lock object to signal when workers all finish. 