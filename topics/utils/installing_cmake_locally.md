---
layout: default
title: Installing CMAKE
nav_order: 2 
# permalink: /topics/utils/target_execute
parent: Utilities
---

## Installing CMAKE Locally 

Having an utpo date [cmake](https://cmake.org) is quite useful when building code. 

<span style="background-color:LightYellow">
Extra Tip: Useful for packages such as YCM </span>


From the [downloads](https://cmake.org/download/) page, get the latest cmake and extract to your desired location. 

then `cd` into the directory 


```bash
tar -xf cmake*.tar.gz
cd cmake*
```

I set the install prefix to local but I's upto you where you keep the installation files. 


```bash
./configure --prefix=$HOME/.local
make 
make install
```

reboot 

these steps will install cmake locally. 

[source](https://pachterlab.github.io/kallisto/local_build.html)




