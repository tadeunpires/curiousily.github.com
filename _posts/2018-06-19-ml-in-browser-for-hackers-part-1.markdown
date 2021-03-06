---
layout: post
title:  "Predicting the next Fibonacci number with Linear Regression in TensorFlow.js - Machine Learning in the Browser for Hackers (Part 1)"
date:   2018-06-20 18:31:00 +0300
categories: ["machine-learning"]
excerpt: Ready to build models that run in the browser using only JavaScript and TensorFlow.js? Let's create a Simple Linear Regression model and pick up some core concepts along the way - Tensors, Optimizers, and Variables. Your mission, should you choose to accept it, is to create a model that given a number from the Fibonacci sequence predicts the next one while only running in the browser!
---

<div class="center">
    <iframe src="https://www.youtube.com/embed/gDK9Phy8GmY" frameborder="0" allowfullscreen></iframe>
    Let’s walk through this tutorial together!
</div>
<br/>

Welcome to the first (or 0th) part of the series! Together we will explore the limits of what is possible (and probably impossible) with the current state of using JavaScript for Machine Learning in the browser!

The complete source code can be found on [GitHub](https://github.com/curiousily/Machine-Learning-in-the-Browser-for-Hackers/tree/master/1-linear-regression) if you want to follow along. Additionally, I've included a gist showing the complete JavaScript code at the end of the post. [Here is a link to a Live Demo](https://htmlpreview.github.io/?https://github.com/curiousily/Machine-Learning-in-the-Browser-for-Hackers/blob/master/1-linear-regression/index.html), you must open your browser console to see the results.

# What are we trying to do?

Given some random number $x$ that belongs to the Fibonacci sequence, we're going to predict the next one.

If you don't know what a Fibonacci number is (shame on you!) you can [take a look here](https://en.wikipedia.org/wiki/Fibonacci_number). The Fibonacci numbers are all-powerful, some say that they even have magical powers (even though they don't contain the number that is the answer to the ultimate question of life, the universe, and everything).

Simply put, the next Fibonacci number is generated by summing the previous two (after the first two). Here are the first couple of numbers:

$ 1, 1, 2, 3, 5, 8, 13, 21 \ldots$

Can we build a model using TensorFlow.js that predicts the next Fibonacci number?

# What is Linear Regression?

Simple Linear Regression is one of the most basic models you can try out. This model operates under the assumption that there is a linear relationship between a variable $Y$ and an independent variable $X$. Here's the equation that describes the model:

$$Y = aX + b$$

What $a$ - "slope" and $b$ - "intercept" are parameters for our model. And TensorFlow.js is going to help us find their values. That is, find values that best describe our data. Want to see what the process looks like? Here is a picture:

{:.center}
![png]({{site.url}}/assets/ml_in_browser_part_1_files/linear_regression_training.gif)
*Source: [Towards Data Science](https://towardsdatascience.com/linear-regression-the-easier-way-6f941aa471ea)*

What you're observing is the process of fitting the straight line through an example dataset. You can see that the line (that is, the model) starts at some crappy position and after some training (indicated by the increasing number of iterations) it is right in the middle of our data. Note that there are points that are far away from our line. Is this an issue? That's a topic for another discussion.

But what is Linear Regression? We only learned about Simple Linear Regression so far. Linear Regression (or Multiple Linear Regression) has two or more independent variables (think $X$s). That's all folks!

Okay, we have a plan now, we will create a Linear Regression model that can predict the next Fibonacci number. For that, we're going to need a powerful tool.

# What is TensorFlow.js

> A JavaScript library for training and deploying ML models in the browser and on Node.js

[TensorFlow.js](https://js.tensorflow.org/) makes it super easy to get started with Machine Learning. But why? 

1) No need to know any of the fancy pantsy languages like C++, Python or Java, just JavaScript. But hey you probably know some already! 

2) Do you know where you can run JavaScript? That's right pretty much everywhere - phones, tablets, PCs, Macs and your grandma bike (just checking if you're still with me). The best part is that you don't need to install anything - simply include a JavaScript file. And yes, you guessed it, that means you can train your models on Android and iOS phones too (PyTorch I am looking at you)!

## Installing TensorFlow.js

Installing TensorFlow.js is simple. Being a JavaScript library, we have to include it in the `<head>` tag of an HTML page. Open up your favorite text editor (as long it is VIM) and create the following HTML file:

```html
<!DOCTYPE html>
<html>
    <head>
        <script src="https://cdn.jsdelivr.net/npm/@tensorflow/tfjs@0.11.6"></script>
        <script src="index.js"></script>
    </head>

    <body>
    </body>
</html>
```

