title: "hadoop-NameNode 的主要数据结构"
date: 2013-03-18 13:32
tags:
- hadoop
categories: 
- Hadoop
---

在之前的分析过程中，我们知道NameNode会将文件和目录的元数据信息存储下来，每次保存或者操作HDFS的操作，都会记录到 editlog文件中，当EditLog达到一定大小的时候，namenode会重新将内存中队整个HDFS的目录树和文件元数据刷到fsImage文件中。

那么NameNode有以下几个问题

1. 元数据信息的数据结构是怎样？
2. 它的EditLog在达到一定大小或者某段保存时间段过后，刷新内存文件到fsImage中是如何做到的呢？

## 元数据信息的数据结构

首先，我们从NameNode启动过程可以看到，NameNode启动时，对`FSNamesystem`进行初始化操作，这个类是HDFS文件系统的执行的核心操作,提供了各种增删改查的操作接口。它内部维护了多个数据结构之间的关系，它的启动过程主要为：

它的方法链如下：

{% img /images/blog/FSNamesystem_method_flow.png FSNamesystem方法链 %}

这里面主要的方法如下

{% codeblock FSNamesystem#initialize lang:java %}
private void initialize(NameNode nn, Configuration conf) throws IOException {
	// 设置检查磁盘可用性的间隔,可以通过 dfs.namenode.resource.check.interval 来设置,默认为5000,单位为毫秒
    resourceRecheckInterval = conf.getLong(DFSConfigKeys.DFS_NAMENODE_RESOURCE_CHECK_INTERVAL_KEY,DFSConfigKeys.DFS_NAMENODE_RESOURCE_CHECK_INTERVAL_DEFAULT);
    // 初始化一个"检查NameNode上可用磁盘配置的所有卷进行检查"的检查类,它的初始化过程一会来说
    nnResourceChecker = new NameNodeResourceChecker(conf);
    // 进行资源检查和缓存结果。就是调用 nnResourceChecker.hasAvailableDiskSpace()，它是通过判断文件使用大小来进行是否可用的
    checkAvailableResources();
    // 获取当前系统时间
    this.systemStart = now();
    // 根据配置对象信息初始化一些配置对象参数
    setConfigurationParameters(conf);
    // 创建一个HDFS的身份令牌的管理者
    dtSecretManager = createDelegationTokenSecretManager(conf);
    // 获取当前NN的地址
    this.nameNodeAddress = nn.getNameNodeAddress();
    // 注册MBean
    this.registerMBean(conf);
    // 初始化 存储文件的目录状态的 管理对象
    this.dir = new FSDirectory(this, conf);
    // 获取NameNode的启动选项，是format 还是其他的选项
    StartupOption startOpt = NameNode.getStartupOption(conf);
    // 根据选项夹在FSImage
    this.dir.loadFSImage(getNamespaceDirs(conf), getNamespaceEditsDirs(conf), startOpt);
    // 获取夹在FSImage的时间
    long timeTakenToLoadFSImage = now() - systemStart;
    LOG.info("Finished loading FSImage in " + timeTakenToLoadFSImage + " msecs");
    // 
    NameNode.getNameNodeMetrics().fsImageLoadTime.set((int) timeTakenToLoadFSImage);
    // 初始化一个安全模式对象
    this.safeMode = new SafeModeInfo(conf);
    // 设置 系统中块的总数
    setBlockTotal();
    // 初始化用于 记录所有的block复制请求 的任务线程
    pendingReplications = new PendingReplicationBlocks( conf.getInt("dfs.replication.pending.timeout.sec", -1) * 1000L);
    if (isAccessTokenEnabled) {
		// 设置 主从模式，这里设置当前机器为主机器
      	accessTokenHandler = new BlockTokenSecretManager(true, accessKeyUpdateInterval, accessTokenLifetime);
    }
    // 初始化 心跳 检查 任务线程
    this.hbthread = new Daemon(new HeartbeatMonitor());
    // 租约管理器LeaseManager负责管理租约
    this.lmthread = new Daemon(leaseManager.new Monitor());
    // 定期调用 computeReplicationWork() 的后台任务; 
    this.replthread = new Daemon(new ReplicationMonitor());
    hbthread.start();
    lmthread.start();
    replthread.start();
    // 定期调用hasAvailableResources的NameNodeResourceChecker，
    // 如果被发现有可用的资源不足，使NN进入安全模式。
    // 如果资源已经恢复到可以接受的水平，这个守护进程将导致NN退出安全模式。
    this.nnrmthread = new Daemon(new NameNodeResourceMonitor());
    nnrmthread.start();
    // 保持 DN和JT与NN 和 TT的连接
    this.hostsReader = new HostsFileReader(conf.get("dfs.hosts",""), conf.get("dfs.hosts.exclude",""));
    // 初始化 DN节点的管理线程
    this.dnthread = new Daemon(new DecommissionManager(this).new Monitor(
        conf.getInt("dfs.namenode.decommission.interval", 30),
        conf.getInt("dfs.namenode.decommission.nodes.per.interval", 5)));
    dnthread.start();
    // 设置 可插拔DNS-name/IP-address RackID解析器。
    this.dnsToSwitchMapping = ReflectionUtils.newInstance(
        conf.getClass("topology.node.switch.mapping.impl", ScriptBasedMapping.class, DNSToSwitchMapping.class), 
        conf);
    // 如果DNS切换缓存映射支持，将这些主机包括列表中的网络位置存储在高速缓存中的映射，在后续的呼叫中能得到快速响应。
    if (dnsToSwitchMapping instanceof CachedDNSToSwitchMapping) {
    	// 主机网络位置存储
      	dnsToSwitchMapping.resolve(new ArrayList<String>(hostsReader.getHosts()));
    }
    // 获取当前NN的通信地址
    InetSocketAddress socAddr = NameNode.getAddress(conf);
    // 设置FSN中的NN的位置信息
    this.nameNodeHostName = socAddr.getHostName();
}
{% endcodeblock %}

在 `NameNodeResourceChecker`中，主要检查的位置包括

1. FSNamesystem.getNamespaceDirs(conf)
	
	监控元数据存储位置，也就是`dfs.name.dir`配置的位置，如果没有配置，默认监控`/tmp/hadoop/dfs/name`
	
2. FSNamesystem.getNamespaceEditsDirs(conf)

	监控HDFS的事物文件存储，也就是editlog日志存储位置，即`dfs.name.edits.dir`配置的位置，如果没有设置，同样默认监控`/tmp/hadoop/dfs/name`
	
3. `dfs.namenode.resource.checked.volumes`配置的卷

{% codeblock FSNamesystem#setConfigurationParameters lang:java %}
private void setConfigurationParameters(Configuration conf) 
                                          throws IOException {
    // 内部位置静态的 FSNamesystem
    fsNamesystemObject = this;
    // 设置文件所有者
    fsOwner = UserGroupInformation.getCurrentUser();
    // 授权用户组，如果没有配置，则默认为supergroup
    this.supergroup = conf.get("dfs.permissions.supergroup", "supergroup");
    // 默认启动授权权限，如果配置dfs.permissions为false，则不启动
    this.isPermissionEnabled = conf.getBoolean("dfs.permissions", true);
    // 这里的授权和Unix/Linux下一致，如果没有配置dfs.upgrade.permission，则默认为777
    short filePermission = (short)conf.getInt("dfs.upgrade.permission", 00777);
    // 存储授权信息
    this.defaultPermission = PermissionStatus.createImmutable(
        fsOwner.getShortUserName(), supergroup, new FsPermission(filePermission));
    //
    this.replicator = new ReplicationTargetChooser(
                         conf.getBoolean("dfs.replication.considerLoad", true),
                         this,
                         clusterMap);
    // 默认的数据块的存储拷贝数量，如果没有设置，默认为3
    this.defaultReplication = conf.getInt("dfs.replication", 3);
    // 最大拷贝数量，默认为512
    this.maxReplication = conf.getInt("dfs.replication.max", 512);
    //  最小拷贝数量，默认为1
    this.minReplication = conf.getInt("dfs.replication.min", 1);
    … //一些判断
    // 
    this.maxReplicationStreams = conf.getInt("dfs.max-repl-streams", 2);
    // 
    long heartbeatInterval = conf.getLong("dfs.heartbeat.interval", 3) * 1000;
    // 
    this.heartbeatRecheckInterval = conf.getInt( "heartbeat.recheck.interval", 5 * 60 * 1000); 
    this.heartbeatExpireInterval = 2 * heartbeatRecheckInterval + 10 * heartbeatInterval;
    this.replicationRecheckInterval =  conf.getInt("dfs.replication.interval", 3) * 1000L;
    this.defaultBlockSize = conf.getLong("dfs.block.size", DEFAULT_BLOCK_SIZE);
    this.maxFsObjects = conf.getLong("dfs.max.objects", 0);
    //default limit
    this.blockInvalidateLimit = Math.max(this.blockInvalidateLimit, 
                                         20*(int)(heartbeatInterval/1000));
    //use conf value if it is set.
    this.blockInvalidateLimit = conf.getInt( DFSConfigKeys.DFS_BLOCK_INVALIDATE_LIMIT_KEY, this.blockInvalidateLimit);
    this.blocksInvalidateWorkPct = conf.getFloat( DFSConfigKeys.DFS_NAMENODE_INVALIDATE_WORK_PCT_PER_ITERATION,
      DFSConfigKeys.DFS_NAMENODE_INVALIDATE_WORK_PCT_PER_ITERATION_DEFAULT);
    Preconditions.checkArgument(
      (this.blocksInvalidateWorkPct > 0),
      DFSConfigKeys.DFS_NAMENODE_INVALIDATE_WORK_PCT_PER_ITERATION +
      " = '" + this.blocksInvalidateWorkPct + "' is invalid. " +
      "It should be a positive, non-zero float value " +
      "indicating a percentage.");
    this.blocksReplWorkMultiplier = conf.getInt(
      DFSConfigKeys.DFS_NAMENODE_REPLICATION_WORK_MULTIPLIER_PER_ITERATION,
      DFSConfigKeys.DFS_NAMENODE_REPLICATION_WORK_MULTIPLIER_PER_ITERATION_DEFAULT);
    Preconditions.checkArgument(
        (this.blocksReplWorkMultiplier > 0),
        DFSConfigKeys.DFS_NAMENODE_REPLICATION_WORK_MULTIPLIER_PER_ITERATION +
        " = '" + this.blocksReplWorkMultiplier + "' is invalid. " +
        "It should be a positive, non-zero integer value.");
    this.accessTimePrecision = conf.getLong("dfs.access.time.precision", 0);
    this.allowBrokenAppend = conf.getBoolean("dfs.support.broken.append", false);
    if (conf.getBoolean("dfs.support.append", false)) {
      LOG.warn("The dfs.support.append option is in your configuration, " +
               "however append is not supported. This configuration option " +
               "is no longer required to enable sync.");
    }
    this.isAccessTokenEnabled = conf.getBoolean( DFSConfigKeys.DFS_BLOCK_ACCESS_TOKEN_ENABLE_KEY, false);
    if (isAccessTokenEnabled) {
      this.accessKeyUpdateInterval = conf.getLong( DFSConfigKeys.DFS_BLOCK_ACCESS_KEY_UPDATE_INTERVAL_KEY, 600) * 60 * 1000L;       	this.accessTokenLifetime = conf.getLong( DFSConfigKeys.DFS_BLOCK_ACCESS_TOKEN_LIFETIME_KEY, 600) * 60 * 1000L; // 10 hrs
    }
    LOG.info("isAccessTokenEnabled=" + isAccessTokenEnabled
        + " accessKeyUpdateInterval=" + accessKeyUpdateInterval / (60 * 1000)
        + " min(s), accessTokenLifetime=" + accessTokenLifetime / (60 * 1000)
        + " min(s)");
}
{% endcodeblock %}

## FSImage类

可以通过 `FSImage#loadFSImage`中得到这些结果

1. 首先是一个image head，其中包含：
	
	* imgVersion(int)：当前image的版本信息
	* namespaceID(int)：用来确保别的HDFS instance中的datanode不会误连上当前NN。
	* numFiles(long)：整个文件系统中包含有多少文件和目录
	* genStamp(long)：生成该image时的时间戳信息。

2. 对每个文件或目录的源数据信息

	主要在`FSImage#saveCurrent`中可以查看到如下逻辑
	
	1. 如果是目录，则包含如下信息

		* path(String)：该目录的路径，如`/user/build/build-index`
		* replications(short)：副本数（目录虽然没有副本，但这里记录的目录副本数也为3）
		* mtime(long)：该目录的修改时间的时间戳信息
		* atime(long)：该目录的访问时间的时间戳信息
		* blocksize(long)：目录的blocksize都为0
		* numBlocks(int)：实际有多少个文件块，目录的该值都为-1，表示该item为目录
		* nsQuota(long)：namespace Quota值，若没加Quota限制则为-1
		* dsQuota(long)：disk Quota值，若没加限制则也为-1
		* username(String)：该目录的所属用户名
		* group(String)：该目录的所属组
		* permission(short)：该目录的permission信息，如644等，有一个short来记录。 
	2. 如果是文件，则包含如下信息
		* blockid(long)：属于该文件的block的blockid，
		* numBytes(long)：该block的大小
		* genStamp(long)：该block的时间戳

> 在 《Hadoop权威指南》中对NameNode存储元数据的大小做了个预计：每个文件、目录和数据块的存储信息大约占150个字节。注意它是放在内存中的。

> HDFS的块(block)，默认大小为64MB，与单一磁盘上的文件系统相似；

