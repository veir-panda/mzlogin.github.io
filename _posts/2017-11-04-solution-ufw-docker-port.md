# 让docker处于ufw防火墙的控制下

最近在使用docker时遇到了一个大坑：运行一个实例时，将宿主机的一个端口映射到docker的一个实例，也就是使用`-p 8080:80`命令运行后，端口呢是能在宿主机上直接访问了，但是我突然发现我在本地居然也能访问服务器上的8080端口号，这就不科学了，因为我通过ufw把8080禁止的呀！

查资料可知，dockerz直接在iptables上进行修改，ufw的设置对它没有作用。下面是解决办法。

# 修改ufw默认的配置

```
vim /etc/default/ufw

#把DEFAULT_FORWARD_POLICY修改为下面
DEFAULT_FORWARD_POLICY="ACCEPT"

```
# 修改docker的默认配置

取消注释DOCKER_OPTS这行，在参数后添加添加`-iptables=false`

```
vim /etc/default/docker

#修改文件
DOCKER_OPTS="--dns 8.8.8.8 --dns 8.8.4.4 -iptables=false"
```
# 修改/etc/ufw/before.rules

```
vim /etc/ufw/before.rules

在`*filter`前面添加下面内容
###########我是分割线############
*nat
:POSTROUTING ACCEPT [0:0]
-A POSTROUTING ! -o docker0 -s 172.17.0.0/16 -j MASQUERADE
COMMIT
###########我是分割线############
```

# 新增文件/etc/docker/daemon.json

```
vim /etc/docker/daemon.json

###########我是分割线############
{
"iptables": false
}
###########我是分割线############
```

# 重启系统

```
reboot
```