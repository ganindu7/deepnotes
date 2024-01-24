---
layout: default
title: Hardware Tips
nav_order: 4
permalink: /topics/utils/hardware
parent: Utilities
---

## Tips for hardware
this is for stuff related to setting up device drivers for MIPI-CSI cameras with a FPD-Link-III hub 

## determining which ports are connected to a camera 

When using LVDS in addition to the CSI lines being bound we need to know which camera is connected to a specific hardware port. 
we can do this in a few ways 

### use `dmesg`

look at the example below 

```
ganindu@ai-ecu:~$ sudo dmesg | grep bound
[    8.697806] tegra-camrtc-capture-vi tegra-capture-vi: subdev 13e10000.host1x:nvcsi@15a00000- bound
[    8.707074] tegra-camrtc-capture-vi tegra-capture-vi: subdev 13e10000.host1x:nvcsi@15a00000- bound
[    8.715621] tegra-camrtc-capture-vi tegra-capture-vi: subdev 13e10000.host1x:nvcsi@15a00000- bound
[    8.724641] tegra-camrtc-capture-vi tegra-capture-vi: subdev 13e10000.host1x:nvcsi@15a00000- bound
[   18.139076] tegra-camrtc-capture-vi tegra-capture-vi: subdev d3-imx390 33-0021 bound
[   18.243614] tegra-camrtc-capture-vi tegra-capture-vi: subdev d3-imx390 43-0021 bound
```

Breaking down the output: In this example only two cameras are connected. the first four lines let us know us about the video input devices being connected to the nvidia csi interface.
the last two lines refer to which LVDS ports the devices are connected to. 

```
tegra-camrtc-capture-vi tegra-capture-vi: subdev d3-imx390 33-0021 bound
tegra-camrtc-capture-vi tegra-capture-vi: subdev d3-imx390 43-0021 bound
```

the entries 33-0021 may refer to the SerDes ic and the port in the IC.  

Now let's see how these device tree bindings are associated with the familiar /dev/VideoN interfaces we may use to assign cameras programmaaticly to our foftware functionalities. in this case we can use the `v4l2-ctl --list-devices` command.


```
ganindu@ai-ecu:~$ v4l2-ctl --list-devices
NVIDIA Tegra Video Input Device (platform:tegra-camrtc-ca):
	/dev/media0

vi-output, d3-imx390 43-0021 (platform:tegra-capture-vi:4):
	/dev/video1

vi-output, d3-imx390 33-0021 (platform:tegra-capture-vi:5):
	/dev/video0

```

here you can see that the device tree element

43-0021 is connected to /dev/video1 interface and vice versa. 






