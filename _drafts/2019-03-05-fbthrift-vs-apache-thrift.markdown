---
layout: post
title: Architectural differences between FBThrift and Apache Thrift
date: '2019-03-05 15:41:40'
tags:
- technical
---

Facebook re-opensourced their fork of Thrift in 2014 after implementing a number of changes. They [claimed](https://code.fb.com/open-source/under-the-hood-building-and-open-sourcing-fbthrift/) the new implementation of Cpp server is superior in performance compared to the old `TNonBlockingServer`. In the past week I have looked through Fbthrift source to see the changes. Here is the summary

## Thread model

## Buffer allocation

## Single connection multiplexing

# References

1. [Setup and run FBThrift](/2019/01/17/fbthrift-for-cpp-service.html).