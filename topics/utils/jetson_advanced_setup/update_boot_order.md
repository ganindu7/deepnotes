---
layout: default
title: Jetson custom boot order
nav_order: 2 
permalink: /topics/utils/jetson_advanced/tweak_boot
parent: JetsonAdvanced
grand_parent: Utilities
---

<!-- ### building a media server  -->

<!-- This article is inspired by [a blog post](https://developer.nvidia.com/blog/building-multi-camera-media-server-ai-processing-jetson/) on the Nvidia developers forum. 

 -->

 <br/> 
 <br/>

 &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp; <mark>  This section is under development or not public. </mark>

Date: 21/06/2022

[Source Document][L4T-3721-BOOT-CONFIG]

The bootloader must be loaded from the internal [eMMC][eMMC]. the Kernel and the DTB can be loaded from the can be loaded from a GPT formatted SD card, USB drive, NVMe SSD or a network drive via DHCP/TFTP.

The CBoot Boot options (CBO) is a device tree that can be used to set certain boot configuration options such as device priority and IP addresses used to boot from a network. 
The name of the CBO device tree node is `boot-configuration`



Thr `cbo.dts` file can be found in the `Linux_for_Tegra/bootloader` directory
```
/*
 * Copyright (c) 2018-2020, NVIDIA CORPORATION.  All rights reserved.
 *
 * NVIDIA CORPORATION and its licensors retain all intellectual property
 * and proprietary rights in and to this software, related documentation
 * and any modifications thereto.  Any use, reproduction, disclosure or
 * distribution of this software and related documentation without an express
 * license agreement from NVIDIA CORPORATION is strictly prohibited.
 */

/dts-v1/;

/ {
    compatible = "nvidia,cboot-options-v1";
    boot-configuration {
        /* nvme boot-order can be in "nvme:C<n>", "nvme:pcie@<addr>", or "nvme" format */
        boot-order = "sd", "usb", "nvme", "emmc", "net";
        tftp-server-ip = /bits/ 8 <192 168 0 1>;
        dhcp-enabled;
        /* remove dhcp-enabled if uncommenting static-ip configuration */
        /* static-ip = /bits/ 8 <0 0 0 0>;
         * ip-netmask = /bits/ 8 <255 255 255 0>;
         * ip-gateway = /bits/ 8 <0 0 0 0>;
         */
    };
};

```



In some cases we may need to modify to force a certain order. Here I've given the `emmc` priority over `nvme` 

```
 boot-order = "sd", "usb", "emmc", "nvme", "net";
```

Now that we've modified the DTS we need to rebuild the DTB.

Follow these steps

<ol>
<li>Locate and <code>cd</code> to the <code> $BSPDIR/Linux_for_Tegra/bootloader/ </code> folder where the <code>cbo.dts</code> file sits.</li> (HINT: <code>$HOME/nvidia/nvidia_sdk/...</code>)

<li> Modify the <code>cbo.dts</code> file as necessary as shown in the example above <a href="https://docs.nvidia.com/jetson/archives/l4t-archived/l4t-3271/index.html#page/Tegra%20Linux%20Driver%20Package%20Development%20Guide/flashing.html">more info here.</a></li>
<li> Enter the command to convert the DTS to DTB (this is run from the same bootloader directory as in step 1.)</li>
<code> dtc -I dts -O dtb -o cbo.dtb cbo.dts </code> 
the <code> dtc </code> executable can be found in <code> $BSPDIR/Linux_for_Tegra/kernel/dtc </code>
<li> Flash the partition CPUBL-CFG. using the flash script </li>
</ol> 


### Flashing the device with changes 

<!-- `sudo ./flash.sh -k kernel-dtb -d ./kernel/dtb/d3-xavier-16x-fpdlink.dtb jetson-xavier mmcblk0p1` -->

click [here][L4T-3721-FLASH-GUIDE] for the original document. apart from the file systems prerequisites listed in the original document when running the usb cable must be connected to the 
recovery usb port of the Jetson xavier. 

The `flash.sh` helper script exists to flash the board with the bootloader and the kernel and optionally flash the root file system to internal or external
storage device. 

`sudo ./flash.sh -r -k CPUBL-CFG -d ./bootloader/cbo.dtb jetson-xavier mmcblk0p1`





## Back up and restore a Jetson Device:

tolls for backup and restore are located in `/Linux_for_Tegra/tools/backup-restore/` instructions are in the README file in the same directory. (Need to install Secureboot)



## boot console 

[see boot messages](https://developer.ridgerun.com/wiki/index.php?title=Xavier%2FIn_Board%2FGetting_in_Board%2FSerial_Console)





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