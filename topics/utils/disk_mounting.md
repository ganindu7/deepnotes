---
layout: default
title: Disk mounting 
parent: Utilities
permalink: /topics/utils/disk_mounting
# permalink: /topics/comms_setup   # adding a permalink broke the internal linking to a topic 
# nav_order: 2
---


# Mounting disks in host computers 

There are multiple ways of automating disk mounting. We will discuss disk mounting using the /etc/fstab file and a mountfs daemon service 


### fstab method 


This is a /etc/fstab file 

I added 
```
# USB Mass storage device 
# /dev/disk/by-uuid/E4C75361C8523252   /media/WD_USB_HDD/  fuseblk  defaults  0  0
# this via `gnome-disks`
/dev/disk/by-uuid/E4C75361C8523252 /media/WD_USB_HD auto nosuid,nodev,nofail,x-gvfs-show 0 0
```



After adding 

```
# /etc/fstab: static file system information.
#
# Use 'blkid' to print the universally unique identifier for a
# device; this may be used with UUID= as a more robust way to name devices
# that works even if disks are added and removed. See fstab(5).
#
# <file system> <mount point>   <type>  <options>       <dump>  <pass>
# / was on /dev/ubuntu-vg/ubuntu-lv during curtin installation
/dev/disk/by-id/dm-uuid-LVM-k5cgR1nZM1b2a4at1ht0fdf4glwuMvq3pQvqh0iVV1bGxU1fbrl7Z3mSyOwIMV5e / ext4 defaults 0 1
# /boot was on /dev/sda2 during curtin installation
/dev/disk/by-uuid/6981be89-9bab-4d00-8aad-118645013431 /boot ext4 defaults 0 1
# /boot/efi was on /dev/sda1 during curtin installation
/dev/disk/by-uuid/3A70-DE26 /boot/efi vfat defaults 0 1
/swap.img       none    swap    sw      0       0
# USB Mass storage device 
# /dev/disk/by-uuid/E4C75361C8523252   /media/WD_USB_HDD/  fuseblk  defaults  0  0


/dev/disk/by-uuid/E4C75361C8523252 /media/WD_USB_HD auto nosuid,nodev,nofail,x-gvfs-show 0 0
```

or use `UUID=...`

```
UUID=fd7766e7-96b5-49d6-9603-b63db21359b2 /mnt/SD_CARD_1 ext4           x-gvfs-show              0 0
```


sometimes you can just mount the path directory. Below is a case where I have just used the `fdisk` utility to 
1. Delete existing partitions. (with option `d`)
2. Crerate a new GNU partitiaon table. (with option `g`)
3. Create a new partition. (option `n` )
4. Set partition type to linux (with option 't')
5. write changes (with `w`)
6. then format the new partition to `ext4`  
7. edit fastab to auto muountthe partition.

*Note that I have used partiton 1*

formatting the partition
```
sudo mkfs.ext4 /dev/nvme0n1p1
```

updates to fstab
```
/dev/nvme0n1p1 /mnt/ssd-disk/ ext4 defaults 0 0
```

checking if mounts work

```
ganindu@ubuntu:/mnt/SD_CARD_1$ sudo mount -av
/                        : ignored
/mnt/SD_CARD_1           : successfully mounted
ganindu@ubuntu:/mnt/SD_CARD_1$ sudo vim /etc/fstab 
```

Note: The directory (mount point) has to be present and empty 

Afterwards you can use `sudo mount -a ` to test these changes

## Note: 
you may ignore these errors 

before reboot <br/>
`[E] unreachable on boot required target: No such file or directory` <br/>
after reboot <br/>
`[E] cannot detect on-disk filesystem type`



### Alternative approach 

usecase: Create a NTFS formatted USB drive for logging 
 * when formattting give it a distinct name e.g. "SENSOR_DATA_LOGGING_USB"

 Then tou will see the label on `/dev/disk/by-label/PM_SENSOR_DATA_LOGGER`

 then add the following line to the fstab file! 

