---
layout: default
title: Custom parsers 
# nav_order: 2 
# permalink: /topics/utils/target_execute
parent: Utilities
---

## DeepStream custom parsers [PyTorch][PYTORCH]
<span style="background-color:LightGreen">
Created : 20/01/2022 | on Linux: 5.4.0-91-generic <br />
Updated: 20/01/2022 | on Linux: 5.4.0-91-generic <br />
Status: Draft
</span>

<span style="background-color:LightYellow"> [**previous topic 1: Starting Development with PyTorch**](../pytorch_walkthrough#Starting-Development-with-PyTorch)  </span> <br  />
<span style="background-color:LightYellow"> [**previous topic 2: Tensors and Data Handling with PyTorch**](../tensors_and_model_input#Tensors-and-Data-Handling-with-PyTorch)  </span>
<span style="background-color:LightYellow"> [**previous topic 3: Building a network in eager mode**](../eager_network_building_blocks#Building-a-network-with-PyTorch)  </span>

### Data

### useful links
[Gst-nvinfer File Configuration Specifications](https://docs.nvidia.com/metropolis/deepstream/dev-guide/text/DS_plugin_gst-nvinfer.html)


convert onnx model to trt.engine [link and more info](https://github.com/NVIDIA/TensorRT/blob/73a7cccd6224035d9008cdde5bc35db77212c402/tools/tensorflow-quantization/examples/README.md)

```
trtexec --my_onnx model.onnx --fp16 --saveEngine my_onnxmodel_created_tensorrt.engine -v
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

<!-- Latex in markdown -->
<script src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML" type="text/javascript"></script>
<!-- $$ \nabla_\boldsymbol{x} J(\boldsymbol{x}) $$ -->