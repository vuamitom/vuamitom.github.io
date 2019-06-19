# Apache Thrift vs FbThrift Benchmark

## 1. Test setup 

Benchmarks are run using [Google Benchmark](https://github.com/google/benchmark), with client and server run as threads in the same process.

Since fbthrift and apache shares the same cpp namespace, we build a separate executable for each. To build equivalent benchmark:

```bash
cd fbthrift # or `cd apache` for apache thrift
mkdir build && cd build
cmake ..
make
```
Client-server communication uses binary protocol for all tests. TThreadPoolServer with 4 worker threads is used for Apache Thrift. 

## 2. Test Result

### 2.a Timing for single client

Timing reported is captured for the single client thread. 

#### Echo a single number

This test service accepts an integer parameter and returns it. Apache Thrift client takes much less time to run than FB client. This is due to the overhead of FB async client having to start a event run loop. 

||Apache|FB|
|--------|--------|--------|
|Time (ns)|14385|162809|
|CPU Time (ns)|8097|75801|

Now, instead of each client making a single request, we have each client sending multiple requests. Again, Apache still outperforms facebook's. 

|no. requests|10|64|512|4096|20000|
|--------|--------|--------|--------|--------|--------|
|Apache CPU time (ns)|81127|518303|4086440|32700670|159816504|
|FB CPU time (ns)|257919|1359218|11271675|90390175|474303323|

![compare multi get answer](imgs/bench_multi_answer.png)

#### Receive large data blob

FBthrift under performs for blob size smaller than 131k bytes, yet starts to surpass Apache perf for larger blob. 

|no. bytes|1024|2048|4096|8192|16384|32768|65536|131072|
|--------|--------|--------|--------|--------|--------|--------|--------|--------|
|Apache CPU time (ns)|12962|14501|16706|20666|30169|47182|83247|146547|
|FB CPU time (ns)|76613|83211|84217|84190|87234|87526|101290|112933|

|no. bytes|262144|524288|1048576|2097152|4194304|8388608|16777216|
|--------|--------|--------|--------|--------|--------|--------|--------|
|Apache CPU time (ns)|272502|532638|1088978|2255047|4909840|9719469|19876943|
|FB CPU time (ns)|139667|192811|336573|691838|1548475|2687792|15087591|

![compare ](imgs/bench_blob.png)

#### Receive complex struct

In this test, service return a list of nested structs. FBThrift performs better here. 

||Apache|FB|
|--------|--------|--------|
|Time (ns)|14843435|13630538|
|CPU Time (ns)|11884881|9171796|

#### Service handler sleep 30ms

Each request to service handler puts the server to sleep 30ms, which emulates behavior of server with high CPU load or large disk read latency. 

|no. requests|10|16|32|64|128|256|512|1000|
|--------|--------|--------|--------|--------|--------|--------|--------|--------|
|Apache CPU time (ns)|359878|630178|1177128|2363526|4992799|10198225|18304162|37331713|
|FB CPU time (ns)|430577|563583|1044424|2076564|3908078|8220933|15792057|29976302|

![compare ](imgs/bench_sleep.png)

### 2.b Multiple clients throughput

Multiple client threads are spawn to send requests concurrently to server. No. of threads are set to no. of cores on test machines.

For no-op requests, throughputs are similar for both libraries with apache impl. a bit more efficient for smaller no. of requests.

|no. reqs per client thread|10000|20000|30000|40000|
|--------|--------|--------|--------|--------|
|Apache req/sec|106.779k|212.927k|208.508k|209.098k|
|FB req/sec|44.3454k|55.337k|70.2522k|64.3128k|

For get_sleep requests, for which service handler sleeps 1ms, fb's throughput is by far higher

|no. reqs per client thread|10000|20000|
|--------|--------|--------|
|Apache req/sec|3.44996k|3.4504k|
|FB req/sec|8.26732k|15.188k|

### Summary 

For really simple services, Apache thrift is simpler to work with and more efficient. However, Fbthrift appears to work better for more complicated use cases. Work need to be done to identify causes of difference in performance in such cases.

An architectural comparison between Thrift Cpp server implementation and FB Thrift Cpp2 is given [here](./ARCH.md). 

## 3. References

1. [http://szelei.me/rpc-benchmark-part1/](http://szelei.me/rpc-benchmark-part1/) 

2. [https://code.fb.com/open-source/under-the-hood-building-and-open-sourcing-fbthrift/](https://code.fb.com/open-source/under-the-hood-building-and-open-sourcing-fbthrift/)

3. Apache Thrift result

![apache result](imgs/apache_result.png)

4. FbThrift result

![fbthrift result](imgs/fb_result.png)