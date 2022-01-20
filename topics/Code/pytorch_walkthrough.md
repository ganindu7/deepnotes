---
layout: default
title: Pytorch Walkthrough Notes
nav_order: 2 
# permalink: /topics/utils/target_execute
parent: Utilities
---

## Starting Development with [PyTorch][PYTORCH]
<span style="background-color:LightGreen">
Created : 18/01/2022 | on Linux: 5.4.0-91-generic <br />
Updated: 18/01/2022 | on Linux: 5.4.0-91-generic <br />
Status: Draft
</span>

<span style="background-color:LightYellow"> Check Installing and configuring PyTorch sections if you haven't already </span>

### Data
PyTorch has two primitives to work with data: 
1. `torch.utils.Dataset`
2. `torch.utils.DataLoader`

*Dataset* stores samples and the corresponding labels while the *DataLoader* wraps an iterable over the *Dataset*. Once a
*DataLoader* wraps over the *Dataset* it can support automated batching, sampling shuffling and nmultiprocesss data loading. 

The code below downloads the FashionMNIST dataset, notice the `train=True` this means what is downloaded (training data in this instance). To get test data 
we can set `train=False` 

```python
training_data = datasets.FashionMNIST(
                                       root="data",
                                       train=True,
                                       download=True,
                                       transform=ToTensor(),
)
```

once we have the data we can wrap it around a *DataLoader* object

```python
batch_size=64
train_dataloader = DataLoader(training_data, batch_size=batch_size)
```

### Creating Models
To define a Neural Network in PyTorch. We need to

1. Create a Class that inherits the `nn.module`.
2. Define the Layers in the `__init__` method of the class defined in the step above.
3. Specify the Data flow in the `forward` method of the class.
4. Move the NN to the GPU if available. 

#### Class Definition

```python
class NeuralNetwork(nn.Module):
    def __init__(self):
        super(NeuralNetwork, self).__init__()
        self.flatten = nn.Flatten()
        self.device = "cuda" if torch.cuda.is_available() else "cpu"
        self.linear_relu_stack = nn.Sequential(
            nn.Linear(28 * 28, 512),
            nn.ReLU(),
            nn.Linear(512, 512),
            nn.ReLU(),
            nn.Linear(512, 10)
        )

        self.loss_fn = nn.CrossEntropyLoss()
        self.optimizer = torch.optim.SGD(self.parameters(), lr=1e-3)


    def forward(self, x):
            x = self.flatten(x)
            logits = self.linear_relu_stack(x)
            return logits


```

#### Train method

```python
def train(dataloader, model, loss_fn, optimizer):
    size = len(dataloader.dataset)
    model.train()
    for batch, (X, y ) in enumerate(dataloader):
        X, y = X.to(model.device), y.to(model.device)

        # compute prediction error 
        pred = model(X)
        loss = loss_fn(pred, y)

        # Backprop
        optimizer.zero_grad()
        loss.backward()
        optimizer.step()

        if batch % 100 == 0:
            loss, current = loss.item(), batch * len(X)
            print(f"loss: {loss:>7f} [{current:>5d}/{size:>5d}]")
```


 <br />
Source: [PyTorch Tutorial][PyTorch-Tutorial]

[JETSON-URL]: https://developer.nvidia.com/embedded/jetson-agx-xavier-developer-kit
[PYTORCH]: https://pytorch.org
[NVIDIA-PYTORCH-GUIDE]: https://forums.developer.nvidia.com/t/pytorch-for-jetson-version-1-9-0-now-available/72048
[PyTorch-Tutorial]: https://pytorch.org/tutorials/beginner/basics/quickstart_tutorial.html

