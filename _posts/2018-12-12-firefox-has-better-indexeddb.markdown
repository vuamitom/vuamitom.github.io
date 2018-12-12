---
layout: post
title: A few things Firefox does better
date: '2018-12-12 10:16:00'
tags:
- engineer
---

Recently, Microsoft has ditched [EdgeHTML](https://en.wikipedia.org/wiki/EdgeHTML) in favor of Chromium rendering engine in its Edge browser. It led to mourning and concern over the uncertain future of the open web, in which, a single big company may have much control over standards and features of the web. Mozilla rallied internet users to [give Firefox another try](https://blog.mozilla.org/blog/2018/12/06/goodbye-edge/). They have a point. The web has always been a messy place with changing standards with which browser vendors catching up at different speed. A nice looking application may look well on Firefox yet broken on Chrome and vice versa. Testing effort is duplicated to ensure consistent look across browsers. Yet, that messiness and duplication of effort creates a larger space for experiments, more opportunities of participation and better chance to make some thing good. Because, I think, innovation has always been done by allowing independent minds to find different approaches to the solve the same problem. And because, in technology, it is always about weighing trade-offs. The fastest implementation is not necessarily the most modular and embeddable one. 

Google has been the force behind innovation. E.g Chrome was the first multi-process browser. And Google will continue to be innovative for the time to come. But the best way to ensure they are motivated to do so for long is some competition. For now, Firefox is the only serious contender, all be it small. And besides the ideological reason to give Firefox another try, there are technical aspects that Firefox did better too. 

Firstly, Firefox renders font better. For a product at work, we used Segoe UI web font. And on Chrome, the text looks blurry despite our team's effort to tweak css like `text-rendering` and `text-shadow`. Text is a lot nicer on Firefox.

Secondly, Firefox's indexeddb implementation is more reliable. Our application is heavily dependent on the browser's ability to store data on the client side. We abused the browser's indexeddb to store Gb of messages, and implemented our own full-text search engine based on it. During the process, we find out that Firefox Indexeddb is more trustworthy. Firefox implements Indexeddb with sqlite underneath, which is battle tested and comes with error recovery mechanism. Meanwhile, Chrome's indexeddb is based on leveldb, which is prone to entire data loss when the host machine crashes. Folks at signal has encountered [this issue](https://github.com/signalapp/Signal-Desktop/issues/718) as well. And Chrome team is [not fixing](https://bugs.chromium.org/p/chromium/issues/detail?id=146284#) anytime soon.

So it is Firefox for me now. I have not missed Chrome after a week with it. 


