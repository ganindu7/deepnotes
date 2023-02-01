---
layout: default
title: Using CAN bus with Nvidia AGX Xavier
parent: Utilities
# permalink: /topics/utils/nvidia-jetson-xavier-can-bus
# permalink: /topics/comms_setup   # adding a permalink broke the internal linking to a topic 
# nav_order: 2
---

# Using CAN bus with Nvidia AGX Xavier devices
{: .no_toc }

CAN is quite useful to communicate with vehicle ECUs and telemetry systems. In this section we discuss how that can be done for Jetson-AGX Xavier devices
{: .fs-6 .fw-300}
<!--
## Table of contents 
{: .no_toc .text-delta}

1. TOC
{:toc}
-->
Note: This is a Draft <br />
<span style="background-color: lightyellow">
CAUTION: go through the the whole document at least once before attempting implementing a full system.  
</span>

*Last updated: Feb/2022*

---

![Wiring setup](jetson-xavier-can.svg)
              Figure 1. Wiring and pin connections




<!-- <p align="center">
    <img src="jetson-xavier-can.svg"
         title="Inputs, graph outputs and labels" />
</p>
 -->
--- 
## 1. Basic Prerequisites

* Jetson AGX-Xavier developer platform module
* Can Transceiver IC (this can be any suitable IC, we use the [SN65HVD230][SN65HVD230] from TI).
* CAN utilities installed `$ sudo apt-get install -y can-utils busybox`

