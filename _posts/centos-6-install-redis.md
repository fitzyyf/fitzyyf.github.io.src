title: "centos 6 Redis 入门及安装"
date: 2013-02-16 13:18
tags: 
- nosql
- centos
- redis
categories: 
- NoSQL
---

安装
===

1. 下载redis并编译

        $ wget http://redis.googlecode.com/files/redis-2.6.10.tar.gz
        $ tar zxvf redis-2.6.10.tar.gz 
        $ cd redis-2.6.10
        $ make
  

2. 测试下，发现提示以下信息

        $ make test 
        
        cd src && make test
        make[1]: Entering directory `/home/yfyang/dev-soft/redis-2.6.10/src'
        You need tcl 8.5 or newer in order to run the Redis test
        make[1]: *** [test] Error 1
        make[1]: Leaving directory `/home/yfyang/dev-soft/redis-2.6.10/src'
        make: *** [test] Error 2
   
   安装提示所说，安装tcl

        $ sudo yum -y install tcl
   
3. 安装完成后，即可执行 在 `redis-2.6.10`目录下，进行安装了。执行`sudo make install`,如果安装不成功，也可以再次执行`make test`，确定下是什么原因。

入门
===
Redis安装成功后，可以分为以下5个命令和一个`redis.conf`配置文件:

1. redis-benchmark
2. redis-check-aof
3. redis-check-dump:
4. redis-cli: redis的命令行客户端工具；
5. redis-server: redis服务的守护进程。

`redis.conf`的主要配置如下：

* daemonize: 是否以守护进程后台启动；
* pidfile: pid文件位置；
* port: redis服务的监听端口；
* timeout: 客户端请求超时时间；
* loglvevl: 日志的信息级别；
* logfile: 日志文件的存储位置；
* databases: 开启数据库的数量；
* save * *: 保存快照的频率，第一个`*`表示多长时间，第二个表示执行多少次写操作；
* rdbcompression: 是否启动压缩机制；
* dbfilename: 数据库快照文件名；
* dir:数据库快照保存的路径；
* appendonly: 是否开启appendonly,开启的话，每次写操作会记录一条log
* appendfsync: 设置appendonly如果同步到磁盘中
    1. 强制调用fsync
    2. 每秒启用一次fsync
    3. 不调用fsync等待系统自己同步

配置
---
1. 设置数据快照文件夹和日志文件
    
        sudo mkdir -p /yfyang/db/redis
        sudo mkdir -p /yfyang/log/redis
    
2. 配置`redis.conf`
    修改 `daemonize` 为`true`，保证后台运行；设置`dir`为 `/yfyang/db/redis`以及`logfile`为`/yfyang/log/redis/redis.log`,设置好应该如下：
    
        daemonize true
        
        dir /yfyang/db/redis
        logfile /yfyang/log/redis/redis.log
    
3. 启动服务 `redis-server /yfyang/etc/redis.conf`

先简单记录到这，下回继续。
 


    
