---
layout: post
title: buntu使用技之开机自启动
categories: linux
description: buntu使用技之开机自启动
keywords: ubuntu, 开机启动
---

# Ubuntu使用技之开机自启动

Ubuntu下开机自启动一个程序有2种方式：

## 写一个开机自启动的脚本添加到系统启动任务中

先写一个脚本，里面写上执行要执行的命令，再使用`
update-rc.d 脚本名 defaults`命令将改脚本添加到系统启动任务

例如。我需要系统开机的时候启动此博客：

```shell
cd /etc/init.d

vim tale

##########文件内容分隔符###############
#!/bin/bash
#program

#先将jdk的环境准备好
export JAVA_HOME=/usr/local/jdk1.8
export JRE=$JAVA_HOME/jre
export CLASSPATH=$JAVA_HOME/lib:$JRE/lib:.
export PATH=$PATH:$JAVA_HOME/bin/:$JRE/bin

#执行命令
nohup java -jar /home/veir/tale/tale-least.jar >/home/veir/tale/logs/tale.log  &

exit 0
##########文件内容分隔符###############

#保存后，给脚本添加可执行权限
chmod +x tale

#添加开机启动服务
update-rc.d tale defaults
#defaults后面可以加一个数字，例如99，它表明一个优先级，越高表示执行的越晚
```

有一个需要**注意**，很多命令的执行需要一个系统的环境，例如上面的`java -jar xxx`就需要jdk的环境支持，然后很有可能，系统在执行此脚本时，jdk的环境还没有加载进来，所以就需要手动在该脚本中声明需要的环境

## 修改/etc/rc.local脚本

/etc/rc.local是Ubuntu启动后自动执行的一个脚本，默认情况下这个脚本里面没有任务。修改这个脚本可以启动你自己的应用。

修改方式很简单，在最后一行`exit 0`之前加上开启服务或应用的命令就好了

下面是我的脚本，我添加了一个开机启动aria2的命令

```shell
#!/bin/sh -e
#
# rc.local
#
# This script is executed at the end of each multiuser runlevel.
# Make sure that the script will "exit 0" on success or any other
# value on error.
#
# In order to enable or disable this script just change the execution
# bits.
#
# By default this script does nothing.

aria2c
exit 0
```