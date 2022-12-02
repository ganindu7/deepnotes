---
layout: default
title: Camera Interfacing
nav_order: 2 
# permalink: /topics/utils/target_execute
parent: Utilities
---

## Camera Interfacing 

We will start with the [libArgus][LARGUS-API], Libargus is for acquiring images and image-metadata from cameras. the fundamental libargus operation is a `capture->processing->image` pipeline. 

**Fundamentals of libArgus**
* Libargus is a frame based API
  * every capture is triggered by an explicit request that specifies how it should be performed.
* Librgus delivers images with EGLStreams (EGLStreams is supported by OpenGL and Cuda and requires no buffer copies when delivering to the consumer because of the extensive support by the system components for EGLStreams)
* Multi Sensor Support (independent and synchronised configuration access)
* Virtual interfaces that remain same through differnt versons and the ability to extend with vendor specifics for portablity without the need to deviate from the uniform pattern)

**Functionality**

* Wide variety of capture settings.
* Optional autocontrol such as exposire and white balance.
* Libraries that cosume the EGLStream can output in defferent encodings and give direct application to images. 
* Metadata delivery via libargus events or EGLStreram metadata.
* Image post processing such as noice reductiona nd sharpening.
* Error Notification
* Event Notifications (image acquisition start, and other events vus synchronous event queues. 














[LARGUS-API]: https://docs.nvidia.com/jetson/l4t-multimedia/group__LibargusAPI.html

