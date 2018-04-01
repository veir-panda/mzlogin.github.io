---
layout: post
title: 查看 Linux 服务器配置信息
categories: linux
description: 查看 Linux 服务器配置信息
keywords: ubuntu, 配置信息
---

# 查看 Linux 服务器配置信息

做测试的时候需要知道服务器的一些配置信息，下面记录查看服务器信息的方法。

## 查询系统版本信息

```shell
root@ubuntu160:~# lsb_release -a
No LSB modules are available.
Distributor ID:    Ubuntu
Description:    Ubuntu 14.04.3 LTS
Release:    14.04
Codename:    trusty
```

## Linux查看物理CPU个数、核数、逻辑CPU个数
```shell
# 总核数 = 物理CPU个数 X 每颗物理CPU的核数 
# 总逻辑CPU数 = 物理CPU个数 X 每颗物理CPU的核数 X 超线程数

# 查看物理CPU个数
cat /proc/cpuinfo| grep "physical id"| sort| uniq| wc -l

# 查看每个物理CPU中core的个数(即核数)
cat /proc/cpuinfo| grep "cpu cores"| uniq

# 查看逻辑CPU的个数
cat /proc/cpuinfo| grep "processor"| wc -l
```

## 查询内存信息

```shell
# 查看概要信息
free -m     #以MB为单位显示，参数还可以是 -b, -k, -g

# 查看内存详细信息
cat /proc/meminfo
```

## 硬盘

### 查看硬盘和分区分布

```shell
lsblk 
#显示信息
NAME                        MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
sda                           8:0    0   150G  0 disk 
├─sda1                        8:1    0   500M  0 part /boot
└─sda2                        8:2    0 149.5G  0 part 
  ├─VolGroup-lv_root (dm-0) 253:0    0    50G  0 lvm  /
  └─VolGroup-lv_swap (dm-1) 253:1    0     4G  0 lvm  [SWAP]
sdb                           8:16   0    18T  0 disk 
└─sdb1                        8:17   0    18T  0 part 
  └─VolGroup-lv_home (dm-2) 253:2    0    16T  0 lvm  /home
sr0                          11:0    1  1024M  0 rom
```

### 查看硬盘和分区的详细信息

```shell
fdisk -l

#显示信息
Disk /dev/sda: 161.1 GB, 161061273600 bytes
255 heads, 63 sectors/track, 19581 cylinders
Units = cylinders of 16065 * 512 = 8225280 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x000cf395

   Device Boot      Start         End      Blocks   Id  System
/dev/sda1   *           1          64      512000   83  Linux
Partition 1 does not end on cylinder boundary.
/dev/sda2              64       19582   156773376   8e  Linux LVM

……
……
```

## 查看文件夹大小

```shell
#以人们可读方式显示当前目录下的第一层目录文件或文件夹大，-h为人类可读，--max-depth为树的深度，点.为当前目录
du -h --max-depth=1 .

du /home    #显示/home及其所有子目录文件大小
```