---
layout: post
title:  Bayesian perspective in ML
date: '2018-09-27 00:16:00'
tags:
- machine learning
---

#### Using least square error

Loss (cost) function with respect to theta

If it is quadratic function with respect to theta, we can find min . Highschool mathematics say it plateau at derivative = 0. 

We can either solve that equation to find theta.

Or another approach is gradient descent. Analogy: instead of having a map of where to go, we follow the light source/ wind source. Here we look at the direction of gradient. 

#### maximum likelihood perspective

To solve theta for likelihood equation to maximize. By sovling it, we will realize that it is equivalent to least square error.

Maximum likelihood vs least square error

In classification problem, we often need to assign a probability to a class of output given input P theta (y | x)

That probability is a Bayesian prior, not a frequentist' probability. The training process is to update our posterior with lost calculation and then feed that back to training. 


NOTE: it's not mandatory to have probability implication 


#### Cost accumulating perspective 

SVM classification and hingeloss function


#### Reference

Pattern and recognition in marchine learning

http://cs231n.github.io/linear-classify/

#### Note to myself


To avoid overflow of exp(x) == 0:

https://www.tensorflow.org/api_docs/python/tf/nn/sigmoid_cross_entropy_with_logits