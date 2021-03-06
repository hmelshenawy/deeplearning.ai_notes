# Specialisation 2: Improving Deep Neural Networks
<!-- TOC -->

- [Specialisation 2: Improving Deep Neural Networks](#specialisation-2-improving-deep-neural-networks)
    - [WEEK 1](#week-1)
        - [Normalizing and initialisation](#normalizing-and-initialisation)
        - [Variance / Bias](#variance--bias)
        - [Regularization](#regularization)
            - [L1 & L2](#l1--l2)
            - [Dropout](#dropout)
        - [Gradient checking](#gradient-checking)
    - [WEEK 2](#week-2)
        - [Mini-batch](#mini-batch)
        - [Exponentially weighted averages](#exponentially-weighted-averages)
        - [Momentum](#momentum)
        - [RMS Prop](#rms-prop)
        - [Adam](#adam)
        - [Learning Rate Decay](#learning-rate-decay)
        - [Local optima](#local-optima)
    - [WEEK 3](#week-3)
        - [Hyperparameter tuning](#hyperparameter-tuning)
        - [Batch norm](#batch-norm)
        - [Multi-class classification](#multi-class-classification)
            - [Softmax](#softmax)
            - [Softmax with a varying temperature (source)](#softmax-with-a-varying-temperature-source)

<!-- /TOC -->

## WEEK 1

### Normalizing and initialisation

We can normelize our example by subtracting the mean and scaling but the variance, by we must apply the same mean and variance to the test set. This helps the gradient descent to find the optima much quickly with less steps.


**Zero initialization:**

In general, initializing all the weights to zero results in the network failing to break symmetry. This means that every neuron in each layer will learn the same thing, so we might as well be training a neural network with n[l]=1 for every layer, and the network is no more powerful than a linear classifier such as logistic regression. It is however okay to initialize the biases <img src="https://latex.codecogs.com/gif.latex?$b^{[l]}$" title="$b^{[l]}$" /> to zeros. Symmetry is still broken so long as the weigths <img src="https://latex.codecogs.com/gif.latex?$W^{[l]}$" title="$W^{[l]}$" /> are initialized randomly.

**Random initialization:**

- The cost starts very high. This is because with large random-valued weights, the last activation (sigmoid) outputs results that are very close to 0 or 1 for some examples, and when it gets that example wrong it incurs a very high loss for that example. Indeed, when <img src="https://latex.codecogs.com/gif.latex?$\log(\hat{y})&space;=&space;\log(0)$" title="$\log(\hat{y}) = \log(0)$" />, the loss goes to infinity.
- Poor initialization can lead to vanishing/exploding gradients, which also slows down the optimization algorithm.
- If we train the network longer we will see better results, but initializing with overly large random numbers slows down the optimization.

**A better initialisation:** given that we'd like to have the activations Z with zero mean and unit variance, for that, it is better to initialize the weights of the network from a guassian distribution and multiply by <img src="https://latex.codecogs.com/gif.latex?\frac{1}{N(l-1)}" title="\frac{1}{N(l-1)}" /> to ensure unit variancein case of sgmoid non linearities, <img src="https://latex.codecogs.com/gif.latex?\frac{2}{N(l-1)}" title="\frac{2}{N(l-1)}" />  works better for Relu, for tanh we use <img src="https://latex.codecogs.com/gif.latex?sqrt(\frac{1}{N(l-1)})" title="sqrt(\frac{1}{N(l-1)})" />, other version is <img src="https://latex.codecogs.com/gif.latex?sqrt(\frac{1}{N(l-1)&space;&plus;&space;N(l)})" title="sqrt(\frac{1}{N(l-1) + N(l)})" />.

In summary:
- Initializing weights to very large random values does not work well. 
- Hopefully intializing with small random values does better and it is used to get the variance equal to one. The important question is: how small should be these random values be? We have different possibilities:
    - Xavier initialization: uses a scaling factor for the weights <img src="https://latex.codecogs.com/gif.latex?$W^{[l]}$" title="$W^{[l]}$" /> of `sqrt(1./layers_dims[l-1])`.
    - He initialization uses `sqrt(2./layers_dims[l-1])`. Works well for networks with ReLU activations.

### Variance / Bias

High variance = overfitting, High bias = underfitting,

If the error in the train set and dev set are both high, then we have both high bias and high variance. This case is very possible in high dimensions, where we have high bias in some regions and high variance in some other regions.

To get rid of high bias, we can train a bigger network until the bias is reduced, if we have high variance, we can try getting more data, applying some regularization, or ytring new and more restrictive architectures.

### Regularization

#### L1 & L2

Using L1 regularization will make w sparse. And what that means is that the matrix w will have a lot of zeros in it. This can help with compressing the model, because a good set of parameters are zero, and we need less memory to store the model. But in practice this does not help that much,

<p align="center"> <img src="figures/reg1.png"> </p>

For Neural network, L2 regulariztion is the Frobenius Norm, also called weight decay (weight decay = A regularization technique (such as L2 regularization) that results in gradient descent shrinking the weights on every iteration).

Regularization helps us train a simpler network, and for tanh per example, it helpes us stay in the linear region (a simpler network not prone to overfitting, like a linear regression).

L2-regularization relies on the assumption that a model with small weights is simpler than a model with large weights. Thus, by penalizing the square values of the weights in the cost function we drive all the weights to smaller values. It becomes too costly for the cost to have large weights! This leads to a smoother model in which the output changes more slowly as the input changes.

When adding regularization, we'll have cost function and back prop by adding the regularization term's gradient (<img src="https://latex.codecogs.com/gif.latex?\frac{d}{dW}&space;(&space;\frac{1}{2}\frac{\lambda}{m}&space;W^2)&space;=&space;\frac{\lambda}{m}&space;W" title="\frac{d}{dW} ( \frac{1}{2}\frac{\lambda}{m} W^2) = \frac{\lambda}{m} W" />).

**Side note [source](https://www.fast.ai/2018/07/02/adam-weight-decay/):**

These types of regulizers are nearly always implemented by adding `lambda * w` to the gradients, rather than actually changing the loss function, given that we don’t want to add more computations by modifying the loss when there is an easier way.

So as a distinction:
* Weight decay: `final_loss = loss + wd * all_weights.pow(2).sum() / 2`,
* L2 regularization, the one implemented in Deep learning frameworks: `w = w - lr * w.grad - lr * lambda * w` 

L2 regularization and weight decay are the same when using vanilla SGD, but as soon as we add momentum, or use a more sophisticated optimizer like Adam, L2  become different than weight decay. For momentum, using L2 regularization consists in adding `lambda * w` to the gradients (as we saw earlier) but the gradients aren’t subtracted from the weights directly. First we compute a moving average:
```
moving_avg = alpha * moving_avg + (1-alpha) * (w.grad + lambda * w)
```
and it’s this moving average that will be multiplied by the learning rate and subtracted from w. So the part linked to the regularization that will be taken from w is `lr* (1-alpha) * lambda * w` plus a combination of the previous weights that were already in `moving_avg`.

On the other hand, weight decay’s update will look like:
```
moving_avg = alpha * moving_avg + (1-alpha) * w.grad 
w = w - lr * moving_avg - lr * lambda * w
```
We can see that the part subtracted from w linked to regularization isn’t the same in the two methods. When using the Adam optimizer, it gets even more different: in the case of L2 regularization we add this `wd*w` to the gradients then compute a moving average of the gradients and their squares before using both of them for the update. Whereas the weight decay method simply consists in doing the update, then subtract to each weight.

#### Dropout

It randomly shuts down some neurons in each iteration from one layer to the next. At each iteration, we shut down (set to zero) each neuron of the previous layer with probability `1−keep_prob` or keep it with probability `keep_prob`. When we shut some neurons down, we actually modify the model. The idea behind dropout is that at each iteration, we train a different model that uses only a subset of the neurons. With dropout, the neurons thus become less sensitive to the activation of one other specific neuron, because that other neuron might be shut down at any time.

Implementing Dropout, called inverted dropout, create a matix `Dl` for a given layer `l`, where each element corresponds to an activation in layer l of the matrix of activations `Al`, each element is random values, if this random values is less than `keep_prob`, then if will be assgined 1, we multiply the two and then devide by the keep_prob to have the same original scale. One down side of dropout, it that the cost function becomes not defined.

For backpropagation: We previously shut down some neurons during forward propagation, by applying a mask D to `A1`. In backpropagation, we will have to shut down the same neurons, by re-applying the same mask D to `dA1`.

A common mistake when using dropout is to use it both in training and testing. we should use dropout (randomly eliminate nodes) only in training, and in inference we multiply the activations by the `1/keep_prob` factor.

Other reularization methods can be used, such as data augmentation, or early stoping.

**Montecarlo dropout**

For normal dropout, at test time the prediction is deterministic. Without other source of randomness, given one test data point, the model will always predict the same label or value.

For Monte Carlo dropout, the dropout is applied at both training and test time. At test time, the prediction is no longer deterministic, but depending on which nodes/links we randomly choose to keep. Therefore, given a same datapoint, the model could predict different values each time.

So the primary goal of Monte Carlo dropout is to generate random predictions and interpret them as samples from a probabilistic distribution. In the authors' words, they call it Bayesian interpretation.

Example: suppose we've trained an dog/cat image classifier with Monte Carlo dropout. If we feed a same image to the classifier again and again, the classifier may be predicting dog 70% of the times while predicting cat 30% of the time. Therefore we can interpret the result in a probabilistic way: with 70% probability, this image shows a dog.

### Gradient checking

To do gradient checking, we take all the parameters W and b and reshape them into one big vecotor theta, same for the derivatives dW, db.

<p align="center"> <img src="figures/grad_check.png"> </p>

Because forward propagation is relatively easy to implement, we can be confident we got that right, and so we're almost 100% sure that we're computing the cost J correctly. Thus, we can use the code for computing J to verify the code for computing <img src="https://latex.codecogs.com/gif.latex?\frac{\partial&space;J}{\partial&space;\theta}" title="\frac{\partial J}{\partial \theta}" />. using the definition of derivatives :

<img src="https://latex.codecogs.com/gif.latex?$$&space;grad_{approx}&space;=&space;\frac{\partial&space;J}{\partial&space;\theta}&space;=&space;\lim_{\varepsilon&space;\to&space;0}&space;\frac{J(\theta&space;&plus;&space;\varepsilon)&space;-&space;J(\theta&space;-&space;\varepsilon)}{2&space;\varepsilon}$$" title="$$ grad_{approx} = \frac{\partial J}{\partial \theta} = \lim_{\varepsilon \to 0} \frac{J(\theta + \varepsilon) - J(\theta - \varepsilon)}{2 \varepsilon}$$" />

We know the following:

- <img src="https://latex.codecogs.com/gif.latex?\frac{\partial&space;J}{\partial&space;\theta}" title="\frac{\partial J}{\partial \theta}" /> is what we want to make sure we're computing correctly.
- We can compute <img src="https://latex.codecogs.com/gif.latex?$J(\theta&space;&plus;&space;\varepsilon)$" title="$J(\theta + \varepsilon)$" /> and <img src="https://latex.codecogs.com/gif.latex?$J(\theta&space;-&space;\varepsilon)$" title="$J(\theta - \varepsilon)$" /> (in the case that $\theta$ is a real number), since we have a correct implementation of J.

Finally, we compute the relative difference between "gradapprox" and the "grad" using the following formula:

<img src="https://latex.codecogs.com/gif.latex?$$&space;difference&space;=&space;\frac&space;{\mid\mid&space;grad&space;-&space;grad_{approx}&space;\mid\mid_2}{\mid\mid&space;grad&space;\mid\mid_2&space;&plus;&space;\mid\mid&space;grad_{approx}&space;\mid\mid_2}$$" title="$$ difference = \frac {\mid\mid grad - grad_{approx} \mid\mid_2}{\mid\mid grad \mid\mid_2 + \mid\mid grad_{approx} \mid\mid_2}$$" />

**Note** 

- Gradient Checking is slow! Approximating the gradient with <img src="https://latex.codecogs.com/gif.latex?$$\frac{\partial&space;J}{\partial&space;\theta}&space;\approx&space;\frac{J(\theta&space;&plus;&space;\varepsilon)&space;-&space;J(\theta&space;-&space;\varepsilon)}{2&space;\varepsilon}$$" title="$$\frac{\partial J}{\partial \theta} \approx \frac{J(\theta + \varepsilon) - J(\theta - \varepsilon)}{2 \varepsilon}$$" /> is computationally costly. For this reason, we don't run gradient checking at every iteration during training. Just a few times to check if the gradient is correct.
- Gradient Checking, at least as we've presented it, doesn't work with dropout. We would usually run the gradient check algorithm without dropout to make sure the backprop is correct, then add dropout.

___

## WEEK 2

### Mini-batch

Mini-batch: instead of going through the whole training set before doing gradient descent and updating the parameters, we can choose only a portion of the training set, a mini batch, and for each mini batch we update the parameters, the updates might be noisy but converge much faster. The cost function might not go down with each iteration, given that some mini batches contain harder examples, yielding a greater cost function.

One Epoch is one pass through the training set.

Given that the size of the mini batch = m;
- If m = size of the whole dataset, then we're talking about regular gradient descent, which takes a lot of time to converge, but with each iteration the cost goes down.
- If size of the mini batch = 1, then we're talking about stochastic gradient descent, in SGD, we use only 1 training example before updating the gradients.

When the training set is large, SGD can be faster. But the parameters will "oscillate" toward the minimum rather than converge smoothly. But we're not using the speed coming from vectorization, the best choice is in between. Typical mini batches are 32, 64, 128, 256, depending on the size of the dataset and how much examples we can store in GPU/CPU memory.

There are two steps to apply Mini-Batch Gradient descent:

1. **Shuffle**: Create a shuffled version of the training set (X, Y). Note that the random shuffling is done synchronously between X and Y. Such that after the shuffling the ith column of X is the example corresponding to the ith label in Y.
2. **Partition**: Partition the shuffled (X, Y) into mini-batches of size `mini_batch_size`. Note that the number of training examples is not always divisible by `mini_batch_size`. So the last mini batch might be smaller.

### Exponentially weighted averages

With EWA instead of taking the last value in a sequences of values (like when doing SGD and calculating gradients), we calculate a new values that also depends on the previous values we've gotten.

<img src="https://latex.codecogs.com/gif.latex?$V_t&space;=&space;\beta&space;V_{t-1}&space;&plus;&space;(1-&space;\beta)&space;\theta_{t}$" title="$V_t = \beta V_{t-1} + (1- \beta) \theta_{t}$" />

Given that for each new value, we're averaging over all the past values, we can say that when the past value becomes 1/e of its original values, it will decay rapidly and will not have any affect, so <img src="https://latex.codecogs.com/gif.latex?\beta&space;^&space;X&space;=&space;\frac{1}{e}" title="\beta ^ X = \frac{1}{e}" /> and it turns out that <img src="https://latex.codecogs.com/gif.latex?X&space;\approx&space;\frac{1}{1-\beta}" title="X \approx \frac{1}{1-\beta}" />.

So depending on the value of Beta, the shape of the weighted average have a different latency. For averaging temperatures, when Beta 0.98 (averagin over <img src="https://latex.codecogs.com/gif.latex?\frac{1}{1-0.98}&space;=&space;50" title="\frac{1}{1-0.98} = 50" /> days) then it's giving a lot of weight to the previous value and a much smaller weight, just 0.02, to whatever we're seeing right now. So when temperature goes up or down, there's exponentially weighted average that adapts more slowly when beta is so large or more rapidly if it is smaller.

<p align="center"> <img src="figures/weighted_avg.png" width="400"> </p>

When we start, we have V0 = 0, so the first weighted averages are not corresponding to the correct values, and it takes a number of iterations to get to the correct averages, for that we can apply bias correction, <img src="https://latex.codecogs.com/gif.latex?\frac{V_t}{1-&space;\beta^t}" title="\frac{V_t}{1- \beta^t}" />. As seen also later in RNN assignement, we can choose a value to assign to the loss V0, like for character level generation : `-np.log(1.0/vocab_size)*seq_length` and then use this first values to smooth the loss with exponenetially weighted average: `loss * 0.999 + cur_loss * 0.001`.

Side note [source](https://sgugger.github.io/how-do-you-find-a-good-learning-rate.html):

How did we get the value of bias correction:

<p align="center"> <img src="figures/smoothed_loss.png" width="700"> </p>

### Momentum
Because mini-batch gradient descent makes a parameter update after seeing just a subset of examples, the direction of the update has some variance, and so the path taken by mini-batch gradient descent will "oscillate" toward convergence. Using momentum can reduce these oscillations. 

Momentum takes into account the past gradients to smooth out the update. We will store the 'direction' of the previous gradients in the variable $v$. Formally, this will be the exponentially weighted average of the gradient on previous steps. We can also think of $v$ as the "velocity" of a ball rolling downhill, building up speed (and momentum) according to the direction of the gradient/slope of the hill. 
On each iteration we compute dW and db, and then:

* <img src="https://latex.codecogs.com/gif.latex?$V_{dW}&space;=&space;\beta&space;V_{dW}&space;&plus;&space;(1-&space;\beta)&space;dW_{t}$" title="$V_{dW} = \beta V_{dW} + (1- \beta) dW_{t}$" />
* <img src="https://latex.codecogs.com/gif.latex?$V_{db}&space;=&space;\beta&space;V_{db}&space;&plus;&space;(1-&space;\beta)&space;db_{t}$" title="$V_{db} = \beta V_{db} + (1- \beta) db_{t}$" />
* Update : <img src="https://latex.codecogs.com/gif.latex?$W&space;=&space;W&space;-&space;\alpha&space;V_{dW}$" title="$W = W - \alpha V_{dW}$" />

In this case, we can make these analogies:
* beta: friction,
* db, dW: acceleration,
* Vdb , Vdw: velocity

This allows us to depen the oscillation and only take step in the direction where the gradients does not cancel each other from one iteration to an other.

**How to choose β ?**

- The larger the momentum β is, the smoother the update because the more we take the past gradients into account. But if β is too big, it could also smooth out the updates too much. 
- Common values for β range from 0.8 to 0.999. β = 0.9 is often a reasonable default. 
- Tuning the optimal β for our model might need trying several values to see what works best in term of reducing the value of the cost function J. 

### RMS Prop

With RMS Propo, we hope that, to accelerate the steps on the direction with small oscillations (thus small gradients) and slowing the steps on the direction with big oscillations, we can divide by the weighted averages, and the bigger the averages are the smaller the updates in the direction with the big oscillations, and to avoid the cancellation of opposite oscillations (like in momentum) we square dW and db.

On each iteration we compute dW and db, and then:

* <img src="https://latex.codecogs.com/gif.latex?$S_{dW}&space;=&space;\beta&space;S_{dW}&space;&plus;&space;(1-&space;\beta)&space;dW_{t}^2$" title="$S_{dW} = \beta S_{dW} + (1- \beta) dW_{t}^2$" />
* <img src="https://latex.codecogs.com/gif.latex?$S_{db}&space;=&space;\beta&space;S_{db}&space;&plus;&space;(1-&space;\beta)&space;db_{t}^2$" title="$S_{db} = \beta S_{db} + (1- \beta) db_{t}^2$" />
* Update : <img src="https://latex.codecogs.com/gif.latex?W&space;=&space;W&space;-&space;\alpha&space;\frac{dW}{\sqrt{S_dW}}" title="W = W - \alpha \frac{dW}{\sqrt{S_dW}}" />


### Adam

Combines both momentum and RMS prop, with three hyperparameters (learning rate, beta1 and beta2)

**How does Adam work?**
1. It calculates an exponentially weighted average of past gradients, and stores it in variables v (before bias correction) and <img src="https://latex.codecogs.com/gif.latex?$v^{corrected}$" title="$v^{corrected}$" /> (with bias correction). 
2. It calculates an exponentially weighted average of the squares of the past gradients, and  stores it in variables $s$ (before bias correction) and $s^{corrected}$ (with bias correction). 
3. It updates parameters in a direction based on combining information from "1" and "2".

<p align="center"> <img src="figures/adam.png"> </p>

where:
- t counts the number of steps taken of Adam 
- L is the number of layers
- β1 and β2 are hyperparameters that control the two exponentially weighted averages. 
- α is the learning rate
- Ɛ is a very small number to avoid dividing by zero

Momentum usually helps, but given a small learning rate and a simplistic dataset, its impact is almost negligeable. Adam on the other hand, clearly outperforms mini-batch gradient descent and Momentum.

Some advantages of Adam include:
- Relatively low memory requirements (though higher than gradient descent and gradient descent with momentum) 
- Usually works well even with little tuning of hyperparameters (except α)

### Learning Rate Decay

By reducing the learning rate, we can afford to take very big state early in the training, and then reduce LR and take small steps to avoid big oscillations later on, some very common LR decay methods are:

<img src="https://latex.codecogs.com/gif.latex?1.&space;\alpha&space;=&space;\frac{1}{&space;1&space;&plus;&space;Decay\_rate&space;&plus;&space;Epoch\_num}&space;\alpha_0&space;\\&space;2.&space;\alpha&space;=&space;0.95^{Epoch\_num}&space;\alpha_0&space;\\&space;3.&space;\alpha&space;=&space;\frac{k}{\sqrt{Epoch\_num}}&space;\alpha_0&space;\\" title="1. \alpha = \frac{1}{ 1 + Decay\_rate + Epoch\_num} \alpha_0 \\ 2. \alpha = 0.95^{Epoch\_num} \alpha_0 \\ 3. \alpha = \frac{k}{\sqrt{Epoch\_num}} \alpha_0 \\" />

### Local optima

In very high dimensions, the problem of local optimas is not very common, to have a local optima, we need to have a convex or a concave like function in each direction, and for very high dimensional spaces, this is very unlikely, instead, most points of zero gradient in a cost function are saddle points.

The real problem in high dimensionnal space are plateaus, where the gradient equals zeros for an extended period of time.

___

## WEEK 3

### Hyperparameter tuning
The comon hyperparameters are: learning rate, beta (momentum) or beta1 and beta2 for adam, #layers, #hidden units, learning rate decay, mini-batch size.

We can use grid search to find, the correct combination of hyperparameter, but instead of trying all the possible pairs, it is better to sample randomly, allowing us to sample over the space of the hypterparameters more efficiently.

But to sample in efficient manner, sometimes we must perform the sampling over the log scale, for example, if we want to find the correct learning rate between 0.0001 and 1, if we sample randomly, 90% of the values are between 0.1 and 1, so it is better to sample in the log scale, given that <img src="https://latex.codecogs.com/gif.latex?$0.0001&space;=&space;10^{-4}$" title="$0.0001 = 10^{-4}$" />, and <img src="https://latex.codecogs.com/gif.latex?$1&space;=&space;10^0$" title="$1 = 10^0$" /> we can sample a random number X between -4 and 0, and then use to obtain a learning rate <img src="https://latex.codecogs.com/gif.latex?$10^X$" title="$10^X$" />. We can use the same method to sample values for Beta (momentum), for Beta between 0.9 and 0.999, we can sample (1 - Beta) which is between 0.1 (<img src="https://latex.codecogs.com/gif.latex?$10^{-1}$" title="$10^{-1}$" />) and 0.0001 (<img src="https://latex.codecogs.com/gif.latex?$10^{-4}$" title="$10^{-4}$" />). This method will help us sample more densly in the regime close to one, given that Beta is very sensitive when it is close to 1 (for 0.999 we average over 1000 examples and for 0.9995 we average over 2000 examples).

There is two schools of thoughts, using only one model when we don't have enough computation, each training iteration we can try a new set of parameters (Babysitting one model), the second approach is training many models in parallel each one with a different set of parameters and compare them.

### Batch norm

Batch normalization makes the hyperparameter search problem much easier and neural network much more robust. This done by normalizing not only the inputs, but also the intermediate activations to have a mean of zero and a variance of one.

Given some intermediate values in the neural net, Z1, ...  , Zn, we compute the mean the variance of these activations, and then we normalize and we scale the activation using the caculated mean and variance, and finaly scale the activation and add to it using two learnable parameters to give the model the possibility to invert the normalization and take advantage of the non-linearity of the activation functions. These parameters can be learned using Adam, Gradient descent with momentum, or RMSprop, not just with gradient descent.

<p align="center"> <img src="figures/batch_norm.png"> </p>

Batch norm also helps with *covariate shift*, so that a model trained on some data might not do well on a new type of data even if there is a same mapping function from X to Y that works well with both. But the model can't find the correct decision boundary just looking at one portion of the data.

<p align="center"> <img src="figures/covariate_shift.png"> </p>

The same thing happen in between the network layers, a given layer L, takes the earlier activations and find a way to map them to Y, trying to learn the parameters to get the correct predictions, but the problem is that the earlier activations are also chaging in the same time, thus we have a covariate shift, but with batch norm, we ensure that the mean and variance of the earlier layers is stable and minimises the affect of the earlier layers on the later ones. So each layer can learn independetly and accelerate the learning speed.

Batch norm also has a slight regularization effect, the scaling by mean/variance within a mini batch adds some noise to each hidden layers's activations.

Note: In batch norm, the bias does not have any impact, given that after adding the bias to get the activations, we substract the mean, so any constant we add before that will be subtracted, and is replaced by beta we add after normelizing the activations.

**Batch norm at test time:** given that in test time, we might want to do the prediction on one example; so we need a different way comming up with the mean and the variance, one solution is to used an exponenetially weighted average of the mean and the variance across the mini batches during training, and at test time, we use the last running average of the mean and variance for the test examples.

**Batch norm in conv nets:**
The output of the convolutional layer is a 4-rank tensor [B, H, W, C], where B is the batch size, (H, W) is the feature map size, C is the number of channels. An index (x, y) where 0 <= x < H and 0 <= y < W is a spatial location.

**Usual batchnorm** 
```python
        # t is the incoming tensor of shape [B, H, W, C]
        # mean and stddev are computed along 0 axis and have shape [H, W, C]
        mean = mean(t, axis=0)
        stddev = stddev(t, axis=0)
        for i in 0..B-1:
            out[i,:,:,:] = norm(t[i,:,:,:], mean, stddev)
```

Basically, it computes H\*W\*C means and  H\*W\*C standard deviations across B elements. We notice that different elements at different spatial locations have their own mean and variance and gather only B values.

**Batchnorm in conv layer**
the convolutional layer has a special property: filter weights are shared across the input image. That's why it's reasonable to normalize the output in the same way, so that each output value takes the mean and variance of B*H*W values, at different locations.

```python
        # t is still the incoming tensor of shape [B, H, W, C]
        # but mean and stddev are computed along (0, 1, 2) axes and have just [C] shape
        mean = mean(t, axis=(0, 1, 2))
        stddev = stddev(t, axis=(0, 1, 2))
        for i in 0..B-1, x in 0..H-1, y in 0..W-1:
            out[i,x,y,:] = norm(t[i,x,y,:], mean, stddev)
```

In total, there are only C means and standard deviations and each one of them is computed over B\*H\*W values. That's what they mean when they say "effective mini-batch": the difference between the two is only in axis selection (or equivalently "mini-batch selection").

### Multi-class classification

#### Softmax

Softmax regression generalizes logistic regression to C classes.

The softmax activation function is often placed at the output layer of a neural network. It’s commonly used in multi-class learning problems where a set of features can be related to one-of-K classes.

Its equation is simple, we just have to compute for the normalized exponential function of all the units in the layer.

<img src="https://latex.codecogs.com/gif.latex?$$S&space;\left(y&space;_&space;{&space;i&space;}&space;\right)&space;=&space;\frac&space;{&space;e^{y_i}&space;}&space;{&space;\sum_{j}&space;e&space;^&space;{&space;y_j&space;}&space;}$$" title="$$S \left(y _ { i } \right) = \frac { e^{y_i} } { \sum_{j} e ^ { y_j } }$$" />

Softmax squashes a vector of size K between 0 and 1. Furthermore, because it is a normalization of the exponential, the sum of this whole vector equates to 1. We can then interpret the output of the softmax as the probabilities that a certain set of features belongs to a certain class.

**Loss function**: the loss is the negative log-likelihood <img src="https://latex.codecogs.com/gif.latex?$L(\hat{y},&space;y)&space;=&space;-&space;\log&space;(y_i)$" title="$L(\hat{y}, y) = - \log (y_i)$" />, given that the target is a one hot vector, we end up with one term of the softmax corresponding to the desired class, and the objective is to increase the predicted probability of this class, thus decreasing the rest of the element of the softmax; the cost is :

<img src="https://latex.codecogs.com/gif.latex?J&space;=&space;\frac{1}{m}\sum^m_{i=1}L(\hat{y},&space;y)" title="J = \frac{1}{m}\sum^m_{i=1}L(\hat{y}, y)" />

#### Softmax with a varying temperature ([source](https://cs.stackexchange.com/questions/79241/what-is-temperature-in-lstm-and-neural-networks-generally))

When the temperature is 1, we compute the softmax directly on the logits (the unscaled output of earlier layers), and using a temperature of 0.6 the model computes the softmax on logits/0.6, resulting in a larger value. Performing softmax on larger values makes the model more confident (less input is needed to activate the output layer) but also more conservative in its sample, it is less likely to sample from unlikely candidates). Using a higher temperature produces a softer probability distribution over the classes, and makes the model more “easily excited” by samples, resulting in more diversity and also more mistakes.

This time the softmax is:

<img src="https://latex.codecogs.com/gif.latex?$$S&space;\left(y&space;_&space;{&space;i&space;}&space;\right)&space;=&space;\frac&space;{&space;e^{y_i}&space;/&space;T}&space;{&space;\sum_{j}&space;e&space;^&space;{&space;y_j&space;/&space;T&space;}&space;}$$" title="$$S \left(y _ { i } \right) = \frac { e^{y_i} / T} { \sum_{j} e ^ { y_j / T } }$$" />

where T is the temperature parameter, normally set to 1.

The softmax function normalizes the candidates at each iteration of the network based on their exponential values by ensuring the network outputs are all between zero and one at every timestep. Temperature therefore increases the sensitivity to low probability candidates

> For high temperatures (τ→∞), all [samples] have nearly the same probability and the lower the temperature, the more expected rewards affect the probability. For a low temperature (τ→0+), the probability of the [sample] with the highest expected reward tends to 1.