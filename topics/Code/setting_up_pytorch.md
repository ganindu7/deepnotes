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


#### Using Docker 

Dockerfile 
```
FROM nvcr.io/nvidia/pytorch:24.04-py3

# Create user and group 'ganindu'
RUN groupadd -g 1000 ganindu && \
    useradd -rm -d /home/ganindu -s /bin/bash -g ganindu -G sudo -u 1000 ganindu

# Set the default user for the container
USER ganindu
WORKDIR /home/ganindu
```

Compose file 
```
version: '3.8'

services:
  inference:
    build: 
      context: ./
      dockerfile: Dockerfile
    # image: custom-pytorch:24.04-py3  # Updated to use the locally built image
    container_name: docker-pytorch
    stdin_open: true  # Equivalent to -i (interactive)
    tty: true          # Equivalent to -t (terminal)
    hostname: docker.dgx
    runtime: nvidia
    ipc: host
    # shm_size: '8gb'
    network_mode: "bridge"  # Using bridge network for networking
    environment:
      - DISPLAY=${DISPLAY}
      - ROOTDIR=/RDR/Home/data
      - NVIDIA_VISIBLE_DEVICES=all
      - NVIDIA_DRIVER_CAPABILITIES=all

    volumes:
      - type: bind
        source: /tmp/.X11-unix
        target: /tmp/.X11-unix
      - type: bind
        source: $HOME/.Xauthority
        target: /root/.Xauthority
        read_only: false
      - type: bind
        source: ./dws
        target: /home/ganindu/workspace/dws  
      - type: bind
        source: $PWD
        target: /home/ganindu/workspace

```


[JETSON-URL]: https://developer.nvidia.com/embedded/jetson-agx-xavier-developer-kit
[PYTORCH]: https://pytorch.org
[NVIDIA-PYTORCH-GUIDE]: https://forums.developer.nvidia.com/t/pytorch-for-jetson-version-1-9-0-now-available/72048

