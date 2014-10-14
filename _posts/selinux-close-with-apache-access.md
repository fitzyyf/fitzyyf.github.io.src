title: "关闭SELinux方法修复apache访问permission的问题"
date: 2013-05-22 09:40
tags: linux
categories:  
- Linux
---

关闭SELinux的方法：
修改`/etc/selinux/config`文件中的SELINUX="" 为 disabled ，然后重启。
如果不想重启系统，使用命令`setenforce 0`

> setenforce 1 设置SELinux 成为enforcing模式
> setenforce 0 设置SELinux 成为permissive模式 

查看selinux状态：
`/usr/bin/setstatus -v `
