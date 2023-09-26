---
title: "Intro to PyTorch for Deep Learning"
date: 2021-05-27T11:39:56Z
type: posts
draft: False 
tags: [Misc, Pytorch, Deep Learning]
description: Pytorch is an open source machine learning framework that accelerates the path from research prototyping to production deployment.
---
## PyTorch

Pytorch is an open source machine learning framework that accelerates the path from research prototyping to production deployment.

## Regression Problem Example

### Boston Housing Dataset

I have used the Boston housing dataset as it is one of the basic and beginner friendly dataset on [Kaggle](https://www.kaggle.com/c/house-prices-advanced-regression-techniques)

### Python Implementation 

The implementation creates a simple neural network using pytorch and compares with a baseline linear regression model.

#### Learnings
Some key lessons learnt are to start with simple datasets and compare with baseline model to see the difference of using a non linear type model.
Data has to be preprocessed into tensors and chunked into batches and then passed to training. 

#### Caveats and Gotchas
Loss Function in pytorch requires to have 2 dimensional tensor for the Y (independent) variable. E.g., for tensor of size 100 the tensor shape should be 
(100, 1)

```python
import imageio
import matplotlib.pyplot as plt
import numpy as np
import pandas as pd
import torch
import torch.nn.functional as F
import torch.utils.data as Data
from torch import tensor
from torch.autograd import Variable

from sklearn.metrics import mean_squared_error
from tqdm import tqdm
from sklearn.preprocessing import MinMaxScaler
torch.manual_seed(1)    # reproducible

datadir = './data/'
# get data from kaggle
df = pd.read_csv(f'{datadir}/boston_train.csv')

y = df['medv'].to_numpy()
x = df[['crim', 'zn', 'indus', 'chas', 'nox', 'age', 'dis', 'rad', 'tax',
        'ptratio', 'black', 'rm', 'lstat']].to_numpy()

# scaling may be enabled not necessary though
# scaler = MinMaxScaler()
# x = scaler.fit_transform(x.astype(np.float32))

x, y = tensor(x.astype(np.float32)), tensor(y.astype(np.float32))
# torch can only train on Variable, so convert them to Variable
x, y = Variable(x), Variable(y)

# another way to define a network
# Linear layers do a linear tranformation i.e matrix multiplication
# with a constant (bias)
# LeakyRelu is an activativation which does non linear transformation
# Input here 13 is number of features in data
net = torch.nn.Sequential(
    torch.nn.Linear(13, 200),
    torch.nn.LeakyReLU(),
    torch.nn.Linear(200, 100),
    torch.nn.LeakyReLU(),
    torch.nn.Linear(100, 1),
)

LR = 0.001

optimizer = torch.optim.Adam(net.parameters(), lr=LR)
# this is for regression mean squared loss
loss_func = torch.nn.MSELoss(reduce='mean')

BATCH_SIZE = 50
EPOCH = 300

torch_dataset = Data.TensorDataset(x, y)

loader = Data.DataLoader(
    dataset=torch_dataset, 
    batch_size=BATCH_SIZE, 
    shuffle=True, num_workers=4,)


def get_mse(model):
    """
    Return the accuracy of the model on the input data and actual ground truth.
    """
    prediction = net(x)     # input x and predict based on x
    return mean_squared_error(y.data.numpy(),  prediction.data.numpy())


epoch_losses = []
training_accuracy = []
# start training
for epoch in range(EPOCH):
    losses = []
    # for each training step
    # steps are equal to total rows / batch size
    for step, (batch_x, batch_y) in enumerate(loader): 
        b_x = Variable(batch_x)
        b_y = Variable(batch_y)
    
        prediction = net(b_x)     # input x and predict based on x

        # must be (1. nn output, 2. target)
        # the target must be 2 dimensional array otherwise model
        # predicts same values
        loss = loss_func(prediction, torch.unsqueeze(b_y, dim=1))

        optimizer.zero_grad()   # clear gradients for next train
        loss.backward()         # backpropagation, compute gradients
        optimizer.step()        # apply gradients

        # save the current training information
        losses.append(float(loss))
    
    epoch_losses.append(sum(losses)/step)
    training_accuracy.append(get_mse(net))

    print(
        f"Epoch #{epoch+1}\tLoss: {epoch_losses[-1]:.3f}\t MSE: {training_accuracy[-1]}")


# plotting
iters = [i for i in range(EPOCH)]
plt.plot(iters, epoch_losses)
plt.title(f"Training Curve (batch_size={BATCH_SIZE}, lr={LR})")
plt.xlabel("Iterations")
plt.ylabel("Loss")
plt.show()

plt.plot(iters, training_accuracy)
plt.title(f"Training Error (batch_size={BATCH_SIZE}, lr={LR})")
plt.xlabel("Iterations")
plt.ylabel("Training Accuracy")
plt.show()


prediction = net(x)     # input x and predict based on x
mean_squared_error(y.data.numpy(),  prediction.data.numpy())


# Compare with Linear regression baseline
from sklearn.linear_model import LinearRegression
lm = LinearRegression()
lm.fit(x, y)
prediction = lm.predict(x)
mean_squared_error(y,  prediction)

```

