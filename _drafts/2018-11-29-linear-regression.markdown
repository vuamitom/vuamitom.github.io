---
layout: post
title:  Perspectives in designing cost function
date: '2018-11-29 00:16:00'
mathjax: true
tags:
- machine learning
---

For many different machine learning problems, finding a solution involves these similar steps. 

- Firstly, one begins with gathering input data \\(x\\). 
- Secondly, work out a hypothesis (model) that maps from input to possible output \\(h_{\theta}(x) = y\\). \\(\theta\\) is trainable (adjustable) parameters of the model. 
- Next, a cost(loss) function \\(J(\theta)\\) which determines how far-off our hypothesis is from the expected output.
- Last but not least, a learning algorithm (e.g gradient descent), which is basically a strategy of updating \\(\theta\\) at eaching training step to reduce cost \\(J(\theta)\\).

At each step, there are design decisions to make. In this post, I'm noting two different views in designing a loss function in step 2. Both ways of explaining work and help one better understand why ML works.

#### Minimum error perspective

The cost function from this point of view is a function that measures the difference between expected output and the model's output. Cost function accumulates as hypothesis gets astray from the expected, and decreases as hypothesis gets closer. Some loss functions designed from this perspective are `mean squared error (MSE)` in `linear regression` and `hinge loss` support vector machine (SVM) classification.

MSE is the scalar difference between hypothesized value and expected value, and square it. (Or it is the difference between L2 norm of hypothesized and expected output vectors). It is pretty straight forward from here. To increase the model's accuracy is to minimize the difference between hypothesized and the expected. Solve for min of \\(J(\theta)\\):

\\[J(\theta) = \frac{1}{2M} \sum_{i=1}^{M} (h_{\theta}(x_{i}) - y_{i})^2\\]

For classification, it is less intuitive to visualize loss in the same way. Reason is, it is hard to quantify difference between hypothesized class and expected output class. Say, a machine learning model to classify animals. How much output cat differs from output dog is not quantitative. Instead, the probability perspective introduced below seem to be a more natural approach. Still, it is possible to devise a cost function 

SVM classification and hingeloss function

#### Maximum likelihood perspective

To solve theta for likelihood equation to maximize. By sovling it, we will realize that it is equivalent to least square error.

Maximum likelihood vs least square error

In classification problem, we often need to assign a probability to a class of output given input P theta (y | x)

That probability is a Bayesian prior, not a frequentist' probability. The training process is to update our posterior with lost calculation and then feed that back to training. 


NOTE: it's not mandatory to have probability implication 

#### Reference

[1] [Pattern and recognition in marchine learning](http://cs231n.github.io/linear-classify/)


#### Note to myself

If it is quadratic function with respect to theta, we can find min . Highschool mathematics say it plateau at derivative = 0. 

We can either solve that equation to find theta.

Or another approach is gradient descent. Analogy: instead of having a map of where to go, we follow the light source/ wind source. Here we look at the direction of gradient. 

To avoid overflow of exp(x) == 0:

https://www.tensorflow.org/api_docs/python/tf/nn/sigmoid_cross_entropy_with_logits