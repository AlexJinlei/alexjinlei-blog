---
title: "Logistic Regression"
date: 2018-06-07T13:39:45-04:00
lastmod: 2018-07-20T12:16:00-04:00
draft: false
categories: ["Technique"]
tags: ["Machine Learning", "Logistic Regression"]
hiddenFromHomePage: false
weight:
---

Logistic regression is an algorithm for linear and binary classification problems. It is a classification, not regression algorithm.

# 1. From odds ratio to sigmoidal logistic function

If $p$ is the probability of a sample being a positive event, then the odds ratio is defined as:
$$\textrm{odds ratio} = \frac{p}{1-p}. \tag{1.1}$$

The logarithm of the odds ratio function is called logit function:
$$z = \textrm{logit}(p) = log(\frac{p}{1-p}). \tag{1.2}$$

Logit function maps the probability $\\{p|p\in[0, 1]\\}$ to the entire real domain $\\{z|z\in(-\infty, +\infty)\\}$.

The inverse of the logit function is logistic function:
$$p = \textrm{logistic}(z) = \frac{1}{1+e^{-z}}. \tag{1.3}$$

Below is the plot of logistic function:
{{% figure class="center" src="/img/logistic-function.pdf" alt="hugo even showcase" title="Logistic Function" %}}

From the definition of the logistic function, we can see that the value of the logistic function is the probability of a sample being a positive event. Logistic function is a sigmoid function. Note that sigmoid function is defined by its shape. A sigmoid function is a mathematical function having a characteristic "S"-shaped curve or sigmoid curve [cited from wiki]. In machine leaning, we usually interchangeably use the two words logistic and sigmoid.

Logistic function has many interesting features. If $z$ is very large, the probablity $p$ will approach to 1, and if $z$ is very small, the probability $p$ will approach to 0. The behavior of logistic function is close to a linear function when $|z|$ is small, and it shows more non-linearity when $|z|$ goes large. Its first order derivatives in near-linear part are much larger than the first order derivatives in the non-linear part. Its first order derivatives approach to 0 when $|z|$ goes large. These features are very important for using logist function as the activation function in neural networks.

# 2. The usage of logistic function in machine leaning

Since the output of the logistic function can be interpreted as the probability of an event being positive, we can use it as the activation function to output probablity. If we use the linear combination of the sample features as the input to logistic function, 

$$z = \textbf{w}^{\textbf{T}} \textbf{x} + b = w_1 x_1 + w_2 x_2 + \cdots + w_m x_m + b.  \tag{2.1}$$

The output will be the probability of the sample $\textbf{x}$ being the positive event, and $\textbf{w}$ is the vector of weights to be learned by minimizing some cost function:

$$p = \frac{1}{1+e^{-(\textbf{w}^{\textbf{T}} \textbf{x}+b)}}.  \tag{2.2}$$

For the sake of simplicity, we use $h_\theta$ to denote the logistic function:

$$h_\theta(\textbf{x}) = \frac{1}{1+e^{-(\textbf{w}^{\textbf{T}} \textbf{x}+b)}}.  \tag{2.3}$$

Remember that, $h_\theta(\textbf{x})$ is the probability of a sample $\textbf{x} = \\{x_1, x_2, ..., x_m\\}$ being a positive event, given the parameters $\theta = \\{\textbf{w}, b \\} = \\{w_1, w_2, ..., w_m, b\\}$.


# 3. Likelihood function and maximum likelihood estimation

Consider a simple statistical model of coin flip. Let's us $p_H$ to denote the probability to get a head. For a perfectly fair coin, $p_H = 0.5$. For one with some basic probablilty knowledge, it's easy to calculate the probability of any head and tail combination. For example, if we flip the coin twice, the probability of getting two heads is:

$$P(HH|p_H=0.5) = p_H*p_H = 0.5^2 = 0.25. \tag{3.1}$$

Expression $(3.1)$ showes the probability of getting two heads in flipping the coin twice, given $p_H = 0.5$. In this experiment, we already know the parameter $p_H$ and the outcome. Let's consider this experiment in another way. If we do not know the value of $p_H$ beforehand, and we got two heads in twice coin flips, how to estimate the value of $p_H$? In the expression $P(HH|p_H=0.5)$, the $p_H$ is a known parameter, and the variable is the outcome $HH$. In our new question, it's more appropriate to take the outcome $HH$ as a known parameter, and consider $p_H$ as a variable. Hence, we define a new function, which is called **likelihood function**:

