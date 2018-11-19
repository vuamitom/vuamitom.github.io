---
layout: post
title: Misconceptions when scale applications
date: '2018-10-25 11:04:00'
tags:
- engineer
---

Once in a while, engineering managers and software engineers would triumphantly tell me to use a particular piece of technology because it is scalable or good for concurrency. Unfortunately, scaling an application means trading off different factors affecting an app's performance and no single technology comes as silver bullet to solve them all. Below are some of the misconceptions I often hear.

##### 1. Just use coroutine

Coroutine is first popularized by Nodejs with its non-blocking programming paradigm. Traditionally, a program routine would occupy CPU's execution context from start to finish even if it has to wait for disk/network access. Parallelizing with routines is relying on spawning multiple threads and the CPU's built-in context switching mechanism. On the contrary, coroutine would relinguish the program flow control to its caller before its finish if it is blocked by disk/network wait.

Since the introduction of Nodejs, dozen other languages like Python, .NET or Java have introduced their set of async or non-blocking api. Async style program is believed to consume less memory since it does not create a lot of threads, and also faster because CPU's context switching is costly. However, before joining the non-blocking cult, one should consider the nature of the application, whether it often fetch data from databases or over network. In case of database-heavy application, non-blocking makes sense as instead of having the program wait for disk rotation, it returns control to the coroutine's caller to continue the event loop, and thus, have the current thread do useful work. Otherwise, re-writing an application in non-blocking style may not make much different, not to mention that it is harder to get right and may sacrifice some readability of the program. Non-blocking program is single threaded and it trusts its subroutine to relinguish control whenever there is a network/disk wait. However, historically, libraries of languages like Java or Python are mostly blocking. A call to JDBC would jeopadize the single threaded application. I once had to develop a non-blocking smtp client in place of the Python's standard smtp lib due to the lack of existing async libraries. 

An async program is also harder to follow. Nodejs program used to be littered with pyramid of callbacks. Fortunately, most programming languages now support async/await pair, which makes program flow look similar to blocking one. 

##### 2. C++ or java is faster.

