---
layout: post
title: C++ '&' operator
date: '2016-03-23 14:10:19'
category: engineering
tags:
- technical
---

Some note on C++ references 

The way I see it, `&` operator in C++ have 2 different meanings. 

#####1) Put another label on storage

```language-cpp
int a = 10; 
int& b = a; // put label 'b' over storage of 'a'
```

Or this 

```language-cpp
void func(int &b) {
   // do not copy 
   // just use the same storage as passed parameter under label 'b'
}

void main() {
   int a = 10; 
   func(a); 
}
```

Either case, nothing is copied. A single storage is given more than one names and compiler treats them the same. 

With this meaning, `&` appears on left side of a statement. 

#####2) Get address of a variable
```language-cpp
int a = 10; 
int *b = &a; //get address of a, copy it to b
```

In this case, memory address of `a` is copied to `b` storage. 

This case, `&` operator appears on right side of a statement. 