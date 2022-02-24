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

### Automatic Differentiation  

Automatic differentiation does a lot of heavy lifting to make backpropagation work. A good working understanding of this tool is essential to understand backprop. Let's jump in!! figure 2 is a graph representation of the 
function $$ f(x_1, x_2) = ln(x_1) + x_1x_2 -sin(x_2) $$, Each node in the graph is an intermediate variable that acts as a building block for the function.



<p align="center">
	<img src="DAG-backwards-AD.svg"
	title="Backwards Automatic differentiation, DAG"
	width="650" height="650" />
</p>
<center> Figure 2. A math function drawn as a graph, <a href="https://arxiv.org/abs/1502.05767"> <em>arXiv:1502.05767 </em></a>  </center>

Notice the $$ v_i $$ and $$ \widetilde{v_i} $$ pairs in Figure 2, *(first pay attention to direct links like $$ v_4 $$ to $$ v_5 $$ or converging links like $$ v_1 \text{ to } v_4 \text{ and } v_2 \text{ to } v_4. $$, when multiple links go out from one node, like $$v_{-1}$$ to $$v_1$$ and $$v_2$$ or $$v_0$$ to $$v_2$$ and $$v_3$$ things are slightly different! we will discuss this in the sections below using Figure 2 and Table 1)* 

We already know that $$v_i$$ is an intermediate variable. $$ \widetilde{v_i} $$ (called an adjoint to $$v_i$$) is the sensitivity of "$$ f $$" to changes in $$v_i$$. 

$$  \widetilde{v_i} = \frac{\partial y_j}{ \partial v_i} \label{1} \qquad (1)$$

After defining $$\widetilde{v_i}$$ our motivation now turns to compute the contribution of each intermediate variable $$ v_i $$ to the change in output $$ f $$. The backwards Automatic Differentiation method that we use to achieve this has two passes. 

1.	**The Forward pass**: Evaluating the function by calculating intermediate variables, and storing some information (for each intermediate variable) needed for the calculations in the subsequent backwards pass.  
2.	**The Backwards pass**: calculating the $$ \widetilde{v_i} $$ using the backpropagated values and the stored values from the forward pass.

The **sensitivity of $$f$$** to a change in an intermediate variable can be expressed mathematically using partial derivatives! Lets take $$v_0$$ from *Figure 2* as an example.

$$ \frac{\partial f}{\partial v_0} = \frac{\partial f}{\partial v_2}\frac{\partial v_2}{\partial v_0} + \frac{\partial f}{\partial v_3}\frac{\partial v_3}{\partial v_0} \; \; \text{  or  } \; \; \widetilde{v_2}\frac{\partial v_2}{\partial v_0} + \widetilde{v_3}\frac{\partial v_3}{\partial v_0} \qquad  (2)$$

By examining *Figure 2* we can see that the gradient inflow to $$v_0$$ during backpropagation is conducted through two channels *(these are the links from $$v_2$$ and $$v_3$$)*. The Equation $$(2)$$ shows how these two inflowing gradients stack up to construct **$$ \widetilde{v_0} =  \frac{\partial f}{\partial v_0} $$**. 

In *Table 1* this is broken down to two stages, $$ \; \widetilde{v_{02}} \; $$ and $$ \; \widetilde{v_{01}}$$.  

$$
\widetilde{v_{02}} = \frac{\partial v_3}{ \partial v_0} \cdot \widetilde{v_3} \quad \text{and} \quad \widetilde{v_{01}} = \frac{\partial v_2}{v_0} \cdot \widetilde{v_2} 
$$

Then accumulated as shown in Equation $$(2)$$.

$$
\widetilde{v_{0}} = \widetilde{v_{01}} + \widetilde{v_{02}} 
$$

We can now solidify this understanding by examining *Table 1*, In this example we have used $$x_1 = 2$$ , $$x_2$$ = 5 and performed a forward pass followed by a backwards pass with value substitutions to 
reach a numerical answer. Note that both $$\widetilde{x_1}$$ and $$\widetilde{x_2}$$ are calculated during the same pass! 

