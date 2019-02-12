---
layout: post
title: On building a fast and reliable client application
date: '2019-02-11 15:41:40'
tags:
- technical
- client
---

The idea of working full-stack has been around for a long while. A full-stack engineer is capable of participating in development of any application module, be it client or server. After all, underlying concepts and principles are shared. Still, there are subtle differences in metrics for which we should optimize when targeting different platforms. Those differences, if appreciated, would greatly improve user experience. If throughput, latency and horizontal scalability are key consideration in server development, preceived responsiveness and resiliency are those of importance to client application. While we have control over our own server's hardware, client devices' capacity varies greatly and connectivity can be unstable at time. This post is a collection of such considerations. 

### 1. Retry on network error

When your application runs on client devices, network connection may not function well all the time. Wifi signal is not always stable. User may be moving to a non receptive corner. Requests to online service may fail even if service is still alive. The lazy way

Stateful vs stateless (or not really stateless, just shorter UI session, quicker restart. (rich app is not the same as website. People just don't F5). Network condition is not reliable. It's wifi or 3g. It's not your typical data center's connection. 

1. Manage your own event queue

Any UI based application is built around an event loop. Granted. Each event loop has a queue of interrupts/callbacks waiting to be executed. If the user clicks on a button which trigger an action, that action is enqueued. A callback from a HTTP request, enqueued. ... Until when the program counter is free from what it is doing and starts to pick up new action again. For most application, under normal workload would just rely on this mechanism to arbitrage computing power between different tasks. However, when the client is under heavy workload, say handling many messages per second, it risk live-lock as messages handler takes up time. Main loop does not have time for touch/click handlers.

Don't just rely on the main event loop queue. It results in un wanted priority. 



2. Try and catch 

The forgiving nature of a client. It often relies on main logics stored on the server. So contrary to the fail fast and fail early of server. Client UI should be forgiving. Catch and log exception. But don't kill the app. Just unload the UI part that caused trouble. Ui should be re-createable from the program state. I mean, some time a program end up being a mess of logics and external dependencies. there is no way to pause the program, unload the component to save on memory and re-create it again (and also shorter state life-time helps avoid lots of problem). On the other hand, there are components that are well encapsulated, but failed to take state into account when re-create. i.e it onlys starts with a blank state. 

For example . Consider a list of message. It is autonomous unit. When it creates, it check it supplied properties. If the list of message is there. It rendered. Else, it fetch messages by itself. Or maybe if it still need to fetch messages for freshness, still display old message first to create the impression of immediateness. 

Pushing fetch logic to components allow better component re-use. And better sync with display cycle. Data need only be fetched when the component is shown.

3. The fewer the better. 

Throttle real-time request. Batch them. 

4. Understanding of business:
critical vs non-critical. Real-time notification vs non realtime but better prepared response. (e.g when receive notification, load it first, and then show it)

Catch event like switching between tab (re-focus) to check cache to see if anything change. 