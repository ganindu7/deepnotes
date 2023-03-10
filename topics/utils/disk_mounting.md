---
layout: default
title: Disk mounting 
parent: Utilities
permalink: /topics/utils/disk_mounting
# permalink: /topics/comms_setup   # adding a permalink broke the internal linking to a topic 
# nav_order: 2
---

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
sudo usermod -aG sudo $NEW_USER

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


### Further reading

[Linux permissions: SUID, SGID, and sticky bit](https://www.redhat.com/sysadmin/suid-sgid-sticky-bit) <br/>
[What is SUID, SGID and Sticky bit ?](https://www.thegeekdiary.com/what-is-suid-sgid-and-sticky-bit/) <br/>
[fstab file](https://wiki.debian.org/fstab) <br/>
