---
layout: post
title: Concurrency Handling
---

#### No concurrency
(Or manual serialization). Another way to dealing with concurrency is to not let it happen. Or at least not when modifying state data. A simple implementation is use a single-thread executor

```language-java
//modify the Monkey class as below
static ExecutorService executor = Executors.newSingleThreadExecutor();
private void drop(){
        executor.submit(
                new Callable<Void>(){
                    public Void call() throws InterruptedException {
                        tree.drops();
                    }
                }
        );
    }
```

Astute readers may notice that, the single-thread executor service must use some underlying synchronised queue to save task waiting to be executed. That should involves intrinsic locks as discussed in the above section. So what exactly is different? The difference here is the granularity of synchronised section. In this example, the updating logic seems trivial. 

In framework such as Akka, mutual state is handled by Actor, which is single-threaded by nature.  



#### Immutability 

(*Source code in this post can be found [here ](https://github.com/vuamitom/Code-Exercises/tree/master/concurrency)*)

### Optimistic lock 
