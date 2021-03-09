
## 内核态与用户态
1. 如何区别
2. 如何相互调用

```
内核态：CPU可以访问内存所有数据, 包括外围设备, 例如硬盘, 网卡. CPU也可以将自己从一个程序切换到另一个程序
```
```
用户态: 只能受限的访问内存, 且不允许访问外围设备. 占用CPU的能力被剥夺, CPU资源可以被其他程序获取
```
```
用户态->内核态：系统调用，如：open(), read(), write()等IO操作
```

## OOM机制

```
Linux 内核有个机制叫OOM killer（Out-Of-Memory killer），该机制会监控那些占用内存过大，尤其是瞬间很快消耗大量内存的进程，为了防止内存耗尽而内核会把该进程杀掉。典型的情况是：某天一台机器突然ssh远程登录不了，但能ping通，说明不是网络的故障，原因是sshd进程被OOM killer杀掉了（多次遇到这样的假死状况）。重启机器后查看系统日志/var/log/messages会发现Out of Memory: Kill process 1865（sshd）类似的错误信息。

防止重要的系统进程触发(OOM)机制而被杀死：可以设置参数/proc/PID/oom_adj为-17，可临时关闭linux内核的OOM机制。内核会通过特定的算法给每个进程计算一个分数来决定杀哪个进程，每个进程的oom分数可以/proc/PID/oom_score中找到。我们运维过程中保护的一般是sshd和一些管理agent。
```

## POSIX标准
```
POSIX是可移植操作系统接口Portable Operating System Interface of UNIX，POSIX标准定义了操作系统应该为应用程序提供的接口标准，是IEEE为要在各种UNIX操作系统上运行的软件而定义的一系列API标准的总称。
```
