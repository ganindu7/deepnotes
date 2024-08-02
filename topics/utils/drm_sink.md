---
layout: default
title: Setting up Direct Rendering for Jetson
nav_order: 2 
permalink: /topics/utils/drm_sink_jetson
parent: Utilities
---

## Setting up the Jetson for DRMSINK output 

<span style="background-color:LightYellow">
Nvidia guide for this operation for L4T_R36.2 can be found [here](https://docs.nvidia.com/jetson/archives/r36.2/DeveloperGuide/SD/Multimedia/AcceleratedGstreamer.html#video-playback-with-gstreamer-1-0)
</span>


Summary:

1. Stop the display manager

```
sudo systemctl stop gdm
```

To disable the displaymanager (this will presisit until it is turned back on)
```
sudo systemctl disable  display-manager
```

2. Terminate the current session (if needed)
```
sudo loginctl terminate-seat seat0
```

3. Load the DRM driver 
```
sudo modprobe nvidia-drm modeset=1
```

4. Run a test pipeline 
```
gst-launch-1.0 videotestsrc ! nvvidconv ! 'video/x-raw(memory:NVMM), format=NV12' ! nvdrmvideosink
```

