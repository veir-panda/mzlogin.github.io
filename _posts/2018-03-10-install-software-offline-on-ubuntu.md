---
layout: post
title: 离线环境下在Ubuntu上安装软件
categories: linux
description: 离线环境下在Ubuntu上安装软件
keywords: ubuntu, 离线安装
---

# 离线环境下在Ubuntu上安装软件

在生产中，经常会遇到客户的环境处于保密或者其他条件限制的情况下，服务器不能联通公网，我们熟悉的例如apt-get/yum等一键安装软件的命令在此时就不能发挥作用。这次，偶然在一次搜索怎样离线安装一个软件时找到了一个方便很多的办法。在此记录，方便以后查阅。

## 安装普通软件包

> 注意：这里只针对Ubuntu环境

这里我以Ubuntu上一般必装的一个开发工具包```build-essential```为例来讲述怎么安装。

* 首先，联网环境下的安装很简单，一条命令即可搞定：```apt-get install build-essential```

> 解释：```apt```命令在安装软件前，会先读取Ubuntu软件源上该软件的信息，其中就包括他的依赖信息，安装的时候会自动安装本机上没有的依赖包。

* 离线安装的过程，实际上就是上面在线安装的手动过程。需要：
1. 下载需要安装的软件包及其所有的依赖包，并复制到服务器上。
2. 根据该软件包的格式，以合适的命令安装

这其中最麻烦的一点就是要找齐该软件所有的依赖包，缺一不可！

### 步骤：


1. 找一台跟你服务器版本一样的Ubuntu系统（可以是你本地的虚拟机），要能联网

2. 找一个目录，输入：
```
sudo apt-get -qq --print-uris install build-essential linux-headers-$(uname -r) | cut -d\' -f 2 > urls.txt
# 该命令会在当前系统上安装该软件时需要下载的软件包的地址存到```urls.txt```。
```

3. 下载软件到当前目录
```
wget -i urls.txt
# 下载urls.txt里面列出的所有软件包到当前目录
```

4. 复制当前目录下的软件包到服务器上的一个目录下

5. 进入服务器存放软件包的目录，安装软件：
```
dpkg -i ./*
```
完工！

### 注意

* 在进行上面的操作前要先**确定能联网的机器上是否已经安装了该软件，已经安装的话请先卸载**：```apt-get autoremove build-essential```,不过这里我建议使用一台全新的，没有安装除系统以外其他软件的系统，不然还是有可能导致找不全依赖包
* 下载其他软件将```build-essential```替换为你要安装的软件名即可,这个名字必须是能通过apt命令一键安装的软件名
* 这里只针对```Ubuntu```系统，```Debian```与```Ubuntu```同源，应该上面的方法应该同样可以，但是没有实践过，很多客户的服务器装的是CentOS，Red Hat企业版或其他在线安装一般使用```yum```的,不在此文讨论范围，后面有空再补充上吧！

### 其他

这里将```Ubuntu 16.04 x86 _64```上的```build-essential```软件包附在这里，以后需要的直接拿去安装就完事了~


[build-essential.7z](https://cloud.veir.me/index.php?share/file&user=1&sid=UCBXp4sJ)

[htop.7z](https://cloud.veir.me/index.php?share/file&user=1&sid=yfN8ekjR)

## 参考

[https://askubuntu.com/questions/334136/how-do-i-install-build-essential-without-an-internet-connection](https://askubuntu.com/questions/334136/how-do-i-install-build-essential-without-an-internet-connection)