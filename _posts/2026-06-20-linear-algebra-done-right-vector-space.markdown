---
layout: post
title: Read Linear Algebra Done Right - Vector Spaces
tags:
- life
---

I came across [the book](https://linear.axler.net/LADR4e.pdf) as suggestion from a Quora answer and decided to give it a try. 
Instead of introducing vector spaces with Decarte plane, it introduces vector spaces as sets. 

The geometry view of the subject can be limited by our ability to visualize objects in more than 3 dimensions. Sets are not. 

Before introducing vector spaces, Field and List come first. 
Field ($\mathbb{F}$) can be either Real or Complex numbers, and is a collection of at least $[0, 1]$ that satisfy arithmetic multiplication and addition properties. 
A list, meanwhile, is an ordered collection of elements in ($\mathbb{F}$) with defined length. A list is a special case of function that maps index $(0,..,n)$ to $(v_i, v_n)$ with $v_i \in R$

A vector space is a collection V of lists, functions or arbitrary objects together with scalar multiplication and addition over V that satisfy:
- Communicative 
- Associative 
- Addition identity & inverse  
- Multiplication identity 
- Distributive 

Vector subspaces are subsets of a vector space that are bounded with addition and scalar multiplication.  

$\mathbf{F}^n$ is a vector space of n length lists of field F.  
$\mathbf{F}^S$ is a vector space of functions that map from S to F. 
And since a list is a special function, $\mathbf{F}^n$ is also a vector subspace of $\mathbf{F}^F$

In introducing vector spaces, the author walked through the process of building up math theories. The first few notattions are defined, stated as is. And the later statemetn can be proved based on the initial constructs. 