$$L(\theta|x_1, ... , x_n) = f(x_1, ... , x_n|\theta), \tag{3.2}$$

where $x_1, ... , x_n$ is the distribution of ramdom variables, $f$ is a statistical model, and $\theta$ is the parameter(s) of the statistical model $f$. In definition $(3.2)$, the value of likelihood function is numerically equal to the value of the probability. However, likelihood and probability are two different conceptions. The likelihood function is a function of the parameters of a statistical model, it describes the likelihood of these parameters.

In our coin flip experiment, the likelihood is:

$$L(p_H=0.5|HH) = P(HH|p_H=0.5) = 0.25. \tag{3.3}$$

Expression $(3.2)$ means that for the defined likelihood function, when two heads are observed in a twice coin flip, the **likelihood** of $p_H = 0.5$ is 0.25. Note that, it does not means that the **probability** of $p_H = 0.5$ is 0.25. Here $p_H$ is the variable of the likelihood function $L$. When $p_H$ changes , the value of the function $L$ will also change. The likelihood of $p_H = 0.6$ is:

$$L(p_H=0.6|HH) = P(HH|p_H=0.6) = 0.36. \tag{3.4}$$

Note that, $L(p_H=0.6|HH) = 0.36 > L(p_H=0.5|HH) = 0.5$, we can believe that as an estimation of $p_H$, $p_H=0.6$ is more convincible than $p_H=0.5$, since the $p_H=0.6$ has the higher likelihood.

In this experiment, the likelihood function is:

$$L(p_H=\theta|HH) = P(HH|p_H=\theta) = \theta^2, \tag{3.5}$$

where  $0 \le p_H \le1$. In order to get the best estimation of parameter $p_H$, we maximize the function $L$, and take the corresponding $p_H$ value as our extimation. This method is called **maximum likelihood extimation**. Function $(3.4)$ reaches its maximum when $p_H=1$, so in this experiment, we can believe that $p_H=1$ is the most reasonable estimation.

Let's do another experiment. Let's flip the coin five times. If the outcome is $HHTTH$. In this second experiment, the likelihood function is:

$$L(p_H=\theta|HHTTH) = P(HHTTH|p_H=\theta) = \theta^2(1-\theta)^2\theta = \theta^3(1-\theta)^2, \tag{3.6}$$

where $0 \le p_H \le1$. By calculating the first order derivative of function $(3.6)$, we can obtain the value of $\theta$ that maximize the likelihood function $L(p_H=\theta|HHTTH) = \theta^3(1-\theta)^2$. 

$$\frac{d}{d\theta}(\theta^3(1-\theta)^2) = 3 (1-\theta)^2 \theta^2 - 2 (1-\theta) \theta^3 \tag{3.7}$$

Solve the equation $3 (1-\theta)^2 \theta^2 - 2 (1-\theta) \theta^3 = 0, 0 \le \theta \le1$, we obtain $\theta = 0$, $\theta = 0.6$, or $\theta = 1$. Function $(3.6)$ reaches its maximum at $\theta = 0.6$. So, in the second experiment, $p_H = 0.6$ is the most reasonable estimation of parameter $p_H$.

# 4. Logistic cost function

The logistic cost function is derived from maximum likelihood estimation. Consider a sample set:

$$ \textbf{X} = \begin{pmatrix} x^{(1)}_1 & x^{(2)}_1 & x^{(3)}_1 & \cdots & x^{(m)}_1 \\\ x^{(1)}_2 & x^{(2)}_2 & x^{(3)}_2 & \cdots & x^{(m)}_2 \\\ x^{(1)}_3 & x^{(2)}_3 & x^{(3)}_3 & \cdots & x^{(m)}_3 \\\ \vdots & \vdots & \vdots & \ddots & \vdots \\\ x^{(1)}_n & x^{(2)}_n & x^{(3)}_n & \cdots & x^{(m)}_n \end{pmatrix} \tag{4.1}$$

and its correspoinding labels:

$$ \textbf{Y} = \begin{pmatrix} y^{(1)} & y^{(2)} & y^{(3)} & \cdots & y^{(m)} \end{pmatrix}, \tag{4.2}$$

where each column is one training example. $x^{(i)}_k$ is the $k$-th feature of the $i$-th training example. Using logistic regression model, we want to predict the label of the $i$-th training example. In order to predict the label, we firstly calculate the probability of one example belonging to a specific label. From equation $(2.3)$, we have

