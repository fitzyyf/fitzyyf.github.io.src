title: "Hadoop工作-安装"
date: 2013-03-02 22:08
tags: 
- hadoop
categories: 
- Hadoop
---

1. 安装

		[root@localhost conf]# wget http://mirror.bit.edu.cn/apache/hadoop/common/hadoop-1.1.1/hadoop-1.1.1.tar.gz
		[root@localhost conf]# mkdir /usr/share/hadoop
		[root@localhost conf]# tar -zvxf hadoop-1.1.1.tar.gz -C /usr/share/hadoop

2. 设置环境变量

		[root@localhost soft]# cd /usr/share/hadoop/hadoop-1.1.1/
		[root@localhost hadoop-1.1.1]# cd conf/
		[root@localhost conf]# vi hadoop-env.sh
		# 修改JAVA_HOME设置
		# The java implementation to use.  Required.
		export JAVA_HOME=/usr/java/jdk1.7.0_15
		# 设置 在任意位置都可以执行hadoop
		[yfyang@localhost conf]$ sudo vi /etc/profile
		# Fixed run hadoop "Warning: $HADOOP_HOME is deprecated."
		export HADOOP_HOME_WARN_SUPPRESS=1
		export HADOOP_HOME=/usr/share/hadoop/hadoop-1.1.1
		export PATH=$PATH:$JAVA_HOME/bin:$JAVA_HOME/jre/bin:$HADOOP_HOME/bin
		export CLASSPATH=.:$JAVA_HOME/lib:$JAVA_HOME/jre/lib:$CLASSPATH

3. 验证

		[yfyang@localhost hadoop-1.1.1]$ hadoop
		Usage: hadoop [--config confdir] COMMAND
		where COMMAND is one of:
		  namenode -format     format the DFS filesystem
		  secondarynamenode    run the DFS secondary namenode
		  namenode             run the DFS namenode
		  datanode             run a DFS datanode
		  dfsadmin             run a DFS admin client
		  mradmin              run a Map-Reduce admin client
		  fsck                 run a DFS filesystem checking utility
		  fs                   run a generic filesystem user client
		  balancer             run a cluster balancing utility
		  fetchdt              fetch a delegation token from the NameNode
		  jobtracker           run the MapReduce job Tracker node
		  pipes                run a Pipes job
		  tasktracker          run a MapReduce task Tracker node
		  historyserver        run job history servers as a standalone daemon
		  job                  manipulate MapReduce jobs
		  queue                get information regarding JobQueues
		  version              print the version
		  jar <jar>            run a jar file
		  distcp <srcurl> <desturl> copy file or directories recursively
		  archive -archiveName NAME -p <parent path> <src>* <dest> create a hadoop archive
		  classpath            prints the class path needed to get the
		                       Hadoop jar and the required libraries
		  daemonlog            get/set the log level for each daemon
		 or
		  CLASSNAME            run the class named CLASSNAME
		Most commands print help when invoked w/o parameters.

	