At the time of this writing, the most recent version of TensorFlow.js is [*0.11.6*](https://github.com/tensorflow/tfjs/releases/tag/v0.11.6). If a newer version is available, when you read these lines, you should consider updating it and fiddling with it if nothing works.

All of our JavaScript is going to be written in `index.js`. Now might be a good time to create it.

#### What is a Tensor?

Tensors are just multidimensional arrays of numbers. They're the main building block you're going to use when building models in TensorFlow (still wondering about the name of the library?). When performing operations, on them, you get new Tensors (that is, they are immutable). Creating a Tensor is easy:

```js
const myFirstTensor = tf.scalar(42)
```

Let's try to print the value of our Tensor:

```js
console.log(myFirstTensor)
```

```
e {isDisposedInternal: false, size: 1, shape: Array(0), dtype: "float32", strides: Array(0), …}
```

Uh, what the crap is that? Welcome to the world of TensorFlow, where everything is just a bit off. It wouldn't be fun if `console.log()` was working as expected, would it?

Ok, here is the solution to our problem:

```js
myFirstTensor.print()
```

```
Tensor
    42
```

And now you know the most well-kept secret of TensorFlow - how to print some values.

Here is how we can create a vector (1d Tensor):

```js
const oneDimTensor = tf.tensor1d([1, 2, 3])
```

Let's practice our printing superpowers:

```js
oneDimTensor.print()
```

```
Tensor
    [1, 2, 3]
```

Depending on your needs you can use other helper functions to create Tensors: `tf.tensor2d()`, `tf.tensor3d()` and `tf.tensor4d()`.

# Predicting the next Fibonacci number

Now that we know what Tensors are we can start building our model. First up - creating the training data.

## Preparing the training data

Remember, our job is to find the values of the parameters $a$ and $b$. Thankfully, we won't have to do this by hand - TensorFlow.js can help us! To do that, we need training data, preferably lots of it. Our model is going to use that data find good values for $a$ and $b$. In our case, "good values" are values that best predict the next Fibonacci number.

Ok, how to create the data? Fortunately, the sequence of Fibonacci numbers $F_n$ can be generated using the following recurrence:

$$F_n = F_{n - 1} + F_{n - 2}$$

We can use that to create an iterative version (why not recursive?) in JavaScript that generates it:

```js

function fibonacci(num){
    var a = 1, b = 0, temp;
    var seq = []

    while (num > 0){
        temp = a;
        a = a + b;
        b = temp;
        seq.push(b)
        num--;
    }

    return seq;
}
```

For our training set, we're going to generate the first 100 Fibonacci numbers:

```js
const fibs = fibonacci(100)
```

Our independent variable $X$ to be a 1D Tensor of the first 99 numbers in that sequence:

```js
const xs = tf.tensor1d(fibs.slice(0, fibs.length - 1))
```

We can obtain our $Y$ by dropping the first number in the sequence, that is the 1D Tensor that contains the values our model is going to predict:

```js
const ys = tf.tensor1d(fibs.slice(1))
```

We can take a glimpse at what our training looks like, by putting the first five values into a table:

| x | y |
|---|---|
| 1 | 1 |
| 1 | 2 |
| 2 | 3 |
| 3 | 5 |
| 5 | 8 |

Basically, we obtain $Y$ by shifting the $X$ values by one to the right.

Using the data as is might not produce a good model, in fact - it won't. We're going to apply a simple hack that will transform (scale) our data:

```js
const xmin = xs.min();
const xmax = xs.max();
const xrange = xmax.sub(xmin);

function norm(x) {
    return x.sub(xmin).div(xrange);
}

xsNorm = norm(xs)
ysNorm = norm(ys)
```

Here are the first couple of $X$ and $Y$ values after applying the operation:

```
X [0, 0,                     4.567816146623912e-21, 9.135632293247824e-21]
Y [0, 4.567816146623912e-21, 9.135632293247824e-21, 1.8271264586495648e-20]
```

Basically, we scaled down the $X$ values in the interval [0, 1]. We almost did the same thing to $Y$ values, except we used the max and min values from the $X$. Can you guess why?

## Building our model

With our data ready to go it is time to create our model. The Simple Linear Regression is well... simple to create. Even in TensorFlow, that is. Let's have a look at the equation describing the model once again:

$$Y = aX + b$$

First, we must initialize our model parameters $a$ and $b$:

```js
const a = tf.variable(tf.scalar(Math.random()))
const b = tf.variable(tf.scalar(Math.random()))
```

What the *Variable* wrapper does it allows the value that it holds to (surprisingly) change. The necessity for using *Variable* comes from the fact that we want to find better values for our parameters $a$ and $b$ and thus change them. Do you know why we initialize $a$ and $b$ with random numbers instead of 0?

Finally, it is time to write our model in TensorFlow.js:

```js
function predict(x) {
    return tf.tidy(() => {
        return a.mul(x).add(b)
    });
}
```

We're going to skip what [`tf.tidy()`](https://js.tensorflow.org/api/latest/index.html#tidy) does and discuss the important part:

```js
a.mul(x).add(b)
```

Here we just follow the formula from the above: multiply $a$ with $X$ and add $b$ to the result. I know, it is a strange syntax for such a simple thing to do, but hey you chose to learn TensorFlow.js!

## Training

Roughly speaking, the training of our model consists of showing data to our model, obtaining a prediction from it, evaluating how good that prediction is and feeding back that information in the training process.

### Loss function

Evaluating the goodness of the prediction is done using a loss (or error) function. We're going to use a rather simple one - Mean Squared Error (MSE):

$$\text{MSE} = \frac{1}{n}\sum_{i=1}^{n}(Y_i - \hat{Y_i}) ^ 2$$

where $n$ is the number of $Y$ values, $Y_i$ is the $i$-th value in $Y$ and $\hat{Y_i}$ is the prediction for $X_i$ from our model.

Roughly speaking, MSE measures the average squared difference between the predicted and real values. The result obtained from MSE is always non-negative. Results closer to 0 indicate that our model can make predictions using the provided data very well.

Here is the MSE formula from above translated into TensorFlow.js lingo:

```js
function loss(predictions, labels) {
    return predictions.sub(labels).square().mean();
}
```

### The training loop

With the `loss()` and `predict()` functions in place you are almost ready to train your first model in TensorFlow.js. The last missing ingredient is the *optimizer*. 

The optimizer is the workhorse behind the process of finding good parameters for your model. The main job of the optimizer is to feedback the signal from the loss function so that your model is (hopefully) continuously improved. Optimization in Machine Learning is an interesting topic that we won't cover in this part of the series, but we will use one right now:

```js
const learningRate = 0.5;
const optimizer = tf.train.sgd(learningRate);
```

We're using `SGD` optimizer with a learning rate set to $0.5$. You can think of the learning rate parameter as a knob that says how fast our model should learn from the data presented to it.  Properly setting this value is still a mystery to some, but we're getting better at it!

The training loop itself is pretty tight:

```js
const numIterations = 10000;
const errors = []

for (let iter = 0; iter < numIterations; iter++) {
    optimizer.minimize(() => {
        const predsYs = predict(xsNorm);
        const e = loss(predsYs, ysNorm);
        errors.push(e.dataSync())
        return e
    });
}
```

First, we set the number of iterations for which our model will see the training data. For each iteration, the error is calculated using the predicted values. The optimizer receives the error and tries to find new parameter values which minimize the error. Additionally, we record all errors for later.

## Making predictions

Did our model learn something? Let's start by checking the first and last value in the errors list:

```js
console.log(errors[0])
console.log(errors[numIterations - 1])
```

```
Float32Array [0.29631567001342773]
Float32Array [2.2385314901642722e-13]
```

> Note that your values might (and probably will) vary but the last value must be pretty close

Would you look at that, initially our error was rather large, but at the end of the training process it is tiny!

Ok, let's pick two numbers from the Fibonacci sequence and ask our model to predict the next one. Here we have them (of the top of my head):

```js
xTest = tf.tensor1d([2, 354224848179262000000])
```

Note that the second number is not in the training data.

Let's unleash our model:

```js
predict(xTest).print()
```

```
Tensor
    [3.2360604, 573146525190143900000]
```

The true values are:

```
    [3, 573147844013817200000]
```

That looks alright for a simple model such as ours. Remember that our task was to learn/find good parameters for our model. The values for $a$ and $b$ found by our optimizer are:

```js
a.print()
b.print()
```

```js
Tensor
    1.6180301904678345

Tensor
    9.997466321465254e-8
```

> Again, your values for $a$ and $b$ might differ, but not widely.

Looks like the important parameter is $a$, while $b$ is pretty much useless.

# Conclusion

You made it! Your first model is successfully running in the browser! It wasn't that hard, wasn't it? Luckily, we’re just getting started. In the next part, we’re going dive deeper in TensorFlow.js and train more complex models.

Please, ask questions or leave feedback in the comments below. Thanks!

P.S. You can find the complete source code on [GitHub](https://github.com/curiousily/Machine-Learning-in-the-Browser-for-Hackers/tree/master/1-linear-regression) or have a look at the gist:

<script src="https://gist.github.com/curiousily/2a8fb08d0e20961256f5a88cb7b8e10d.js"></script>
