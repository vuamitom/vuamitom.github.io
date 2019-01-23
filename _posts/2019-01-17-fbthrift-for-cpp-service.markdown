---
layout: post
title: Basic FBThrift example
date: '2019-01-17 15:41:40'
tags:
- technical
---

Facebook re-opensourced their fork of Apache Thrift some 4 years ago. Yet there is relatively little documentation and independent comparison to see if there is any performance bonus to gain by moving from Apache Thrift to FBThrift. The two are no drop-in replacement, thus, replacing one with another requires effort. This post first looks at setting up and running FBThrift. Complete code example used in this post can be found [here](https://github.com/vuamitom/Code-Exercises/tree/master/blog/fbthrift).

## Build FBThrift

FBThrift source can be obtained from their [github page](https://github.com/facebook/fbthrift). Installing procedure is typical and relatively straight forward. One catch was that the README did not mention [rsocket-cpp](https://github.com/rsocket/rsocket-cpp) in dependencies but that is needed as well. (If you encounter 'internal compiler error: cpp1ypl' when building rsocket-cpp, try to run make without `-j`).

After successfully building FBThrift, the thrift compiler is built to `/path/to/fbthrift/build/bin/thrift1`, which can be used to generate C++ service source. Seem like most of online guides on using FBThrift compiler are outdated, which refers to the python module `thrift_compiler`. Rather, this [github issue](https://github.com/facebook/fbthrift/issues/303) is helpful in making it work. To generate C++ source files from idle file `example.thrift`, run:

```
/path/to/fbthrift/build/bin/thrift1 --gen mstch_cpp2 --templates /path/to/fbthrift/thrift/compiler/generate/templates  example.thrift
```

Content of `example.thrift`

```thrift
# example.thrift
namespace cpp tamvm

service ExampleService {
  i32 get_number(1:i32 number);  
}
```

## Implement ExampleService

Implementing the service code is not much different from Apache Thrift. Output C++ files are generated in `./gen-cpp2` by default. Pay attention to the generated file `ExampleService.h`, which contains the interface `ExampleServiceSvIf` which we need to override with actual service logic. 

```c++
class ExampleHandler: public ExampleServiceSvIf {
public:
	int32_t get_number(int32_t n) override {
		printf ("server: receive %d", n);
		return n;
	}
};
```

## Create client and server instances

For the purpose of this demo, we are going to have a FBThrift client and a server running in one process. 

```c++
int main(int argc, char *argv[]) {
	LOG(INFO) << "Starting test ...";
	FLAGS_logtostderr = 1;	
	folly::init(&argc, &argv);
		
	// starting server on a separate thread
	std::thread server_thread([] {	    
	    auto server = newServer(thrift_port);
	    LOG(INFO) << "server: starts";	    
		server->serve();
	});
	server_thread.detach();

	// wait for a short while 
	// enough for socket opening 
	std::this_thread::sleep_for(std::chrono::milliseconds(50));

	// create event runloop, to run on this thread
	folly::EventBase eb;
	folly::SocketAddress addr("127.0.0.1", thrift_port);
	// creating client
	auto client = newHeaderClient(&eb, addr);
	std::vector<folly::Future<folly::Unit>> futs;
	for (int32_t i = 10; i < 14; i++) {
		LOG(INFO) << "client: send number " << i;
		auto f = client->future_get_number(i);
		futs.push_back(std::move(f).thenValue(onReply).thenError<std::exception>(onError));
	}
	collectAll(futs.begin(), futs.end()).thenValue([&eb](std::vector<folly::Try<folly::Unit>>&& v){
		LOG(INFO) << "client: received all responses";
		eb.terminateLoopSoon();
	});

	// libevent/epoll loop which keeps main thread from existing.
	eb.loopForever();	
	return 0;
}
```
The above code uses `folly::Future` based api for clarify. The `RequestCallback` api will do just as well. `folly::EventBase` is the main run loop on which async callbacks are scheduled. The call to `client->future_get_number` does not actually send outbound request right away. Rather, it schedules the task on EventBase run loop to be excuted in batch [1](https://github.com/facebook/wangle/blob/master/wangle/channel/OutputBufferingHandler.h). As a result, request is only actually sent when caller method gives up program control (whether it returns or waits as coroutine) [2](https://github.com/facebook/fbthrift/issues/307). If the actual time when request is sent is important, the `RequestCallback` api is useful. 

Notice the last line `eb.loopForever` is to keep the program from quitting before collecting all responses. You can replace it with other non-blocking wait call. (`std::this_thread::sleep` is **not** one of them. `sleep` blocks the main thread and EventBase run loop can't proceed).

## Build and Run

Build and run is basic:

```bash
mkdir build && cd build
cmake ..
make 
./fbthrift_ex
```

If you encounter this error `exception specification in declaration does not match previous declaration`, change the compiler to g++ instead of clang++.

Final output: 

```
WARNING: Logging before InitGoogleLogging() is written to STDERR
I0123 17:05:29.276188 26497 main.cc:103] Starting test ...
I0123 17:05:29.276881 26498 main.cc:110] server: starts
I0123 17:05:29.327250 26497 main.cc:121] client: send number 10
I0123 17:05:29.327659 26497 main.cc:121] client: send number 11
I0123 17:05:29.327802 26497 main.cc:121] client: send number 12
I0123 17:05:29.327927 26497 main.cc:121] client: send number 13
I0123 17:05:29.328977 26512 ExampleHandler.h:11] server: receive 11
I0123 17:05:29.328977 26511 ExampleHandler.h:11] server: receive 10
I0123 17:05:29.329150 26510 ExampleHandler.h:11] server: receive 12
I0123 17:05:29.329188 26509 ExampleHandler.h:11] server: receive 13
I0123 17:05:29.329497 26497 main.cc:95] client: get response 10
I0123 17:05:29.329609 26497 main.cc:95] client: get response 11
I0123 17:05:29.329762 26497 main.cc:95] client: get response 12
I0123 17:05:29.329864 26497 main.cc:95] client: get response 13
I0123 17:05:29.329892 26497 main.cc:126] client: received all responses
```

Again, complete source code of example is here: [https://github.com/vuamitom/Code-Exercises/tree/master/blog/fbthrift](https://github.com/vuamitom/Code-Exercises/tree/master/blog/fbthrift)