$$P(y^{(i)}=1|\textbf{x}^{(i)}, \theta) = h_\theta(\textbf{x}^{(i)}), \tag{4.3}$$

and

$$P(y^{(i)}=0|\textbf{x}^{(i)}, \theta) = 1 - h_\theta(\textbf{x}^{(i)}), \tag{4.4}$$

where $y^{(i)}$ is the label of the $i$-th training example. $y^{(i)} = 1$ means that the training example $\textbf{x}^{(i)}$ is a positive event, and $y^{(i)} = 0$ means that the training example $\textbf{x}^{(i)}$ is a negative event.

Combine expressions $(4.3)$ and $(4.4)$, we have

$$P( y^{(i)} | \textbf{x}^{(i)}, \theta) = (h_0(\textbf{x}^{(i)}))^{y^{(i)}} (1-h_0(\textbf{x}^{(i)}))^{(1-y^{(i)})}. \tag{4.5}$$

(Note: In $h_0$, the subscript $0$ should be $\theta$, but Mathjax cannot render it correctly, I have to use $h_0$ to denote it.) In expression $(4.5)$, when $y^{(i)} = 1$, $(4.5)$ reduces to $(4.3)$, and when $y^{(i)} = 0$, $(4.5)$ reduces to $(4.4)$. 

We can take each traning sample as one independent event, and multiply each $P( y^{(i)} | \textbf{x}^{(i)}, \theta)$ to get the probability that the entire training set happens.

$$P( \textbf{Y} | \textbf{X}, \theta) = \prod_{i=1}^{m} P( y^{(i)} | \textbf{x}^{(i)}, \theta) = (h_0(\textbf{x}^{(i)}))^{y^{(i)}} (1-h_0(\textbf{x}^{(i)}))^{(1-y^{(i)})}\tag{4.6}$$

Since the likelihood function and the probability function are identical, the likelihood function is

$$L(\theta) = \prod_{i=1}^{m}(h_0(\textbf{x}^{(i)}))^{y^{(i)}} (1-h_0(\textbf{x}^{(i)}))^{(1-y^{(i)})}\tag{4.7}$$

Keeping the methodology of maximum likelihood estimation in mind, since the events in our training set happened, we want to maximize the probability that it happens. We will find the parameter $\theta$ that maximizes the likelihood function $L$. 

In logistic regression model, we want to predict the label $\hat{y}^{(i)}$ from the input $\textbf{x}^{(i)}$. The predicted label is calculated by Equation $(2.3)$. The parameters $\theta = \\{\textbf{w}, b \\} = \\{w_1, w_2, ..., w_m, b\\}$ are to be learned. Rewriting $h_0(\textbf{x}^{(i)})$ to $\hat{y}^{(i)}$, the likelihood function is

$$L(\theta) = \prod_{i=1}^{m}({\hat{y}^{(i)}})^{y^{(i)}}({1-\hat{y}^{(i)}})^{(1-y^{(i)})}\tag{4.8}$$

In practice, the calculation will be simpler if we adopt the logarithm form of the loss function, the log-likelihood function:

$$l(\theta) = log(L(\theta)) = \sum_{i=1}^{m}[y^{(i)}log(\hat{y}^{(i)}) + (1-y^{(i)})log(1-\hat{y}^{(i)})]\tag{4.9}$$

By adding a minus sign, we can get a cost function $-l(\theta)$ which can be minimized using gradient descent. Since we have $m$ training sample, we can normalize the cost function by multiplying a factor $\frac{1}{m}$. The overall cost function can be write as:

$$J(\theta) = -\frac{1}{m}l(\theta) = -\frac{1}{m}\sum_{i=1}^{m}[y^{(i)}log(\hat{y}^{(i)}) + (1-y^{(i)})log(1-\hat{y}^{(i)})] \tag{4.10}$$

Since we've obtained the cost function, we are ready to let the model learn parameters by minimizing this cost function. Because the cost function is derived from logistic function, the model that uses this cost function is called logistic model.

(A loss function measures the discrepancy between the prediction $\hat{y}^{(i)}$ and the desired output $y^{(i)}$. In other words, the loss function computes the error for a single training example. The cost function is the average of the loss function of the entire training set.) -- From Andrew Ng's lecture notes.











