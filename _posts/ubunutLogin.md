title: Ubuntu设置环境变量错误导致无法登录的解决办法
description: 设置环境变量导致Path出现问题，而无法在输入正确的用户名和密码时登录Ubuntu
tags: 
- linux
- Ubuntu
category: 
- Linux
---
1.在登录界面，按`ctrl+alt+f1`进入命令行界面，按照管理员用户名和密码登录，使用如下命令
	/usr/bin/sudo /usr/bin/vi /etc/profile     #环境变量
	/usr/bin/sudo /usr/bin/vi /etc/environment #系统属性（如果没有修改这个，则无需修改）
删除不正确的path设置。一般为新增的一些path变量。

2.如果发现修改了profile，而可能忘记修改了environment，那么可以通过将ubunt的iso文件启动（光盘或者U盘），然后进入系统，将相关文件进行修改即可。

＃ 如果是虚拟机，设置Iso为启动的设置方法

1. 在虚拟机列表右键菜单中，`Power On to BIOS`
2. 设置Boot（＋、-），移动CD驱动到第一位
3. 重新启动
