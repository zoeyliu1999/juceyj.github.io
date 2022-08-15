---
title: 'Pytorch Notes'
date: 2019-02-01 16:31:13
layout: archive
author_profile: true
permalink: /blog/pytorch/
---

```python
import torch
import torch.nn as nn
import torch.nn.functional as F
import torch.optim as optim
import numpy as np
```



# Dataloader

首先是pytorch的导入数据API，这部分API我觉得整体思想和`tf.data`非常类似，都是将数据集合迭代器搭配使用。

```python
from torch.utils.data import DataLoader #loader库
from torch.utils.data import sampler  # 不是很重要的取样API

import torchvision.datasets as dset # 从dataset中生成loader
import torchvision.transforms as T # dataset的对应预处理操作

transform = T.Compose([
                T.ToTensor(),
                T.Normalize((0.4914, 0.4822, 0.4465), (0.2023, 0.1994, 0.2010))
            ])

cifar10_train = dset.CIFAR10('./cs231n/datasets', train=True, download=True,
                             transform=transform)
loader_train = DataLoader(cifar10_train, batch_size=64, 
                          sampler=sampler.SubsetRandomSampler(range(NUM_TRAIN)))

cifar10_val = dset.CIFAR10('./cs231n/datasets', train=True, download=True,
                           transform=transform)
loader_val = DataLoader(cifar10_val, batch_size=64, 
                        sampler=sampler.SubsetRandomSampler(range(NUM_TRAIN, 50000)))

cifar10_test = dset.CIFAR10('./cs231n/datasets', train=False, download=True, 
                            transform=transform)
loader_test = DataLoader(cifar10_test, batch_size=64)

# 1.
for t, (x, y) in enumerate(loader_train):
# 2. 
for x, y in loader:
```

