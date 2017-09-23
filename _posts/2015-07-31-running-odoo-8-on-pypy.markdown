---
layout: post
title: Running Odoo 8 on Pypy
date: '2015-07-31 05:19:00'
category: engineering
tags:
- odoo
- pypy
- technical
---

Pypy has not reached the point of compatibility that one could simply switch the interpreter. Some libraries need to be replaced, or installed with latest development code. Still, one can get the Odoo web running with a few changes. 

Changes that were made are pretty simple: 
##### **1. gevent and greenlet**
Install from the latest development code to avoid smth like this: 

	gevent/callbacks.c:8:18: error: ‘PyThreadState’ has no member named ‘curexc_type’
	
	gevent/callbacks.c:11:19: error: ‘PyThreadState’ has no member named ‘curexc_value’
	
	gevent/callbacks.c:12:23: error: ‘PyThreadState’ has no member named ‘curexc_traceback’

To install from github

	pip install cython  git+git://github.com/gevent/gevent.git#egg=gevent
	
	pip install git+git://github.com/python-greenlet/greenlet.git#egg=greenlet
	
##### **2. psycopg2**
Instead of psycopg, install: 

	pip install psycopg2cffi psycopg2cffi-compat
	
Here's the link to complete pip requirements.txt

[Github gist](https://gist.github.com/vuamitom/eece12bb4a4cf78378d6)
