Linear Regression is a statistical technique used to find the relationship between variables. In an ML context, linear regression finds the relationship between features and a label.

> **feature** - an input variable to a machine learning model.
> 
> **label** - the target output or correct answer that a model tries to predict.

For example, suppose we want to predict a car's fuel efficiency in miles per gallon based on how heavy the car is, and we have the following dataset:

| **Pounds in 1000s (feature)** | **Miles per gallon (label)** |
| ----------------------------- | ---------------------------- |
| 3.5                           | 18                           |
| 3.69                          | 15                           |
| 3.44                          | 18                           |
| 3.43                          | 16                           |
| 4.34                          | 15                           |
| 4.42                          | 14                           |
| 2.37                          | 24                           |

If we plotted these points, we'd get the following graph:

![](../../assets/Pasted%20image%2020260814165011.png)

**Fig 1.** Car heaviness (in pounds) versus miles per gallon rating. As a car gets heavier, its miles per gallon rating generally decreases.

We could create our own model by drawing a best fit line through the points:

![](../../assets/Pasted%20image%2020260814165051.png)

**Fig 2.** A best fit line drawn through the data from the previous figure.

## Linear regression equation 

In algebraic terms, the model would be defined as $y = mx + b$, where 
- $y$ is miles per gallon - the value we want to predict.
- $m$ is the slope of the line.
- $x$ is pounds - our input value.
- $b$ is the y-intercept.

In ML, we write the equation for a linear regression model as follows:

$$
y' = b + w_1 x_1
$$

where:

