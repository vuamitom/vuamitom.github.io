---
layout: post
title: Tornado Non-blocking Smtp Client
date: '2014-07-28 05:16:00'
tags:
- tornado
- python
- smtp
- technical
---

Recently, I was developing a tornado-based web application at work. The idea of using Tornado is to base the whole web application on Tornado's single-threaded IOloop, which enables the application to handle higher load compared to using other multi-threaded model (given the same hardware). Since there is no additional thread spawned to handle concurrent requests, no memory overhead. And besides, that help avoids context-switching cost when the program control is passed between threads.


For my application, most of request serving time was spent on IO (reading data from ElasticSearch index) without much computation. Thus, non-blocking IO loop is ideal for this situation. 


However, for Tornado application to achieve performance, all 3rd party libraries need to be non-blocking as well. One of the app's function is to send email to users, which, normally could be done with Python's standard smtplib. Unfortunately, smtplib is blocking library, using which can be detrimental to the overall performance. 


Since there was no readily available solution around, I rolled my own port of Python smtplib, which makes use of Tornado's asynchronous IOStream. Since it's a port, the syntax is almost the same as that of smtplib, except that most of the api returns a Future. 


```language-python
client = SMTPAsync() 
yield client.connect('host',587) 
```

Installation can be done:
> pip install tornado-smtpclient 

I tested with Tornado 3.x and 4.0. And the port needs python3.3 and above to run. Here's the [github][1] page 



  [1]: http://github.com/vuamitom/tornado-smtpclient