---
layout: default
title: Tensorflow Notes 
# nav_order: 2 
# permalink: /topics/utils/target_execute
parent: Development
---

## Insert Topic Here [PyTorch][PYTORCH]
<span style="background-color:LightGreen">
Created : 20/01/2022 | on Linux: 5.4.0-91-generic <br />
Updated: 20/01/2022 | on Linux: 5.4.0-91-generic <br />
Status: Draft
</span>

<span style="background-color:LightYellow"> [**previous topic 1: Starting Development with PyTorch**](../pytorch_walkthrough#Starting-Development-with-PyTorch)  </span> <br  />
<span style="background-color:LightYellow"> [**previous topic 2: Tensors and Data Handling with PyTorch**](../tensors_and_model_input#Tensors-and-Data-Handling-with-PyTorch)  </span>
<span style="background-color:LightYellow"> [**previous topic 3: Building a network in eager mode**](../eager_network_building_blocks#Building-a-network-with-PyTorch)  </span>

### Typical Workflow 

Teensorflow 1 is based on a graph style programing model. A typical top level workflow can be summarised into 4 key slots. 

1. Defining graph elements.
2. Defining connectivty (layout the graph).
3. Defining functionality (algorithmic components). 
4. Running the graph and interpriting results.


### Graph Elements 

Basic graph element objects are 

* Constants    : Holds the value once declared. 
* Placeholders : Placeholders need placement at somepoint using a feed dictionary. 
* Variables    : Variables need proper initialisation and can change within graph execution.  


Within Tensorflow these data elements are usually called Tensors. We can say that Tensor attributes vary depending on the element types. This differents can be a mix of qualititative and quantative attributes. 

for example, A constant, placeholder or a Variable  can be of different type and differnt shape, we could also designate where they reside in computer memory.















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

<!-- Latex in markdown -->
<script src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML" type="text/javascript"></script>
<!-- $$ \nabla_\boldsymbol{x} J(\boldsymbol{x}) $$ -->
