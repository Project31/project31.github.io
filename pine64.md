---
layout: page
title: Pine64
permalink: /pine64/
---

# 1. Install Centos-7 to the Pine64

The Pine64 board should be able to run Aarch64/ARM64 images. The official Centos-7 supports these 
images at <http://mirror.centos.org/altarch/7/isos/aarch64/>. Unfortunately currently that kernel does
not run as is on the Pine64. However there are really great instructions by [Uli Middelberg](https://github.com/umiddelb/aarch64/wiki/Install-CentOS-7-on-your-favourite-ARMv8-ARM64-AArch64-board) how to
fix this. Alternative you can download a ready to go image from [mypine64.com](https://www.mypine64.com/forums/viewtopic.php?f=30&t=5).

![Pine64](https://cdn-images-1.medium.com/max/800/1*rzKvW06sxv6u-hblFgmEhw.jpeg)

# 2. Flash the image
You will need a 8 GB microSD card or bigger. On OSX stick the microSD card into an adapter and into your macbook. It will automount. Open the Disk Utility and check the Device setting, which should be of form `disk<n>s1`, so something like ‘disk2s1’ for n=2. Now unmount the disk and run

~~~~
sudo dd bs=1m if=CentOS-aarch64-pine64.img of=/dev/rdisk2
~~~~

Warning: in ‘rdisk2’ the ‘r’ means raw disk and writes orders of magnitude faster on OSX and 2 is the value found above for n.
You can use control-t to monitor the progress. Once done you should update the uEnv.txt file on the microSD card. Add the following line

~~~~
ethaddr=00:06:dc:xx:xx:xx
~~~~

where the setting is the second number on the sticker on your pine board, if it says something like 0006dc8bce10 then use ‘ethaddr=00:06:dc:8b:ce:10’ [[Pine64–1]]. Note that 00:06:dc is the vendor id. Once done you can eject the card and insert into the Pine64 board. 

# 3. Boot up your Pine64

Stick your microSD card in the board power it up. I had no luck getting an HDMI monitor to work so I used the following command to see which new SSH server came online:

~~~~
nmap -p 22 — open -sV 192.168.1.0/24
~~~~

Make sure to adjust the network setting you are expecting on your network. This command will give you a list of IP address that run SSH servers and you should be able to find IP address for your Pine64 board. Now you can ssh in using ssh `root@<ipaddress>` and use the default password of ‘centos’. Check that the /boot/uEnv.txt file contains the ethaddr setting you added, or add it now [Pine64–2].

The initial size of the microSD card is 8 GB. To use the full size of your microSD card [StackX] you need to use

~~~~
sudo fdisk /dev/mmcblk0
~~~~

and type ‘p’ to obtain the list of partitions.

~~~~
/dev/mmcblk0p1 40960 143359 51200 c W95 FAT32 (LBA)
/dev/mmcblk0p2 143360 15138815 7497728 83 Linux
~~~~

To enlarge the second partition first delete it using ‘d’ and then ‘2’. Then ’n’ for createing a new partition and ‘p’ for ‘primary’. Enter ‘143360’ from the start of the second partition above and accept the default for the end, and type ‘w’ to save the partition settings. Now reboot and finally

~~~~
resize2fs /dev/mmcblk0p2
~~~~

and once done reboot again. Now

~~~~
df -h .
~~~~

should reflect the full capacity of the card. Finally you should run

~~~~
yum update
~~~~

to bring Centos-7 distro fully up to date.

# 4. Optional configuration tips

## 4.1 NTP

For most application it is crucial the time of your machine is set correctly so it makes sense to run
the ntp service so you no longer have to worry about that.

~~~~
yum -y install ntp
systemctl enable ntpd
systemctl start ntpd
~~~~

## 4.2 Switch out NetworkManager for the good old network service

If you're going to use your Pine64 as a server platform then it makes sense to swap out the (too) dynamic NetworkManager
for the good old `network` service. You're going to have to be a bit careful if you don't have a monitor hooked
up like me that you don't loose the network while doing this.

Bring up the network service

~~~~
echo "NETWORKING=yes" > /etc/sysconfig/network
~~~~

Edit the `/etc/sysconfig/network-scripts/ifcfg-eth0` file and set `NM_CONTROLLED=no`

~~~~
NAME=eth0
DEVICE=eth0
ONBOOT=yes
BOOTPROTO=dhcp
NM_CONTROLLED=no
~~~~

NetworkManager runs with a dhcp-help which you need to shutdown before you can startup the network service

~~~~
killall -9 nm-dhcp-helper
systemctl enable network
systemctl start network
system disable NetworkManager
system stop NetworkManager
~~~~

## 4.3 ifconfig

~~~~
yum -y install net-tools
~~~~

## 4.4 NFS

If you want to mount an external NFS share you need to run autofs

~~~~
yum -y install autofs nfs-utils
systemctl enable autofs
systemctl start autofs
~~~~

before you can reference your share in your `auto.myshare` file, for example

~~~~
sdb2         -rw,soft,intr,rsize=8192,wsize=8192   192.168.1.11:/mnt/sdb2
~~~~

referenced in the /etc/auto.master file as

~~~~
/export /etc/auto.myshare
~~~~

will mount as `/export/mnt/sdb2`. Make sure the `/export` directory mountpoint exists.

## 4.5 Docker and Kubernetes

You can install docker, etcd, flanned and kubernetes from rpm as well but some of these packages are too old, so
don't install these from rpm. We will manage these from ansible.


# 5. References
1. [MyPine] My Pine64 Unofficial PINE64 Site. <https://www.mypine64.com/>
2. [UMidd] Install CentOS 7 on your favourite ARMv8 ARM64 AArch64 board, Uli Middelberg, <https://github.com/umiddelb/aarch64/wiki/Install-CentOS-7-on-your-favourite-ARMv8-ARM64-AArch64-board>
3. [Pine64–1] pine64 mac_address: <http://forum.pine64.org/showthread.php?tid=743&pid=6939#pid6939>
4. [Pine64–2] pine64 subforum on ethernet port: <http://forum.pine64.org/showthread.php?tid=2049>
5. [StackX] Resize the root partition: <http://raspberrypi.stackexchange.com/questions/499/how-can-i-resize-my-root-partition>