$$
\begin{array}{|l||l|}
\hline \\
\text{Forward Trace} & \text{Backwards trace for adjoint variables} \\ 
\hline  \\
\downarrow v_{-1} = x_1 = 2 & \uparrow  \widetilde{x_1} = \widetilde{v_{-1}} = 5.5 \\
\downarrow v_0 = x_2 = 5 & \uparrow   \widetilde{x_2} = \widetilde{v_0} \approx 1.7163\\ 
\hline \\
& \uparrow \widetilde{v_{-1}} = \widetilde{v_{-11}} + \widetilde{v_{-12}} = 5.5\\ 
& \uparrow \widetilde{v_{-12}} = \frac{\partial v_2}{\partial v_{-1}} \cdot \widetilde{v_2} = [\frac{\partial  }{\partial v_{-1}}(v_{-1} \cdot v_0)] \widetilde{v_2} = v_0 \cdot \widetilde{v_2} = 5\\ 
& \uparrow \widetilde{v_{-11}} = \frac{\partial v_1}{\partial v_{-1}} \cdot \widetilde{v_1} = \frac{ \partial}{ \partial v_{-1}}ln(v_{-1}) = \frac{1}{v_{-1}} \cdot \widetilde{v_1} = 0.5\\ 
& \uparrow \widetilde{v_{0}} = \widetilde{v_{01}} + \widetilde{v_{02}} \approx 1.7163 \\
& \uparrow \widetilde{v_{02}} = \frac{\partial v_3}{ \partial v_0} \cdot \widetilde{v_3} =  \frac{\partial}{ \partial v_0}sin(v_0) \cdot \widetilde{v_3} = cos(v_0) \times -1  \approx -0.2836\\
& \uparrow \widetilde{v_{01}} = \frac{\partial v_2}{v_0} \cdot \widetilde{v_2} = [\frac{\partial  }{\partial v_0}(v_{-1} \cdot v_0)] \widetilde{v_2} = v_{-1} \cdot \widetilde{v_2} = 2 \\
\downarrow v_1 = ln(v_{-1}) = ln2 & \uparrow \widetilde{v_1} = \frac{ \partial(v_1 + v_2)}{\partial v_1} \cdot \widetilde{v_4} = 1  \\
\downarrow v_2 = v_{-1} \cdot v_{0} = 2 \times 5 & \uparrow \widetilde{v_2} = \frac{ \partial(v_1 + v_2)}{\partial v_2} \cdot \widetilde{v_4} = 1 \\
\downarrow v_3 = sin(v_0) = sin(5)  &  \uparrow \widetilde{v_3} = \frac{ \partial v_5}{ \partial v_3} \cdot \frac{ \partial f}{ \partial v_5} =  (\frac{ \partial v_4}{ \partial v_3} - \frac{ \partial v_3}{ \partial v_3}) \cdot \widetilde{v_5} = -1 \\
\downarrow v_4 = v_1 + v_2 = ln2 + 10 & \uparrow \widetilde{v_4} = \frac{ \partial v_5}{ \partial v_4} \cdot \frac{ \partial f}{ \partial v_5} =  (\frac{ \partial v_4}{ \partial v_4} - \frac{ \partial v_3}{ \partial v_4}) \cdot \widetilde{v_5} = 1 \\
\downarrow v_5 = v_4 - v_3 = ln2 + 10 - sin(5)  \\ 
\hline \\
\downarrow y = v_5 \approx 11.6521 & \uparrow \widetilde{v_5} = \frac{ \partial f}{ \partial v_5} = \frac{ \partial v_5}{ \partial v_5} = 1 \\
\hline
\end{array}  
$$
<center> Table 1. Forward and Backward traces for an Autodiff function, <a href="https://arxiv.org/abs/1502.05767"> arXiv:1502.05767</a> (angles are in radians) </center>

The function used in *Table 1* ($$ ln(x_1) + x_1x_2 - sin(x_2) $$) is a scalar valued function. All the adjoints we calculated are in respect to the output of this scalar valued function. If we maintain the current structure and happened to evaluate a tensor valued function we will have to maintain tensor intermediate variables (and their adjoints) and backpropagate multiple times (once per each output) per each single forward pass. I think this is an uncommon example where in the Machine Learning context we train the same model blueprint to detect various things simultaneously. 

In the general case usually we have a large number of inputs and a scalar valued Objective function (an Error function, $$\mathbf{E}$$) at the end that is based on the comparison of the model output and the training label value.

The Jacobain for a function $$f:\mathbb{R}^n \rightarrow \mathbb{R}^m $$ with input $$x$$ and output $$y$$ such that $$f(x_1, \dots, x_n) = [y_1, \dots, y_m]$$ can be written as 

$$
J = \begin{bmatrix}
    \frac{\partial y_1}{\partial x_1} & 
    \dots & 
  \frac{\partial y_1}{\partial x_n} 

                  \\
   \vdots & 
   \ddots & 
 \vdots 
\\
   \frac{\partial y_m}{\partial x_1} & 
   \dots & 
\frac{\partial y_m}{\partial x_n}
\end{bmatrix}_{m \times n}  \qquad  (3)
$$


When we perform backwards AD we get 

$$
\begin{bmatrix}

\widetilde{x_1} \\
\vdots \\
\widetilde{x_n} 


\end{bmatrix}
 

 = \begin{bmatrix}

\frac{\partial yj}{\partial {x_1}} \\
\vdots \\
\frac{\partial yj}{\partial {x_n}}


\end{bmatrix}_{n \times 1}
$$

If we compare this with the $$J$$ in equation $$3$$ we can see that this is a column of $$J^T$$ (transpose of the Jacobian) Therefore we can see that with reverse AD we can easily get 

$$
J^T = \begin{bmatrix}
    \frac{\partial y_1}{\partial x_1} & 
    \dots & 
  \frac{\partial y_m}{\partial x_1} 

                  \\
   \vdots & 
   \ddots & 
 \vdots 
\\
   \frac{\partial y_1}{\partial x_n} & 
   \dots & 
\frac{\partial y_m}{\partial x_n}
\end{bmatrix}_{n \times m}  \qquad  (4)
$$


<!-- The number of operations needed to calculate the Jacobian is proportional to *$$m$$*. This is a huge benefit when we have less outputs compared to inputs like Neural Networks.  This makes Backwards AD more efficient when compared to forward AD (more details on that can be found [here][ML-NOTES-2] at sections $$3.1$$ and $$3.2$$)
 -->



<p align="center">
	<img src="fwd_backwd.svg"
	title="activations and gradient flow"
	width="650" height="650" />
</p>
<center> Figure 3. Activations flowing forward and gradient flowing backwards  <a href="https://arxiv.org/abs/1502.05767"> <em>arXiv:1502.05767 </em></a>  </center>




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
