---
layout: default
title: Building a network in eager mode 
nav_order: 2 
# permalink: /topics/utils/target_execute
parent: Utilities
---

## Building a network with [PyTorch][PYTORCH]
<span style="background-color:LightGreen">
Created : 08/02/2022 | on Linux: 5.4.0-91-generic <br />
Updated: 08/02/2022 | on Linux: 5.4.0-91-generic <br />
Status: Draft
</span>

<span style="background-color:LightYellow"> [**previous topic 1: Starting Development with PyTorch**](../pytorch_walkthrough#Starting-Development-with-PyTorch)  </span> <br  />
<span style="background-color:LightYellow"> [**previous topic 2: Tensors and Data Handling with PyTorch**](../tensors_and_model_input#Building-a-network)  </span>

### Modules 

the [torch.nn][TORCH-NN] namespace provides the key building blocks to build the network. Every module in [pytorch][PYTORCH] subclasses the [nn.module][NN-MODULE].
The neural network itself is a nested module-inside-module entity. This mostly refers to to the layers being modules inside the larger module.

```python

import os
import torch 
from torch import nn
from torch.utils.data import DataLoader
from torchvision import datasets, transforms 


```

### Select device 

If [torch.cuda][TORCH-CUDA] is available we can run our code in a cuda gpu for a significantly high performance boost.

```python
device = 'cuda' if torch.cuda.is_available() else `cpu`
printf(f'using {device}')
```

### Class definition

Let's examine the python code below to examine a *NeuralNetwork* Class!

```python
class NeuralNetwork(nn.Module):
    def __init__(self):
        super(NeuralNetwork, self).__init__()
        self.flatten = nn.Flatten()
        self.linear_relu_stack = nn.Sequential(
            nn.Linear(28*28, 512),
            nn.ReLU(),
            nn.Linear(512, 512),
            nn.ReLU(),
            nn.Linear(512, 10),
        )

    def forward(self, x):
        x = self.flatten(x)
        logits = self.linear_relu_stack(x)
        return logits
```

The Class defined above subclasses the `nn.Module`. The layers of the network are sequentially defined inside the body of the `__init__()` method using call to `nn.Sequential()`.

It is important to note that all classes that inherit from the `nn.Module` consists of a `forward` method that defines the operations carried out on the input data. 

If we summarise:

* define the network layout in the `__init__` method.
* define operations on input data in the `forward` method.

### Instantiate the Model and do basic operations.

We can now instantiate the class defined above! At this point we can interrogate the instantiated class to learn about its structure.

```python
model = NeuralNetwork().to(device)
print(model)
```

Output:

```shell
NeuralNetwork(
  (flatten): Flatten(start_dim=1, end_dim=-1)
  (linear_relu_stack): Sequential(
    (0): Linear(in_features=784, out_features=512, bias=True)
    (1): ReLU()
    (2): Linear(in_features=512, out_features=512, bias=True)
    (3): ReLU()
    (4): Linear(in_features=512, out_features=10, bias=True)
  )
)
```

This shows the 4(0 indexed) layers of our neural network, please note the line for the final output layer (`(4): Linear(in_features=512, out_features=10, bias=True)`) *the final output size is 10.*

For the purpose of the tutorial we generate some random data representing a 28x28 single channel image. 

```python
X = torch.rand(1, 28, 28, device=device)
``` 

Now we feed it into the model that executes the `forward` method we defined earlier and [some background operations][BACKROUND-OPS] that we will need to differentiate, optimise and update the weights later on. 

The resulting model returns the 10 output features (logits) from the last layer of the network we defined following the linear transformation with the form *($$ \boldsymbol{xW^T + b} $$)*.

```python
logits = model(X)
```

then we want to convert the logits into probabilities using the [Softmax][SOFTMAX-F] function and choose the one with the highest probability as the predicted outcome. 

```python
pred_prob = nn.Softmax(dim=1)(logits)
y_pred = pred_prob.argmax(1)
print(f"Prediction: {y_pred}")
```






<br />

<span style="background-color:LightYellow"> Check the [**next topic**](../pytorch_walkthrough#Starting-Development-with-PyTorch)  </span>

Source: [PyTorch Tutorial][PyTorch-Tutorial]

---
*Click [here][ERRORS-SUGGESTIONS] to report Errors, make Suggestions or Comments!*

[JETSON-URL]: https://developer.nvidia.com/embedded/jetson-agx-xavier-developer-kit
[PYTORCH]: https://pytorch.org
[NVIDIA-PYTORCH-GUIDE]: https://forums.developer.nvidia.com/t/pytorch-for-jetson-version-1-9-0-now-available/72048
[PyTorch-Tutorial]: https://pytorch.org/tutorials/beginner/basics/quickstart_tutorial.html
[FashonMnist-dataset]: https://github.com/zalandoresearch/fashion-mnist
[ERRORS-SUGGESTIONS]: https://github.com/ganindu7/deepnotes/issues
[TORCH-NN]: https://pytorch.org/docs/stable/nn.html
[NN-MODULE]: https://pytorch.org/docs/stable/generated/torch.nn.Module.html
[TORCH-CUDA]: https://pytorch.org/docs/stable/notes/cuda.html
[BACKROUND-OPS]: https://github.com/pytorch/pytorch/blob/270111b7b611d174967ed204776985cefca9c144/torch/nn/modules/module.py#L866
[LOGITS]: https://stackoverflow.com/questions/34240703/what-are-logits-what-is-the-difference-between-softmax-and-softmax-cross-entrop
[SOFTMAX-F]: https://en.wikipedia.org/wiki/Softmax_function

<!-- Latex in markdown -->
<script src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML" type="text/javascript"></script>
<!-- $$ \nabla_\boldsymbol{x} J(\boldsymbol{x}) $$ -->