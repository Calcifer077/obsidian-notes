---
title: Classification
source: https://developers.google.com/machine-learning/crash-course/classification
created: 2026-10-20
tags:
  - aiml
---
In the [logistic regression](../Logistic%20Regression/Logistic%20Regression.md) module, we learned how to use the **sigmoid function** to convert raw model output to a value between $0$ and $1$ to make probabilistic prediction - for example, predicting that a given email has a $75\%$ change of being spam. But what if your goal is not to output probability but a category - for example, predicting whether a given email is "spam" or "not spam"?

**Classification** is the task of predicting which set of classes (categories) an example belongs to. 

# Thresholds and the confusion matrix 

Let's say you have a logistic regression model for spam-email detection that predicts a value between $0$ and $1$, representing the probability that a given email is spam. A prediction of $0.50$ signifies a $50\%$ chance that the email is spam, a prediction of $0.75$ signifies a $75\%$  chance that the email is spam, and so on.

