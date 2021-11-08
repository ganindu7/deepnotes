---
layout: default
title: setting up PyTorch
nav_order: 2 
# permalink: /topics/utils/target_execute
parent: Utilities
---

## Setting up [PyTorch][PYTORCH] in [Nvidia Jetson][JETSON-URL]


for Nvidia JetPack 4.4 


Note: I got the error below when attempting to install `torchaudio` but I didn't chase after it because audio isn't a part of our project at the moment. 
```

ERROR: Could not find a version that satisfies the requirement torchaudio (from versions: none)
ERROR: No matching distribution found for torchaudio

```

Below is a slightly modified version of the Nvidia's [guide][NVIDIA-PYTORCH-GUIDE] 

#### Instructions for Installing:

Note: I am using a python 3.6.15 virtualenv. check [this](setting_up_pyenv.md#fixing-broken-python-wheel) in case you encounter `pip` related problems. 


```

wget https://nvidia.box.com/shared/static/h1z9sw4bb1ybi0rm3tu8qdj8hs05ljbm.whl -O torch-1.9.0-cp36-cp36m-linux_aarch64.whl
sudo apt-get install python3-pip libopenblas-base libopenmpi-dev 
pip install Cython

```

Then you can proceed to install torch.

```
pip install numpy==1.19.4 torch-1.9.0-cp36-cp36m-linux_aarch64.whl torchvideo

```

To use latest  Numpy.

```
pip install pip install numpy torch-1.9.0-cp36-cp36m-linux_aarch64.whl torchvideo

```
if you have numpy 1.19.5 and get a runtime error please try referring to this [forum answer](https://forums.developer.nvidia.com/t/illegal-instruction-core-dumped/165488/16) athat suggests adding `export OPENBLAS_CORETYPE=ARMV8` to your `.bashrc` file


[JETSON-URL]: https://developer.nvidia.com/embedded/jetson-agx-xavier-developer-kit
[PYTORCH]: https://pytorch.org
[NVIDIA-PYTORCH-GUIDE]: https://forums.developer.nvidia.com/t/pytorch-for-jetson-version-1-9-0-now-available/72048

