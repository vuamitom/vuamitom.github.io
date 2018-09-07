---
layout: post
title: Shortcomings of web technologies in building a robust desktop application.
date: '2018-09-06 23:16:00'
tags:
- chromium
- indexeddb
---

As more computation power and data are shifting to the edge, web applications are acting more and more like installed desktop applications with (limited) file system access, threading, offline storages... Frameworks such as Electron, which allow browser-based application to be packaged as standalone app, are gaining popularity. However, with great power comes great responsibility. Javascript, html, css have always been used to build webpages, which is a stateless and forgiving environment. A broken webpage can be remedied by hitting refresh button. The web is not used to be fast, users' expectation for it is lower than for an installed application. 

Me and my team has recently had a chance to test the limit of what web technologies can do and how fast it can run by building an desktop application that serves millions of users a day. Web technologies enabled us to deliver fast and to iterate on the product much faster than developing an equivalent C++ application. Feedback time between code changes and UI updates is short. Besides benefits, we experienced its limits as well. Let's not mention issues that are not obvious to novice users, such as the bloated RAM usage, and occasional CPU hike. And fast improving end users' machines are making that somewhat acceptable. Rather, below are problems that are impactful on end users' experiences and not likely to be fixed anytime soon.  


##### Chromium Indexeddb is not reliable.

Though the web comes with 3 storage options: localStorage, WebSQL, Indexeddb, it's not much of a choice in practice. LocalStorage is simple key-value without indexing support. WebSQL is discontinued. Indexeddb is the only contender for serious database. (In hybrid environments like Electron, sqlite as node module is available, but it won't work for the browser version) 

Our application would push users' generated data down to the browser, encrypt it and store in indexeddb. General expectation of a database is that it store the data. That's it.  

However, it's insane that under many circumstances, Chrome's indexeddb take the liberty of deleting the data. 

1. When user's disk full 
2. when user's host machine crashes. 

No.2 maybe due to design choice, which favors speed over durability. However, no.1 is just weird, shows a lack of respect for data and broke the contract of a database, which is to store data, not to make decision on users' behalf about what to do with those data. 

So do not rely on Chrome for keeping important data if you can help it. 

##### First pain is slow

##### Overlaying task is cumblesome and bad in performance.