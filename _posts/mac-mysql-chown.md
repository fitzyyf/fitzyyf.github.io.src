title: mysql安装目录权限导致mysql无法启动
tags: mysql
categories: 
- MySQL
---
昨天安装完brew，将/usr/local下的所有文件的权限都给当前用户了。没有注意。
今天早上起来，准备启动mysql，无法启动，打开系统设置中的Mysql启动项。
提示权限被修改无法启动。

修改用户权限

输入如下命令OK（在 stackoverflow 上找到的），

		sudo chown -RL root:mysql /usr/local/mysql
		sudo chown -RL mysql:mysql /usr/local/mysql/data
		sudo /usr/local/mysql/support-files/mysql.server start
一切OK.
