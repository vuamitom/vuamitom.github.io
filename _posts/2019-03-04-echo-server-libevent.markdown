---
layout: post
title: Echo server with libevent
date: '2019-03-04 22:11:00'
tags:
- technical
---

Network programming is one area where non-blocking IO can be used to achieve higher performance. A typical server needs to handle a few hundreds to a few thousands connections at a time. With the thread-pool based blocking model, when a new connection is established, a server's thread serving that connection will trigger kernel system call to read data from socket file descriptor, be blocked until data are available. Thus, to handle say 200 connections concurrently, the sever needs to spawn 200 threads. 

On the other hand, with the non-blocking IO model, the application code just informs kernel of file descriptors that it is interested in. And instead of waiting for data, it will continue doing other tasks. Kernel will inform it when data are available. Compared with the thread-pool based model, a single thread can handle IO operations for a higher no. of connections, which is assumed to reduce CPU context-switching overhead. Ideally, number of event loops equals number of cores on the host machine. 

[FbThrift](https://github.com/facebook/fbthrift)'s Cpp2 server is one production-grade example of non-blocking architecture. They has a single event loop that binds to server socket and dispatches incomming connections to a pool of IO event loops. These IO event loops each handles read from/write to a bunch of client sockets. After an IO event loop reads data from a client socket, it then dispatches the actual request to be processed in another worker thread-pool to avoid event loop blocking. A basic FbThrift example can be found [here](/2019/01/17/fbthrift-for-cpp-service.html).

Thanks to [libevent](https://libevent.org/), putting together a non-blocking server is fairly simple. The code is [here](https://github.com/vuamitom/Code-Exercises/tree/master/blog/libevent/server). To build:

```bash
# build
mkdir build && cd build
cmake ..
make

# run server
./echo_server
```

To test it. `netcat` is handy. 

Right now, our server treats whatever incoming data as a single stream, not caring what it is. So the next step would be devise a protocol on top of this stream in order to invoke different service methods. And after that, it would be nice to multiplex requests on a single TCP connection. Will update the code next time :). 

#### References

1. [Generic multithreading advice](https://github.com/facebook/folly/blob/master/folly/io/async/README.md#generic-multithreading-advice)