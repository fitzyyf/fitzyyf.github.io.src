title: "uwsgi使用记录"
date: 2013-08-08 10:08
tags: uwsgi
categories:  
- 开发笔记
---

uwsgi 的命令行配置，命令行参数是通过`--`前缀来指定的。常用参数有

1. socket 

		--socket /tmp/uwsgi.sock 		# 绑定到/tmp/uwsgi.sock 指定的UNIX socket
		-s 127.0.0.1:15210 				# 绑定到IP地址为 127.0.0.1 的 15210端口

2. protocol

		--protocol <protocol> 			# 设置默认的通信协议（uwsgi，http，fastcgi）
		
3. processes or workers

		--processes 8 					# 使用8个工作进程
		--workers 4 						# 使用4个工程进行
		-p 8 								# 第一个 --processes 的简化

4. daemonize

		--daemonize /tmp/uwsgi.log 	# 使进程在后台运行，并将日志打到指定的日志文件或者udp服务器

5. master(M)

		--master    						# 开启Master进程
		-M 									# 开启Master进程(简化)

6. threads

		--threads 						# 开启线程操作模式

7. buffer-size(b)

		--buffer-size 					# 设置用于uwsgi包解析的内部缓存区大小 默认是4k。可简化为 -b

8. daemonize2

		--daemonize2 						# 程序加载成功后，以守护进程方式运行

9. 
