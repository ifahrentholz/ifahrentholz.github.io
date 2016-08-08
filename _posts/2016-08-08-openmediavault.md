---
title:  "Use the Raspberry Pi as NAS server"
date:   2016-01-08 15:04:23
categories: [raspberry]
tags: [raspberry, nas, openmediavault, omv]
---

First of all - due to the crapy data throughput of the Raspberry Pi you should not expect high
performance data transfer. A 1GB file will take ~20 minutes to be transfered.

That said lets dive directly into the setup.

## Prerequisites:
- sudo rights on your working station
- external hard disks (one or more)
- [OpenMediaVault image for your Pi](https://sourceforge.net/projects/openmediavault/files/)

## Install OMV OS
* Insert the SD card onto your computer
* Get the device name of the SD card - should be something like: **/dev/disk2**

```bash
➜  ~ diskutil list
/dev/disk2 (internal, physical):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:     FDisk_partition_scheme                        *31.9 GB    disk2
   1:             Windows_FAT_32 boot                    58.7 MB    disk2s1
   2:                      Linux                         3.5 GB     disk2s2
   3:                      Linux                         67.1 MB    disk2s3
```

* Open the terminal and open the folder where you downloaded your OMV Os to.
* Unzip the file:

```
gunzip omv_3.0.24_beta_rpi2_rpi3.img.gz
```

* Now you need to unmount your SD card an install the img (be sure you substitue _/dev/disk2_ with your device name of the SD card):

```bash
sudo diskutil unmountDisk /dev/disk2
sudo dd bs=1m if=omv_3.0.24_beta_rpi2_rpi3.img of=/dev/disk2
```

* Put the SD card into your Raspberry Pi and start it - but make sure the external hard disks and the ethernet cable are connected
* Now you'll be prompted with the login:

```bash
raspberrypi login: root
Password: openmediavault
```

* Get the IP address of your Raspberry Pi (ifconfig):

```bash
root@raspberrypi:~# ifconfig
eth0      Link encap:Ethernet  HWaddr b8:27:eb:f0:ea:18
          inet addr:***.***.*.**  Bcast:***.***.*.***  Mask:***.***.***.*
```

* On your workstation you can open a webbrowser and as url type: http://[your-ip]
* The default weblogin is:

```bash
Username: admin
Password: openmediavault
```

* The first thing you should do is to change the default password to something secure:

```bash
sudo passwd
```
