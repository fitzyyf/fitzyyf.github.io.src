title: "Apache Pig 使用"
date: 2013-06-21 10:34
tags: 
- hadoop
- pig
categories: 
- Hadoop
---

很久没有写博客了，最近忙着这事那事，加之敬佩的领导也离开公司出去“散心”了。所以本来计划的“大数据”研究的事情也不了了之了。

不过，最近部门安排到公司的研究院做相关工作，这边不少需要大数据处理的事情，原来所设想的发展方向又可以好好的折腾把了。

生命在于折腾嘛。

最新一个工作就是了解和熟悉Apach Pig的使用以及其一些工作原理，刚和研究院的同事讨论这个Pig的使用问题，由于我之前在行软的工作方式和这边的不同，讨论了不少时间总说不明白，故此，在这以博文记之。

# 啥是Apach Pig

之前看`Apache Hadoop`时，对于`MapReduce`使用关注较少，但是对于MR的一些服务产品，包括`Hive`等工具也略知一二。这次来到新的工作岗位的第一个任务就是对于这种`MR`工具的使用进行一些数据挖掘和分析。这里他们已经对`Apache Pig`有了一定的使用，所以就选型方面就选了它。那么这周就对`Apache Pig`进行了一番了解。

`Apache Pig`是对处理超大型数据集的抽象层，在`MapReduce`中的框架中有`map`和`reduce`两个函数，如果我们自己去写一个MapReduce的实现，从代码编写到编译以及部署到Hadoop中执行的话，是非常耗时间的，无法做到高效率。`Pig`可以简化`MapReduce`的开发，而且还可以对不同的数据进行转换，同时可以将结果存储到各个介质中。另外它的每个过程都是很清晰开放的，如果我们有需要自己实现加载数据的`LAOD`或者自己存储的`STORGE`以及`UDFs`，都是较简单的。

# Apach Pig 组成部分

`Apache Pig`自带两个模式，`local`模式和`mapreduce`的方式，默认采用`mr`的方式。如果需要在本地测试的话，可以用`local`模式来进行。生产的话，就用`mr`方式。

`Apache Pig`包括一套类似SQL的脚本和执行脚本的工具。类似SQL的脚本在Pig中我们称之为`Pig Latin`，而执行脚本的工具也就是Pig本身了。

Pig的实现有如下5个主要部分组成

1. Pig Latin

	一种相对简单的语言，它可以执行语句，一条语句就是一个操作，它需要输入一些内容（比如代表一个元组集的包，也就是一些数据来源，比如文本文件等），病发出另一个包作为其输出。在这里一个包代表就是一个关系，与关系数据库中的表类似。（元组表示行，每个元组都由字段组成）。
	
	用`Pig Latin`编写的脚本一般都是这样
	
	* 从文件系统（HDFS）中读取数据；
	* 对数据进行一系列操作（filter，group）；
	* 得到的结果关系存储到文件系统(HDFS)中。

2. Zebra Library

	Pig与Hadoop/HDFS的中间层，Zebra是MapReduce作业编写的客户端，Zerbra用结构化的语言实现了对hadoop物理存储元数据的管理也是对Hadoop的数据抽象层；
	
	在Zebra中有2个核心的类 `TableStore`(写)/`TableLoad`(读)对Hadoop上的数据进行操作	

3. Pig中的Streaming

	分为4个组件:
	1. Pig Latin，也就是上面介绍的；
	2. 逻辑层(Logical Layer)；
	3. 物理层(Physical Layer)；
	4. Streaming具体实现(Implementation)，Streaming会创建一个Map/Reduce作业，并把它发送给合适的集群，同时监视这个作业的在集群环境中的整个执行过程

4. MapReduce
	
	hadoop执行Pig产生的Mr程序

5. HDFS 最终存储数据文件系统


# PigLatin实例

[这里更加详细介绍](http://www.ibm.com/developerworks/cn/linux/l-apachepigdataquery/)

	
	-- 注册自己的UDF以及数据库驱动
	REGISTER 'mysql-connector-java-5.1.25.jar'
	REGISTER 'cqa_pig.jar'
	-- 新增用户分析Pig脚本
	LOGS = LOAD 'cqa_logs/{cqa.log-*}' USING com.iflytek.cqa.PigUserLogLoader;
	-- 根据时间排序，14表示时间的位置
	LOG_ORDER_BY_TIME = ORDER LOGS BY $14 DESC;
	-- 根据第三个字段也就是imsi号来分组排序
	LOG_GROUP_BY_IMSI = GROUP LOG_ORDER_BY_TIME by $3;
	-- 分组后删除多余的数据，只留下来一条数据，每个帐号下
	IMSI_LIMT_FIRST_USR = FOREACH LOG_GROUP_BY_IMSI {
		top_rec = LIMIT LOG_ORDER_BY_TIME 1;
		GENERATE FLATTEN(top_rec);
	}
	-- 将新增用户写入Mysql数据库
	STORE NEW_USER INTO 'NEWUSR' USING DBStorage('com.mysql.jdbc.Driver','jdbc:mysql://127.0.0.1:3306/cqa?useUnicode=true&characterEncoding=utf-8&zeroDateTimeBehavior=convertToNull','cqa','cqa_1234','INSERT INTO NEW_USER (uuid,q,category,imei,imsi,time,client_os) VALUES (?,?,?,?,?,?,?)');	

# 一些问题记录
1. 我的日志文件是按照每天存储到Hadoop的HDFS中，如何做到在一个脚本中同时LOAD所有的日志文件；

		# pig -p input=/user/yfyang/log_*,也可以通过正则匹配
		logs = LOAD '$input' USING com.iflytek.cqa.PigUserLogLoader;
	

2. 如何连接远程Hadoop集群，以客户机的方式去访问计算Hadoop？

		$ vim $PIG_HOME/bin/pig
		$ #修改内容
		$ JAVA_HOME=`/usr/libexec/java_home -v 1.6`
		# 设置Hadoop的配置，让这个配置去访问Hadoop集群，最好与集群中的Hadoop配置一致
		$ PIG_CLASSPATH=$HADOOP_HOME/conf 
		$ CLASSPATH=${CLASSPATH}:$PIG_HOME/pig-0.11.1.jar


3. Pig执行临时文件的目录权限问题
	默认是写到 `/tmp`，如果hdfs运行和pig运行用户不一致，会出现权限无法写或者读的问题。可以到 `PIG_HOME/conf/pig.properties`中设定如下内容
		
		pig.temp.dir=/user/yfyang/tmp #指定当前运行Pig的用户可在HDFS中访问和读写的目录即可
4. 
