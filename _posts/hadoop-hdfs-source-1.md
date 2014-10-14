title: "Hadoop HDFS-NameNode启动分析篇"
date: 2013-03-15 14:06
tags: hadoop
categories: 
- Hadoop
---

> 最近公司可能在存储方面要使用Hadoop，也就是HDFS的相关功能。正好之前也准备对其进行研究，就暂时停止了去看Spring如何与Hadoop以及MapReduce方面的内容；而专心去看HDFS。这里将个人在学习和研究的步骤记录于此。

前两天围观博客，发现了这张图，感觉不错：

{% img /images/blog/hadoop-final-4.jpg "Hadoop生态圈" %}

[前往真身](http://gigaom.com/2013/03/05/the-hadoop-ecosystem-the-welcome-elephant-in-the-room-infographic/),每个项目都是可以点击的哦。。。

### 版本选择
在版本选择上，我们可以看出，Hadoop有很多的发行版本。

通过了解公司的一些其他部门的使用情况，选择了Cloudera的CDH3U5版本。（前面的环境搭建其实是选择的是Apache Hadoop的发行版本）

### 源码下载

	$ wget http://archive.cloudera.com/cdh/3/hadoop-0.20.2-cdh3u5.tar.gz
	$ tar -zxvf hadoop-0.20.2-cdh3u5.tar.gz
	$ cd hadoop-0.20.2-cdh3u5
	
	
接下来，我们就可以去了解其内部实现方面的东西。因为环境其实已经搭建OK，这里省去XX搭建环境的过程。没有啥东西可以说的，和Hadoop搭建一样。

## 代码结构

    .
    ├── bin             # 启动相关脚本
    ├── build           # 构建目录
    ├── c++             # C++ 客户端
    ├── cloudera        # cloudera 补丁等信息
    ├── conf            # 示例配置信息
    ├── contrib         # 第三方的一些依赖包及工程
    ├── docs            # 文档说明文件（API等）
    ├── example-confs   # 多环境配置示例
    ├── ivy             # ivy2的配置依赖
    ├── lib             # 第三方依赖包
    ├── pids            # pid文件存放文件夹
    ├── sbin            # 不大清楚 好像是一些linux下不同环境编译工具
    ├── src             # 源码目录，我们要关注的
    └── webapps         # webapp相关信息，就是对监控web页面的配置源码

## ─➤出发-从NameNode的说起

查看命令行，可以得到以下一张截图

{% img /images/blog/run-namenode-command-1.png %}

从这里可以看到 `nameNode`其实是由`org.apache.hadoop.hdfs.server.namenode.NameNode`启动的。

在目录`hadoop-0.20.2-cdh3u5`下的`src`目录下的`hdfs`目录中可以找到这个类；`main`中主要内容如下：

{% codeblock NameNode#main lang:java %}
try {
  // 这个方法的作用是，调用`StringUtils`类中的打印相关信息的工具方法，就是打印启动信息，方法当中获取了本地主机的名称`final String hostname = getHostname();`；
  StringUtils.startupShutdownMessage(NameNode.class, argv, LOG);
  // 创建NameNode
  NameNode namenode = createNameNode(argv, null);
  if (namenode != null)
    // 创建NameNode成功，等待RPC通信服务停止
    namenode.join();
} catch (Throwable e) {
  LOG.error(StringUtils.stringifyException(e));
  // 出现异常，退出JVM
  System.exit(-1);
}

{% endcodeblock %}

  在第5行中，创建NameNode，大致如下代码：

{% codeblock NameNode#createNameNode lang:java %}
// 初始化NameNode的配置信息；
if (conf == null)
  conf = new Configuration();
// 解析参数信息（arg），封装为参数信息对象 `StartupOption`；
StartupOption startOpt = parseArguments(argv);
if (startOpt == null) {
  printUsage();
  return null;
}
// 将参数信息设置到变量`dfs.namenode.startup`中；
setStartupOption(conf, startOpt);

switch (startOpt) {
  case FORMAT:
    // 如果传入参数为`-format`，则对其进行配置的校验，判断配置目录是否存在，然后以命令行交互方法确认，对HDFS目录进行格式化。
    // 依据成功后是否退出JVM信息；这一步应当对应命令的`hadoop namenode -format`,也就是对其HDFS进行格式化；这里后续在进入详细了解。
    boolean aborted = format(conf, true);
    System.exit(aborted ? 1 : 0);
  case FINALIZE:
    // 如果传入参数是 `-finalize`，则是需要删除以前升级的一些记录状态，
    // 会导致roolback无法使用，同样也会以交互的方式提示确认是否进行删除；
    aborted = finalize(conf, true);
    System.exit(aborted ? 1 : 0);
  case RECOVER:
    // 如果传入参数`-recover`，表示选择元数据恢复模式（这是啥模式？）看了下代码中的提示信息：
    // 大致意思如下：*这种模式的目的是恢复丢失的文件系统损坏的元数据。元数据恢复模式往往会永久删除您的HDFS文件系统的数据。请备份日志和文件镜像，然后再尝试！*，就不解释了吧。它说的很清楚了。
    NameNode.doRecovery(startOpt, conf);
    return null;
  default:
}
NameNode namenode = new NameNode(conf);
return namenode;

{% endcodeblock %}
  
  从代码来看的话，这中间的`format`、`finalize`、`doRecovery`都是对`FSNamesystem`中的属性`FSDirectory`中的`FSImage`内容进行操作，包括格式化等。相信这个应该就是NameNode的存储数据格式。（好吧 它就是，有人已经分析过了。引用[HDFS源码学习（1）——NameNode主要数据结构](http://jiangbo.me/blog/2012/10/18/hdfs-namenode-datastructure/)，我这里还是按我的阅读方式进行下去吧。）

最后第29行根据配置信息创建`new NameNode(conf)`，它是通过构造函数来初始化的，我们可以看看它到底做初始化了哪些信息：
      	
{% codeblock NameNode#initialize lang:java %}
// 首先使用`InetSocketAddress`来绑定一个套接字Socket，从这里看NameNode可能是采用TCP/IP协议来进行通信。这步做了大量的事情，待会看下去；
InetSocketAddress socAddr = NameNode.getAddress(conf);
// 根据配置信息设置Hadoop的用户和用户组（`UserGroupInformation`），并设置证书信息，通过配置中的`dfs.namenode.keytab.file`和`dfs.namenode.kerberos.principal`
UserGroupInformation.setConfiguration(conf);
SecurityUtil.login(conf, DFSConfigKeys.DFS_NAMENODE_KEYTAB_FILE_KEY, DFSConfigKeys.DFS_NAMENODE_USER_NAME_KEY, socAddr.getHostName());
            
// 设置服务器端的线程数量，如果配置了`dfs.namenode.handler.count`，则使用它，否则默认为10；
int handlerCount = conf.getInt("dfs.namenode.handler.count", 10);
            
// 创建`NameNode`的统计数据的统计变量`NameNodeMetrics`；
myMetrics = new NameNodeMetrics(conf, this);
// 从`fsimage`和`edits log `创建用来维护与Datanode相关的重要列表的`FSNamesystem`，同时也是存储文件元数据信息的对象；
this.namesystem = new FSNamesystem(this, conf);
// 如果用户组是通过`kerheros`或者简单的身份验证的话，那么启动`DelegationTokenSecretManager`守护进程，这个好像是提供授权认证的管理器？不大确认？
if (UserGroupInformation.isSecurityEnabled()) {
    namesystem.activateSecretManager();
}

// 创建其他节点与HDFS的通信RPC服务，这个服务如果被配置的话，那么BackupNode、DataNode和其他所有的服务都应该连接这个RPC服务，客户端只连接NodeNode的RPC通信。通过配置 `dfs.namenode.servicerpc-address`来指定的。
// 在这里 我的理解是，serviceRPCAddress在集群中其他节点的通信都由它来管理，而Client的通信则是this.server来管理，将通信的方式进行隔离。
InetSocketAddress dnSocketAddr = getServiceRpcServerAddress(conf);
if (dnSocketAddr != null) {
    // 如果创建失败的话，则重新创建serviceRpcServer
    // 获取默认的rpc线程数，可以在配置中通过`dfs.namenode.service.handler.count`来设置，默认是10
    int serviceHandlerCount = conf.getInt(DFSConfigKeys.DFS_NAMENODE_SERVICE_HANDLER_COUNT_KEY,
                                DFSConfigKeys.DFS_NAMENODE_SERVICE_HANDLER_COUNT_DEFAULT);
    // 重新创建与其他节点通信的RPC服务
    this.serviceRpcServer = RPC.getServer(this, dnSocketAddr.getHostName(), 
                  dnSocketAddr.getPort(), serviceHandlerCount,
                  false, conf, namesystem.getDelegationTokenSecretManager());
    this.serviceRPCAddress = this.serviceRpcServer.getListenerAddress();
        setRpcServiceServerAddress(conf);
}
// 创建RPC服务,默认的rpc线程数是10，默认端口是8020
this.server = RPC.getServer(this, socAddr.getHostName(),
                socAddr.getPort(), handlerCount, false, conf, namesystem
                .getDelegationTokenSecretManager());

// 设置安全策略
if (serviceAuthEnabled = conf.getBoolean(CommonConfigurationKeys.HADOOP_SECURITY_AUTHORIZATION, false)) {
    this.server.refreshServiceAcl(conf, new HDFSPolicyProvider());
    if (this.server != null) {
        this.server.refreshServiceAcl(conf, new HDFSPolicyProvider());
    }
}

// 监听RPC通信
this.serverAddress = this.server.getListenerAddress(); 
// 设置默认的HDFS访问URL，也就是hdfs协议访问
FileSystem.setDefaultUri(conf, getUri(serverAddress));
LOG.info("Namenode up at: " + this.serverAddress);
        
// 启动http服务器，启动后可以通过http://namenode:50070 访问hdfs的管理页面
startHttpServer(conf);
// 启动当前的RPC服务
this.server.start(); 
            
if (serviceRpcServer != null) {
  // 启动节点通信RPC服务
  serviceRpcServer.start();
}
// 启动回收站清理线程，清理过期文件。注意是真的删除文件彻底删除
startTrashEmptier(conf);
// 注册插件，比如调度方面，通过`dfs.namenode.plugins`来设置
pluginDispatcher = PluginDispatcher.createFromConfiguration(
                conf, DFSConfigKeys.DFS_NAMENODE_PLUGINS_KEY, NamenodePlugin.class);
    pluginDispatcher.dispatchStart(this);
{% endcodeblock %}


  大致上 NameNode的启动 如上所述，这里面还有 RPC协议的问题没有分析，fsImage的结构未分析以及授权身份验证的问题；先到这里,要下班了，后续...

  
