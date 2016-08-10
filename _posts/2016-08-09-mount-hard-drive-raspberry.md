---
title:  "Mount USB HDD on Raspberry"
date:   2016-08-09 10:00:00
categories: [raspberry, mount]
tags: [raspberry, mount]
---


In this article I want to document how to mount a external HDD device on a Raspberry Pi.


### Configure your Pi
```bash
sudo raspi-config
```

1. Expand Filesystem
2. Change user password
3. Change hostname
4. Set boot to CLI
5. Set GPU Memory to 16
6. Update your localisation settings


#### Update your system
```
sudo apt-get update && sudo apt-get upgrade && sudo apt-get dist-upgrade
```


### Mount your external HDD

Now we can mount our storage. Please not that I'm using in this tutorial **usbstorage**,
**sda1** and as user **pi**. These are variables which you have to adjust depending on your system configuration and needs.

#### Prepare the Mount Point

First make a directory where your storage should be mounted.

```
sudo mkdir /mnt/usbstorage
```

Set the user permissions

```
sudo chown -R pi:pi /mnt/usbstorage
sudo chmod -R 775 /mnt/usbstorage
```

Set all future permissions for the mount point.

```
sudo setfacl -Rdm g:pi:rwx /mnt/usbstorage
sudo setfacl -Rm g:pi:rwx /mnt/usbstorage
```

#### Determine the USB Hard Drive Format

You need to knwo the file system the drive is formatted with

```
sudo blkid

/dev/mmcblk0p1: SEC_TYPE="msdos" LABEL="boot" UUID="787C-2FD4" TYPE="vfat"
/dev/mmcblk0p2: UUID="3d81d9e2-7d1b-4015-8c2c-29ec0875f762" TYPE="ext4"
/dev/sda1: LABEL="TrekStor" UUID="3d9d92e2-7d1b-4015-8c2c" TYPE="exfat"
```

Note the **TYPE="exfat"** at the end, you will need this for the fstab file.

Now mount the usb stick. If it is NTFS you will need to install some utilities first.

```
sudo apt-get install ntfs-3g -y
```

If the drive is exfat install these utilities

```
sudo apt-get install exfat-utils -y
```

For all drive types mount the usb with this command, -o insures pi is the owner which should avoid permission issues

```
sudo mount -o uid=pi,gid=pi /dev/sda1 /mnt/usbstorage
```

If you get an error use this syntax

```
sudo mount -t uid=pi,gid=pi /dev/sda1 /mnt/usbstorage
```

If the mount -t command returns an error then use this syntax

```
sudo mount uid=pi,gid=pi /dev/sda1 /mnt/usbstorage
```


### Automount the USB Hard Drive

/mnt/usbstorage will be the folder in which you store your media. We want it to be automounted on boot The best way to do this is through the UUID. Get the UUID by using this commmand

```
sudo ls -l /dev/disk/by-uuid/
```

You will see some output like this. The UUID you want is formatted like this XXXX-XXXX for the sda1 drive. If the drive is NTFS it can have a longer format like UUID="BABA3C2CBA3BE413". Note this UUID, for me it is BA8F-FFE8

```
total 0
lrwxrwxrwx 1 root root 15 Jan  1  1970 3d81d9e2-7d1b-4015-8c2c-29ec0875f762 -> ../../mmcblk0p2
lrwxrwxrwx 1 root root 15 Jan  1  1970 787C-2FD4 -> ../../mmcblk0p1
lrwxrwxrwx 1 root root 10 Oct 26 21:10 BA8F-FFE8 -> ../../sda1
```

Now we will edit fstab to mount the USB by UUID on boot

```
sudo nano /etc/fstab
```

Add the line in red to the bottom, replace XXXX-XXXX with your UUID and exfat with your type if it is different (e.g. ntfs, vfat, ext4). You may or may not need the quotation marks wrapped around the UID, you do not need quotation marks wrapped around the file system type (ext4, vfat, NTFS etc).

The umask 0002 sets 775 permissions so the pi user and group can read, write and execute files on the external USB drive. To completely eliminate permission issues you can set the umask to 0000 which equals 777 permissions so anybody can read, write and execute. Note that 777 permissions are considered a security risk.

If you have issues here then try replacing uid=pi,gid=pi with just the word defaults (typical for ext4). You can also try replacing the UUID with the /dev/sda1 line.

This is an example for exfat

```
/dev/mmcblk0p1 /boot vfat defaults 0 2
/dev/mmcblk0p2 / ext4 errors=remount-ro,noatime 0 1

UUID=XXXX-XXXX  /mnt/usbstorage exfat   nofail,uid=pi,gid=pi   0   0
```

For NTFS, note that it is ntfs and not ntfs-3g

```
/dev/mmcblk0p1 /boot vfat defaults 0 2
/dev/mmcblk0p2 / ext4 errors=remount-ro,noatime 0 1

UUID=XXXX-XXXX    /mnt/usbstorage    ntfs   nofail,uid=pi,gid=pi    0   0
```

for ext4 using uid and gid is not recommended so use at your own risk as it could cause
issues.

```
/dev/mmcblk0p1 /boot vfat defaults 0 2
/dev/mmcblk0p2 / ext4 errors=remount-ro,noatime 0 1

UUID=XXXX-XXXX    /mnt/usbstorage    ext4   nofail,uid=pi,gid=pi    0   0
```

If you get any errors you can replace uid=pi,gid=pi with defaults or remove it entirely

```
/dev/mmcblk0p1 /boot vfat defaults 0 2
/dev/mmcblk0p2 / ext4 errors=remount-ro,noatime 0 1

UUID=XXXX-XXXX    /mnt/usbstorage    ext4   nofail,defaults    0   0
```

For using /dev/sda1 and defaults if you have troubles with UUID

```
/dev/mmcblk0p1 /boot vfat defaults 0 2
/dev/mmcblk0p2 / ext4 errors=remount-ro,noatime 0 1

/dev/sda1    /mnt/usbstorage    ext4   nofail    0   0
```

Now test if the fstab file works

```
sudo mount -a
```

If you didn’t get errors reboot, otherwise try the suggestions above to get it working then mount -a again until it succeeds

```
sudo reboot
```

You should be able to access the mounted USB drive and list its contents

```
cd /mnt/usbstorage
ls
```

Every time you reboot, the drives will be mounted as long as the UUID remains the same. If you delete the partitions or format the USB hard drive or stick the UUID changes so bear this in mind. You can always repeat the process for additional hard drives in the future.

If you have multiple hard drives you will have to make separate mount points (e.g. /mnt/usbstorage2) for each drive’s partition


### Fix Raspberry Pi 2 Mounting Issues

Apparently there is a bug in the Pi 2 that messes up automounting. You can fix it by creating a delay.

Open up the /boot/cmdline.txt files

```
sudo nano /boot/cmdline.txt
```

Add this line to the bottom, you can increase this delay if necessary

```
rootdelay=5
```

Hit Ctrl+X, Y and Enter to save and exit, then reboot to see if it automounts now.
If the Raspberry Pi hard drive still does not automount we can use rc.local

```
sudo nano /etc/rc.local
```

Add this lines before the exit line

```
sleep 30
sudo mount -a
exit
```

Ctrl+X, Y and Enter to save

Reboot again to test


### Appendix

- Unmount the disk
```
sudo umount /mnt/usbstorage
```



