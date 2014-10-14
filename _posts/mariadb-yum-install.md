title: "Centos 使用YUM安装MariaDB"
date: 2013-03-02 15:54
tags: mysql
categories: 
- MySQL
---

1. 在 `/etc/yum/repos.d/` 下建立 MariaDB.repo,内容如下:

	另外，如果是其他的操作系统，可以在[这里](https://downloads.mariadb.org/mariadb/repositories/)找到相关信息

   	MariaDB.repo:

       	[yfyang@localhost ~]$ cat /etc/yum.repos.d/MariaDB.repo 
		# MariaDB 5.5 CentOS repository list - created 2013-03-02 07:37 UTC
		# http://mariadb.org/mariadb/repositories/
		[mariadb]
		name = MariaDB
		baseurl = http://yum.mariadb.org/5.5/centos6-amd64
		gpgkey=https://yum.mariadb.org/RPM-GPG-KEY-MariaDB
		gpgcheck=1
		[yfyang@localhost ~]$ 

2. 使用YUM安装MariaDB

		yum install MariaDB-server MariaDB-client

3. 安装 MariaDB Cluster(注意，与 MariaDB-server不能共存)

		yum -y install MariaDB-Galera-server MariaDB-client galera

4. 启动数据库

		[yfyang@localhost ~]$ service mysql start
		Starting MySQL SUCCESS! 

5. 修改Root的密码
	
		[yfyang@localhost ~]$ mysqladmin -u root password 'yf@yang'

6. 配置远程访问
	
		[yfyang@localhost ~]$ mysql -uroot -p'yf@yang'
		Welcome to the MariaDB monitor.  Commands end with ; or \g.
		Your MariaDB connection id is 6
		Server version: 5.5.29-MariaDB MariaDB Server

		Copyright (c) 2000, 2012, Oracle, Monty Program Ab and others.

		Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
		
		MariaDB [mysql]> GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'yf@yang' WITH GRANT OPTION;
		Query OK, 0 rows affected (0.00 sec)

		MariaDB [mysql]> FLUSH PRIVILEGES;
		Query OK, 0 rows affected (0.00 sec)

## 碰到了yum时出现错误Error: database disk image is malformed
	
[解决办法](http://mythempire.blogspot.com/2012/06/yumerror-database-disk-image-is.html)
	
主要步骤：

	[yfyang@localhost ~]$ sudo yum clean metadata
	[yfyang@localhost ~]$ sudo yum clean dbcache
	[yfyang@localhost ~]$ sudo yum makecache

## 另外你可能会碰到防火墙关闭端口的问题

1. 最简单，关闭防火墙（不建议）

		[root@localhost /]# chkconfig --level 35 iptables off
		[root@localhost /]# /etc/init.d/iptables stop
	
2. 设置防火墙端口
	
		[root@localhost /]# vi /etc/sysconfig/iptables
		# 增加一行 允许 端口 3306 可以防范
		-A INPUT -m state --state NEW -m tcp -p tcp --dport 3306 -j ACCEPT
		# 重启防火墙
		[root@localhost /]# /etc/init.d/iptables stop
		[root@localhost /]# /etc/init.d/iptables start

其他的配置和Mysql是一样的。

with yfyang
	
