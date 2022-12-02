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
Updated: 14/02/2022 | on Linux: 5.4.0-91-generic <br />
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

Code for the steps discussed above.

<script src="https://gist.github.com/ganindu7/351906087bd899193c9115c2be8b9187.js?file=eager_model.py"></script>
<br />


### Model layers 

Now we look at the layers of the model. To make things more transparent we are going to get some images from the [FashionMNIST dataset][FashonMnist-dataset] to use in the next steps. 

```python
import torch
from torch import nn
from torchvision import datasets
from torchvision.transforms import ToTensor
import matplotlib.pyplot as plt
import  numpy as n

training_data = datasets.FashionMNIST(root="data",
                                      train="True",
                                      download=True,
                                      transform=ToTensor()
                                     )

listsize = 3
image_list = []
for i in range(1, listsize + 1):
    sample_idx = torch.randint(len(training_data), size=(1,)).item()
    img, label = training_data[sample_idx]
    image_list.append(img)

images = torch.squeeze(torch.stack(image_list,0), 1) # image tensor 
print(images.size())
``` 
The code above will download (if necessary) the dataset and create a tensor of tensor of images. 

### Prepare the input

When we are using 1D stacked neuron layers we need to flatten the input to to match the $$ \boldsymbol{xW^T + b} $$ input shape for the linear transform. Here we initialise the [nn.Flatten][NN-FLATTEN] to convert the 2D 28x28 image into a contiguous array of 784 pixel values. 

```python
flatten = nn.Flatten()
flat_image = flatten(images)
```

The resulting tensor consisting of stacked image pixel values is of dimensions $$ 3\times784 $$, to get the desired $$ 3\times10$$ discrete probabilities output our $$ W^T $$ needs to be $$ 784\times10$$.

$$\boldsymbol{X} = \begin{pmatrix} 
x_{0,0} & x_{0,1} & ... & x_{0,783} \\
x_{1,0} & x_{1,1} & ... & x_{1,783} \\
x_{2,0} & x_{2,1} & ... & x_{2,783}
\end{pmatrix}$$ 

<!-- $\begin{bmatrix}a & b\\c & d\end{bmatrix}$ -->

```python
print(flat_image.size())
```

```shell
torch.Size([3, 784])
```

### [nn.Linear][NN-LINEAR]

The [linear layer][NN-LINEAR] applies the linear transformation $$ \boldsymbol{xW^T + b} $$  to the input using its stored weights and biases.

```python
layer1 =nn.Linear(in_features=28*28, out_features=512)
hidden1 = layer1(flat_image)
print(hidden1.size())
```

```
torch.Size([3, 512])
```

### Using [Rectified Linear Units][RELU-TDS]

The currently widely popular activation function (that is a linear ramp for values greater than 0 and clamped to zero otherwise) serves as a non-linear mapping between inputs to the model and its outputs.


```python
hidden1 = nn.ReLU()(hidden1)
```


```shell
Before ReLU: tensor([[-0.0316, -0.0214,  0.0713,  ...,  0.0804,  0.1832, -0.0583],
        [-0.2158, -0.3031,  0.2640,  ..., -0.1948, -0.1070, -0.1727],
        [-0.0818,  0.1435, -0.0153,  ...,  0.0534,  0.3041, -0.0452]],
       grad_fn=<AddmmBackward0>)
After ReLU: tensor([[0.0000, 0.0000, 0.0713,  ..., 0.0804, 0.1832, 0.0000],
        [0.0000, 0.0000, 0.2640,  ..., 0.0000, 0.0000, 0.0000],
        [0.0000, 0.1435, 0.0000,  ..., 0.0534, 0.3041, 0.0000]],
       grad_fn=<ReluBackward0>)

```

### [nn.Sequential][NN-SEQUENTIAL]

As the name suggests [nn.Sequential][NN-SEQUENTIAL] provides a container to layout a ordered list of layers. The data is propagated to the proposed sequential order.
Below is a quick example for a quick sequential NN made from the blocks we discussed earlier 

```python
seq_modules = nn.Sequential(flatten,  # flatten 28x28 = 784 feature tensors
                            layer1,   # input: flattened features, output 512 units
                            nn.ReLU(),
                            nn.Linear(512, 10)
                            )

logits = seq_modules(images)
```

### [nn.Softmax][NN-SOFTMAX]

logits returned from the [nn.Linear][NN-LINEAR] are in the range of [-inf, +inf]. We convert them to values in [0, 1] to represent probabilities associated with corresponding predicted outcomes,

```python
softmax = nn.Softmax(dim=1) # `dim` indicated the dimension along the values must sum to 1
predicted_probabilities = softmax(logits)   
print(f"predicted probababilities = {predicted_probabilities}")
```

Please find the code for the topics we discussed in this section.

<script src="https://gist.github.com/ganindu7/351906087bd899193c9115c2be8b9187.js?file=network_components.py"></script>

## Model Parameters 

when data passes through a Neural Network structure the base input data is transformed by conducting mathematical operations with values residing inside the structure. Each layer consists of numerous parameters such as weights and biases. 

the [nn.Module][NN-MODULE] automatically tracks all the fields defined inside the model object and makes all parameters accessible using the model's `parameters()` or `named_parameters()` methods.

Check below for a complete code example that you can run and experiment with.

<script src="https://gist.github.com/ganindu7/351906087bd899193c9115c2be8b9187.js?file=model_params.py"></script>

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
[NN-FLATTEN]: https://pytorch.org/docs/stable/generated/torch.nn.Flatten.html
[NN-LINEAR]: https://pytorch.org/docs/stable/generated/torch.nn.Linear.html
[RELU-TDS]: https://towardsdatascience.com/understanding-relu-the-most-popular-activation-function-in-5-minutes-459e3a2124f
[NN-RELU]: https://pytorch.org/docs/stable/generated/torch.nn.ReLU.html
[NN-SEQUENTIAL]: https://pytorch.org/docs/stable/generated/torch.nn.Sequential.html
[NN-SOFTMAX]: https://pytorch.org/docs/stable/generated/torch.nn.Softmax.html




<!-- Latex in markdown -->
<script src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML" type="text/javascript"></script>
<!-- $$ \nabla_\boldsymbol{x} J(\boldsymbol{x}) $$ -->