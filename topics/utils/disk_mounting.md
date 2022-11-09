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
