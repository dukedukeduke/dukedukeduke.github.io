---
layout: post
title:  "查看Linux 硬盘信息"
date:   2019-02-01 19:27:02 +0800
comments: true
tags:
- linux
- disk device
- 硬盘信息
---

### fdisk -l

```
fdisk -l

Disk /dev/sda: 199.8 GB, 199848099840 bytes
255 heads, 63 sectors/track, 24296 cylinders, total 390328320 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x00034cda

   Device Boot      Start         End      Blocks   Id  System
/dev/sda1   *        2048      679935      338944   83  Linux
/dev/sda2          681982   390326271   194822145    5  Extended
/dev/sda5          681984    32679935    15998976   82  Linux swap / Solaris
/dev/sda6        32681984   126679039    46998528   83  Linux
/dev/sda7       126681088   390326271   131822592   83  Linux

Disk /dev/sdb: 199.8 GB, 199848099840 bytes
255 heads, 63 sectors/track, 24296 cylinders, total 390328320 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x00000000

Disk /dev/sdb doesn't contain a valid partition table
```

### 查看硬盘ID

```
root@db2:/etc/ha.d# cd /dev/disk/by-id
root@db2:/dev/disk/by-id# ls -al
total 0
drwxr-xr-x 2 root root 280 Mar  8  2017 .
drwxr-xr-x 5 root root 100 Mar  8  2017 ..
lrwxrwxrwx 1 root root   9 Mar  8  2017 ata-DV-28S-W_11102221065739 -> ../../sr0
lrwxrwxrwx 1 root root  10 Mar  8  2017 dm-name-vg-data -> ../../dm-0
lrwxrwxrwx 1 root root  10 Mar  8  2017 dm-uuid-LVM-u1bboFqZ1T7eszXRXG1NnInP3RzWFwNDUkBrJJpnOuy4jcEHG0eWnIvQVuT8gc8r -> ../../dm-0
lrwxrwxrwx 1 root root   9 Mar  8  2017 scsi-SAdaptec_JBOD-A_DCA674A4 -> ../../sda
lrwxrwxrwx 1 root root  10 Mar  8  2017 scsi-SAdaptec_JBOD-A_DCA674A4-part1 -> ../../sda1
lrwxrwxrwx 1 root root  10 Mar  8  2017 scsi-SAdaptec_JBOD-A_DCA674A4-part2 -> ../../sda2
lrwxrwxrwx 1 root root  10 Mar  8  2017 scsi-SAdaptec_JBOD-A_DCA674A4-part5 -> ../../sda5
lrwxrwxrwx 1 root root  10 Mar  8  2017 scsi-SAdaptec_JBOD-A_DCA674A4-part6 -> ../../sda6
lrwxrwxrwx 1 root root  10 Mar  8  2017 scsi-SAdaptec_JBOD-A_DCA674A4-part7 -> ../../sda7
lrwxrwxrwx 1 root root   9 Mar  8  2017 scsi-SAdaptec_JBOD-B_57CE74A4 -> ../../sdb
```
