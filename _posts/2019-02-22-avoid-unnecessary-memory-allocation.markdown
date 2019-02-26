---
layout: post
title: The underestimated cost of memory allocation
date: '2019-02-22 22:30:00'
tags:
- technical
---

In languages like Java and C++, memory allocation is explicit and obvious. Programming with arrays in those languages mean thinking about size in advance before allocation. If there is a need for flexible size list, the standard library is also explicit about whether the list is backed by an array or linked list, so that programmers are mindful about operation complexity. 

Moving to scripting language like Javascript, memory allocation is not as evident, which is probably one of the reason why novice programmers make many mistakes that result in performance penalites. Below are some common oversights I came across while reviewing code of programmers in my team. 


### 1. Not making use of array size if you know it in advance

In javascript, it's easy to create a new list like `let a = []`. Under the hood, the V8 engine would allocates an array buffer of fixed size to hold the list's elements. Once that buffer is filled, it would be copied to a bigger one. That copy operation is costly. One help avoid memory copy altogether by specifying a large enough size before hand. 

```javascript
function fixSizeArray() {
    let iter = 10000,
        size = 100000,
        dur;
    dur = time(() => {
        let ar = [];
        for (let s = 0; s < size; s++)
            ar.push(s);
    }, iter);
    console.log(`pushing into array takes ${dur}ms`);

    dur = time(() => {
        let ar = new Array(size);
        for (let s = 0; s < size; s++)
            ar[s] = s;
    }, iter);

    console.log(`pre-allocate array takes ${dur}ms`);
}

// Output:
// pushing into array takes 12222ms
// pre-allocate array takes 4689ms
``` 
From the above benchmark, it is apparent that specifying a large enough size before hand reduces array initilization time by 60%. 

### 2. map vs forEach

Javascript list comes with `map` and `forEach` methods. Both of which allow running a lambda on each element of the list. The difference is that `map` would aggregate returned value from each lambda's invokation into a result list, which cost CPU time even if memory would soon be freed. I don't know if this is a common mistake among Javascript programmers, but it was there during my team's code review. 

```javascript
function mapVsForeach() {
    let iter = 10000,
        ar = new Array(100000),
        dur;
    dur = time(() => {
        ar.map(a => a)
    }, iter);
    console.log(`using 'map' takes ${dur}ms`);

    dur = time(() => {
        ar.forEach(a => a)
    }, iter);

    console.log(`using 'forEach' takes ${dur}ms`);
}

// Output:
// using 'map' takes 6800ms
// using 'forEach' takes 2716ms
```

So if you don't need the returned value, use `forEach` every time. 

## A little twist.

Talking about memory allocation reminds me of a text-book problem. In Java for example, `java.util.String` is an immutable object, which means whenever a string concatenation `+` operator is used, char arrays are copied and joined in a new array buffer, which is costly. So in Java, classes like `StringBuilder` is used to avoid such wasteful memory copy. Coming from that background, I'm pretty mindful of string concatenation in Javascript as well. However, there is no `StringBuilder` in javascript. One of the solution may be to push all string tokens into a temp array and join them once for all. At least with that approach, underlying char arrays would be copied only once, right?

In fact, modern Javascript engine may have optimized string concatenation operator `+` and `+=` (according to this [mail thread](https://www.mail-archive.com/es-discuss@mozilla.org/msg10125.html)) to the point that they are faster. These below benchmark code shows it.

```javascript
function stringConcate() {
    let iter = 100000,
        size = 10,
        dur;
    

    dur = time(() => {
        let a = new Array(size);
        for (let i = 0; i < size; i++)
            a[i] = 'a'
        a = a.join('');
        return a;
    }, iter);

    console.log(`using [].join takes ${dur}ms`);   

    dur = time(() => {
        let a = '';
        for (let i = 0; i < size; i++)
            a += 'a';
        return a;
    }, iter);
    console.log(`using '+' on string takes ${dur}ms`);
}

// Output:
// using [].join takes 224ms
// using '+' on string takes 90ms
```

So, just use `+` for string concatenation in Javascript because it is fast. 

Benchmark code used in this post can be found [here](https://github.com/vuamitom/Code-Exercises/blob/master/blog/mem-alloc.js).