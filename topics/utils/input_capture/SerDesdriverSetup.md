---
layout: default
title: Driver setup
nav_order: 1 
permalink: /topics/utils/input_capture/drvSetup
parent: InputSetup
grand_parent: Utilities
---

### Setting up the board 

<p>To connect the FPD-Link-III SerDes board to get camera input. We need the driver to be installed on our target. Before going ahead
make sure the host is connected to the target (SSH) and the target is connected to the Internet. 
</p>

#### set up ssh key 

* create a keypair `ssh-keygen -t rsa -b 4096 -C 'ganindu@email.com'` and save the key as `~.ssh/Jetson`
* Edit the `~/.ssh/config` file

```
# this is if you have added a github key previously
Host github.com
	Hostname ssh.github.com
	Port 446

# This is for the Jetson board
Host Jetson 
	# I'm using IPv6, note the double "%" before the interface name 
	Hostname fe80::b444:7974:e88:53f1%%enp0s31f6
	# Username for the Jetson board 
	User a
	IdentityFile ~/.ssh/Jetson
	StrictHostKeyChecking no
	# Not great practice but this is a dev board
	UserKnownHostsFile /dev/null

```

Then copy the newly minted ssh public key to the target <br>
`ssh-copy-id -i ~/.ssh/Jetson a@fe80::b444:7974:e88:53f1%enp0s31f6`


<span style="background-color:LightYellow">
refer to the ***setting up communications*** section fore more about  IPv6 setup, This can be done with IPv4 too. In that case you can omit the network interface name e.g. `Hostname 192.168.55.1` and the copying is simply `ssh-copy-id -i ~/.ssh/Jetson a@192.168.55.1` </span>

Now check if you can log in via SSH without the password 

`ssh a@fe80::b444:7974:e88:53f1%enp0s31f6` or simply `ssh a@Jetson`

#### setting up dependencies 

install the following dependencies with the command
	
```sudo apt-get install ack build-essential cmake libexpat1-dev libgstreamer1.0-dev libgtk-3-dev libjpeg-dev libx11-dev mplayer pkg-config silversearcher-ag strace tree vlc```

#### install OS support 

setting up rsync 

```
ssh-copy-id @USERNAME_AT@"$TARGET"
ssh @USERNAME_SSH@ -t "$TARGET" 'sudo apt-get -y update && sudo apt-get install -y rsync'
make sync-debug
ssh @USERNAME_SSH@ -t "$TARGET" 'debug/disable-screen-lock'
```




	




