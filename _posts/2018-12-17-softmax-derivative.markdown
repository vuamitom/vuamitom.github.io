---
layout: post
title:  Derivative of loss function in softmax classification
date: '2018-12-17 22:16:00'
mathjax: true
tags:
- machine learning
- technical
---

Though frameworks like Tensorflow, Pytorch has done the heavy lifting of implementing gradient descent, it helps to understand the nuts and bolts of how it works. After all, neural network is pretty much a series of derivative functions. In this blog post, let's look at getting gradient of the lost function used in multi-class logistic regression.   

Softmax function is used to describe the posterior conditional probability \\(P(y^{(i)}\|x^{(i)}, w)\\). Using maximum likelihood approach, the loss function is derived as:

\\[ J_{(w)} = - \frac{1}{m} \sum_{i=1}^{m} \sum_{k=1}^{K} y_{k}^{(i)} log s_{k}^{(i)} \\]

To get gradient, the trick is to get partial derivative of \\(J_{w}\\) with respect to \\(w_{jz}\\), which is the weight connecting input \\(x_{j}\\) and output class \\(z\\)

\\[ \begin{align}
\frac{\partial J_{(w)}}{\partial w_{jz}} & = - \frac{1}{m} \sum_{i=1}^{m} \sum_{k=1}^{K} y_{k}^{(i)} \frac{\partial log s_{k}^{(i)}}{\partial w_{jz}} \\\\ 
	& = - \frac{1}{m} \sum_{i=1}^{m} \sum_{k=1}^{K} \frac{y_{k}^{(i)}}{s_{k}^{(i)}} \frac{\partial s_{k}^{(i)}}{\partial w_{jz}}
\end{align} \\] 

With \\( s_{k}^{(i)} = \frac{e^{w_{k}^{T}x^{(i)}}}{\sum_{j=1}^{K} e^{w_{j}^{T}x^{(i)}}} \\), we can expand the partial derivative of \\( s_{k}^{(i)} \\) as below:

\\[ \frac{\partial s_{k}^{(i)}}{\partial w_{jz}} =  
\begin{cases}
x_{j}^{(i)}s_{k}^{(i)}(1 - s_{z}^{(i)}), \text{if } k=z \\\\\\
(-x_{j}^{(i)}s_{k}^{(i)}s_{z}^{(i)}), \text{if } k \neq z
\end{cases}\\]

Putting pieces together, we have:

\\[ \begin{align}
\frac{\partial J_{(w)}}{\partial w_{jz}} & = - \frac{1}{m} \sum_{i=1}^{m} (y_{z}^{(i)}x_{j}^{(i)}(1 - s_{z}^{(i)}) - \sum_{k \neq z}^{K}y_{k}^{(i)}x_{j}^{(i)}s_{z}^{(i)}) \\\\\\
& = - \frac{1}{m} \sum_{i=1}^{m} (y_{z}^{(i)}x_{j}^{(i)} - s_{z}^{(i)}x_{j}^{(i)}\sum_{k=1}^{K}y_{k}^{(i)})
\end{align} \\] 

Since it is a single class problem, only one \\(y_{k}\\) is 1 for an input i, hence \\(\sum_{k=1}^{K}y_{k}^{(i)} = 1 \\). 

\\[\frac{\partial J_{(w)}}{\partial w_{jz}} = - \frac{1}{m} \sum_{i=1}^{m} x_{j}^{(i)} (y_{z}^{(i)} - s_{z}^{(i)}) \\]

Finally, we have the numpy friendly vector form:

\\[\frac{\partial J_{(w)}}{\partial w} = - \frac{1}{m} x^{T} (y - s) \\]