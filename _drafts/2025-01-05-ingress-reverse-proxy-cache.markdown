---
layout: post
title: Enable Kubernetes's ingress reverse proxy cache
date: '2024-01-05 22:02:00'
tags:
- technical
---

Kubernetes's ingress is built on Nginx
As with Nginx it can be used as a reverse proxy cache. 
It's less straightforward. if you're using Bitnami nginx chart for ingress. There are two things. 

Adding a config map to declare the cache area 

Adding location configuration block 

Fix cache MISS problem 

Reference 

https://stackoverflow.com/questions/72516747/kubernetes-ingress-response-caching
https://stackoverflow.com/questions/62245119/ingress-nginx-cache 