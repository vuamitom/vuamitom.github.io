# Cpp vs Cpp2 server architecture

Below are high level differences between the 2 cpp implementation. 

### 1. Parallel handling of requests per client connection.

**FBThrift** handles requests from the same client concurrently while **Apache**'s does not. 

#### FbThrift

FBThrift enable concurrency per client. With it comes the need for out of order responses. Say a request that arrives later but takes less time to handle should be returned immeditately even if the prior request is still in processing. 

FBThrift `ThriftServer` allocates a `ioThreadPool_` from which an `ThriftAcceptorFactory` creates `Cpp2Worker` to handle incoming requests. Ideally, `ThriftServer` creates a thread per core, and one single `Cpp2Worker` per thread. `Cpp2Worker` schedules handlers on eventBase run loop of the thread on which it runs. To ensure that handlers' code do not block eventBase run loop, handlers are wrapped into a future-based non-blocking async interface by the generated Cpp code. 

Thus if a handling routine is waiting on IO, subsequent requests are run instead of waiting for prior request to complete. FBThrift `ThriftServer` impl is heavily dependent on `folly` and `wangle` libraries. 

An instance of `Cpp2Worker` (Acceptor) handle multiple connections. In other words, connections from clients are distributed among IO threads. This aspect is similar to Apache's. `folly::AsyncServerSocket::dispatchSocket` is the method that is responsible for distributing **new connection** event to acceptor's queue (see `RemoteAcceptor` and `NotificationQueue`) in a round-robin manner (`callbacks_.nextcallback()`). 

TODO: `folly::AsyncSocket` is responsible for reading to IOBufQueue. Confirm that it runs on io_pool 

#### Apache

For Apache, requests are processed sequentially for each connection (look at `TNonblockingServer::TConnection::transition`, see that state machine is associated with each connection instead of request). Even for `TThreadedServer`, clients' requests are shared among threads. But workload within a given client is processed sequentially. 

Further more, Apache client is blocking. Multiplexing requests to one connection is only possible with the use of thread-based concurrent client api (which is said to be slower).

Thread-wise, TNonBlockingServer is similar to FBThrift server in that it uses one IOThread pool to read from sockets (htol, memcopy) and one Thread pool to execute tasks. IOThread pool also acts as Acceptor pool of FBThrift. 

TODO: look into wangle::ServerBootstrap for schedule, balance, port reuse logic. (method: `pipeline` vs `childPipeline` and some config)
### 2. Buffer Allocation

For **Apache**, looking at the code of `TServerFramework.cpp`, there is a single instance of `TTransport` created for each client. As a result, requests from the same client share a single buffer. 

TODO: check how folly::IOBuf is shared. 

### 2. Non-blocking handling 

Cpp2 server vs TNonblockingServer of Apache

### 3. Duplex server

A server instance that acts as both client and server. The server then shares client's eventbase run loop. (Look at `ThriftServer.h` code). This does not look important for now.


### 4. Serialization

Comparing serialization performance of FBthrift and Apache thrift can be found [here](./SERIAL.md);

### 6. Accepter vs IOWorker threads vs CPUWorker threads

FBThrift server makes use of 3 sets of threads: 
- Acceptor thread pool
- IO thread pool (which drives Cpp2Worker). Refer to [github](https://github.com/facebook/wangle/blob/master/wangle/bootstrap/ServerBootstrap.h#L117). A `ServerWorkerPool` instance acts as observer to iopool, creates new acceptor (`Cpp2Worker`) whenever a thread is started. This new acceptor handles a part of socket events on its own thread. 
- CPU worker thread manager, which wraps and executes service processors to avoid blocking run loops. 

Acceptor thread pool is single thread accepting connection. And acceptors would then distributes tasks to IO threads in a round-robin fashion. (see `folly::AsyncServerSocket`, which put msg into `NotificationQueue`, which in turned is shared among acceptors).

Other than that, there is ThreadManager instance to manage CPU worker threads (to actually do the request handling?)

`setNumCPUWorkerThreads` vs `setNumIOWorkerThreads`

 * Cpp2Worker drives the actual I/O for ThriftServer connections.
 *
 * The ThriftServer itself accepts incoming connections, then hands off each
 * connection to a Cpp2Worker running in another thread.  There should
 * typically be around one Cpp2Worker thread per core.


One thing to note about FBThrift **Acceptor**. Everytime a libevent OPEN handler is triggered, it tried to call `accept` N times, will that faster handle concurrent connections? Or will it is because of correctness only (e.g when fd read fires, accept is only called once, missing out some other connections)

### 7. Socket handling

TODO: For FBThrift looks at ServerBootstrap ServerSocketFactory

 * ServerBootstrap is a parent class intended to set up a
 * high-performance TCP accepting server.  It will manage a pool of
 * accepting threads, any number of accepting sockets, a pool of
 * IO-worker threads, and connection pool for each IO thread for you.

### 8. TCP Fast open option?


### 9. Handling of Buffer

folly::AsyncServerSocket

```
bool AsyncServerSocket::setZeroCopy(bool enable) {
  if (msgErrQueueSupported) {
    int fd = getSocket();
    int val = enable ? 1 : 0;
    int ret = setsockopt(fd, SOL_SOCKET, SO_ZEROCOPY, &val, sizeof(val));

    return (0 == ret);
  }

  return false;
}
```

#### Read buffer

**fbthrift** does not keep separate buffer for each connection. Instead AsyncTransport::AsyncReader::ReadCallback only get buffer when data is available. 

```
* This method allows the ReadCallback to delay buffer allocation until
* data becomes available.  This allows applications to manage large
* numbers of idle connections, without having to maintain a separate read
* buffer for each idle connection.
```
meanwhile, **apache** allocate dedicated buffer per connection, and doubles buffer size every time it is not enough. If TConnection object stays alive, buffer grows indefinitely and memory is not freed. 

### 10. HHWheelTimer 

### 11. Inter thread communication 

#### FBthrift

use notification queue with lock `folly::SpinLockGuard`. Only wake up consumer that is not running. TODO: check if Cpp2Worker acceptor is always on??
fb thrift relies on the piping between event loop too. But it adds a msg queue buffer, so if consumer is already running (no piping is necessary, just put msg on the queue). Avoid the overhead of writing msg data over pipe? 

#### Apache

add fd to event loop. notify by pipe signal to the fd. But actually, apache only write pointer (8 bytes at most) to pipe; Maybe mem management will be trickier since the responsibility of freeing the pointer lies somewhere 



### 12. Inter-thread communication

FBthrift use eventfd, fallback to pipe if not supported. And the queue in front to reduce pipe call (is that necessary?)

Apache use pipe (evutil pair socket api)


eventfd is said to be lightweight alternative to pipe

http://www.sourcexr.com/articles/2013/10/26/lightweight-inter-process-signaling-with-eventfd

### References

- [https://code.fb.com/open-source/under-the-hood-building-and-open-sourcing-fbthrift/](https://code.fb.com/open-source/under-the-hood-building-and-open-sourcing-fbthrift/)
 
 http://www.wangafu.net/~nickm/libevent-book/TOC.html
