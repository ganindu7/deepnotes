---
layout: default
title: Backpropagation  
nav_order: 2 
# permalink: /topics/utils/target_execute
parent: Utilities
---

## Backpropagation 
<span style="background-color:LightGreen">
Created : 16/02/2022 | on Linux: 5.4.0-91-generic <br />
Updated : 16/02/2022 | on Linux: 5.4.0-91-generic <br />
Status: Draft
</span>


### Introduction

<!-- <span style="background-color:lightgoldenrodyellow;">
***"The training effect on a single parameter inside a neural net flows from the <u>sensitivity of the total error of the network</u> to a <u>change in that parameter</u>."*** </span> -->

<span style="background-color:lightgoldenrodyellow;">
Observing the sensitivity of the network loss (*or the error*) in response to a change in single parameter during a single pass cycle is a key fundamental step in training neural networks. </span>

Once we know the end effect of that particular parameter *(weight or bias)* we can make adjustments to drive that loss down. Repeating this process of observing and making adjustments until we get a satisfactory set of parameters is our goal in training neural networks.   

### Loss
 
Understanding the gradient flow into an arbitrary node is much easier if we start from the beginning! <span style="background-color:lightgoldenrodyellow;"> The "Beginning" for backpropagation is at the very end of the forward pass </span> *(however this comes with a little caveat we will discuss shortly, but for now just focus on the end)* 



At the end of the forward pass the neural network either spits out a scalar or a tensor valued output at the end of the graph (*based on the design of the network*). This output is then compared against a ***ground truth*** (or a known *label* as we sometimes call it). We Call this comparison the loss($$ \textbf{L} $$) of the network.

<p align="center">
	<img src="intro.svg"
	     title="Inputs, graph outputs and labels"
	     width="450" height="450" />
</p>
<center> <em> Figure 1. Inputs, Network graph, outputs and labels  </em> </center>

the word "Loss" sounds like an implicit term for ***"Error"*** *(At least I feel like that coming from a controls engineering background)* However there is a distinction, Error is a more explicitly comparable in terms of tangible quantities that are more similar in nature *(think of a control system regulating position or speed).* In a neural network the output of a graph and a ground truth cannot be compared to that level of similarity without making serious assumptions. So instead we use the ***"Loss"*** which is a mathematical function with useful properties to make *NN*s work. Having said that we will soon figure out that the *"Loss"* tracks the *"Error"* very closely. *i.e. If we have a large loss we can assume that we have a large error and vice versa.*   


If the output of the network is "$$ \textbf{z} $$" and the ground truth is "$$ \textbf{y} $$", We can formalise the loss, "$$ \textbf{L} $$" as a function of $$\textbf{z}$$ and $$\textbf{y}$$,  "$$ \textbf{L(z, y)} $$". <br />

<span style="background-color:powderblue;">
*The internal composition of this loss function can be one of many forms as long as it is differentiable with respect to* $$ \textbf{z} $$. 
</span>

### Sensitivity and Gradients 

In the section above we talked about the Loss, Now it is time to think of ways to use the loss to update the parameters (weights and biases) of our network to minimise the loss.

Usually **we start with a random set of values for our parameters and change them over and over again until the <em>"Loss"</em> reaches a satisfactory minimum.**

To preform this efficiently 



<p align="center">
	<img src="graph0.svg"
	     title="abstract-example"
	     width="650" height="650" />
</p>
<center> <em> Figure 2. Gradient flow for a single computational unit in a Neural Network </em> </center>

<p align="center">
	<img src="graph1.svg"
	title="multiply unit"
	width="650" height="650" />
</p>
<center> Figure 3. PCAN View CAN trace </center>

<p align="center">
	<img src="graph2.svg"
	title="worked example"
	width="650" height="650" />
</p>
<center> Figure 4. PCAN View CAN trace </center>





 -->


<br />

<span style="background-color:LightYellow"> Check the [**next topic**](../pytorch_walkthrough#Starting-Development-with-PyTorch)  </span>


---
*Click [here][ERRORS-SUGGESTIONS] to report Errors, make Suggestions or Comments!*

[JETSON-URL]: https://developer.nvidia.com/embedded/jetson-agx-xavier-developer-kit
[PYTORCH]: https://pytorch.org
[NVIDIA-PYTORCH-GUIDE]: https://forums.developer.nvidia.com/t/pytorch-for-jetson-version-1-9-0-now-available/72048
[PyTorch-Tutorial]: https://pytorch.org/tutorials/beginner/basics/quickstart_tutorial.html
[FashonMnist-dataset]: https://github.com/zalandoresearch/fashion-mnist
[ERRORS-SUGGESTIONS]: https://github.com/ganindu7/deepnotes/issues
[GRADIENT-DESCENT-1]: https://towardsdatascience.com/gradient-descent-algorithm-a-deep-dive-cf04e8115f21
[BACKPROP-NOTES]: https://cs231n.github.io/optimization-2/
[ML-NOTES-2]: https://arxiv.org/abs/1502.05767

<!-- Latex in markdown -->
<script src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML" type="text/javascript"></script>
<!-- $$ \nabla_\boldsymbol{x} J(\boldsymbol{x}) $$ -->