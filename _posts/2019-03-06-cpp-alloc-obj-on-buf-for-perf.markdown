---
layout: post
title: Allocate objects on memory buffer for performance gain 
date: '2019-03-06 22:11:00'
tags:
- technical
---

I wrote about the cost of memory allocation in [a recent post](/2019/02/22/avoid-unnecessary-memory-allocation.html). Given a fixed amount of memory needed, reserve a large chunk in one go is cheaper than grabing smaller chunks one at a time. I did not realize that Cpp has the facility to take advantage of that until reading through the code of [folly::IOBuf](https://github.com/facebook/folly/blob/master/folly/io/IOBuf.h). 

For example, we have a class `Obj` and want to reserve an array of N instances of `Obj`. 

```c++
class Obj {
	int i1;
	int i2;
	int i3; 
	char a2;
	long l1;
	long ar[10];
};
```

Normal approach would be to run a loop and create each object with **new** operator. Everytime **new** is invoked, memory allocator looks for area in heap that is big enough to contain the object and reseve that area of memory.  

```c++
static Obj** create_objs() {
	int c = COUNT;
	Obj** r = new Obj*[c];
	for (int i = 0; i < c; i++) {	
		// let memory allocator find free memory in heap.
		r[i] = new Obj();	
	}
	return r;
}
```

With C++, say for 100 instances, instead of looking for free space 100 times, the program can reserve a buffer large enough for all objects in one go. **new** operator can then be instructed to initiate object on that buffer. Below code does just that. 

```c++
static Obj** create_objs_on_buf() {
	int c = COUNT, s = sizeof(Obj);
	int buf_size = s * c;
	Obj** r = new Obj*[c];

	// reserve a buffer large enough for all new objects
	uint8_t* buf = static_cast<uint8_t*>(malloc(buf_size));
	uint8_t* start = buf;
	for (int i = 0; i < c; i++) {
		// allocate new object on buffer
		r[i] = new (start) Obj;
		start += s;
	} 	
	return r;		
}
```

Lets compare the efficiency of each approach with `COUNT = 100000`. 

```c++
int main(int argc, char *argv[]) {
	std::clock_t start;
    start = std::clock();
	auto r = create_objs_on_buf(); // or create_objs()
	std::cout << "Time: " << (std::clock() - start) << " ticks" << std::endl;
    return 0;
}

// output of create_objs_on_buf
// Time: 1936 ticks
// output of create_objs
// Time: 9114 ticks
```
`create_objs` takes 5 times more CPU ticks than `create_objs_on_buf`. Point has been made. However, in reality, the trouble may not be worthwhile. The later approach results in program complexity where object can't be freed independently, thus only suitable for scenario when objects' life cycles are tied together. Secondly, reserving a continuous area in memory can lead to larger heap in case of memory fragmentation. And after all, though 5 time faster sounds like a lot, it may not result in visible performance impact. The demo code allocated a fairly large no. of objects, yet timing difference is only a few ns on a 1.6Ghz core. 

