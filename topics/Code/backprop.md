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
The art of training a neural network can be described in many ways. At the heart of training is a response to a sensitivity observation. We can even say it is an iterative reactionary response or
<span style="background-color:lightgoldenrodyellow;">
***<u>reacting to the sensitivity of a network due to a change in a single parameter </u> in a manner that drives the error down.*** </span>

Once we know the end effect of that particular parameter *(weight or bias)* we can make adjustments to drive that loss down. Repeating this process of observing and making adjustments until we get a satisfactory set of parameters is our goal in training neural networks.   

### Loss
 
Understanding the gradient flow into an arbitrary node is much easier if we start from the point where an "Error" value is calculated. <span style="background-color:lightgoldenrodyellow;"> The "Beginning" for backpropagation as the name suggests is at the very end of the forward pass. </span> 



At the end of the forward pass the neural network either spits out a scalar or a tensor valued output at the end nodes of the graph. This output is then compared against a ***ground truth*** (or a known *label* as we sometimes call it). We Call this comparison the loss($$ \textbf{L} $$) or the Error($$ \textbf{E} $$)of the network.


<p align="center">
	<img src="intro.svg"
	     title="Inputs, graph outputs and labels"
	     width="450" height="450" />
</p>
<center> <em> Figure 1. Inputs, Network graph, outputs and labels  </em> </center>

<em>the word "Loss" sounds like an implicit term for ***"Error"*** *(At least I feel like that coming from a controls engineering background)* In this text I will use the terms interchangeably referring to the same thing.</em> 

*Notation:* 
If the output of the network is "$$ \textbf{y} $$" and the ground truth is "$$ \textbf{t} $$", We can formalise the loss, "$$ \textbf{L} $$" as a function of $$\textbf{y}$$ and $$\textbf{t}$$,  "$$ \textbf{L(y, t)} $$". <br />

<span style="background-color:powderblue;">
*The internal composition of this loss function can be one of many forms as long as it is differentiable with respect to* $$ \textbf{z} $$. 
</span>

### Backwards Automatic Differentiation  

<p align="center">
	<img src="function_dag_1.svg"
	title="worked example"
	width="650" height="650" />
</p>
<center> Figure 2. a DAG representing a math function, arrows pointing towards the forward direction.  </center>


let's say $$ x_1 = 2, x_2 = 5 $$

$$
\begin{array}{|l||c|c|}
\hline \\
\text{Forward Trace} & \text{Backwards trace} \\ 
\hline  \\
v_{-1} = x_1 = 2 \\
v_0 = x_2 = 5  \\ 
\hline \\
v_1 = ln(v_{-1}) = ln2 \\
v_2 = v_{-1} \cdot v_{0} = 2 \times 5 \\
v_3 = sin(v_0) = sin(5) \\
v_4 = v_1 + v_2 = ln2 + 10 \\
v_5 = v_4 - v_3 = ln2 + 10 - sin(5) \\
\hline \\
y = v_5  &  \frac{ \partial v_5}{ \partial v_5} = \frac{ \partial y}{ \partial y} = 1 \\
\hline
\end{array}  
$$




<p align="center">
	<img src="fwd_backwd.svg"
	title="activations and gradient flow"
	width="650" height="650" />
</p>
<center> Figure 3. Activations flowing forward and gradient flowing backwards  [source][ML-NOTES-2]</center>




<p align="center">
	<img src="graph0.svg"
	     title="abstract-example"
	     width="650" height="650" />
</p>
<center> <em> Figure 4. Gradient flow for a single computational unit in a Neural Network </em> </center>

<p align="center">
	<img src="graph1.svg"
	title="multiply unit"
	width="650" height="650" />
</p>
<center> Figure 5. PCAN View CAN trace </center>

<p align="center">
	<img src="graph2.svg"
	title="worked example"
	width="650" height="650" />
</p>
<center> Figure 6. PCAN View CAN trace </center>


<!-- <p align="center">
	<img src="function_dag_1.svg"
	title="worked example"
	width="650" height="650" />
</p>
<center> Figure 7. PCAN View CAN trace </center>

 -->

 [Jacobian][JACOBIAN-MATRIX]


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
[JACOBIAN-MATRIX]: https://math.stackexchange.com/q/1127350

<!-- Latex in markdown -->
<script src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML" type="text/javascript"></script>
<!-- $$ \nabla_\boldsymbol{x} J(\boldsymbol{x}) $$ -->