- torchvision是独立于pytorch的关于图像操作的一些方便工具库。

  torchvision主要包括一下几个包：

  - [vision.datasets](https://pypi.org/project/torchvision/0.1.8/#datasets) : 几个常用视觉数据集，可以下载和加载
  - [vision.models](https://pypi.org/project/torchvision/0.1.8/#models) : 流行的模型，例如 AlexNet, VGG, and ResNet 以及 与训练好的参数。
  - [vision.transforms](https://pypi.org/project/torchvision/0.1.8/#transforms) : 常用的图像操作，例如：随机切割，旋转等。
  - [vision.utils](https://pypi.org/project/torchvision/0.1.8/#utils) : 用于把形似 (3 x H x W) 的张量保存到硬盘中，给一个mini-batch的图像可以产生一个图像格网。

- 首先`torchvision.transform`：图像预处理转化，这里采取的主要就是均值归一化之后将数据转为torch.tensor

- `torchvision.dataset`：常用的就上述的四个参数：`root`表示数据集的存储位置，`train`用来区分训练集和测试集，`download`和`transorm`如语意所示

- `Dataloader`：	

  - dataset (Dataset) – dataset from which to load the data.
  - batch_size (int, optional) – how many samples per batch to load (default: 1).
  - shuffle (bool, optional) – set to True to have the data reshuffled at every epoch (default: False).
  - sampler (Sampler, optional) – defines the strategy to draw samples from the dataset. If specified, shuffle must be False.

- 两种调用loader的方式，和_iter__()很类似



# Device

```python
device = torch.device('cuda')
device = torch.device('cpu')
```


# torch.Tensor

torch.Tensor几乎是综合了tensorflow中所有的constant，variable，tensor等等变量，其中有几个重要的attribute：

- **data**(array_like)– The returned Tensor copies `data`.
- **dtype** ([`torch.dtype`](https://pytorch.org/docs/stable/tensor_attributes.html#torch.torch.dtype), optional) – the desired type of returned tensor. Default: if None, same [`torch.dtype`](https://pytorch.org/docs/stable/tensor_attributes.html#torch.torch.dtype) as this tensor.
- **device** ([`torch.device`](https://pytorch.org/docs/stable/tensor_attributes.html#torch.torch.device), optional) – the desired device of returned tensor. Default: if None, same [`torch.device`](https://pytorch.org/docs/stable/tensor_attributes.html#torch.torch.device) as this tensor.
- **requires_grad** ([*bool*](https://docs.python.org/3/library/functions.html#bool)*,* *optional*) – If autograd should record operations on the returned tensor. Default: `False`.

    ```python
    x = torch.tensor([[1., -1.], [1., 1.]], requires_grad=True)
    out = x.pow(2).sum()
    out.backward()
    x.grad
    >>>tensor([[ 2.0000, -2.0000],
        [ 2.0000,  2.0000]])
    ```
- tensor可以通过item()（标量）和numpy()（数组）来访问数据，每个variable有两个重要的属性：data()（返回tensor）和grad()（返回梯度，numpy array）。

- GPU的两种使用方式

  - 使用可以在使用的时候将tensor移动到指定的device

  ```python
  for x, y in loader:
    	x = x.to(device=device, dtype=dtype)  # move to device, e.g. GPU
   	y = y.to(device=device, dtype=torch.int64)
      scores = model_fn(x, params)
  ```

  - 设定dtype

  ```python
  dtype = torch.cuda.FloatTensor
  x = x.type(dtype)
  ```



- .view()和.reshape()类似

# no.grad()

```python
with torch.no_grad():
   	for w in params:
     	w -= learning_rate * w.grad
        # Manually zero the gradients after running the backward pass
        w.grad.zero_()
```

在不需要计算梯度的情况下关闭梯度，并且每次迭代都清空参数的梯度



# torch.nn

- torch.nn和torch.nn.functional，就是相同的操作，一个是class API一个是函数式API
- `torch.nn.init`：提供的初始化真是骚爆了。重点是每一次nn Module的初始化都要调用nn.init的初始化包

```python
class TwoLayerFC(nn.Module):
    def __init__(self, input_size, hidden_size, num_classes):
        super().__init__()
        self.fc1 = nn.Linear(input_size, hidden_size)
        nn.init.kaiming_normal_(self.fc1.weight)
        self.fc2 = nn.Linear(hidden_size, num_classes)
        nn.init.kaiming_normal_(self.fc2.weight)
    # 调用函数位forward
    def forward(self, x):
        x = flatten(x)
        scores = self.fc2(F.relu(self.fc1(x)))
        return scores
```

- 训练过程：**每一次都要手动清空梯度，否则梯度会自动累计**。而且在开始之前还可以设定model的training/testing状态（应该是提供给dropout的），当然必须是`nn.Module`的继承类才可以。
- .detach()可以强制使得这个点为叶节点，从而截断梯度往下传递 

```python
for t, (x, y) in enumerate(loader_train):
    model.train()  # put model to training mode
    # model.eval() put model to evaluation mode
    x = x.to(device=device, dtype=dtype)  # move to device, e.g. GPU
    y = y.to(device=device, dtype=torch.long)

    scores = model(x)
    scores = scores.reshape((-1, 10))
    loss = F.cross_entropy(scores, y)

   	optimizer.zero_grad()
	loss.backward()
	optimizer.step()
```

- `nn.Sequential`和tf.keras.Sequential一模一样
- 需要移动到device的有：model、x、y



# torch.save

```python
def save_checkpoint(checkpoint_path, model, optimizer):
    state = {'state_dict': model.state_dict(),
             'optimizer' : optimizer.state_dict()}
    torch.save(state, checkpoint_path)
    print('model saved to %s' % checkpoint_path)
    
def load_checkpoint(checkpoint_path, model, optimizer):
    state = torch.load(checkpoint_path)
    model.load_state_dict(state['state_dict'])
    optimizer.load_state_dict(state['optimizer'])
    print('model loaded from %s' % checkpoint_path)
```

- model.state_dict()
- optimizer.state_dict()