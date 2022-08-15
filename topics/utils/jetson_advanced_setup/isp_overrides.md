---
layout: default
title: ISP overriding for cameras (Tested on AGX XAVIER) 
nav_order: 2 
permalink: /topics/utils/jetson_advanced/isp_override
parent: JetsonAdvanced
grand_parent: Utilities
---


 <br/> 
 <br/>

 &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp; <mark>  This section is under development or not public. </mark>

Date: 22/06/2022

ISP override symlinks can be placed in the `/var/nvidia/nvcam/settings` directory! 

eg.  If your imx390 camera is in camera port 2

`ln -sT imx390.isp d3_2_imx390.isp`


if your omnivision camera is in port 3 

`ln -sT ov10640.isp d3_3_ov10640.isp`


you can have overrides for the same port lets say you can have imx390 on port 3 as well 

you can have both

`ln -sT imx390.isp d3_3_imx390.isp`
`ln -sT ov10640.isp d3_3_ov10640.isp`




 *Click [here][ERRORS-SUGGESTIONS] to report Errors, make Suggestions or Comments!*

[L4T-3721-BOOT-CONFIG]: https://docs.nvidia.com/jetson/archives/l4t-archived/l4t-3271/index.html#page/Tegra%20Linux%20Driver%20Package%20Development%20Guide/flashing.html
[L4T-3721-FLASH-GUIDE]: https://docs.nvidia.com/jetson/archives/l4t-archived/l4t-3271/index.html#page/Tegra%20Linux%20Driver%20Package%20Development%20Guide/flashing.html#
[JETSON-URL]: https://developer.nvidia.com/embedded/jetson-agx-xavier-developer-kit
[ERRORS-SUGGESTIONS]: https://github.com/ganindu7/deepnotes/issues
[eMMC]: https://semiconductor.samsung.com/newsroom/tech-blog/introducing-emmc-5-1-the-next-step-in-relentless-flash-innovation/

<!-- Latex in markdown -->
<script src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML" type="text/javascript"></script>
<!-- $$ \nabla_\boldsymbol{x} J(\boldsymbol{x}) $$ -->

<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/highlight.js/11.5.0/styles/default.min.css">
<script src="https://cdnjs.cloudflare.com/ajax/libs/highlight.js/11.5.0/highlight.min.js"></script>
<script>hljs.initHighlightingOnLoad();</script>