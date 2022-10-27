---
layout: default
title:  port mapping using ssh tunneling
nav_order: 2 
# permalink: /topics/utils/target_execute
parent: Utilities
---

## Port mapping between local and remote targets
Tested on a Linux 5.4.0-94-generic on 18/01/2022

When we are developing software using several machines we want to access resources across several machines. <span style="background-color:LightYellow"> (e.g. access a Jupyter notebook server running on a docker container on a remote host) </span>

I will use the example use case above for the walk-through shown below. 

This line below is run from my local PC <br />

```
ssh -N -L localport:ip_of_the_resource_seen_by_the_remote_host:port username@ip_address
ssh -N -L 8887:localhost:8888 ganindu@192.168.0.33
```
<br />
*IPV6*
```
ssh -N -L localport:ip_of_the_resource_seen_by_the_remote_host:port username@ip_address%network_interface
ssh -N -L 8887:localhost:8888 g@fe80::4bdc:e743:27d1:bf03%enp0s41f7
```



`8887` : Port in my local PC that I want to map to <br />
`localhost:8888`: Source IP:PORT as as seen by my host computer <br />
`-N`: Prevent executing other shell commands (helps to keep the shell standalone) <br />
`-L`: Local mapping setup (check manual entry for ssh)


#### Use-case example with a slightly different syntax: <br/>
Imagine this deepnotes website is hosted on a remote PC's localhost (e.g.`127.0.0.1:4000/deepnotes/`) and this remote pc is accesible via `ssh jon@654.32.1.87` <br/>
Type `ssh -L 127.0.0.1:4000:127.0.0.1:4000 jon@654.321.1.87` in a new terminal to create the tunnel (keep terminal active to keep the tunnel alive).<br/> then access the remote webpage by inserting `127.0.0.1:4000/deepnotes/` in the adress bar from the local pc. <br/> <br/>

*Note: This is same as* `ssh -N  -L 4000:localhost:4000 jon@654.321.1.87`

You can extend this to other use cases as Jupyter notebook servers or ipython kernels.



P.S. 

Alternatively you can do something like `ssh -N -L localhost:8887:localhost:8888 username@computer1` or `ssh -N -L 8887:localhost:8888 username@computerx` too.



[source 1: Connecting to a Jupyter Notebook on a remote Linux machine with an SSH tunnel](https://towardsdatascience.com/connecting-to-a-jupyter-notebook-on-a-remote-linux-machine-277cef04abb7) <br />
[source 2: Accessing a Jupyter notebook server through reverse port forwarding](https://michaelgoerz.net/notes/accessing-a-jupyter-notebook-server-through-reverse-port-forwarding.html)




