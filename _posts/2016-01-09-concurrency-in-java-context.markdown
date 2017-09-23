---
layout: post
title: Concurrency in Java context
date: '2016-01-09 17:25:40'
category: engineering
tags:
- technical
- java
- concurrency
---

Concurrency is an unavoidable fact in web development if the page ever gets pass more than one user (which is pretty much any service out there). But concurrency also poses a problem to data consistency. This post is a back-to-the-basics summary of techniques I'm aware of. 

Consider a program to simulate a forest, in which there are big crazy banana-loving monkeys surrounding a big banana tree. Since these monkeys love banana like crazy, they jump at the banana tree all at once until there are no bananas left. 

#### Naive implementation 

```language-java
/* Tree.java */
public class Tree{
    private int bananas;

    public Tree(int b){
        this.bananas = b;
    }

    public void grows(){
        this.bananas += 1;
    }

    public boolean drops(){
        this.bananas -= 1;
        return true; 
    }
}

/* Monkey.java */
public class Monkey implements Runnable{

    Tree tree;
    public Monkey(Tree tree){
        this.tree = tree;
    }

    public void run(){
        try{
            for(int i = 0; i <= 10; i++){
                Thread.sleep(10);
                this.tree.drops();
            }
        }
        catch(InterruptedException e){
            // ...
        }
    }

}
```

