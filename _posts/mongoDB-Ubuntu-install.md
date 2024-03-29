title: MongoDB的Ubuntu 配置
tags: mongodb
categories:  
- NoSQL
---
1. 下载MongoDB
		
		wget http://fastdl.mongodb.org/linux/mongodb-linux-x86_64-2.0.5.tgz #64位
		wget http://fastdl.mongodb.org/linux/mongodb-linux-i686-2.0.5.tgz #32位
		
2. 安装运行MongoDB

		#解压缩命令 z-用gzip来压缩/解压缩文档；x-从档案文档中释放文档；v-详细报告tar处理的文档信息；f-使用档案文档或设备
		tar zxvf mongodb-linux-i686-2.0.5.tgz 
		# 进去mongodb安装文件
		cd mongodb/
		# 创建 数据文件
		mkdir datas
		# 创建 日志文件夹
		mkdir logs
		# 创建配置文件文件夹
		mkdir etc
		# 设置配置信息
		vi etc/mongodb.conf
		
3. 最近安装方式（2013-01-27更新）

		# 下载源码包以及相关软件
		# mongoDB的构建工具
		wget http://sourceforge.net/projects/scons/files/scons/2.2.0/scons-2.2.0.tar.gz/download
		wget http://ftp.mozilla.org/pub/mozilla.org/js/js-1.7.0.tar.gz
		wget http://sourceforge.net/projects/pcre/files/pcre/8.32/pcre-8.32.tar.gz/download
		wget http://downloads.mongodb.org/src/mongodb-src-r2.2.2.tar.gz
		# 1. 安装scons
		tar zxvf scons-2.2.0.tar.gz
		cd scons-2.2.0
		python setup.py install
		cd ..
		# 2. 安装pcre
		tar zxvf pcre-8.32.tar.gz
		cd pcre-8.32
		./configure
		make
		make install
		cd ..
		# 3. 安装Spider MonKey
		tar zxvf js-1.7.0.tar.gz
		cd js/src
		export CFLAGS="-DJS_C_STRINGS_ARE_UTF8"
		make -f Makefile.ref
		JS_DST=/usr make -f Makefile.ref export
		cd ../..
		# 4. 安装boost
		yum -y install boost boost-devel
		# 5. 安装mongoDB
		tar zxf mongodb-src-r2.2.2.tar.gz
		cd mongodb-src-r2.2.2
		scons all
		scons --prefix=/usr/local/mongodb --full install
	
	mongodb.conf文件内容
	---
		# 日志文件地址
		logpath=/home/poplar/devloper/mongodb/logs/mongod.log
		# 以守护进程的方式运行MongoDB，创建服务器进程
		fork=true
		#数据库文件
		dbpath=/home/poplar/devloper/mongodb/datas
	其他的参数		
	---
	* 基本参数
			
	
			--quiet                # 安静输出
			--port arg             # 指定服务端口号，默认端口27017
			--bind_ip arg          # 绑定服务IP，若绑定127.0.0.1，则只能本机访问，不指定默认本地所有IP 
			--logpath arg 		  # 指定MongoDB日志文件，注意是指定文件不是目录
			--logappend            # 使用追加的方式写日志
			--pidfilepath arg      # PID File 的完整路径，如果没有设置，则没有PID文件
			--keyFile arg          # 集群的私钥的完整路径，只对于Replica Set 架构有效
			--unixSocketPrefix arg # UNIX域套接字替代目录,(默认为 /tmp)
			--fork                 # 以守护进程的方式运行MongoDB，创建服务器进程
			--auth                 # 启用验证
			--cpu                  # 定期显示CPU的CPU利用率和iowait
			--dbpath arg           # 指定数据库路径
			--diaglog arg          # diaglog选项 0=off 1=W 2=R 3=both 7=W+some reads
			--directoryperdb       # 设置每个数据库将被保存在一个单独的目录
			--journal              # 启用日志选项，MongoDB的数据操作将会写入到journal文件夹的文件里
			--journalOptions arg   # 启用日志诊断选项
			--ipv6                 # 启用IPv6选项
			--jsonp                # 允许JSONP形式通过HTTP访问（有安全影响）
			--maxConns arg         # 最大同时连接数 默认2000
			--noauth               # 不启用验证
			--nohttpinterface      # 关闭http接口，默认关闭27018端口访问
			--noprealloc           # 禁用数据文件预分配(往往影响性能)
			--noscripting          # 禁用脚本引擎
			--notablescan          # 不允许表扫描
			--nounixsocket         # 禁用Unix套接字监听
			--nssize arg (=16)     # 设置信数据库.ns文件大小(MB)
			--objcheck             # 在收到客户数据,检查的有效性
			--profile arg          # 档案参数 0=off 1=slow, 2=all
			--quota                # 限制每个数据库的文件数，设置默认为8
			--quotaFiles arg       #  number of files allower per db, requires --quota
			--rest                 # 开启简单的rest API
			--repair               # 修复所有数据库run repair on all dbs
			--repairpath arg       # 修复库生成的文件的目录,默认为目录名称dbpath
			--slowms arg (=100)    # value of slow for profile and console log
			--smallfiles           # 使用较小的默认文件
			--syncdelay arg (=60)  # 数据写入磁盘的时间秒数(0=never,不推荐)
			--sysinfo              # 打印一些诊断系统信息
			--upgrade              # 如果需要升级数据库
	* Replicaton 参数
			
			--fastsync             # 从一个dbpath里启用从库复制服务，该dbpath的数据库是主库的快照，可用于快速启用同步
			--autoresync           # 如果从库与主库同步数据差得多，自动重新同步
			--oplogSize arg        # 设置oplog的大小(MB)
	* 主/从参数
			
			--master               # 主库模式
			--slave                # 从库模式
			--source arg           # 从库端口号
			--only arg             # 指定单一的数据库复制
			--slavedelay arg       # 设置从库同步主库的延迟时间
	* Sharding(分片)选项
			
			 --configsvr           # 声明这是一个集群的config服务,默认端口27019，默认目录/data/configdb
			 --shardsvr            # 声明这是一个集群的分片,默认端口27018
			 --noMoveParanoia      # 关闭偏执为moveChunk数据保存
	* Replica set(副本集)选项
			
			--replSet arg          # 设置副本集名称
	以上配置都可以在配置文件中增加
	
	修改系统允许的最大连接数
	---
	最大连接数目的限制原因是Linux系统默认最大文件打开数目为1024,用`ulimit -a`查看
![image](/postfiles/images/ulimit-a.png)
		
	修改 `sudo vi /etc/security/limits.conf`
	增加以下代码：
		
		soft nofile 3000  #软限制 可以超过的配置数。
		hard nofile 20000 #硬限制 最大不能超过的配置数
	
	启动
		
		# 指定配置文件启动
		./bin/mongod --config /home/poplar/devloper/mongodb/etc/mongodb.conf
