---
layout: default
title: Vedio device control 
nav_order: 2 
# permalink: /topics/utils/target_execute
parent: Utilities
---

## Insert Topic Here [PyTorch][PYTORCH]
<span style="background-color:LightGreen">
Created : 23/02/2023 | on Linux: 5.4.0-139-generic <br />
Updated: 23/02/2023 | on Linux: 5.4.0-139-generic <br />
Status: Draft
</span>


### Using virtual video devices to redirect streams.

The aim of this exercise is to take a network stream (usually defined with an *IP* and a *PORT* with some information about the stream attributes) and place it in a virtual video device node (*or a video loopback device*) so it can be accessed similar to as a camera. e.g. share LVDS camera video via microsoft teams if you want to debug a camera over a video call.


#### Steps 

1. Create a virtual device. 
2. Link up the stream to the virtual device created in *STEP 1*.
3. Test with a simple webcam application.

##### Prerequisites 

Install *[v4l-utils](https://manpages.ubuntu.com/manpages/xenial/man1/v4l2-ctl.1.html)*
```
sudo apt install v4l-utils
```

###### Method 1

install `v4l2loopback` kernel modules (if you may get `modprobe: FATAL: Module v4l2loopback not found in directory /lib/modules/x.x.x-xxx-generic
` you may want to try the second method)

```
sudo apt install v4l2loopback-dkms
```
###### Method 2 

This time we install the module by compiling it. 
first get rid if any previous artifacts od `v4l2loopback` with `sudo apt purge v4l2loopback-dkms`

```
git clone git@github.com:umlaeute/v4l2loopback.git
```


<br/>
Make sure you have met the [dependencies](https://github.com/umlaeute/v4l2loopback#dependencies)
<br/>

follow the build [instructions](https://github.com/umlaeute/v4l2loopback#dependencies)

Note: run `make clean` if you have already run make before (in case of kernel change)

```
make && sudo make install
sudo depmod -a
```

***Testing Installation***

In the terminal  

```
sudo modprobe v4l2loopback
```

If you want to assign a number and a  name;

```
sudo modprobe v4l2loopback video_nr=10 card_label="virtual_lvds_camera"
```


exclusive caps (for input devices)
```
sudo modprobe v4l2loopback exclusive_caps=1
```

suggested command:

more info on the options can be found in the [repository](https://github.com/umlaeute/v4l2loopback) 

```
sudo modprobe v4l2loopback video_nr=10 card_label="virtual_lvds_camera" exclusive_caps=1
```

With this method you will also get the `v4l2loopback-ctl` tool (compiled and added to your search path with the `make install step`). This will give you the ability to control FPS, image timeouts, placeholder picture, and setup caps (gstreamer caps)



##### 1. Create a virtual device. 

Create a device with 
device number:10
label: virtual_lvds_camera

```
sudo modprobe v4l2loopback video_nr=10 card_label="virtual_lvds_camera" exclusive_caps=1

```

#### 2. Test the device 

list devices 
```
 v4l2-ctl --list-devices
```

check attributes
```
v4l2-ctl --device /dev/video0 --all
```
you may want to change the number (`0` uses in the example) depending on the `video_nr` value used.


##### 3. Creating and Accessing streams 

Create a stream in a jetson device with `STREAM_PORT` and `STREAM_IP` set to the receiving device (i.e IP address and a port of the device we want to playback or create a virtual video device node)

```
gst-launch-1.0 v4l2src device=/dev/video0  !  nvvideoconvert  ! nvv4l2h264enc insert-sps-pps=true  iframeinterval=5 control-rate=1 maxperf-enable=true name=encoder ! h264parse config-interval=5 ! udpsink port=$STREAM_PORT host=$STREAM_IP sync=false

```

Then from the receiving device open a terminal and run the following to test the stream.

```
 gst-launch-1.0 udpsrc address=$STREAM_IP port=$STREAM_PORT ! h264parse ! avdec_h264 ! videoconvert ! autovideosink
```

At the time of writing there is a bug in `v4l2sink` that does not copy new buffers. one workaround is too add a `tee`
please refer to [this issue](https://github.com/umlaeute/v4l2loopback/issues/519) 

```
gst-launch-1.0 videotestsrc ! tee name=t ! v4l2sink device=/dev/video0 

```

However this is not an ideal fix as this causes the frame to freeze.

Next we can create a pipeline with a preview stream and a `v4l2stream` 


setting caps for the v4l2 loopback device

```
v4l2loopback-ctl set-caps "video/x-raw, format=(string)YUY2, width=1280, height=720, fps=30/1" /dev/video0
```


```
gst-launch-1.0 udpsrc address=$STREAM_IP port=$STREAM_PORT ! h264parse ! avdec_h264 ! videoconvert ! tee name=t ! autovideosink   t. ! videoconvert  !   v4l2sink device=/dev/video0  async
```





<br />

<span style="background-color:LightYellow"> Check the [**next topic**](../pytorch_walkthrough#Starting-Development-with-PyTorch)  </span>

Source: [PyTorch Tutorial][PyTorch-Tutorial]

---
*Click [here][ERRORS-SUGGESTIONS] to report Errors, make Suggestions or Comments!*

[JETSON-URL]: https://developer.nvidia.com/embedded/jetson-agx-xavier-developer-kit
[PYTORCH]: https://pytorch.org
[NVIDIA-PYTORCH-GUIDE]: https://forums.developer.nvidia.com/t/pytorch-for-jetson-version-1-9-0-now-available/72048
[PyTorch-Tutorial]: https://pytorch.org/tutorials/beginner/basics/quickstart_tutorial.html
[FashonMnist-dataset]: https://github.com/zalandoresearch/fashion-mnist
[ERRORS-SUGGESTIONS]: https://github.com/ganindu7/deepnotes/issues

<!-- Latex in markdown -->
<script src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML" type="text/javascript"></script>
<!-- $$ \nabla_\boldsymbol{x} J(\boldsymbol{x}) $$ -->