Looks good? No. Say, there are 2 monkeys trying to grasp the fruit. The line *this.bananas -= 1;* does not happen simply in one CPU clock cycle, but rather consists of smaller steps: read-update-write. These smaller steps are interleaved such that monkeys can proceed concurrently (but not necessary simultaneously). One possible scenario, which is commonly referred to as [race condition](https://en.wikipedia.org/wiki/Race_condition): 

Monkey A reads  : 5 (bananas) 
Monkey B reads  : 5
Monkey A updates: 5 - 1 = 4
Monkey A writes : 4 
Monkey B updates: 5 - 1 = 4
Monkey B writes : 4

In the end, there are 4 bananas left in stead of 3 as it should be. 

#### Atomic access
The above problem arise due to updating a variable in Java is not atomic, but spans over multiple CPU clock cycles. So, the simplest solution would be to make this update operation *atomic*, i.e happens all at once. 

For writing to primitive type variables, Java provides the `volatile` keyword, which ensures every write happens-before subsequent read on the same memory area. (See [docs](https://docs.oracle.com/javase/tutorial/essential/concurrency/atomic.html)). The above Tree class can be modified as: 

```language-java
/* Tree.java */
public class Tree{
    private volatile int bananas;
``` 
Besides, Java 1.5 and above comes with `java.util.concurrent.atomic` package which provides atomic update operation for reference types as well ([docs](https://docs.oracle.com/javase/7/docs/api/java/util/concurrent/atomic/package-summary.html)). 

Problem solved? No (again). The situation we are modelling is not that simple. The banana tree represents a kind of limited resource. When no. of bananas is zero, monkeys stop taking. Let's add a check in the *drops* function: 

```language-java
    /* inside Tree class */
    public boolean drops(){
        if(this.bananas > 0){
           this.bananas -= 1;
           return true;
        }
        return false; 
    }
```
Unfortunately, adding that check makes the *drops* operation no longer atomic. Again, we see a 3 steps read-check-write operation, which is prone to race-condition as the read-update-write operation above, just at a higher logical level. 

In order to solve this problem, we will be grouping these read-check-write steps into one single logical block in one of the most commonly used approach in dealing with concurrency, serialization. 

#### Serialization 
Imagine that there is a big locked-gate to the banana tree. Monkey who wants to get banana must acquire the lock. Since there is only one lock, only one monkey can get to the tree at a time and he can do the whole of read-check-write operation without any other monkeys interfering in between. 

Traditionally, serialization is done via a mechanism called intrinsic lock in Java, using keyword `synchronized`. Monkey class run method can be rewritten as: 

```language-java
   public void run(){
        try{
            for(int i = 0; i <= 10; i++){
                Thread.sleep(10);
                
                // only one thread can execute the synchronized block at a time. 
                // threads that can't acquire the lock are blocked until the lock is released. 
                synchronized(this.tree){
                    this.tree.drops();
                }
            }
        }
        catch(InterruptedException e){
            // ...
        }
    }
``` 

In this example, *this.tree* is used as the intrinsic lock. We can use any other objects as lock, but make sure that players are synchronized on the same lock. 
 
Or, alternatively, we can make the drops method of Tree class a synchronized method (which is synonym to a synchronized block using `this` as the lock object)

```language-java
    /* inside Tree class */
    public synchronized boolean drops(){
        if(this.bananas > 0){
           this.bananas -= 1;
           return true; 
        }
        return false;
    }
```

The later appears to be cleaner, since it synchronizes right at the exposed public method level. Otherwise it would fall to the caller of *drops* method to make sure that the call is synchronized. Consider, sometimes later, a Bird class comes along and fails to call *drops* within a synchronized with the same lock (i.e tree object), race-condition still happens.

However, in some case, the former approach would be preferred. If we modify the problem statement a little bit, instead of just having Monkey and Tree, we are now having Monkey, Tree and Nature. Tree is the passive datasource whereas Monkey and Nature are each actively reducing or growing no. of bananas. Now we have a traditional [Consumer-Producer](https://en.wikipedia.org/wiki/Producer%E2%80%93consumer_problem) problem. In the end, the consumer and producer will need to collaborate using a synchronization mechanism, which makes synchronizing the *drops* and *grows* methods themselves redundant and not efficient. That's probably the reason why Java `Hashtable` is obsolete in favor of `HashMap` ([docs](http://docs.oracle.com/javase/7/docs/api/java/util/Hashtable.html)). 

Mechanism like `synchronized`, `Object.wait` and `Object.notify` is considered low-level and hard to get right. It can give rise to a host of possible problems like *deadlock, livelock ...*. Therefore, Java 1.5 adds better high level support for serializing concurrency. Find more about it [here](http://www.oracle.com/technetwork/articles/javase/index-140767.html). 

Serialization is not ideal because it creates efficiency problem as acquiring lock is not free. Executing threads may be busy waiting for the lock to be released. Moreover, [context-switching](https://en.wikipedia.org/wiki/Context_switch#Performance) between threads results in overheads. 

Following sections discuss ideas of reducing the overhead of serialisation, by making the synchronised logic minimal. 

#### Use single thread

When it comes to modifying state data, allow only one thread to do it. 

```language-java
//modify the Monkey class as below
static ExecutorService executor = Executors.newSingleThreadExecutor();
...
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

Astute readers may notice that, the single-thread executor service must use some underlying synchronised queue to save task waiting to be executed. That should involves intrinsic locks as discussed in the above section. So what exactly is different? The difference here is the granularity of synchronised section. In this example, executing threads are blocked only for the period of inserting tasks into Executor service's queue, not for the whole computation period.   

In framework such as Akka, shared state is often handled by a Actor class instances, which are single-threaded by nature. 

```language-java
//re-write Tree class using Akka actor
public class Tree
    extends UntypedActor{
    public static enum Msg {
        DROP, GROW;
    }

    ...
    public void onReceive(Object msg){
        if(msg == Msg.DROP){
            drops();
        }
        else if(msg == Msg.GROW){
            grows(); 
        }
        else{
            //...
        }
    }
}

//Monkey class get injected with only the Actor Reference of Tree Actor, not the Tree object 
public class Monkey implements Runnable{
     public Monkey(ActorRef treeRef){
         this.treeRef = treeRef
     }

     public void run(){
        try{
            for(int i = 0; i <= 10; i++){
                Thread.sleep(10);
                // tell create a record in ActorSystem message queue.
                this.treeRef.tell(Tree.Msg.DROP)
            }
        }
        catch(InterruptedException e){
            // ...
        }
     }
}
``` 
#### Optimistic lock 
Say, the monkey takes really long time to eat a banana, overheads would be unjustifiable to synchronised the entire read-update-write section since update is time-consuming. 

The idea is, let each monkey read-update independent of each other. Just before writing banana value back to mutual state, check if the value it reads before update has been modified since. If yes, fails the write and have it re-try. 



(*Source code in this post can be found [here ](https://github.com/vuamitom/Code-Exercises/tree/master/concurrency)*)