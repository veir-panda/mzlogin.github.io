ubuntu 16版本没有了`/var/log/boot.log`这个系统启动日志16版可以使用下面的命令查看：

```
journalctl -b0 SYSLOG_PID=1
```

如果发现系统启动中有什么启动失败的问题，老版本可以直接查看`/var/log/boot.log`文件，新版本使用上面的命令查看