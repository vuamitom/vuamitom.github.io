---
layout: post
title: Misconceptions when scale applications
date: '2018-10-25 11:04:00'
tags:
- engineer
---

Once in a while, engineering managers and software engineers would triumphantly tell me to use a particular piece of technology because it is scalable or good for concurrency. Unfortunately, scaling an application means trading off different factors affecting an app's performance and no single technology comes as silver bullet to solve them all. Below are some of the misconceptions I often hear.

##### 1. Just use coroutine/non-blocking/async code.

Coroutine is first popularized by Nodejs with its non-blocking programming paradigm. Traditionally, a program routine would occupy CPU's execution context from start to finish even if it has to wait for disk/network access. Parallelizing with routines is relying on spawning multiple threads and the CPU's built-in context switching mechanism. On the contrary, coroutine would relinguish the program flow control to its caller before its finish if it is blocked by disk/network wait.

Since the introduction of Nodejs, dozen other languages like Python, .NET or Java have introduced their set of async or non-blocking api. Async style program is believed to consume less memory since it does not create a lot of threads, and also faster because CPU's context switching is costly. However, before joining the non-blocking cult, one should consider the nature of the application, whether it often fetch data from databases or over network. In case of database-heavy application, non-blocking makes sense as instead of having the program wait for disk rotation, it returns control to the coroutine's caller to continue the event loop, and thus, have the current thread do useful work. Otherwise, re-writing an application in non-blocking style may not make much different, not to mention that it is harder to get right and may sacrifice some readability of the program. Non-blocking program is single threaded and it trusts its subroutine to relinguish control whenever there is a network/disk wait. However, historically, libraries of languages like Java or Python are mostly blocking. A call to JDBC would jeopadize the single threaded application. I once had to develop a non-blocking smtp client in place of the Python's standard smtp lib due to the lack of existing async libraries. 

An async program is also harder to follow. Nodejs program used to be littered with pyramid of callbacks. Fortunately, most programming languages now support async/await pair, which makes program flow look similar to blocking one. 

##### 2. Just use a faster programming language.

Compiled language such as Java/C++ are believed to be faster, which is generally true since the job of translating program to bytecode has been done in advance. Besides, java/C++ is statically typed, which gives compiler a lot of hints for further optimization. However, the gain in performance can be marginal if there is not a lot use of CPU. Most of Saas now aday do not do heavy computation or complex image processing. Instead most of processing time is spent on fetching data from database/networked server. Therefore, a little gain in performance may not worth the trouble of using C++ for development which often entails larger amount of code for the same feature. And even if the application is computationally heavy, it is recommended that we separate the computational part from the rest to optimize. SOA (microservice or whatever you call it) architecture would facilitate that nicely. Python has used this approach successfully for years where they replace only the program's hot-path (which is the most time consuming part) with C++ implementation. 

##### Conclusion

It's trade-off. To borrow the no-free-lunch theorem from data science and use it here, there is no single technology that is inherently better than one another. Each has its pros and cons and suitable use cases. Single threaded non-blocking application is suitable for database/network bounded applications, dependencies of which must be non-blocking as well. Choice of programming languages hardly matter to performance's gain but rather determined by human resource and time constraints. 

Last but not least, it is premature to optimize too early in the development process. Unless you know well the expected usage, keep it simple first.

