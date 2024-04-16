---
layout: default
title: setting up vscode
# nav_order: 3 
permalink: /topics/utils/wireguard
parent: Utilities
---

## Setting up [WireGuard][WIREGUARD-SITE]

### On a client

1. instll wireguard 

```
sudo apt install wireguard
```

2. this is basically the same as the tutorial on website 

```[Interface]
PrivateKey = <private kay of the client>
Address = 10.0.0.3/24  # This IP should be unique to this client within the VPN network

[Peer]
PublicKey = <pubkey of server>
Endpoint = <server ip>:<server port>  # Raspberry Pi's local network IP and WireGuard port
AllowedIPs =  x.x.x.x/24   # Route only some Traffic through the internal network e.g. 10.0.0.0/24
PersistentKeepalive = 25  # Keeps the connection alive
```

Then if you want to forward packets across network interfaces in the clinent enable this setting 

```
sudo sysctl -w net.ipv4.ip_forward=1
``` 

to make it persistant, uncomment the same line on 

```
/etc/sysctl.conf
```

and run `sudo sysctl -p` to load settings from file


-> Apply NAT (Network Address Translation) 

This is needed for the client to hide the source of packets when forwarding them to the pc across the ethernet interface to the wireguard interface 

```
sudo iptables -t nat -A POSTROUTING -o enp0s31f6 -j MASQUERADE
```

for the iptables setting s to persist install `iptables-persistent` and run `sudo netfilter-persistent save`

```
sudo apt-get update
sudo apt-get install iptables-persistent

sudo netfilter-persistent save
```

to check if the changes have stuck use 

```
sudo iptables -t nat -L POSTROUTING -v
```





#### Troubleshooting 

`The terminal process failed to launch: Path to shell executable "/bin/bash --login c" does not exist.`

This may happen even if you correctly set the settings file in the remote container this may be a bug) 

* goto: File > Preferences > Settings
* Click on Terminal 
* Scroll to Integrated > Default profile: Linux 
* set it to bash (usually default value is `None`)
* then again in remote settings you will notice that the integrated profile is now changed to `bash`

For me at this point the error goes away and If I have set the `gbash` profile definition as above I change the value from `bash to `gbash`. However I'm not sure if this makes any effect.    


click on Terminal 






Read [this article][VSCODE-INDEXER-ERRORS] for more info on debugging indexer errors by fixing IntelliSense bugs.


[WIREGUARD-SITE]: https://www.wireguard.com/