In this example I've used single core PCV thin wall cables (1mm) which is standard in most automotive settings. but you can use 0.5 mm or lower based on the connector and crimp sizes. 
To create twisted pairs you can use a [vice](https://en.wikipedia.org/wiki/Vise) to hold one end of the pair and a drill to twist **slowly** (or hand twist of course). To verify can traffic you can use [PCAN USB PRO][PCAN-USB-PRO] or a [Vector CAN interface][VECTOR-CAN] or check [here][LINUX-CAN-STUFF] for some recommendations from the [embedded Linux community][ELINUX-ORG]. In the Example I've used a [PCAN View][PCAN-VIEW] with a PCAN USB device in a Linux machine( there is a graphical windows interface for MS Windows). 

## Connecting and Configuring the Nvidia Jetson AGX-Xavier 

### Kernel DTB

first you need to check if the CAN nodes are enabled in the kernel device tree binary that is flashed in to the device. 

```shell
$ cat /proc/device-tree/mttcan\@c310000/status
``` 

### Pinmux

A CAN configured Pinmux can be set in different ways.

* Update the pinmux config files pre-flashing.
* Use the graphical [Jetson-IO tool][JETSON-IO-TOOL]
* Use a [busybox devmem tool][BUSYBOX-DEVMEM] to write to hardware registers. (Changes will persist until next system boot) <br />
 Install busybox `$ sudo apt install busybox`
 `busybox devmem <register> w <value>` ( you can pick the register values from Table 1 below )
e.g. 
```shell
$ sudo busybox devmem 0x0c303010 w 0x400
$ sudo busybox devmem 0x0c303018 w 0x458
```


 | CAN peripheral          | Controller base address | pins on board | port addr |register    | value | chip conn (SN65HVD230) |
 |-------------------------|-------------------------| ------------- |-----------|------------| ------| -----------------------|
 |  <center>CAN0</center>  | mttcan@c310000          | 29            | can0_din  | 0x0c303018 | 0x458 | R (RX)                 |
 |  <center>"</center>     | <center>"</center>      | 31            | can0_dout | 0x0c303010 | 0x400 | D (TX)                 |
 |  <center>CAN1</center>  | mttcan@c320000          | 37            | can1_din  | 0x0c303008 | 0x458 | R (RX)                 |
 |  <center>"</center>     | <center>"</center>      | 33            | can1_dout | 0x0c303000 | 0x400 | D (TX)                 |

<center> Table 1. Configuring the CAN peripheral on NVIDIA Jetson AGX-Xavier  </center>


### Loading Kernel Drivers
load all the necessary kernel drivers in the following order.
- *Note: Use `sudo` mode if necessary. *
1. `$ modprobe can`
2. `$ modprobe can_raw`
3. `$ modprobe mttcan`

### Network interface setup 

Example to set up a 250Kbps network. with presumed acknowledgement. *(suited to a test network with one or two nodes to prevent BUSHEAVY conditions)*

```shell
$ sudo ip link set can0 type can bitrate 250000 presume-ack on restart-ms 2000
$ sudo ip link set can0 up
```

You can use the shell script below to set all the steps in one go. <br />
<span style="background-color: lightyellow">
you can run `$ chmod +x scriptname.sh` (maybe with sudo) and then `./scriptname.sh`
</span>

### Setting up Register values and enabling CAN

The shell script below will update registers, set up drivers and bring the `can0` interface up. The same process cab be modified to activate `can1`.

<script src="https://gist.github.com/ganindu7/fb8fa77394ecd22516567bf8cf2fe957.js?file=run.sh"></script>


#### **Disabling CAN**
<span style="background-color: lightgoldenrodyellow;">
If you want to disable CAN for debugging (e.g. prior to updating parameters) use the script below. 
This unloads the drives and brings the CAN down, however the registers will remain unchanged until explicitly done so.   
</span>
<script src="https://gist.github.com/ganindu7/fb8fa77394ecd22516567bf8cf2fe957.js?file=disable_can.sh"></script>


## Testing and running code

<span style="background-color: lightgoldenrodyellow;">
This section uses [python-can][PYTHON-CAN] with [socketcan][SOCKETCAN]. You can install the pythin package via pip with the following command </span>

If you are using the `system-python` 

```
sudo apt install python3-wrapt
sudo apt install python3-can
```
```
pip3 install python-can
```  

The python examples below are tested on <br/>


| ------ | ----- |
| python | 3.6.9 |
| python3-wrapt | 1.9.0-3 |
| python-can | 3.3.4 |




Now your python environment is ready to work with *[SocketCAN][SOCKETCAN]* 


Once the can drivers and the interface is up you can use commands like [cansend][CANSENND] and [candump][CANDUMP] alongside potential already existing bus traffic to inject and read CAN messages.
At this point you have to be aware of the clock compatibility of the participating CAN nodes or you may get various faulty BUS ERROR conditions. 

Following code example and the image is from running the python script from the Jetson and capturing the outfrom from a CAN bus monitoring software (also with `candump` from the sending end) 

<script src="https://gist.github.com/ganindu7/fb8fa77394ecd22516567bf8cf2fe957.js?file=test.py"></script>

![cross trace](can_bus_jetson_xavier_files/can-capture-good.gif)
<center> Figure 3. Top left: can dump (jetson), top right: python script executing (jetson), <br /> bottom: CAN trace (pc with PCAN interface)</center>


<!-- ![bus trace](can_bus_jetson_xavier_files/can-capture.gif)
<center> Figure 2. PCAN View CAN trace </center>
 -->
If we trace the bus with a logic analyser scope when our CAN frames are flying through it we can see the resulting waveform. (here I have probed stage between the transceiver and the Jetson CAN peripheral, there are other special tools that can go in the can L and H on the physical differential CAN BUS too). you will notice that the dominant level is the logic LOW while in idle state the bus voltages float up. Also Notice the ACK/NACK bits that represent `acknowledgements` (this is quite helpful as if some nodes do not see the ACK they will report errors such as [BUSHEAVY][BUSHEAVY-PCAN] conditions.)

![trace](can_bus_jetson_xavier_files/can-sig-trace.svg)
<center> Figure 2. Can Signal Trace (open in new tab for better resolution) </center>
<span style="background-color: lightyellow">
Note: If you don't provide a node on the bus that does not acknowledge or do not put anything other than the sending node and the tracer the acknowledge will not be present as shown above. 
adding instruction lines like e.g. `sudo ip link set can0 type can bitrate 250000 presume-ack on restart-ms 2000` will tell the interface to cope with out and ACK and even reset the bus in case of 
a "crazy chatty" node.
</span>

## Connecting to the CAN bus.

The CAN L and H from the transceiver is connected to the CAN bus as shown below. It is Important to use the right terminations at the right place to avoid reflections that may cause the bus to not perform correctly. Usually we use 120 ohm resistors in the end or in the case of a split termination (usually done in board level design)




![layout example, source: datasheet](can_bus_jetson_xavier_files/can-bus.png)
            Figure 4. CAN bus, source: [datasheet][SN65HVD230]


## Circuit design ideas 


![layout example, source: datasheet](can_bus_jetson_xavier_files/can-term.png)
            Figure 5. PCB Can termination, source: [datasheet][SN65HVD230]

## PCB design ideas 

Because ESD and EFT transients can be between 2Mhz and 3GHz approximately High frequency layout guidelines must be applied to the PCB design. 
To work in machines or vehicle plants external transient protection devices must be used at bus connectors to prevent these transients from propagating into the PCB.

* Use power and ground planes to provide low inductance paths for high frequency signals to dissipate into (rather than onto signal paths) and provide shielding. 
* Design bus protection components in the direction of the signal path. (do not force transients to divert from the signal path to reach the protection device e.g. capacitor) 
* It is advised to use transient voltage suppression devices (TVS) bi directional diodes or varistors and bus filter capacitors, bus transient devices should be placed close to the connector to stop them from coming in. 
* You may use CAN termination on the board but however if it is used the device should be an end node and should not be removed without adding a substitute termination to the bus. (split termination provides common mode filtering from the bus)
* bypass and bulk capacitors must be placed as close to the supply pins of the transceiver. 
* use at least two vias for vcc and ground connections of bypass capacitors and protection devices to minimise trace and via inductance. 
* to limit current via digital lines serial resistors may be used. 
* to filter noise on the digital IO lines a capacitor may be used to close the input side the IO as shown by C1 and C4.
* since the internal pull up and pull down biasing of the device is weak in the face of transients for floating pins an external 1k to 10 k pull up/down should be used to bias the state of the pin nore strongly against the noise during transient events. 
* If D(TxD) is driven via an open drain/collector it should be pulled up to vcc when it is not shorted to GND through the driving gate. (look at R1)
* if the device is only operating on normal mode or slope control R£ is not needed and C4 pads can be used for the pulldown resistor to ground. 
if pin5 is not used it can be left floating. 



![layout example, source: datasheet](can_bus_jetson_xavier_files/footprint-ckt.png)
<center>Figure 6. PCB footprint suggestion, source: datasheet]</center> 



