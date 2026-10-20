---
title: Logisitic Regression
source: https://developers.google.com/machine-learning/crash-course/logistic-regression
created: 2026-10-20
tags:
  - aiml
---
**Logistic Regression** model is a type of regression model (a model that predicts  a numeric value) that predicts a probability. It works for things like "Will it rain today?" or "Is this email spam?"

# Calculating a probability with the sigmoid function 

Many problems require a probability estimate as output. Logistic regression helps us with that. You can use the returned probability in either of the following two ways:
- Applied "as is". For example, if a spam-prediction model takes an email as input and outputs a value of $0.932$, this implies a $93.2\%$ probability that the email is spam.
- Converted to binary category such as `True` or `False`, `Spam` or `Not Spam`.

## Sigmoid function 

But how does a logistic regression model calculates the probability while ensuring that the result is always between $0$ and $1$. There's a family of functions called **logistic functions** whose output has the same characteristics. The standard logistic function, also known as the **sigmoid function** (_sigmoid_, meaning "s-shaped", has the formula:

$$
f(x) = \frac{1}{1 + e^{-x}}
$$

where:
- $f(x)$ is the output of the sigmoid function.
- $e$ is Euler's number, with value of $2.71828$.
- $x$ is the input to the sigmoid function.

![](../../assets/Pasted%20image%2020261020140916.png)

**Fig.** shows the output of the sigmoid function.

As you can see in the above figure, the output never reaches $1$ or $0$ and just stays in between.

## Transforming linear output using the sigmoid function 

The following equation represents the linear component of a logistic regression model:

$$
z = b + w_1 x_1 + w_2 x_2 + \cdots + w_N x_N
$$

where:
- $z$ is the output of the linear equation, also called the **log odds**.
- $b$ is the bias.
- The $w$ values are the model's learned weights.
- The $x$ values are the feature values for a particular example.

To obtain the logistic regression prediction, the $z$ value is then passed to the sigmoid function, yielding a value (a probability) between $0$ and $1$:

$$
y' = \frac{1}{1 + e^{-z}}
$$

where: 
- $y'$ is the output of the logistic regression model.
- e is the Euler number.
- $z$ is the linear output.

![](../../assets/Pasted%20image%2020261020142137.png)

**Fig.** shows how linear output is transformed to logistic regression output.

In the above figure, a linear equation becomes input to the sigmoid function, which bends the straight line into an s-shape. Notice, that the linear equation have very large values up to $-15$ but the sigmoid function converts it to $0<=value<=1$.

First you need to calculate the values of $z$ like you do in linear regression and than put that value in sigmoid function.

# Loss and regularization 

Logistic regression models are trained using the same process as [linear regression](../Linear%20Regression/Linear%20Regression.md) models, with two key distinctions:
- Logistic regression models use [**Log Loss**](https://developers.google.com/machine-learning/glossary#Log_Loss) as the loss function instead of [**squared loss**](https://developers.google.com/machine-learning/glossary#l2-loss).
- Applying [regularization](https://developers.google.com/machine-learning/crash-course/overfitting/regularization) is critical to prevent [**overfitting**](https://developers.google.com/machine-learning/glossary#overfitting).

## Log Loss 

In Linear regression, we used squared loss (also called $L_2$ loss) as the loss function. Squared loss works well for a linear model where the rate of change of the output values is constant. 

However, the rate of change of a logistic regression model is _not_ constant. As we saw above, the **sigmoid** curve is _s-shaped_ rather than linear. 

The loss function for logistic regression is Log Loss. The Log Loss equation returns the logarithm of the magnitude of the change, rather than just the distance from data to prediction. Log Loss is calculated as follows:

$$\text{Log Loss} = -\frac{1}{N} \sum_{i=1}^{N} \left[ y_i \log(y'_i) + (1 - y_i) \log(1 - y'_i) \right]$$

where:
- $N$ is the number of labeled examples in the dataset.
- $i$ is index.
- $y_i$ is the label for the $i$th example. Since this is logistic regression, $y_i$ must either be $0$ or $1$.
- $y'_i$ is your model's prediction for the $i$th example (somewhere between $0$ and $1$) .

## Regularization in logistic regression 

[**Regularization**](https://developers.google.com/machine-learning/glossary#regularization), in logistic regression is a technique that discourages the model from fitting the training data too closely by adding a penalty on the size of the coefficients to the loss function. Without regularization, the asymptotic nature of logistic regression would keep driving loss towards $0$ in cases where the model has a large number of features. Consequently, most logistic regression models use one of the following two strategies to decrease model complexity:

- [L2 regularization](https://developers.google.com/machine-learning/crash-course/overfitting/regularization)
- [Early stopping](https://developers.google.com/machine-learning/crash-course/overfitting/regularization#early_stopping_an_alternative_to_complexity-based_regularization): Limiting the number of training steps to halt training while loss is still decreasing.

> We will learn more about regularization in later modules.

# Further Reading 

- [IBM - What is logistic regression](https://www.ibm.com/think/topics/logistic-regression)