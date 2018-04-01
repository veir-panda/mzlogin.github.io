---
layout: post
title: 离线环境下安装Python环境及相关库文件
categories: Python
description: 离线环境下安装Python环境及相关库文件
keywords: Python环境, 离线安装
---

# 离线环境下安装Python环境及相关库文件

Python版本分为2和3两大类，在一台机子上直接安装多个版本的Python会不利于切换，这里推荐一个Python包管理系统[Anaconda](https://www.anaconda.com/download/#linux),通过它能方便的安装Python环境和第三方库，切换不同的Python版本，还能很方便的离线安装。

## 下载

到这里去下载：[https://www.anaconda.com/download/#linux](https://www.anaconda.com/download/#linux)

不同的系统下有对应的包，注意区分系统和Python版本号

## 安装与使用

……

算了不写了，这种小白的步骤我就懒得写了，贴上一篇地址，自己去看吧：
[初学python者自学anaconda的正确姿势是什么？？ - 猴子的回答 - 知乎](https://www.zhihu.com/question/58033789/answer/254673663)

这里重点记录怎么将开发环境下的软件包迁移到另一台离线环境去（同时也适用于备份当前环境）：

## 离线环境安装软件包（也适用于备份当前环境）

> 先在当前有网的环境下安装好软件

有两种备份当前环境包的方法，选择哪个取决于你安装软件包时候的方式

### pip方式安装的

1. 打开终端或cmd，切换到需要保存的环境后，输入：```pip freeze > requirements.txt```生成一个txt文件，这个件就是github上的python项目中经常看到的，该项目的依赖包。

2. 输入```pip download -r requirements.txt -d .```，将requirements.txt文件中的软件下载到当前目录。

3. 备份该目录:package

4. 如果需要在离线环境下安装的话，将上面下载好的软件包复制到离线环境，然后输入命令：pip install package/*

### conda方式安装的

```
# 如果目标环境可以联网:在本机打开终端或cmd，切换到需要保存的虚拟环境后:

#将当前环境导出到yaml文件中
conda env export > environment.yaml     

# 复制environment.yaml文件到目标环境，创建一个同名的虚拟环境之后激活此环境，然后安装相同的环境：
conda env update -f=/path/environment.yaml

# 注意：这种方式只针对目标环境有网，且所有包都是通过conda方式安装的，使用pip安装的不会被导出到文件中去
```

根据经验，conda的[官方软件仓库](https://anaconda.org/)有大部分经过验证、打包的Python库，但还是有些库在上面是找不到的，这时只能通过pip来安装，这种方式安装的库不能使用下面的方式备份，只能使用上面pip的列出库文件命令；还有自己开发的那种，没有上传到pip源的，一般通过setup.py安装的，只能手动重新复制过去安装。

**一般能使用conda方式的都能使用上面pip方式，且pip方式更简单**

通过conda离线安装Python库的方式有两种：

* 到conda的[官方软件仓库](https://anaconda.org/)去搜索下载需要的库到本地，一般为一个tar.gz文件，然后在目标环境下：conda install ./*.tar.gz --offline

    这种方式适用于目标环境的包较少的情况

* 在一台没有安装其他Python库的联网环境上：```conda install scrapy --download-only```,这时会下载所有tar.gz包到anaconda的安装目录下pkgs目录（该目录下原本有很多其他库文件），复制该目录下的所有包到目标机器相同的目录，再离线安装:```conda install scrapy --offline```

通过以上可以看出通过pip的方式导出Python库并下载更方便，所以这里推荐pip方式导出。