---
layout: post
title: Shortcomings of web technologies in building a robust desktop application.
date: '2018-09-06 23:16:00'
tags:
- chromium
- indexeddb
- webapp
---

As computation power and data are shifting to the edge, web applications are more and more like installed desktop applications with (limited) file system access, threading, offline storages... Frameworks such as Electron, which allow browser-based application to be packaged as standalone app, are gaining popularity. However, with great power comes great responsibility. Javascript, html, css have always been used to build webpages, which is a stateless and forgiving environment. A broken webpage can be remedied by hitting refresh button. The web is not used to be fast, users' expectation for it is lower than for an installed application. 

Me and my team has recently had a chance to test the limit of what web technologies can do and how fast it can run by building an desktop application that serves millions of users a day. Web technologies enabled us to deliver fast and to iterate on the product much faster than developing an equivalent C++ application. Feedback time between code changes and UI updates is short. Besides benefits, we experienced its limits as well. Let's not mention issues that are not obvious to novice users, such as the bloated RAM usage, and occasional CPU hike. And fast improving end users' machines are making that somewhat acceptable. Rather, below are problems that are impactful on end users' experiences and not likely to be fixed anytime soon.  

##### Chrome's Indexeddb is not reliable.

Though the web comes with 3 storage options: localStorage, WebSQL, Indexeddb, it's not much of a choice in practice. LocalStorage is simple key-value without indexing support. WebSQL is discontinued. Indexeddb is the only contender for serious database. (In hybrid environments like Electron, sqlite as node module is available, but it won't work for the browser environment) 

In our case, Indexeddb is not just for cached data but rather main users' storage. Past users' generated data, which are not too important to occupy much server's storage space but at the sametime not discardable, are stored entirely on client machine. In other words, Indexeddb holds all of users' history. We would push down data to browser and expect Indexeddb to do what databases do, keep it. Unfortunately, we discovered later that Chrome (Electron) automatically wipes Indexeddb clean under a few circumstances.

- To reclaim disk space when storage is strained. Though there is a so-called persistant api (documented [here](https://developers.google.com/web/updates/2016/06/persistent-storage)) which is supposed to request permission from user for examption from this policy, Chrome automatically denies the request if page is neither bookmarked nor highly engaged. Firefox handles persistent api much better by asking end users explicitly. Electron seems to grant persistent request just fine. 
- To recover from previously corrupted state. This happens when host machine crashes while Chrome Indexeddb is opened, Indexedb is very likely to be corrupted. This is a [known bug](https://bugs.chromium.org/p/chromium/issues/detail?id=146284#). Chrome team won't fix it anytime soon.

##### Webish UI

Web-based UI requires more effort to attain the sleek feel of a native app. We encountered problem with font rendering. Segoe UI typeface just looks blurry on Chrome and there is no way to fix it yet. Firefox, again, does font rendering better.

When an web-based app loads for the first time, there is always this instant when the underlying browser needs to parse, layout and render before anything appears. And to avoid screen flickering, a host of techniques must be applied, such as inline css background color for the Body DOM node. Or if you're using Electron, it's a good idea to preload the application window before the need to show it.

##### Image manipulation is severely lacking

For many image-related tasks, working with Image DOM element is pretty cumblesome. Our application would download images by setting `src` of `<img/>` element, which is not limited by same domain policy. It works fine until it's no longer just about displaying the image. For example, when source is rotated by 90 degree, there is no access to the image's bytestream, which has been downloaded by the Image DOM. Or when it comes to resizing large images, we would need to draw the image on to a hidden Canvas element before any manipulation is done, which technically means duplicating the bitmap in memory. Besides Image DOM element lack mechanism for progress reporting, retrying on error...

It's understandable where things stand now, as `<img/>` was created before javascript was contrieved for the web. Javascript apis to work with images feel like an afterthought. 

##### Concluding thoughts

Above are things that I hope would have been better. But in retrospect, giving up on them for the incredible speed of development and reduced cost for multi-platform deployment was a trade-off worth making. Web technologies are still at early stage and evolving fast and the boundary between web apps and native apps is blurring. 
