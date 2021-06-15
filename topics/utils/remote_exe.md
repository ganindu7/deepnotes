---
layout: default
title: Execute on target device
nav_order: 2 
permalink: /topics/utils/execute_on_target
parent: Utilities
---

## Running code in the Jetson
Once the code is built you can either direcly log on to the target device and execute the code or do it over ssh. If the executable has visual elements and if you haven't [forwarded the X session at the begining](https://en.wikipedia.org/wiki/SSH_(Secure_Shell)#Uses) (if your target has a disply attached) you can attach the output to the target's display by typing `$ export DISPLAY=:1` and then running the executable.

Note: These ssh pipes can be broken if the host machine locks itself and halts the underlying process. Take care when running processes over long periods of time i.e. overnight simulations. 


## Configuring SSH even more for Development

The many options for SSH configurations can be found [here](https://www.ssh.com/academy/ssh/config). For our purposes we suggest the use of X11 forwarding and agent forwarding. In our development computers we can always change the ~/.ssh/config file instead of the system wide /etc/ssh/ssh_config file.

forwarding the SSH agent allows you to use the development machine without having to set up a key in the target machine, you can pull code from remote repositories using your hosts identity. More information can be found [here](https://www.cloudsavvyit.com/25/what-is-ssh-agent-forwarding-and-how-do-you-use-it/) 


The Unix way of displaying graphics is the X Server-client architecture developed by the MIT lab in 1987 which has been in use across every Unix like system to date (current gen X11). Behind every program with a GUI there is an X server separating the display and execution functionality of the underlying program, the display server and the program talk to each other to keep what's on display relevant to the program's latest state. 

Our application (i.e display annotated sensor data along with some interactive controls) in this context is called a "x client" is running on our Jetson hardware and we want it's output to be rendered by the X server running on our development PC. We use the X forwarding to achieve this by sending out input to the client (running on Jetson HW) and relaying it's response to our X server. For more details please check this [article](https://medium.com/mindorks/x-server-client-what-the-hell-305bd0dc857f)

### Summary

* open the `~/.ssh/config` file


Check the example below and make necessary changes. the key parameters here are `ForwardAgent` and `ForwardX11`

```
Host MyHostName 
	Hostname ip.address.to.target
	User	 myUserName
	ForwardAgent yes
	ForwardX11 yes
	TCPKeepAlive yes
	IdentityFile ~/.ssh/IdentityFileForTheKey
```

* restart ssh with `sudo systemctl restart ssh.service`

## Disclaimer 

In instances where the gpu is doing the display work the remote may not be able to show that (due to the inability to perform the GPU rendering), in that case the solution is "[vino](https://en.wikipedia.org/wiki/Vino_(VNC_server))". check this [link](https://developer.nvidia.com/embedded/learn/tutorials/vnc-setup) out for detailed steps. 


## To redirect output to the the screen attached to Jetson

ssh into the target and type `export DISPLAY=:1`  in the relevant shell. Now the graphics will run there. (If unsure you can always find out with `printenv | grep DISPLAY` command).

*If your OpenSSH version is 7.8+ you can even add that to the ssh config file. given that your targets `/etc/ssh/sshd_config` file accepts that
by adding that to the accepted list. `AcceptEnv LANG LC_* DISPLAY` (in this case `DISPLAY` variable)*