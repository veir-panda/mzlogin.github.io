# Centos7.4安装CUDA9.1

## 环境与说明

Centos 版本号为7.2升级到7.4的，内核为3.10.0
CUDA取最新版本
GPU：K80*2

## 安装前

### 验证你的系统是否能识别上面的GPU
 前提机器上面有支持CUDA的Nvidia GPU，查看支持CUDA的GPU列表:
[https://developer.nvidia.com/cuda-gpus](https://developer.nvidia.com/cuda-gpus)
```shell
lspci | grep -i nvidia
```
正常应该显示Nvidia显卡的型号，没有任何显示需要更新pci硬件库`update-pciids`

### 验证系统是否是受支持的Linux版本
```
uname -m && cat /etc/*release
```
到这里查看受支持的Linux版本：[http://docs.nvidia.com/cuda/cuda-installation-guide-linux/index.html#system-requirements](http://docs.nvidia.com/cuda/cuda-installation-guide-linux/index.html#system-requirements)

### 验证系统是否有GCC编译环境

```
gcc -v
```
没有的话需要先安装GCC，Centos7的最小化安装一般勾选上开发软件都会自动安装GCC

### 验证系统是否安装了正确的内核头文件和开发包
```
sudo yum install kernel-devel-$(uname -r) kernel-headers-$(uname -r)
```
我还没出现过没有的情况--

## 开始安装

CUDA安装方式有两种可以选择，一是通过各Linux发行版的deb/rpm包直接安装，另一种通过通用的runfile文件安装。如果可以的话，deb/rpm包安装更方便，但之前有过一次在我本地Ubuntu上安装失败的经验，这里选择runfile文件方式安装

### 下载

- CUDA

9.1版本地址：[https://developer.nvidia.com/cuda-downloads](https://developer.nvidia.com/cuda-downloads)
所有版本地址：[https://developer.nvidia.com/cuda-toolkit-archive](https://developer.nvidia.com/cuda-toolkit-archive)

- ~~Nvidia驱动~~（runfiel程序自带Nvidia官方驱动程序，这里不需要再手动安装）

根据官方页面介绍，安装CUDA还需要安装Nvidia驱动:[www.nvidia.com/drivers](www.nvidia.com/drivers)

根据需要选择下载

### 卸载

如果已经安装过CUDA了，那在本次安装之前应卸载之前的安装
也可以参考官方对此的说明：
[http://docs.nvidia.com/cuda/cuda-installation-guide-linux/index.html#handle-uninstallation](http://docs.nvidia.com/cuda/cuda-installation-guide-linux/index.html#handle-uninstallation)

### 禁用nouveau

nouveau是一个第三方开源的Nvidia驱动，一般Linux安装的时候默认会安装这个驱动。
这个驱动会与Nvidia官方的驱动冲突，在安装Nvidia驱动和和CUDA之前应先禁用nouveau

查看系统是否正在使用nouveau

```
lsmod | grep nouveau
```
如果有显示内容，则进行以下的步骤：
Centos7禁用方法

```
#新建一个配置文件
sudo vim /etc/modprobe.d/blacklist-nouveau.conf
#sudo vim /usr/lib/modprobe.d/dist-blacklist.conf #上面的的命令最后失败的话使用这个

#写入以下内容
blacklist nouveau
options nouveau modeset=0

#保存并退出
:wq

#备份当前的镜像
sudo mv /boot/initramfs-$(uname -r).img /boot/initramfs-$(uname -r).img.bak

#建立新的镜像
sudo dracut /boot/initramfs-$(uname -r).img $(uname -r)

#重启
sudo reboot

#最后输入上面的命令验证
lsmod | grep nouveau

#如果还不行可以尝试下面的命令再重启：
sudo dracut --force
```
总之最后一定要保证`lsmod | grep nouveau`命令没有任何输出内容。
这也是runfile文件安装方式相对rpm/deb方式麻烦的一点，如果本节禁用nouveau有问题，可以尝试rpm/deb方式安装,rpm/deb方式安装会自动禁用

### ~~安装Nvidia驱动~~（runfiel程序自带Nvidia官方驱动程序，这里不需要再手动安装）

```
sudo rpm -ivh nvidia-diag-driver-local-repo-rhel7-390.30-1.0-1.x86_64.rpm
```


### 安装CUDA相关包

从这里开始进入命令行模式，如果当前为图形化界面，请先切换到命令行模式:`init 3`,我这里没有GUI界面的环境，具体自行百度

通常在CUDA下载页除了一个大点的`xxx.run`包之外，还有一些其他的包，一般是升级或增强或补丁之类的，建议一起下载安装

进入到CUDA安装文件`xxx.run`所在的目录

安装CUDA基础包：
```
sudo sh cuda_9.1.85_387.26_linux.run

#刚开始安装会进入more模式，一直按`空格`即可

#Do you accept the previously read EULA?
#是否接受协议
accept

#Install NVIDIA Accelerated Graphics Driver for Linux-x86_64 387.26?
#是否安装Nvida驱动，需要
y

#Do you want to install the OpenGL libraries?
#是否需要安装OpenGL，不需要
n

#Do you want to run nvidia-xconfig?
回车

#Install the CUDA 9.1 Toolkit?
y

#Enter Toolkit Location
#输入Toolkit的安装目录
#一般默认即可
回车

#Do you want to install a symbolic link at /usr/local/cuda?
#创建一个软连接，我选择是
y

#Install the CUDA 9.1 Samples?
#安装CUDA官方示例包
y

#Enter CUDA Samples Location
#输入示例包的安装目录，我选的默认路径，可以根据实际情况选择输入
回车





#如果版本不同，提示也可能不同，根据情况输入或选择
```

安装其他包：
```
sudo sh sh cuda_9.1.85.1_linux.run
sudo sh sh cuda_9.1.85.2_linux.run
sudo sh sh cuda_9.1.85.3_linux.run

#参考上面的情况进行选择
```

### 验证

CUDA安装完之后，检查`/dev`目录下是否有几个以`nvidia`开头的文件
```
ls /dev | grep nvidia
```

没有输出的话，在某个目录下新建一个文件

```
#新建一个文件
sudo vim fix.sh

#输入下面的内容
#!/bin/bash 
/sbin/modprobe nvidia 

if [ "$?" -eq 0 ]; then 
	# Count the number of NVIDIA controllers found. 
	NVDEVS=`lspci | grep -i NVIDIA` 
	N3D=`echo "$NVDEVS" | grep "3D controller" | wc -l` 
	NVGA=`echo "$NVDEVS" | grep "VGA compatible controller" | wc -l` 

	N=`expr $N3D + $NVGA - 1` 
	for i in `seq 0 $N`; do 
	mknod -m 666 /dev/nvidia$i c 195 $i 
	done 

	mknod -m 666 /dev/nvidiactl c 195 255 
else 
	exit 1 
fi 

/sbin/modprobe nvidia-uvm 

if [ "$?" -eq 0 ]; then 
	# Find out the major device number used by the nvidia-uvm driver 
	D=`grep nvidia-uvm /proc/devices | awk '{print $1}'` 

	mknod -m 666 /dev/nvidia-uvm c $D 0 
else 
	exit 1 
fi

#保存并退出

#执行
sudo sh fix.sh
```

再次检查

### 配置环境变量

如果有注意的话，cuda安装完之后有个提示，要配置环境变量

```
#修改系统配置文件
sudo vim /etc/profile

#在最后面加入以下内容
export PATH=/usr/local/cuda/bin:$PATH
export LD_LIBRARY_PATH=/usr/local/cuda/lib64:$LD_LIBRARY_PATH

#保存并退出
:wq

#使配置文件生效
sudo source /etc/profile
```

### 其他

在cuda的官方手册中推荐安装Nvdia的守护进程
```
sudo /usr/bin/nvidia-persistenced --verbose
```

如果找不到CUDA的官方示例或之前没有选择安装，可以现在安装
```
sudo cuda-install-samples-9.1.sh 目录
```

### 验证安装

显示NVDIA Driver的版本号和gcc的版本号
```
cat /proc/driver/nvidia/version
```

检查 CUDA Toolkit是否安装成功
```
nvcc -V

#显示了以下内容
nvcc: NVIDIA (R) Cuda compiler driver
Copyright (c) 2005-2017 NVIDIA Corporation
Built on Fri_Nov__3_21:07:56_CDT_2017
Cuda compilation tools, release 9.1, V9.1.85
```

通过Samples示例检验cuda
```
#进入CUDA官方示例目录 
cd xxx/NVIDIA_CUDA-9.1_Samples/

#编译示例程序
make

#...
#...
#...
#...
#是不是等了很久？反正我是等了很久很久...
#最后编译完成
#Finished building CUDA samples

#进入编译好的目录
cd bin/x86_64/linux/release/

#运行
./deviceQuery

#出现一大段结果，最后是
Result = PASS
#代表通过了

#最后检查
./bandwidthTest

#显示Result = PASS
```

## 参考

[http://docs.nvidia.com/cuda/cuda-installation-guide-linux/index.html#abstract](http://docs.nvidia.com/cuda/cuda-installation-guide-linux/index.html#abstract)