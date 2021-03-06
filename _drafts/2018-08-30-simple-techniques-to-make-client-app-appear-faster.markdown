---
layout: post
title: Simple techniques to make a client app appears faster and reliable
date: '2018-08-30 17:43:00'
tags:
- rich client
- technical
---

During the course of building a fast and reliable client application that serves millions of users daily, me and my team at work use some simple tweaks to bring about a better perceived speed and performance.

Here they are, in no particular order. 

1. Inline css background and spinner

A lot of web app shows a spinner before loading complete to signal progress. It's best to base64 encode the resource file and embed it directly in the html page via data url. The browser won't waste another round trip to fetch the resource.

2. Use throttling 

Everything is expected to be instant. Search is one of them. Google make its an expectation that results get shown and updated as you type. However, it's a common mistake to keep hitting server with partial queries on every key stroke. Best to delay it a few ms after each keystroke such that subsequent query that are too close can be batched.

3. delay spinner

showing a spinner when a widget is first shown. It creates a flicker in layout. Better wait for a short while. E.g data load from localstorage (indexeddb)

4. Cached image as blob

All in all, they are based on the principle of 
- perceived throughput
- avoid relayout
- do work early

5. Technique number 

Or the best way to do it it's just not doing it at all. By doing nothing, you have the speed of nothingness. How long is nothingness, is it instantaneous or is it forever. Nothing is right here right now. But there is no end to nothingness, it can be said to have last forever. If there is nothing to use, there is no trouble about speed and then problem is solved. Everyone is happy. It's ok, it was as bad as it used to be 2 years ago when I first joined. Let think that starting over frequently is a good exercise. At the same time it's troubling. 2 years and starting over. It's my decision, let's own it and follow through to the end.


What I talk about when I talk about responsive application

It is becoming increasingly simple to put together a working application in a short amount of time. Firstly, a large number of frontend frameworks and tools offer to solve the gound work of setting up basic base. It takes a few minutes to pull Bootstrap and draw an app's layout with predefined css classes. An equally short amount of time to add React, which loosely enforce a interface update cycle that help avoiding performance penalty of abusing DOM updates. And any framework, minimal or opinionated, help development faster by eliminate the multipilicity of choices. Secondly, faster computers make programmers' time more valuable compared to computers' time. 

However, when building applications that serves millions of users a day and loaded up with data, it pays to get details right to provide the most robust and responsive experience. Below are bullet points that I think might help alot had they been considered upfront. I have always approached application building by building what is absolute minimum necessary for an function to work and scale up from there. But in restropect, sometimes I underestimate the benefit of those principles(??), which as a result cost me hours of rusing to fix before launch date. 

These bullet points are in no particular order.    


Working on frontend's performance means focus of perceived performance vs actual performance. 

When you scale an application to the platform's limit. It's important to take control of everything. From downloading an image to how to render them. There are so many legacies in the web platform that it will have a hard time catching up with native dev platform. One of them is how image is downloaded and handle. Even though image bitmap has already been downloaded and rendered to <img/> object, it not possible for javascript to read from there and manipulate bitmap freely without first drawing to a canvas. <img /> does not display according to BOM header. we used a separate thread to read byte

There are multiple uses of indexeddb. However, we push indexeddb to its limit by inserting thousands of messages to it. Other data as well. When number of users is large, there are edge cases. Case when users happen to have a almost full hard-disk. Chrome indexeddb does not support a persistence database. Which means that it delete all the data without users consent. Not only that. Sometimes data is lost when host machine crashes. (known issue with Chrome indexeddb), and users end up with a corrupted db. Then Chrome would automatically delete it to open a new one just as well. 

Some example includes
- pre-load above the fold. 
- load part of the data first. 
- breaking down processing into smaller chunk (like react fibre)
- JVM garbage collection strategy (mark sweep gc vs ...)


With all that has been said. I'm tryng to put down a comprehensive list of things that could have saved me the troubles. These are stuffs I learned along the way and was fortunate enough to lead those important projects. Had I known it from the beginning, I could have designed a better application. 

1. have your own event queue to better arbitrage different action

2. throttle request that happens too often while only the last one matters

3. localStorage can acts as an cache

4. indexeddb don't insert too large a record. it will get stored as blob

5. use virtualized list 

6. design with worker thread first.

7. it pays to have some form of arbiter at interfaces such as databases, network, callback executioner... such that you have control of priority and better awareness of what's going on. 

8. Some form of placeholder to reduce re-layout helps. E.g server can communicate total no of images first. Client display accordingly before actually getting the picture.

9. Loading spinner should not be there if wait time is too short. 

10. Retry with network failures. But only with errors due to network. Not with server error codes. Do not retry on server error code. As it may overload the server further. 

11. Show offline data first before fetching online one. 

12. Merging offline and online state: a locally unique id is often helpful in deduping and joining result after ward.

13. Making a cut between config batch saving and auto saving per config line can be really different. (this is more of UX, it's about communicate error on time and at the right place)

14. Retry with photos and all. Coming from web background, a lot of people take it for granted that users would just hit the refresh button. Web is stateless. A long live application is stateful