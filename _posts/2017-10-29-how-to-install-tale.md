---
layout: post
title: Ubuntu搭建Tale个人博客
categories: other
description: Ubuntu搭建Tale个人博客
keywords: Ubuntu, Tale, 个人博客
---

# Ubuntu搭建Tale个人博客

一直以来，Java类型的动态博客相比静态博客都是处于劣势，即便现在也是一样。但作为一个写Java的码农，总是更青睐于Java自己写的博客。

Tale博客基于Java语言编写，最新版只需要Java运行环境即可，Github[主页](https://github.com/otale/tale)有详细的介绍。这里附下我的安装过程供参考。

> Tale博客我已经搭建好半年之久了，搭建过程也遇到了一些问题，但是当时没有记录，Tale[作者](https://biezhi.me/)自己写的教程比较简单，附的教程也是别人写的，这次受到国外服务器访问风波，换了一个服务器，重新搭建了新版的Tale，在此记录搭建过程。有疑问或对本文内容有不同意见的欢迎留言。

## 服务器信息

操作系统：Ubuntu 16.04 X86_64

## 安装前须知

本人使用的是Windows系统，如果你也是，你需要必备以下的基础才能继续操作

- 知道Windows怎么连接linux，这里我用的为Shell
- 知道怎么上传文件到服务器，我用的WinSCP
- 基本了解Linux的常用命令操作，尤其是vim操作（虽然本文附出大部分命令，但如果你一点不了解，你会寸步难行）
- 本系列教程全部使用root账户操作，进行操作前**请将账户切换为root**。命令:`su`,也可不切换，但必须在下面每条命令前添加`sudo`

## JDK1.8安装

### 下载

因为oracle官网的jdk下载地址有权限验证，需要自己手动到oracle官网去下载jdk1.8， 下载地址：[这里](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)

> 如果服务器为国外的服务器，直接在服务器上下载会比你在本地下载再上传到服务器快很多
> 下载方式：进入上面的下载地址下载，我下载的是`Java SE Development Kit 8u152`版本，再到浏览器的下载管理右键复制链接，再到shell终端输入命令 `wget -O jdk1.8.tar.gz 复制的链接地址`即可下载到当前目录并保存文件名为“jdk1.8.tar.gz”

假设已将压缩包下载到`/home/veir`目录下
```shell
进入目录
cd /home/veir
#解压
tar -zxvf jdk1.8.tar.gz
#将jdk文件夹复制到/usr/local目录下，同时重命名为jdk1.8
mv jdk1.8.0_152 /usr/local/jdk1.8
#解压后即可删除原压缩包，如果怕后面还需要，也可暂时不删除
rm -rf jdk1.8.tar.gz
```
### 环境变量配置

```shell
#编辑环境变量配置文件
vim /etc/profile
#在文件的最后加入下面的配置信息，注意JAVA_HOME路径要与jdk实际路径匹配
export JAVA_HOME=/usr/local/jdk1.8
export JRE_HOME=${JAVA_HOME}/jre
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib
export PATH=${JAVA_HOME}/bin:$PATH
#使环境变量生效
source /etc/profile
```
### 验证

要验证是否安装成功很简单，在终端输入
`java -version`,显示出java版本号代表配置成功

## 安装Tale

### 下载

在Tale的github主页[下载](http://static.biezhi.me/tale-least.zip?10281)最新版博客，上传到服务器，我将文件上传到服务器的`/home/veir`目录下

### 启动

使用前先解压
```
unzip unzip tale-least.zip
```

Tale博客启动特别简单，进入到tale目录一条命令即可搞定：`java -jar tale-least.jar`，但这里不推荐这种启动方式，因为这种方式启动，一旦关闭终端，Tale也就停了

新版的博客自带了一个脚本`tale-cli`，查看该脚本的用法:
```shell
COMMANDS:
    start    启动tale
    stop     停止当前tale实例
    reload   重新启动当前tale实例
    log      查看当前tale日志
    status   查看当前tale状态
    upgrade  升级当前的tale版本
    help, h  Shows a list of commands or help for one command
```

使用脚本的启动方式就为

```
cd /home/veir/tale
#给脚本执行权限
chmod +x tale-cli
#启动Tale
./tale-cli start
```
现在即可看到tale成功启动的提示：
```
root@ubuntu:/home/veir/tale# ./tale-cli start
Tale 启动成功, 可以使用 ./tale-cli log 命令查看日志.
```
### 安装

接下来就更简单了，访问`http://ip地址:9000`,开始傻瓜式操作，填写必要的信息，完成即可。

### 开机自启动

服务器一般不必重启，所以一般也不必配置开机自启动；但是我自己经常鼓捣我的服务器，会不可避免的重启，所以还是配置一下开机自启动为好。

```shell
#写一个自启动脚本
cd /etc/init.d
vim tale

#在文件加入以下内容
#######文件内容分割线#######
#!/bin/bash
#program

export JAVA_HOME=/usr/local/jdk1.8
export JRE=$JAVA_HOME/jre
export CLASSPATH=$JAVA_HOME/lib:$JRE/lib:.
export PATH=$PATH:$JAVA_HOME/bin/:$JRE/bin

nohup java -jar /home/veir/tale/tale-least.jar >/home/veir/tale/logs/tale.log  &

exit 0
#######文件内容分割线#######

#给tale可执行权限
chmod +x tale
#配置开机自启动此脚本 99表明一个优先级，越高表示执行的越晚
update-rc.d tale defaults 99
#移除Ubuntu开机脚本
#update-rc.d -f tale remove
```
现在`reboot`试试，看看在开机之后能否直接访问之前的地址

安装过程本来到现在为止也就完了，但我们是在服务器上访问，我总不能一直输入ip+端口号来访问吧？所以我们要配置域名访问

## 域名访问（Nginx代理）

平时我们以类似于`www.veir.site`这样的域名方式来访问，这就要看下面的过程。

> **注意**：要以域名访问，如果服务器在国内，那必须得在工信部备案才能正常访问，但是部分小众域名在国内暂时又是不能备案的；国外的服务器就可以免去这些烦恼。

### 域名解析
要配置域名访问，首先得要有一个域名，我是在阿里云购买的域名，[地址](https://wanwang.aliyun.com/)，当然其他地方购买的也可以

购买好后，进入控制台管理刚注册的域名，点击"添加解析"，添加一个"A"记录解析到你服务器地址
```
记录类型：A
主机记录：@
解析线路：默认
记录值：IP地址
TTL值：10分钟（默认就好）
```

上面的解析是将这样的域名解析到服务器ip地址上（比如：veir.site解析到94.xx.xx.xx）
如果你想`veir.site`访问，那到此为止，域名解析也就完了；如果你想以这样的域名`www.veir.site`来访问，则需要添加二级域名解析：

```
记录类型：CNAME
主机记录：www
解析线路：默认
记录值：域名.后缀   （例如：veir.site）
TTL值：10分钟（默认就好）
```

这样，就将`www.veir.site`解析到你到服务器上面了

### 免端口访问

现在请求是能到达你的服务器了，但是现在还要在域名后加一个端口号“:9000”才能访问，因为浏览器访问网站的默认端口为80（HTTPS端口为443），想要不加端口号也能访问，有两种方式：
- 修改Tale的默认端口号为80:
修改tale/resources/目录下`app.properties`文件,添加一行
```
server.port=80
```
- 使用Nginx代理，这也是本文推荐的方式

使用Nginx代理有很多优点，除了可以代理服务器上除Tale之外的应用，使二级域名访问不同的应用外，还可以对网站启用`SSL`访问，简直不能太好！

### Nginx安装

先在服务器上安装Nginx
```shell
apt-get update
apt-get install nginx
```
此时Ngin应该在你的服务器上安装好了，我们使用`systemctl status nginx`查看Nginx是否启动
如果看到“Active: active (running)”字样说明已经成功启动，否则可以尝试`systemctl start nginx`来启动Nginx，还是失败的后可能是80端口被占用，可以尝试下面的方式解决

```shell
netstat -ntlp | grep 80     #查看占用的进程
kill -9 进程号     #杀死进程
```

启动成功后可以在浏览器输入`http://服务器IP`，成功访问会看到“Welcome To Nginx”字样

附上Nginx的常用命令：

```
systemctl start nginx   #启动
systemctl stop nginx    #停止
systemctl restart nginx #重新启动
systemctl reload nginx  #修改配置文件后重新加载配置文件
systemctl disable nginx #关掉开机自启动Nginx
systemctl enable nginx  #开启开机自启动
```
### Nginx代理Tale

我们写一个Nginx配置文件，为了方便管理和查看，文件名就为要访问的域名，一个域名一个文件

```shell
vim /etc/nginx/sites-available/www.veir.site
#写入下面的内容

#######文件内容分隔线#########
server {
    server_name www.veir.site;
    listen 80;
    location / {
        proxy_redirect off;
        proxy_pass http://127.0.0.1:9000;
        proxy_http_version 1.1;
        proxy_set_header Host $http_host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header REMOTE-HOST $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

    }
}

#如果想输入veir.site也能直接访问www.veir.site的话，再添加下面的，否则就不用了
server {
    server_name veir.site;
    return 301 $scheme://www.$host$request_uri;
}
#######文件内容分隔线#########

#建立一个软链接到nginx生效的目录
ln -s /etc/nginx/sites-available/www.veir.site /etc/nginx/sites-enabled/www.veir.site
#使配置文件生效
systemctl reload nginx.service  #Ubuntu 16 版本方式
#service nginx reload #Ubuntu 14 版本方式
```
这时，在Nginx和Tale都已经启动了，并且域名管理的地方也做了域名解析并生效之后，我们就可以直接输入域名访问Tale了。试试看!

## 给网站添加HTTPS

SSL协议主要用来提供对用户和服务器的认证，激活SSL协议，实现数据信息在客户端和服务器之间的加密传输，可以防止数据信息的泄露。保证了双方传递信息的安全性，而且用户可以通过服务器证书验证他所访问的网站是否是真实可靠。————以上摘自百科。

当我们使用浏览器访问类似于百度、谷歌这样的大型网站的时候，我们会在浏览器的地址栏左边看到一个“绿色加锁”的符号，chrome还会显示“安全”字样，这样的网站代表他是受浏览器信任的，现在还有部分网站没有开启SSL证书访问，这样的网站通常会以一个感叹号来标示，而证书有问题的网站，chrome默认会禁止访问并给出访问警告。所以，给自己的网站开启SSL访问时很有必要的。

一般SSL证书是要给钱的，但还好有我们的开源世界，他就是`Let's Encrypt`,具体的大家自行去搜索了解就够了，下面给出安装`Let's Encrypt`证书的过程。

```shell
apt-get update
apt-get install software-properties-common
add-apt-repository ppa:certbot/certbot
apt-get update
apt-get install python-certbot-nginx

#自动配置Nginx，根据提示进行操作
certbot --nginx
```

这时你再访问你的网站，看看`https://域名`能否正常访问，在chrome等浏览器中地址栏左边是否出现了“绿锁”图标，是的话，恭喜你，大功告成了！

遗憾的是，`Let's Encrypt`每次申请的SSL证书只有90天的有效期，我们可以通过以下命令设置证书自动续订

```shell
#测试自动续订能否工作
certbot renew --dry-run
#设置自动续订
certbot renew
```
## 其他

Ubuntu服务器默认没有开启防火墙，但是作为一台服务器，为了安全着想，防火墙最好还是开启为好。

ufw是ubuntu自带的防火墙管理工具，相比linux默认的iptables配置更简单，这里以ufw为例。如果运行下面的命令出现“找不到ufw”命令，使用`apt-get install ufw`安装。

```shell
#先给ufw允许访问的list中添加一些常用端口，尤其是你shell连接到服务器的端口
ufw allow 80    #允许80端口访问
ufw allow 22    #如果SSH端口为默认的22，开启此端口的访问
ufw allow 443    #允许HTTPS的443端口访问
……
#其他类似
ufw allow 3000-3003    #开启3000到3003之间的所有端口
ufw enable  #开启防火墙并开机启动
ufw default deny    #默认拒绝所有端口访问
ufw status      #查看ufw允许访问的端口列表
```
如果要删除之前建立的规则
```shell
ufw status numbered     #带行号显示
ufw delete 行号      #删除某条规则
```

## 结束

恭喜你完成了个人博客的搭建。但搭建只是一方面，个人博客更重要的是内容，所以经常写博客才是更重要的事。

如果你在安装过程中出现了其他问题，欢迎给我留言或给我发邮件，QQ直接找我请备注好信息。