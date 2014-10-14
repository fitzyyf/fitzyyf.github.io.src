title: "更换CeontOS的启动方式"
date: 2014-03-17 11:34
tags: 
- linux
- centos
categories: 
- Linux
---

最近安装几台`CentOS`的服务器，在安装时，安装了个啥子桌面环境，非常不爽，就想重装为服务器命令模式；让同事找了下，发现可以修改`/etc/inittab`文件来解决这个问题。

解决办法：

	$ sudo vim /etc/inittab
	id:3:initdefault:	

`/etc/inittab`的格式如下： `[设定项目]:[run level]:[init执行的动作]:[指令项目]`

找了下资料，`run level`包含如下几个级别：
	
- 0 - halt (系统直接进行关机)
- 1 - single user mode (单人维护方式)
- 2 - Multi-user, without NFS (与第三级别类似，但无网络（NFS）服务)
- 3 - Full multi-user mode (完整的命令行模式)
- 4 - unused (系统保留功能)
- 5 - X11 (与 `3` 级别一致，增加了窗口图形化界面)
- 6 - reboot (重新启动)

内容来自[鸟哥Linux的私房菜](http://linux.vbird.org/linux_basic/0510osloader.php#startup_init)