-  $y'$ is the predicted label—the output.
-  $b$ is the [**bias**](https://developers.google.com/machine-learning/glossary#bias-math-or-bias-term) of the model. Bias is the same concept as the y-intercept in the algebraic equation for a line. In ML, bias is sometimes referred to as . Bias is a [**parameter**](https://developers.google.com/machine-learning/glossary#parameter) of the model and is calculated during training.
-  $w_1$ is the [**weight**](https://developers.google.com/machine-learning/glossary#weight) of the feature. Weight is the same concept as the slope  in the algebraic equation for a line. Weight is a [**parameter**](https://developers.google.com/machine-learning/glossary#parameter) of the model and is calculated during training.
-  $x_1$ is a [**feature**](https://developers.google.com/machine-learning/glossary#feature)—the input.

During training, the model calculates the weight and bias that produce the best model.

![](../../assets/Pasted%20image%2020260814165547.png)

**Fig 3.** Mathematical representation of a linear model.

In our example, we'd calculate the weight and bias from the line we drew. The bias is 34 (where the line intersects the y-axis), and the weight is –4.6 (the slope of the line). The model would be defined as $y'  = 34 + (-4.6)(x_1)$, and we could use it to make predictions. For instance, using this model, a 4,000-pound car would have a predicted fuel efficiency of 15.6 miles per gallon.

![](../../assets/Pasted%20image%2020260814165706.png)

**Fig 4.** Using the model, a 4000 pound car has a predicted fuel efficiency of 15.6 miles per gallon. 

> $y'$ - 15.6
> $x_1$ - 4000

## Models with multiple features 

Although the example in this section uses only one feature - the heaviness of the car, a more sophisticated model might rely on multiple features, each having a separate weight ($w_1$, $w_2$, etc.). For example, a model that relies on five features would be written as follows:

$$
y' = b + w_1x_1 + w_2x_2 + w_3x_3 + w_4x_4 + w_5x_5
$$

For example, a model that predicts gas mileage could additionally use features such as the following:

- Engine displacement
- Acceleration
- Number of cylinders
- Horsepower

This model would be written as follows:

![](../../assets/Pasted%20image%2020260814170027.png)

**Fig 5.** A model with five features to predict a car's miles per gallon rating.

By graphing a couple of these additional features, we can see that they also have a linear relationship to the label, miles per gallon:

![](../../assets/Pasted%20image%2020260814170148.png)

**Fig 6.** A car's displacement in cubic centimeters and its miles per gallon rating. As a car's engine gets bigger, its miles per gallon rating generally decreases.

![](../../assets/Pasted%20image%2020260814170201.png)

**Fig 7.** A car's acceleration and its miles per gallon rating. As a car's acceleration takes longer, the miles per gallon rating generally increases.

# Loss 

Loss is a numerical metric that describes how wrong a model's predictions are. Loss measures the distance between the model's predictions and the actual labels. It is the difference between the actual value and the predicted value. The goal of training a model is to minimize the loss, reducing it to its lowest possible value.

In the following image, you can visualize loss as arrows drawn from the data points to the model. The arrows show how far the model's predictions are from the actual values.

![](../../assets/Pasted%20image%2020260814170524.png)

**Fig.** Loss is measured from the actual value to the predicted value 

## Distance of loss

In statistics and machine learning, loss measures the difference between the predicted and actual values. Loss focuses on the _distance_ between the values, not the direction (you ignore the negative sign). 

The two most common methods to remove the sign are the following:

- Take the absolute value of the difference
- Square the difference

## Types of loss 

In linear regression, there are five main types of loss, which are outlined in the following table.

| Loss type                                                                                                                  | Definition                                                                                           | Equation                                                       |
| -------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------- | -------------------------------------------------------------- |
| **[L1 loss](https://developers.google.com/machine-learning/glossary#l1-loss)**                                             | The sum of the absolute values of the difference between the predicted values and the actual values. | $\sum \|actual\ value - predicted\ value\|$                    |
| **[Mean absolute error (MAE)](https://developers.google.com/machine-learning/glossary#mean-absolute-error-mae)**           | The average of L1 losses across a set of _N_ examples.                                               | $\frac{1}{N} \sum \|actual\ value - predicted\ value\|$        |
| **[L2 loss](https://developers.google.com/machine-learning/glossary#l2-loss)**                                             | The sum of the squared difference between the predicted values and the actual values.                | $\sum (actual\ value - predicted\ value)^2$                    |
| **[Mean squared error (MSE)](https://developers.google.com/machine-learning/glossary#mean-squared-error-mse)**             | The average of L2 losses across a set of _N_ examples.                                               | $\frac{1}{N} \sum (actual\ value - predicted\ value)^2$        |
| **[Root mean squared error (RMSE)](https://developers.google.com/machine-learning/glossary#root-mean-squared-error-rmse)** | The square root of the mean squared error (MSE).                                                     | $\sqrt{\frac{1}{N} \sum (actual\ value - predicted\ value)^2}$ |

The functional difference between $L_1$ loss and $L_2$ loss (or between MAE/RMSE and MSE) is squaring. When the difference between the prediction and label is large, squaring makes the loss even larger. When the difference is small (less than 1), squaring makes the loss even smaller.

Loss metrics like MAE and RMSE may be preferable to $L_2$ loss or MSE in some use cases because they tend to be more human-interpretable, as they measure error using the same scale as the model's predicted value.

> Note: MAE and RMSE can differ quite widely. MAE represents the average prediction error, whereas RMSE represents the "spread" of the errors, and is more skewed by larger errors.

When processing multiple examples at once, it is recommended averaging the losses across all the examples, whether using MAE, MSE, or RMSE.

## Calculating loss example 

Above, we created the following model to predict the fuel efficiency based on car heaviness:

- Model: $y' = 34 + (-4.6)(x_1)$
	- Weight: -4.6
	- Bias: 34

If the model predicts that a 2,370 pound car gets 23.1 miles per gallon, but it actually gets 24 miles per gallon, we would calculate the $L_2$ loss as follows:

| Value        | Equation                                                            | Result |
| ------------ | ------------------------------------------------------------------- | ------ |
| Prediction   | $$bias + (weight * feature\ value)$$<br>$$34 + (-4.6 \times 2.37)$$ | $23.1$ |
| Actual value | $label$                                                             | $24$   |
| L2 loss      | $$(actual\ value - predicted\ value)^2$$<br><br>$$(24 - 23.1)^2$$   | $0.81$ |
In this example, the $L_2$ loss for that single data point is 0.81.

## Choosing a loss 

Deciding whether to use MAE or MSE can depend on the dataset and the way you want to handle certain predictions. Most feature values in a dataset typically fall within a distinct range. For example, cars are normally between 2000 and 5000 pounds and get between 8 to 50 miles per gallon. An 8,000-pound car, or a car that gets 100 miles per gallon, is outside the typical range and would be considered an [**outlier**](https://developers.google.com/machine-learning/glossary#outliers).

An outlier can also refer to how far off a model's predictions are from the real values. For instance, 3,000 pounds is within the typical car-weight range, and 40 miles per gallon is within the typical fuel-efficiency range. However, a 3,000-pound car that gets 40 miles per gallon would be an outlier in terms of the model's prediction because the model would predict that a 3,000-pound car would get around 20 miles per gallon.

When choosing the best loss function, consider how you want the model to treat outliers. For instance, MSE moves the model more toward the outliers, while MAE doesn't. L2 loss incurs a much higher penalty for an outlier than L1 loss. For example, the following images show a model trained using MAE and a model trained using MSE. The red line represents a fully trained model that will be used to make predictions. The outliers are closer to the model trained with MSE than to the model trained with MAE.

![](../../assets/Pasted%20image%2020260814174004.png)

**Fig .** MSE loss moves the model closer to the outliers 

![](../../assets/Pasted%20image%2020260814174027.png)

**Fig .** MAE loss keeps the model farther from the outliers 

Note the relationship between the model and the data:

- **MSE**. The model is closer to the outliers but further away from most of the other data points.
- **MAE**. The model is further away from the outliers but closer to most of the other data points.

### Choose MSE:

- If you want to heavily penalize large errors.
- If you believe the outliers are important and indicative of true data variance that the model should account for.

### Choose MAE:

- If your dataset has significant outliers that you don't want to overly influence the model. MAE is more robust.
- If you prefer a loss function that is more directly interpretable as the average error magnitude.

# Gradient descent 

Gradient descent is a mathematical technique that iteratively finds the weights and bias that produce the model with lowest loss. It finds the best weight and bias by repeating the following process for a number of user-defined iterations.

The model begins training with randomized weights and biases near zero, and then repeats the following steps:

1. Calculate the loss with the current weight and bias
2. Determine the direction to move the weights and bias that reduce loss 
3. Move the weight and bias values a small amount in the direction that reduce loss 
4. Return to step one and repeat the process until the model can't reduce the loss any further 

The diagram below outlines the iterative steps for gradient descent:

![](../../assets/Pasted%20image%2020260816153437.png)

**Fig .** Iterative process of gradient descent to produce weight and bias that have the lowest loss 

## Math behind gradient descent 

Below is a example dataset:

| **Pounds in 1000s (feature)** | **Miles per gallon (label)** |
| ----------------------------- | ---------------------------- |
| 3.5                           | 18                           |
| 3.69                          | 15                           |
| 3.44                          | 18                           |
| 3.43                          | 16                           |
| 4.34                          | 15                           |
| 4.42                          | 14                           |
| 2.37                          | 24                           |

1. The model starts training by setting the weights and bias to zero:
$weight$ = $0$, $bias$ = $0$

Formula of linear gradient:

$$
y' = b + w_1 x_1
$$

Here, $y'$ is label and $x_1$ is input or feature.

Putting initial values into this equation gives us:

$$
y = 0 + 0(x_1)
$$
So the output (predicted value) is $0$.

2. Calculate MSE loss with the current model parameters:

$$
Loss = \frac{(18-0)^2 + (15-0)^2 + (18-0)^2 + (16-0)^2 + (15-0)^2 + (14-0)^2 + (24-0)^2}{7}
$$

$$
Loss = 303.71
$$

3. Calculate the slop of weight and bias:

It uses partial derivates, which I don't know yet (don't want to learn the again, will update after I have learned them).

The main formula goes like this:
$$
{Weight slope} = \frac{\partial L}{\partial w}
=
\frac{2}{n}\sum_{i=1}^{n} x_i \cdot (y_{pred,i} - y_i)
$$
$$
{Bias slope} = \frac{\partial L}{\partial b}
=
\frac{2}{n}\sum_{i=1}^{n}(y_{pred,i} - y_i)
$$

Our $y_{pred}$ was $0$, so replacing it in above formulas gives:

$$
\frac{\partial L}{\partial w} = -\frac{2}{n}\sum x_i y_i
\qquad
\frac{\partial L}{\partial b} = -\frac{2}{n}\sum y_i
$$
Calculating $\sum(x.y)$:


| **x** | **y** | **x.y** |
| ----- | ----- | ------- |
| 3.5   | 18    | 63.00   |
| 3.69  | 15    | 55.35   |
| 3.44  | 18    | 61.92   |
| 3.43  | 16    | 54.88   |
| 4.34  | 15    | 65.10   |
| 4.42  | 14    | 61.88   |
| 2.37  | 24    | 56.88   |

$\sum(x.y) = 419.01$ 

$\sum(y) = 120$

Putting these values into formulas above we get:

$$
\frac{\partial L}{\partial w}
=
-\frac{2}{7}(419.01)
=
-119.72
$$

$$
\frac{\partial L}{\partial b}
=
-\frac{2}{7}(120)
=
-34.29
$$
Weight slope: $-119.72$
Bias slope: $-34.29$

4. Move a small amount 

Move a small amount in the direction of the negative slope to get the next weight and bias. For now, we'll use "small amount" as $0.01$:

$$
{New \ weight} = old \ weight - (small \ amount * weight \ slope)
$$

$$
{New \ bias} = {old \ bias} - (small \ amount * bias \ slope)
$$

$$
{New \ weight} = 0 - (0.01) * (-119.7)
$$

$$
{New \ bias} = 0 - (0.01) * (-34.3)
$$

$$
{New \ weight} = 1.2
$$

$$
{New \ bias} = 0.34
$$

Use the new weight and bias to calculate the loss and repeat. Completing the process for six iterations, we'd get the following weights, biases, and losses:

| **Iteration** | **Weight** | **Bias** | **Loss (MSE)** |
| ------------- | ---------- | -------- | -------------- |
| 1             | 0          | 0        | 303.71         |
| 2             | 1.20       | 0.34     | 170.84         |
| 3             | 2.05       | 0.59     | 103.17         |
| 4             | 2.66       | 0.78     | 68.70          |
| 5             | 3.09       | 0.91     | 51.13          |
| 6             | 3.40       | 1.01     | 42.17          |

You can see that the loss gets lower with each updated weight and bias. In this example, we stopped after six iterations. In practice, a model trains until it _converges_. When a model converges, additional iterations don't reduce loss more because gradient descent has found the weights and bias that nearly minimize the loss. 

If the model continues to train past convergence, loss begins to fluctuate in small amounts as the model continually updates the parameters around their lowest values. This can make it hard to verify that the model has actually converged. To confirm the model has converged, you'll want to continue training until the loss has stabilized.

## Model convergence and loss curves 

When training a model, you'll often look at a loss curve to determine if the model has converged. The loss curve shows how the loss changes as the model trains. Following figure is what a loss curve typically look like:

![](../../assets/Pasted%20image%2020260816170053.png)

**Fig .** Loss curve showing the model converging around the 1,000th iteration.

You can see that loss dramatically decreases during the first few iterations, then gradually decreases before flattening out around the 1,000th-iteration mark. After 1,000 iterations, we can be mostly certain that the model has converged.

In the following figures, we draw the model at three points during the training process: the beginning, the middle, and the end. Visualizing the model's state at snapshots during the training process solidifies the link between updating the weights and bias, reducing loss, and model convergence.

![](../../assets/Pasted%20image%2020260816170245.png)

**Fig .** Loss curve and snapshot of the model at the beginning of the training process.

![](../../assets/Pasted%20image%2020260816170316.png)

**Fig .** Loss curve and snapshot of model about midway through training.

![](../../assets/Pasted%20image%2020260816170343.png)

**Fig .** Loss curve and snapshot of the model near the end of the training process

### Convergence and convex functions 

The loss functions for linear models always produce a convex surface. As a result of this property, when a linear regression model converges, we know the model has found the weights and bias that produce the lowest loss. 

A convex shape will look something like this:

![](../../assets/Pasted%20image%2020260816170723.png)

**Fig .** Loss surface that shows its convex shape.

A linear model converges when it's found the minimum loss. If we graphed the weights and bias points during gradient descent, the points would look like a ball rolling down a hill, finally stopping at the point where there's no more downward slope. 

![](../../assets/Pasted%20image%2020260816170851.png)

**Fig .** Loss graph showing gradient descent points stopping at the lowest point on the graph.

# Hyperparameters 

_Hyperparameters_ are variables that control different aspects of training. Three common hyperparameters are:
- Learning rate
- Batch size 
- Epochs 

In contrast, _parameters_ are the variables, like the weight and bias, that are part of the model itself. In other words, hyperparameters are values that you control; parameters are values that the model calculates during training.

## Learning rate 

_Learning rate_ is a floating point number you set that influences how quickly the model converges. If the learning rate is too low, the model can take a long time to converge. However, if the learning rate is too high, the model never converges, but instead bounces around the weights and bias that minimize the loss. The goal is to pick a learning rate that's not too high nor too low so that the model converges quickly.

The learning rate determines the magnitude of the changes to make to the weights and bias during each step of the gradient descent process. The model multiplies the gradient by the learning rate to determine the model's parameters (weight and bias values) for the next iteration. In the third step of gradient descent, the "small amount" to move in the direction of negative slope refers to the learning rate.

The difference between the old model parameters and the new model parameters is  proportional to the slope of the loss function. For example, if the slope is large, the model takes a large step. If small, it takes a small step. For example, if the gradient's magnitude is 2.5 and the learning rate is 0.01, then the model will change the parameter by 0.025.

The ideal learning rate helps the model to converge within a reasonable number of iterations. In below figure, the loss curve shows the model significantly improving during the first 20 iterations before beginning to converge:

![](../../assets/Pasted%20image%2020260816210638.png)

**Fig .** Loss graph showing a model trained with a learning rate that converges quickly.

In contrast, a learning rate that's too small can take too many iterations to converge. This model will show very minor improvements after each iteration.

![](../../assets/Pasted%20image%2020260816210736.png)

**Fig .** Loss graph showing a model trained with a small learning rate.

A learning rate that's too large never converges because each iteration either causes the loss to bounce around or continually increase. 

![](../../assets/Pasted%20image%2020260816210935.png)

**Fig .** Loss graph showing a model trained with a learning rate that's too big, where the loss curve fluctuates wildly, going up and down as the iterations increase.

![](../../assets/Pasted%20image%2020260816211012.png)

**Fig .** Loss graph showing a model trained with a learning rate that's too big, where the loss curve drastically increases in later iterations.

## Batch size 

_Batch size_ is a hyperparameter that refers to the number of examples the model processes before updating its weights and bias. You might think that the model should calculate the loss for _every_ example in the dataset before updating the weights and bias. However, when a dataset contains hundreds of thousands or even millions of examples, using the full batch isn't practical.

Two common techniques to get the right gradient on average without needing to look at every example in the dataset are _stochastic gradient descent_ and _min-batch stochastic gradient descent_:

- **Stochastic gradient descent (SGD)**: This uses only a single example (a batch size of one) per iteration. Given enough iterations, SGD works but is very noisy. "Noise" refers to variations during training that cause the loss to increase rather than decrease during an iteration. The term "stochastic" indicates that the one example comprising each batch is chosen at random.

  Notice in the following image how loss slightly fluctuates as the model updates its weights and bias during SGD, which can lead to noise in the loss graph:

![](../../assets/Pasted%20image%2020260816211834.png)

**Fig .** Model trained with stochastic gradient descent showing noise in the loss curve.

- **Mini-batch stochastic gradient descent**: This technique is a compromise between full-batch and SGD. For $N$ number of data points, the batch size can be any number greater than $1$ and less than $N$. The model choose the examples included in each batch at random, averages their gradients, and then updates the weights and bias once per iteration.

  Determining the number of examples for each batch depends on the dataset and the available compute resources. In general, small batch sizes behaves like SGD, and larger batch sizes behaves like full-batch gradient descent.

![](../../assets/Pasted%20image%2020260816212223.png)

**Fig .** Model trained with min-batch SGD

When training a model, you might think that noise is an undesirable characteristic that should be eliminated. However, a certain amount of noise can be a good thing.

## Epochs 

During training, an _epoch_ means that the model has processed every example in the training set _once_. For example, given a training set with $1000$ examples and a mini-batch size of $100$ examples, it will take the model $10$ iterations to complete one epoch.

Training typically requires many epochs. That is, the system needs to process every example in the training set multiple times.

The number of epochs is a hyperparameter you set before the model begins training. In many cases, you'll need to experiment with how many epochs it takes for the model to converge. In general, more epochs produces a better model, but also takes more time to train.

The following table describes how batch size and epochs  relate to the number of times a model updates its parameters.

| **Batch type**                         | **When weights and bias updates occur**                                                                                                                                                                                           |
| -------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Full batch                             | After the model looks at all the examples in the dataset. For instance, if a dataset contains $1,000$ examples and the model trains for $20$ epochs, the model updates the weights and bias $20$ times, once per epoch.           |
| Stochastic gradient descent            | After the model looks at a single example from the dataset. For instance, if a dataset contains $1,000$ examples and trains for $20$ epochs, the model updates the weights and bias $20,000$ times.                               |
| Mini-batch stochastic gradient descent | After the model looks at the examples in each batch. For instance, if a dataset contains $1,000$ examples, and the batch size is $100$, and the model trains for $20$ epochs, the model updates the weights and bias $200$ times. |


