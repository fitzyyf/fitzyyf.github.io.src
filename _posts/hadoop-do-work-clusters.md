title: "hadoop 集群配置"
date: 2013-03-03 11:42
tags: 
- hadoop
categories: 
- Hadoop
---

> 注意：防火墙先关闭 [防火墙关闭](http://yfyang.github.com/blog/mariadb-yum-install/) 文章最后节


## Hadoop的安装配置 见前一篇
## 集群主从Node安排

节点类型 | 节点IP  | 节点hostName	| 节点对应Hadoop的节点及作用
------  | ----------  | ---------- | ---------------------
master	 | 192.168.1.102 | hadoop-master| 集群主节点，驻留NameNode和JobTracker守护进程 
backup	 | 192.168.1.104 | hadoop-backup| 驻留SNN守护进程的节点
slave-1 | 192.168.1.102 | hadoop-master | 集群的从节点，驻留DataNode和TaskTracker守护进程
slave-2 | 192.168.1.103 | hadoop-slave | 集群的从节点，驻留DataNode和TaskTracker守护进程
slave-3 | 192.168.1.104 | hadoop-backup | 集群的从节点，驻留DataNode和TaskTracker守护进程
	
## 网络配置
修改 `/etc/hosts` 配置 host-name，并在每台机器上复制过去。
	
	[yfyang@localhost ~]$ sudo vi /etc/hosts
	192.168.1.102 hadoop-master
	192.168.1.103 hadoop-slave
	192.168.1.104 hadoop-backup
	
## 定义公共帐号
从一个节点的用户帐号到目标机上的另一个用户帐号。对于Hadoop，所有的集群节点上的帐号都应该又相同的用户名，我的安装过程中就是用`yfyang`，级别是普通用户级别。
	
以下参考《Hadoop实战》
	
### 验证SSH（省略了，Goole一堆结果）
### 生成SSH密钥
	主节点上生成ssh密钥，分发到其他从机器上
				
		[yfyang@localhost ~]$ ssh-keygen -t rsa
		# 提示密钥文件保存路径和密码（注意，Hadoop使用的是无密码的密钥，所以不要输入密码）
			
### 将公钥分布并登录验证
1. 使用`scp`命令将公钥复制到各个从节点上
		
		[yfyang@localhost ~]$ scp .ssh/id_rsa.pub yfyang@192.168.1.103:~/master_key
		[yfyang@localhost ~]$ scp .ssh/id_rsa.pub yfyang@192.168.1.104:~/master_key
			
2. 手动登录到目标节点，设置主节点的密钥为授权密码（如有其他授权，则采用追加的方式）
		
		[yfyang@localhost ~]$ mv master_key ~/.ssh/authorized_keys 
		# 追加授权的话
		[yfyang@localhost ~]$ cat master_key >> .ssh/authorized_keys
			
3. 验证是否成功
		
		[yfyang@localhost ~]$ ssh hadoop-slave
		he authenticity of host 'hadoop-slave (192.168.1.103)' can't be established.
		RSA key fingerprint is 68:90:d5:ba:66:db:74:57:e7:e4:a0:3b:db:f3:7b:49.
		Are you sure you want to continue connecting (yes/no)? yes
		Warning: Permanently added 'hadoop-slave' (RSA) to the list of known hosts.
		Last login: Sun Mar  3 11:32:41 2013 from 192.168.1.100
			
### 配置 Hadoop
#### 1-进入 `HADOOP_HOME`目录下

		[yfyang@localhost ~]$ cd /usr/share/hadoop/hadoop-1.1.1/
		[yfyang@localhost hadoop-1.1.1]$ ls
		bin          hadoop-ant-1.1.1.jar          ivy          sbin
		build.xml    hadoop-client-1.1.1.jar       ivy.xml      share
		c++          hadoop-core-1.1.1.jar         lib          src
		CHANGES.txt  hadoop-examples-1.1.1.jar     libexec      webapps
		conf         hadoop-minicluster-1.1.1.jar  LICENSE.txt
		contrib      hadoop-test-1.1.1.jar         NOTICE.txt
		docs         hadoop-tools-1.1.1.jar        README.txt
		[yfyang@localhost hadoop-1.1.1]$ cd conf/
		[yfyang@localhost conf]$ ls
		capacity-scheduler.xml      hadoop-policy.xml      slaves
		configuration.xsl           hdfs-site.xml          ssl-client.xml.example
		core-site.xml               log4j.properties       ssl-server.xml.example
		fair-scheduler.xml          mapred-queue-acls.xml  taskcontroller.cfg
		hadoop-env.sh               mapred-site.xml
		hadoop-metrics2.properties  masters

#### 2-配置`core-site.xml`

		[yfyang@localhost conf]$ vi core-site.xml 

{% codeblock core-site.xml lang:xml %}
<?xml version="1.0"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
	<property>
		<!-- 设置namenode所在主机，端口号是9000 -->
		<name>fs.default.name</name>
		<value>hdfs://hadoop-master:9000</value>
	</property>
	<property>
		<!-- hadoop分布式文件系统文件存放位置都是基于hadoop.tmp.dir目录的，namenode的名字空间存放地方就是 ${hadoop.tmp.dir}/dfs/name, datanode数据块的存放地方就是 ${hadoop.tmp.dir}/dfs/data，所以设置好hadoop.tmp.dir目录后，其他的重要目录都是在这个目录下面，这是一个根目录。-->
		<name>Hadoop.tmp.dir</name> 
		<value>/tmp/hadoop</value>
	</property>
</configuration>
{% endcodeblock %}
		
#### 3-配置`mapred-site.xml`
		
		[yfyang@localhost conf]$ sudo vi mapred-site.xml
		
{% codeblock mapred-site.xml lang:xml %}
<?xml version="1.0"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
	<property>
		<!-- 定义JobTracker所在的主节点。 -->
		<name>mapred.job.tracker</name>
		<value>hadoop-master:9001</value>
	</property>
</configuration>
{% endcodeblock %}
		
#### 4-配置 `hdfs-site.xml`
		
		[yfyang@localhost conf]$ sudo vi hdfs-site.xml
		
{% codeblock hdfs-site.xml lang:xml %}
<?xml version="1.0"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
	<property>
		<!-- 增大HDFS的备份参数 -->
		<name>dfs.replication</name>
		<value>3</value>
	</property>
</configuration>
{% endcodeblock %}
	
#### 5-修改 `msters`
		
		[yfyang@localhost conf]$ sudo vi masters
		hadoop-backup
	
#### 6-修改 `slaves`
	
		[yfyang@localhost conf]$ sudo vi slaves 
		hadoop-master
		hadoop-slave
		hadoop-backup

#### 7-复制各个配置到其他从节点上:
	
		[yfyang@localhost conf]$ scp core-site.xml hadoop-slave:/usr/share/hadoop/hadoop-1.1.1/conf/core-site.xml
		[yfyang@localhost conf]$ scp hdfs-site.xml root@hadoop-slave:/usr/share/hadoop/hadoop-1.1.1/conf/hdfs-site.xml 
		[yfyang@localhost conf]$ scp mapred-site.xml root@hadoop-slave:/usr/share/hadoop/hadoop-1.1.1/conf/mapred-site.xml
		[yfyang@localhost conf]$ scp masters root@hadoop-slave:/usr/share/hadoop/hadoop-1.1.1/conf/masters 
		[yfyang@localhost conf]$ scp slaves root@hadoop-slave:/usr/share/hadoop/hadoop-1.1.1/conf/slaves
		# 其他机器复制...
		
#### 8-格式化HDFS准备好存储数据(这步一定要进行)
	
		[yfyang@localhost conf]$ hadoop namenode -format
	
#### 9-启动守护进程
		
		[yfyang@localhost ~]$ start-all.sh

> 这里需要注意一点，如果用户是不是root的话，比如我这里就是yfyang ，需要将 Hadoop_home的文件夹及子文件都授权给当前用户,当然，如果Hadoop_Home在当前用户的目录下面也就是`~`目录下，可以忽略这个问题
> `sudo chown yfyang /usr/share/hadoop/hadoop-1.1.1/`
	
#### 10-关闭Hadoop(关闭命令)
		
		[yfyang@localhost ~]$ stop-all.sh
	
#### 11-确认是否成功(在其他节点也执行查看)
		
		[yfyang@localhost ~]$ jps
		
#### 12-最后成功的话应当是这样的
	
{% img /images/blog/hadoop-clusters.png %}

### 查看hdfs的分布式文件系统

详细命令 在[这里](http://hadoop.apache.org/docs/r1.1.1/commands_manual.html)

#### 1-查看目录结构命令

	hadoop fs -ls /

#### 2-创建文件夹

	hadoop fs -mkdir /fs-dir

### 基于Web的集群用户界面

#### 1- HDFS的WEB界面快照

当集群环境搭建完成后，可以通过NameNode的50070端口来查看信息，它会提供一些常规报告，描绘了集群上HDFS的状态视图。

#### 2- MapReduce的WEB界面快照

通过JobTracker提供作业运行状态的近似视图,可以监控活跃的mapReduce作业，并访问每个map和reduce任务的日志。还可以活得以前提交作业的日志，以帮助我们程序调试。

{% img /images/blog/clusters-hadoop-mapreduce.png %}


> 下面搞搞Spring-Hadoop的整合方式以及运行开发模式


