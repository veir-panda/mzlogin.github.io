---
layout: post
title: Centos 离线安装软件
categories: linux
description: Centos 离线安装软件
keywords: centos, 离线安装
---

# Centos 离线安装软件

在前面的一篇文章中，我写了怎样在Ubuntu上离线安装软件，以及离线安装Python包的两篇文章，这里再加上一篇在Centos上离线安装的文章吧，作为这一系列的最后一篇

## 安装方法

具体的安装方法这里就不详细写了，贴上一篇文章链接（不过我还是怕想看的时候那个博客打不开了怎么办？？）

文章地址：[https://www.yanning.wang/archives/664.html](https://www.yanning.wang/archives/664.html)

---

。。。算了，我还是粘贴过来好了，免得到时候打不开就头疼了，也免去新开一个页面的麻烦。

### yum离线下载程序包

关于这个需求，我们可以安装一个yum插件：“yum-plugin-downloadonly”去实现它。

我们首先用yum安装这个插件：

```Shell
yum install -y yum-plugin-downloadonly
```

安装完成之后就在使用yum的时候使用“--downloadonly”参数。这个参数将告诉yum只下载程序包，不进行安装。

例如我想下载supervisor这个程序包，就只要使用下面的命令：

> 我注：实测Centos7.4已经不需要安装上面的插件了，不知道是不是版本的问题？

```
yum install -y --downloadonly supervisor
```

请注意：--downloadonly参数将自动下载程序包安装时所需要的所有依赖，所以建议在全新的系统中使用本命令，因为在已经安装过部分依赖的系统上，yum不会将所有需要的依赖下载完全。

当然，您也可以使用yum resolvedep命令手动查看并下载依赖，只不过这样比较麻烦就是了。

使用--downloadonly参数之后，yum在下载完程序包后就会显示一句“exiting because "Download Only" specified”并自动退出，此时要下载的程序包已经被放置到了yum的默认存放位置，在CentOS 7 x64下，这个默认路径是：

```
/var/cache/yum/x86_64/7/<repo>/packages/
```

如果要指定yum的下载目录，还需要一个“--downloaddir”参数，例如我要将supervisor程序包安装到当前文件夹下，就使用下面的命令：

```
yum install -y --downloadonly --downloaddir=. supervisor
```

等待下载完成后，用ls命令就可以看到当前目录下已经出现了supervisor及其依赖的程序包rpm文件。

### yum离线安装程序包

在刚刚第二步中我们使用了yum --downloadonly命令离线下载了想要的程序包安装文件，现在我们有一台因为种种原因而不能上网的系统，我们现在要把刚刚下载下来的程序包安装到这台电脑上该怎么办呢？

很简单，用yum localinstall命令。

首先将我们下载下来的程序包及其依赖去使用U盘等方式拷贝到这台不能上网的电脑中，然后进入程序包存放目录，执行下面的命令（仍以supervisor为例）：

```
yum localinstall -y --nogpgcheck supervisor-3.1.4-1.el7.noarch.rpm \
       python-backports-1.0-8.el7.x86_64.rpm \
       python-backports-ssl_match_hostname-3.4.0.2-4.el7.noarch.rpm \
       python-meld3-0.6.10-1.el7.x86_64.rpm \
       python-setuptools-0.9.8-7.el7.noarch.rpm
```

几个注意点：

1. 使用yum localinstall命令需要的程序包时需要同时安装程序包所有的依赖项目，否则还是会尝试联网去下载缺少的依赖项目；

2. “--nogpgcheck”参数主要是为了不让yum对程序包进行GPG验证；

3. 除了yum localinstall命令以外，还可以使用rpm -ivh命令安装rpm包，这里不再单独讨论。

---

## 常用包下载

[Development-Tools.zip](https://cloud.veir.me/index.php?share/file&user=1&sid=k69tQ7EB)(类似于Ubuntu平台的build-essential)

[Htop.zip](https://cloud.veir.me/index.php?share/file&user=1&sid=eryQzsYu)

## 参考

[https://www.yanning.wang/archives/664.html](https://www.yanning.wang/archives/664.html)
[https://anshengme.github.io/blog/article/use-the-offline-yum-installation-package-on-centos.html](https://anshengme.github.io/blog/article/use-the-offline-yum-installation-package-on-centos.html)