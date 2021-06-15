---
layout: default
title: Updating C++ compiler
nav_order: 2 
# permalink: /topics/utils/target_execute
parent: Utilities
---

## selecting a custom gcc/g++ compiler 

```bash

$sudo apt-get install build-essential

# to install a gcc. then we want to install the compiler we want (gcc/g++ 8 in this case)

$ sudo apt-get install gcc-8 g++-8

# then check version to make sure what it active:

$ gcc --version 
$ g++ --version 

then we want to create an update alternatives entry 

sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-7 7
sudo update-alternatives --install /usr/bin/g++ g++ /usr/bin/g++-7 7
sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-8 8
sudo update-alternatives --install /usr/bin/g++ g++ /usr/bin/g++-8 8


then we can open a selector menu and choose e.g for gcc


sudo update-alternatives --config gcc 

```