```
 LABEL=PM_SENSOR_DATA_LOGGER /mnt/PM_LOGGING_USB/ auto nosuid,nodev,nofail,x-gvfs-show 0 0
```


### directory permissions for other users 

You can create a user group and give the group permission for a directory 

create a group 

```
sudo groupadd new.group
```

add users to the group 

```
sudo usermod -aG sudo $USER

```

to check memebers in a group use `getent`
e.g. 
```
getent group adm
```



then change group of the directory


```
sudo usermod -aG new.group $NEW_USER
or
chown -R owners_username:crazy_user_group the_directory_to_give_permissions/
or
sudo chgrp -R new.group /path/to/the/directory

```

give correct permissions

```
sudo chmod -R g+rwx /path/to/the/directory
```

to make all new files and directories inherit the ownership and permissions

```
sudo find /path/to/the/directory -type d -exec chmod 2775 {} \;    
``` 

to inherit read write access 

```
sudo find /path/to/the/directory -type f -exec chmod ug+rwx {} \;
```

[source](https://superuser.com/questions/19318/how-can-i-give-write-access-of-a-folder-to-all-users-in-linux)



### Service daemon method 

Service Daemons are background processes that can be defined by the user and managed by the operating system. In this example our mount is a network share, our goal is to mount this 
as a file accessible as a mounted folder in a systm resource optimal way. 


resource: `172.16.1.39:/DS/myshare`  <br/>
target: `/mnt/my_share_mount`
<br/>

create a file named `mnt-my_share_mount.mount` in the `/etc/systemd/system` directory. <br/>

Note the naming of the file in respect to the directory (forward slashes convert to dashes)


```
[Unit]
Description=Mount NFS Share at boot

[Mount]
What=172.16.1.39:/DS/myshare
Where=/mnt/my_share_mount
Type=nfs
Options=defaults

[Install]
WantedBy=multi-user.target
```

Move the file to `/etc/systemd/system` with <br/>

`sudo mv mnt-my_share_mount.mount /etc/systemd/system/`

Reload the daemon

```
sudo systemctl daemon-reload
```

Enable and srart the service

```
sudo systemctl enable mnt-my_share_mount.mount
sudo systemctl start mnt-my_share_mount.mount
```


output of `sudo systemctl enable mnt-my_share_mount.mount`
```
Created symlink /etc/systemd/system/multi-user.target.wants/mnt-my_share_mount.mount → /etc/systemd/system/mnt-my_share_mount.mount.
```



Check status with `sudo systemctl status mnt-qnap_ganindu.mount` <br/>

you will see something simlilar as the response if everything went well

```
● mnt-qnap_ganindu.mount - Mount NFS Share at boot
     Loaded: loaded (/etc/systemd/system/mnt-my_share_mount.mount; enabled; vendor preset: enabled)
     Active: active (mounted) since Mon 2023-09-11 17:00:39 BST; 4s ago
      Where: /mnt/my_share_mount
       What: 172.16.1.39:/DS/myshare
      Tasks: 0 (limit: 618847)
     Memory: 140.0K
        CPU: 5ms
     CGroup: /system.slice/mnt-my_share_mount.mount

Sep 11 17:00:38 dgx systemd[1]: Mounting Mount NFS Share at boot...
Sep 11 17:00:39 dgx systemd[1]: Mounted Mount NFS Share at boot.
```


you can check it further with 

```
df -h
```

or the `mount` command





 






### Further reading

[Linux permissions: SUID, SGID, and sticky bit](https://www.redhat.com/sysadmin/suid-sgid-sticky-bit) <br/>
[What is SUID, SGID and Sticky bit ?](https://www.thegeekdiary.com/what-is-suid-sgid-and-sticky-bit/) <br/>
[fstab file](https://wiki.debian.org/fstab) <br/>
