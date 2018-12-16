---
layout: post
title:  Perspectives in designing ML cost function
date: '2018-12-16 00:16:00'
mathjax: true
tags:
- machine learning
- technical
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

For classification, it is less intuitive to visualize loss in the same way. An somewhat simplified reason is, it is hard to quantify difference between hypothesized class and expected output class. Say, a machine learning model to classify animals. How much output cat differs from output dog is not quantitative. The simplest possible approach is that, we can use discriminant functions to map from input to a scalar value e.g \\(f_{theta}(x) = y\\), and then use a threshold value to assign output class e.g positive output if \\(y > 0\\). MSE can then be used here as the loss function. Though it suffers some drawbacks like being sensitive to outliers, gradient vanishing and limited to guassian distribution of output \[1\].

Still, it is possible to devise a cost function 

Another example is SVM classification and hingeloss function. Hingeloss is a loss function which accumulates when the difference between score of correct class and an incorrect one is below certain threshold \\(\delta\\).

\\[J(\theta) = \frac{1}{M} \sum_{i=1}^{M} \sum_{k=1}^{K} max(0, s_{k} - s_{y_{i}} + \delta)\\]

#### Maximum likelihood perspective

This approach seem to be more intuitive for classification problem. Given an input \\(x\\), K possible classes and expected output \\(C_{j}\\) 1 <= j <= K, the model should maximize the conditional probability of \\(P_{\theta}(C_{j}\|x)\\). To determine this class conditional probability, one can either uses the `generative modelling` or `discriminative modelling` approach. The former involves finding out the prior \\(P(x\|C_{j})\\) and input distribution \\(P(x)\\) and then infer posteri via Bayesian rule:

\\[ P_{\theta}(C_{j}\|x) = \frac{P_{\theta}(x\|C_{j}) P(C_{j})}{P(x)}\\]

However, determining the input distribution \\(P(x)\\) can be costly for some dataset. So we often directly model posterior \\(P_{\theta}(C_{j}\|x)\\) and tweak \\(\theta\\) to maximize it (discriminative modelling). With m as number of samples:

\\[ P_{\theta}(Y\|X) = \prod_{i=1}^{m} P_{\theta}(y_{i} \|x_{i}) \\]

Maximizing the above likelihood is the same as minimizing the negative of likelihood, arriving at our cost function. And since probability is small, log is often taken. Hence, cost function becomes:

\\[ J_{\theta} = - \frac{1}{m} \sum_{i=1}^{m} log P_{\theta}(y_{i} \|x_{i}) \\]

For example, if we model \\(P_{\theta}(y_{i}\|x_{i})\\) with softmax function, we can have the cost function as:

\\[ J_{\theta} = - \frac{1}{m} \sum_{i=1}^{m} softmax(y_{i}, \theta, x_{i}) \\]


#### Reference

1. [Pattern and recognition in marchine learning](http://cs231n.github.io/linear-classify/)