## Troubleshooting CAN

Setting up the physical can bus and get the electrical and software components working altogether can be challenging at times. This section is to address some of these problems in a Q&A
style writeup.

### CAN interface is not up 

*Note: (this can usually can happen with embedded platforms)*
(e.g. `candump can0` complains that the interface is not up) <br />
first [disble CAN](#disabling-can) and try replacing `sudo ip link set can0 type can bitrate 250000 presume-ack on restart-ms 2000` on the [setup script](#setting-up-register-values-and-enabling-can)    with 
`sudo ip link set can0 up type can bitrate 250000 berr-reporting on ` 


### the setup stage works fine but can't see any messages on the bus. 

This could be either an internal or external issue. first of all we need to eliminate the possibility of a failure on the jetson side (before testing external things). in order to do this we can do a loopback test.

if CAN is already set up please [disable](#disabling-can) before making modifications.

<script src="https://gist.github.com/ganindu7/fb8fa77394ecd22516567bf8cf2fe957.js?file=loopback_can.sh"></script>

at this point you can invoke the previously mentioned [python script](#testing-and-running-code) or a `cansend` command alongside [candump][CANDUMP] (e.g. run `candump can0` in a different shell)

If the jetson internals are working fine you should get something similar to what is shown below (we've run the [script](#testing-and-running-code) in the *testing and running section* )

![loopback](can_bus_jetson_xavier_files/can-loopback.gif)
<center> Figure 7. Enabling loopback mode on the can interface </center>

if you get the output above and still got problems it seems that the software is running fine. However we still can't rule out clock issues that may originate from the Jetson board that may not help with sampling issues and synchronization.

## Related [Nvidia Developer forum][DEV-FORUM] Questions

* Error due to the nodes on CAN bus having different clocks. [link](https://forums.developer.nvidia.com/t/agx-xavier-can-bus-not-working-in-jetpack-4-6/202219?u=ganindu1) <br/>


### Appendix 
[L4T 32.7.1 CAN DOCS](https://docs.nvidia.com/jetson/archives/l4t-archived/l4t-3271/index.html#page/Tegra%20Linux%20Driver%20Package%20Development%20Guide/hw_setup_jetson_can.html) <br/>
to change pinmux and update kernel look  at `$BSP/Linux_for_Tegra/kernel/pinmux/t19x/README.txt` <br/>
[L4T 32.7.1 IO-CONFIG-TOOL](https://docs.nvidia.com/jetson/archives/l4t-archived/l4t-3271/index.html#page/Tegra%20Linux%20Driver%20Package%20Development%20Guide/hw_setup_jetson_io.html#wwpID0E0ZE0HA) <br/>

---
*Click [here][ERRORS-SUGGESTIONS] for Errors, Suggestions or Comments!*

[SN65HVD230]: https://www.ti.com/lit/ds/symlink/sn65hvd230.pdf
[PCAN-USB-PRO]: https://www.peak-system.com/PCAN-USB-Pro-FD.366.0.html?&L=1
[VECTOR-CAN]: https://www.vector.com/int/en/products/products-a-z/hardware/network-interfaces/vn7640
[LINUX-CAN-STUFF]: https://elinux.org/CAN_Bus
[ELINUX-ORG]: https://elinux.org/
[PCAN-VIEW]: https://www.peak-system.com/PCAN-View.242.0.html?&L=1
[JETSON-IO-TOOL]: https://docs.nvidia.com/jetson/l4t/Tegra%20Linux%20Driver%20Package%20Development%20Guide/hw_setup_jetson_io.html#wwpID0E0ZE0HA
[BUSYBOX-DEVMEM]: https://www.busybox.net/downloads/BusyBox.html
[CANDUMP]: http://manpages.ubuntu.com/manpages/bionic/man1/candump.1.html
[CANSEND]: http://manpages.ubuntu.com/manpages/bionic/man1/cansend.1.html
[DEV-FORUM]: https://forums.developer.nvidia.com/
[BUSHEAVY-PCAN]: https://forum.peak-system.com/viewtopic.php?f=120&t=39
[ERRORS-SUGGESTIONS]: https://github.com/ganindu7/deepnotes/issues
[SOCKETCAN]: https://docs.kernel.org/networking/can.html
[PYTHON-CAN]: https://python-can.readthedocs.io/en/master/installation.html


