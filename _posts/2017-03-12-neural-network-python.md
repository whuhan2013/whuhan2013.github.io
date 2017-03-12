---
layout: post
title: Neural Netork之python实现
date: 2017-3-12
categories: blog
tags: [计算机视觉]
description: 计算机视觉
---

本篇主要实现一个神经网络并应用于CIFAR-10数据集。           

**计算损失函数与梯度**      

```
    W1, b1 = self.params['W1'], self.params['b1']
    W2, b2 = self.params['W2'], self.params['b2']
    N, D = X.shape

    # Compute the forward pass
    scores = None

    hidden_layer = np.maximum(0, np.dot(X, W1) + b1) #ReLU
    scores = np.dot(hidden_layer, W2) + b2
    # print scores
    
    # If the targets are not given then jump out, we're done
    if y is None:
      return scores

    # Compute the loss
    loss = None

    # visual prob
    exp_scores = np.exp(scores)
    probs = exp_scores / np.sum(exp_scores, axis=1, keepdims=True) # [N x K]

    #calculate loss
    corect_logprobs = -np.log(probs[range(N),y])
    data_loss = np.sum(corect_logprobs)/N
    reg_loss = 0.5*reg*np.sum(W1*W1) + 0.5*reg*np.sum(W2*W2)
    loss = data_loss + reg_loss

    # Backward pass: compute gradients
    grads = {}
    # calcute gradient
    dscores = probs
    dscores[range(N),y] -= 1
    dscores /= N

    # graident backforwar
    dW2 = np.dot(hidden_layer.T, dscores)
    db2 = np.sum(dscores, axis=0, keepdims=False)

    dhidden = np.dot(dscores, W2.T)

    dhidden[hidden_layer <= 0] = 0
    # get w b gradient
    dW1 = np.dot(X.T, dhidden)
    db1 = np.sum(dhidden, axis=0, keepdims=False)

    # add regular
    dW2 += reg * W2
    dW1 += reg * W1
    grads['W1']=dW1
    grads['W2']=dW2
    grads['b1']=db1
    grads['b2']=db2

    return loss, grads
```

**检验损失函数与梯度**      

```
def rel_error(x, y):
  """ returns relative error """
  return np.max(np.abs(x - y) / (np.maximum(1e-8, np.abs(x) + np.abs(y))))

print 'Difference between your scores and correct scores:'
print np.sum(np.abs(scores - correct_scores))


loss, _ = net.loss(X, y, reg=0.1)
correct_loss = 1.30378789133

# should be very small, we get < 1e-12
print 'Difference between your loss and correct loss:'
print np.sum(np.abs(loss - correct_loss))

loss, grads = net.loss(X, y, reg=0.1)

# these should all be less than 1e-8 or so
for param_name in grads:
  
  f = lambda j: net.loss(X, y, reg=0.1)[0]
  #print type(f),type(net.loss(X, y, reg=0.1)[0])
  param_grad_num = eval_numerical_gradient(f, net.params[param_name], verbose=False)
  print '%s max relative error: %e' % (param_name, rel_error(param_grad_num, grads[param_name]))
```

**训练神经网络**      

```
def train(self, X, y, X_val, y_val,
            learning_rate=1e-3, learning_rate_decay=0.95,
            reg=1e-5, num_iters=100,
            batch_size=200, verbose=False):
    num_train = X.shape[0]
    iterations_per_epoch = max(num_train / batch_size, 1)

    # Use SGD to optimize the parameters in self.model
    loss_history = []
    train_acc_history = []
    val_acc_history = []

    for it in xrange(num_iters):
      X_batch = None
      y_batch = None

      idx = np.random.choice(num_train, batch_size, replace=True)
      X_batch = X[idx]
      y_batch = y[idx]


      # Compute loss and gradients using the current minibatch
      loss, grads = self.loss(X_batch, y=y_batch, reg=reg)
      loss_history.append(loss)


      self.params['W1'] += -learning_rate * grads['W1']
      self.params['b1'] += -learning_rate * grads['b1']
      self.params['W2'] += -learning_rate * grads['W2']
      self.params['b2'] += -learning_rate * grads['b2']


      if verbose and it % 100 == 0:
        print 'iteration %d / %d: loss %f' % (it, num_iters, loss)

      # Every epoch, check train and val accuracy and decay learning rate.
      if it % iterations_per_epoch == 0:
        # Check accuracy
        train_acc = (self.predict(X_batch) == y_batch).mean()
        val_acc = (self.predict(X_val) == y_val).mean()
        train_acc_history.append(train_acc)
        val_acc_history.append(val_acc)

        # Decay learning rate
        learning_rate *= learning_rate_decay

    return {
      'loss_history': loss_history,
      'train_acc_history': train_acc_history,
      'val_acc_history': val_acc_history,
    }

def predict(self, X):
    y_pred = None

 
    hidden_layer = np.maximum(0, np.dot(X, self.params['W1']) + self.params['b1'])
    scores = np.dot(hidden_layer, self.params['W2']) + self.params['b2']
    y_pred = np.argmax(scores, axis=1)
    return y_pred
```

**画出损失函数变化图**      

```
net = init_toy_model()
stats = net.train(X, y, X, y,
            learning_rate=1e-1, reg=1e-5,
            num_iters=100, verbose=False)

print 'Final training loss: ', stats['loss_history'][-1]

# plot the loss history
plt.plot(stats['loss_history'])
plt.xlabel('iteration')
plt.ylabel('training loss')
plt.title('Training Loss history')
plt.show()
```

![](https://raw.githubusercontent.com/whuhan2013/myImage/master/cs231n/chapter9/p4.png)





