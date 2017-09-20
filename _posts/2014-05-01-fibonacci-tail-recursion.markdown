---
layout: post
title: Fibonacci Tail Recursion
date: '2014-05-01 04:43:00'
tags:
- fibonacci
- tail-recursion
- haskell
- functional-programming
- technical
---

*(Documenting my progress with Haskell. little by little)*


Haskell, or functional programming language in general, is without the variable-stored states often seen in other imperative languages. Therefore, it requires a little thinking shift in crafting an algorithmic solution. 


Consider handling an array of elements. In Python, Java or C#..., a **for** loop comes to mind naturally. The blueprint goes like: having initial value stored in a variable X, and then for each iteration of **for** loop, we do calculations on X such that at the end of the loop, X contains the value we need.


However, **for** loop is not present in the Haskell's arsenal. In fact, recursion forms the very basis of functional programming, not loop. This seems unnatural at first. Let look at the Fibonacci example to see how we do it with recursion. 


A naive approach would be to implement it exactly as how it's defined.  

```language-haskell
fib 1 = 0 
fib 2 = 1
fib n = fib (n-1) + fib (n-2)
```

Hmm, let'see. We're good. The program yields results as expected. But problem starts to surface when n gets to value of >= 40. The code takes seconds to return, too much for simple purpose. Compile the program with profile flags ([Real world Haskell][1])

```language-bash 
ghc --make ./fibonacci-tail.hs  -prof -auto-all -caf-all
time ./fibonacci 40 +RTS -hc -p
```

Here's what we got: 

total time  =       33.06 secs   (33057 ticks @ 1000 us, 1 processor)
    total alloc = 36,408,208,176 bytes  (excludes profiling overheads)
    

33.06 secs, that's ourageous!!. Not to mention the huge memory allocated. Let's look at the recursive call, the execution flow would be as below:  


fib n =  fib (n -1) + fib (n-2) 
fib n = (fib (n-2) + fib (n-3) ) + (fib (n-3) + fib (n -4))
fib n = ((fib (n-3) + fib (n-4)) + (fib(n-4) + fib(n-5)) + (fib (n-4) + fib (n-5) + fib (n-5) + fib(n-6)))


That explains. A given **fib** call would not return until the very last call in the chain returns, resulting in a large number of literals being pushed into the program's memory stack. As n increases, memory use increases exponentially. Besides, there are a large number of duplicated functional (e.g fib (n-3) is evaluated thrice). 

With imperative language such as Python, part of the problem could be solved by using a cache such that subsequent calls to fib(n-3) won't require re-evaluating the whole thing. Unfortunately, I don't know how to use cache in Haskell yet, or does Haskell even have the notion of cache ( since it has no state ). To solve the issue 'functionally', we need something called **tail-recursion**

In the recursive code we just did, a given function call waits for result from functions deeper in the recursive chain to return, does the calculation and then returns. In tail recursion, a function does it calculation first, pass the result as parameter to subsequent recursive call. And when the very last recursive call returns, the final result has already been obtained. 

The Fibonacci code can be re-written tail recursively as : 

```language-haskell
f 1 p1 p2 = p2 
f 2 p1 p2 = p1 
f n p1 p2 = f (n-1) (p1+p2) p1
fib n = f n 1 0 
```
And the code execution was a breeze: 

total time  =        0.00 secs   (0 ticks @ 1000 us, 1 processor)
total alloc =      67,952 bytes  (excludes profiling overheads)


Great, so where did the gain come from. Firstly, Haskell has **tail call optimization** mechanism. For a given tail recursive call, it returns exactly the result of adjacent call. In other words,  recursive call is the last statement in a given tail recursion call. Therefore, context such as arguments can be freed up from mem stack even before the call returns. 


Secondly, this implementation is stateful, just that 'state' is not stored in any variables but passed as arguments to each recursive call, which helps memorizing value of Fibonacci of lower order and thus avoids redundant evaluation. Conceptually, it's like a for loop with the last 2 Fibonacci values kept in p1, p2. 


Wait a minute, did we just go all this length with functional programming just to achieve a very simple for loop? Is it worth the trouble? I don't know. Maybe once we stay with functional programming long enough, our programmer's instinct will accomodate tail recursion, normalize it and make it natural and simple the way **for** loop is. 

  [1]: http://book.realworldhaskell.org/read/profiling-and-optimization.html