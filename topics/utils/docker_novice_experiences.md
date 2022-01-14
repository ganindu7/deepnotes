---
layout: default
title: Stranger to docker
parent: Utilities
# permalink: /topics/utils/communication_setup
# permalink: /topics/comms_setup   # adding a permalink broke the internal linking to a topic 
# nav_order: 2
---
Last Updated : 6/11/2021
# Why [Docker](https://en.wikipedia.org/wiki/Docker_(software))
{: .no_toc }

With a comparatively small overhead docker simply give the luxury of virtually attaching a workspace to an isolated (potentially even ephemeral) sandbox engine for developers.
{: .fs-5 .fw-300}
When things are ready for deployment we can create lean, robust and portable containers. 
{: .fs-5 .fw-300}
<!--
{: .no_toc .text-delta}
1. TOC
{:toc}
-->

<span style="background-color:LightYellow">
Docker is not a virtual machine, with hardware/firmware providers presenting their own runtimes such as the [nvidia-container-runtime](https://developer.nvidia.com/nvidia-container-runtime) they can be more closer to the machine while still having the advantages containers provide.
{: .fs-4 .fw-300}
<span style="background-color:LightYellow">
Also Docker is not the only container solution, there are others too. e.g. [LXD](https://linuxcontainers.org/), [podman](https://podman.io/), [kubernetes](https://kubernetes.io/) and [OpenVz](https://openvz.org/)
{: .fs-4 .fw-300}


--- 
begin