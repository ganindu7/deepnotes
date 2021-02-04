---
layout: default
title: Execute on target device
nav_order: 2 
permalink: /topics/utils/execute_on_target
parent: Utilities
---

## Running code in the Jetson
Once the code is built you can either direcly log on to the target device and execute the code or do it over ssh. If the executable has visual elements and if you hasn't [forwarded the X session at the begining](https://en.wikipedia.org/wiki/SSH_(Secure_Shell)#Uses) (if your target has a disply attached) you can attach the output to the target's display by typing `$ export DISPLAY=:1` and then running the executable.


