Mount the filesystem of the host box in rescue mode

Enable rescue mode in the console, make note of password, and reboot box

Then

```bash
ssh root@xxx.xxx.xxx.xxx
```

The server will return some information about the box on connection

```bash
-------------------------------------------------------------------

  Welcome to the Hetzner Rescue System.

  This Rescue System is based on Debian 8.0 (jessie) with a newer
  kernel. You can install software as in a normal system.

  To install a new operating system from one of our prebuilt
  images, run 'installimage' and follow the instructions.

  More information at http://wiki.hetzner.de

-------------------------------------------------------------------

Hardware data:

   CPU1: AMD Ryzen 7 1700X Eight-Core Processor (Cores 16)
   Memory:  64370 MB
   Disk /dev/sda: 512 GB (=> 476 GiB)
   Disk /dev/sdb: 512 GB (=> 476 GiB)
   Total capacity 953 GiB with 2 Disks

Network data:
   eth0  LINK: yes
         MAC:  00:00:00:00:00:00
         IP:   xxx.xxx.xxx.xxx
         IPv6: 1001:1001:1001:1001::2/64
         RealTek RTL-8169 Gigabit Ethernet driver
```

Use vg scan to find disk

```bash
root@rescue ~ # vgscan
  Reading all physical volumes.  This may take a while...
  Found volume group "vg0" using metadata type lvm2

root@rescue ~ # pvscan
  PV /dev/md1   VG vg0   lvm2 [476,31 GiB / 341,31 GiB free]
  Total: 1 [476,31 GiB] / in use: 1 [476,31 GiB] / in no VG: 0 [0   ]
```

Activate vg0

```bash
root@rescue ~ # lvm vgchange -a y
4 logical volume(s) in volume group "vg0" now active
```

List partitions in the LVM

```bash
root@rescue ~ # lvscan
ACTIVE            '/dev/vg0/root' [10,00 GiB] inherit
ACTIVE            '/dev/vg0/tmp' [5,00 GiB] inherit
ACTIVE            '/dev/vg0/home' [20,00 GiB] inherit
```

List Mountable devices

```bash
root@rescue ~ # ls -l /dev/mapper/vg*
lrwxrwxrwx 1 root root 7 Sep 11 07:26 /dev/mapper/vg0-home -> ../dm-2
lrwxrwxrwx 1 root root 7 Sep 11 07:26 /dev/mapper/vg0-root -> ../dm-0
lrwxrwxrwx 1 root root 7 Sep 11 07:26 /dev/mapper/vg0-tmp -> ../dm-1
```

Mount root (/) on /mnt

```bash
root@rescue ~ # mount /dev/mapper/vg0-root /mnt
```

Now work with files normally as if you were on the box

```bash
root@rescue ~ # ls -l /mnt
total 1,2M
lrwxrwxrwx  1 root root    7 May 14 08:39 bin -> usr/bin
drwxr-xr-x  2 root root 4,0K Sep 10 16:08 boot
drwxr-xr-x  2 root root 4,0K Sep 10 16:08 dev
drwxr-xr-x 71 root root 4,0K Sep 10 16:09 etc
drwxr-xr-x  2 root root 4,0K Sep 10 16:08 home
-rw-r-----  1 root root  733 Sep 10 16:09 installimage.conf
-rw-r-----  1 root root 1,2M Sep 10 16:09 installimage.debug
lrwxrwxrwx  1 root root    7 May 14 08:39 lib -> usr/lib
lrwxrwxrwx  1 root root    9 May 14 08:39 lib64 -> usr/lib64
drwx------  2 root root  16K Sep 10 16:07 lost+found
drwxr-xr-x  2 root root 4,0K Apr 11 06:59 media
drwxr-xr-x  2 root root 4,0K Apr 11 06:59 mnt
drwxr-xr-x  2 root root 4,0K Apr 11 06:59 opt
drwxr-xr-x  2 root root 4,0K Sep 10 16:08 proc
dr-xr-x---  4 root root 4,0K Sep 10 16:08 root
drwxr-xr-x  3 root root 4,0K Sep 10 16:08 run
lrwxrwxrwx  1 root root    8 May 14 08:39 sbin -> usr/sbin
drwxr-xr-x  2 root root 4,0K Apr 11 06:59 srv
drwxr-xr-x  2 root root 4,0K Sep 10 16:08 sys
drwxr-xr-x  2 root root 4,0K Sep 10 16:08 tmp
drwxr-xr-x 13 root root 4,0K May 14 08:39 usr
drwxr-xr-x 19 root root 4,0K May 14 08:39 var
```

when finished - unmount

```bash
root@rescue ~ # umount /dev/mapper/vg0-root
```

And deactivate vg0

```bash
root@rescue ~ # lvm vgchange -a n vg0
  0 logical volume(s) in volume group "vg0" now active
```

Then reboot, to get back to normal mode, and profit!

```bash
root@rescue ~ # reboot

Broadcast message from root@rescue on pts/0 (Tue 2018-09-11 08:58:23 CEST):

The system is going down for reboot NOW!
```
