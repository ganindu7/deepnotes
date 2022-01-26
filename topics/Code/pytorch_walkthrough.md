---
layout: default
title: Pytorch Walkthrough Notes
nav_order: 2 
# permalink: /topics/utils/target_execute
parent: Development
---

## Starting Development with [PyTorch][PYTORCH]
<span style="background-color:LightGreen">
Created : 18/01/2022 | on Linux: 5.4.0-91-generic <br />
Updated: 18/01/2022 | on Linux: 5.4.0-91-generic <br />
Status: Draft
</span>

<span style="background-color:LightYellow"> Check Installing and configuring PyTorch sections if you haven't already </span>

### Data
PyTorch has two primitives to work with data, these are:
1. `torch.utils.Dataset`
2. `torch.utils.DataLoader`

*Dataset* stores samples and the corresponding labels while the *DataLoader* wraps an iterable over the *Dataset*. Once a
*DataLoader* wraps over the *Dataset* it can support automated batching, sampling shuffling and multiprocess data loading. 

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

in the code above we are referring to the FashionMNIST dataset. If it is unavailable `datasets` will download the dataset and save it under the name pointed by `root`.

once we have the data we can wrap it around a *DataLoader* object as shown in the code snippet below 

```python
batch_size=64
train_dataloader = DataLoader(training_data, batch_size=batch_size)
```

### Creating Models
To define a Neural Network in PyTorch. We need to

1. Create a Class that inherits the `nn.module`.
2. Define the Layers in the `__init__` method of the class defined in the step above.
3. Specify the Data flow in the `forward` method of the class.
4. Move the NN to the GPU if the resource is available to us. 

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

Train method uses the training set to make predictions, compare with the labels and backpropagate the prediction error to update the weights. 
the `loss function` here is used to get the error (difference between model output and the training labels) and the optimiser determines the semantics of the navigation through the error 
plane. (how the error is optimally reduced) 


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

Putting everything together. The python script below downloads training and test data from the [FashonMnist dataset][FashonMnist-dataset] 
batches into bundles of size n (e.g. 64) and runs some training and test loops so we can see the accuracy grow as every iteration. As its final action the model is saves as a state dictionary into a ".pth" file.

{: .mx-1}
<script src="https://gist.github.com/ganindu7/351906087bd899193c9115c2be8b9187.js?file=quickstart.py"></script>
{: .mx-1}


#### Loading models

Loading models can be easily done if we have the class definition for the models and the ".pth" file that consists the learned parameters. 
In the following 

<script src="https://gist.github.com/ganindu7/351906087bd899193c9115c2be8b9187.js?file=loading_models.py"></script>



Source: [PyTorch Tutorial][PyTorch-Tutorial]

[JETSON-URL]: https://developer.nvidia.com/embedded/jetson-agx-xavier-developer-kit
[PYTORCH]: https://pytorch.org
[NVIDIA-PYTORCH-GUIDE]: https://forums.developer.nvidia.com/t/pytorch-for-jetson-version-1-9-0-now-available/72048
[PyTorch-Tutorial]: https://pytorch.org/tutorials/beginner/basics/quickstart_tutorial.html
[FashonMnist-dataset]: https://github.com/zalandoresearch/fashion-mnist

