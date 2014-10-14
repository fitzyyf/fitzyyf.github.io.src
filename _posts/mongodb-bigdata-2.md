title: "大数据之mongoDB:环境搭建"
date: 2013-01-27 22:01
tags: mongodb
categories:  
- NoSQL
---

接[上篇](/blog/mongodb-bigdata-1)

## 环境说明

上篇回顾了下**mongoDB**的一些概念，接下来，为了验证一些处理海量数据的能力，我们需要建立一个分布式集群的环境。在这里，我们将相关环境简单说明下：

1. mongo-1主备机器,ip为`192.168.1.102`；
2. mongo-2副机器,ip为`192.168.1.103`。

机器操作系统为 `Ubuntu 12.04`，内存为 `1G`(这里都是虚拟机的方式，没实体机器。)

## 安装mongoDB

当我们说安装mongoDB时，主要是构建核心的数据库服务器`mongod`。它可以作为单个服务器、主从节点、副本集的成员，还可以当作片。

最先稳定版本为`2.2.2`，在[mongoDB Downloads](http://www.mongodb.org/downloads)可以下载到各个系统下的二进制包。

这里我们主要介绍下linux或者unix下安装mongoDB的源码编译和包管理器安装的方式：

1. 源码编译

   [mongoDB的Uuntu配置](/blog/mongoDB-Ubuntu-install)

2. 包管理器（linux-yum\apt,mac ox s-brew）
        
        # brew install mongodb
        yum install mongodb

3. 测试
        
        ./mongod --dbpath /tmp

运行成功。一般建议使用`包管理器`来安装，这样可以得到更好的依赖保证。

然后分别到mongo-1和mongo-2机器上都安装OK好mongoDB。

## 主从复制

主从复制是一种常见的横向扩展方式，在关系数据库中也是如此。它也是mongoDB最常用的复制方式，这种方式非常灵活，可用于备份、故障恢复、读扩展等。在这里我们先进行搭配一个从节点的主节点。

1. 启动主节点机器

        ssh 192.168.1.102
        
        mkdir -p /iflytek/mongodb/master
        mongod --dbpath /iflytek/mongodb/master --master
        
    这样就完成了主服务器的启动。
    
2. 启动从节点机器
        
        ssh 192.168.1.103
        
        mkdir -p /iflytek/mongodb/slave
        mongod --dbpath /iflytek/mongodb/slave --slave --source 192.168.1.102（master_address）
        
    完成从节点服务器的启动。
    
当然，这种方式适合在学习和演练的情况下使用，在生产环境下，可以通过配置文件的方式来启动，具体配置方式[mongoDB的Uuntu配置](../../../2012/06/09/mongoDB-Ubuntu-install)。

### 选项
主从复制一些游泳的选项

* `--only`

    在从节点上指定只复制特定某个数据库（默认的情况是复制所有数据库）
    
* `--slavedelay`

    用在从节点服务器上，当应用主节点的操作时增加延时（单位是秒）。这种节点对用户无意中删除重要文档或者插入了垃圾数据等一些事故有很重要的防护作用。

* `--fastsync`

    以主节点的数据快照为基础启动从节点。

* `--autoresync`

    如果从节点与主节点不同步了，则自动重新同步。

* `--oplogSize`

    主节点oplog的大小（单位为MB）。
    
### 添加及删除源
启动从节点时可以用过`--source`来指定主节点，也可以通过`shell`中来配置这个源。

    use local
    # 将192.168.1.102作为源添加到从节点上
    db.sources.insert({"host":"192.168.1.102"})
    # 删除192.168.1.102:27017源
    db.sources.remove({"host":"192.168.1.102:27017"})

    
## 分片
建立分片有两步：启动实际的服务器，然后决定怎么切分数据

### 分片组成：
* 片

    保存子集合数据的容器。可以为单个的mongod服务器，也可以是副本。

* mongos

    mongos就是mongoDB的路由器进程。它路由所有的请求，然后将结果聚合。本身不存储数据或者配置的信息。

* 配置服务器
     
    用于存储集群的配置信息；数据和片的对应关系。

### 启动服务器

1. 启动配置服务器
    
    它需要最先启动，因为mongos需要使用配置信息。和启动mongod一样
    
        mkdir -p /iflytek/mongodb/config
        mongod --dbpath /iflytek/mongodb/config --port 20000
    
    它不需要很多的空间和资源。（大概200M实际数据=1KB的配置空间）

2. 启动mongos

        mongos --port 30000 --configdb localhost:20000
        
3. 添加片

    上述就说过，片其实就是mongod实例（或者副本）
    
        makir -p /iflytek/mongodb/shard1
        mongodb --dbpath /iflytek/mongodb/shard1 --port 10000
    
    链接mongos，为集群添加一个片
        
        mongo localhost:30000/admin
        
    确定链接的是mongos而不是mongod后，就可以使用`addshard`命令来添加片了:
    
        > db.runCommand({addshard:"localhost:10000",allowLocal:true})
        
### 切分数据

mongoDB不会将存储的每一条数据都直接发布，需要先在数据库和集合的级别上将分片功能打开。
    
    > db.runCommand({"enablesharding":"iflytek_data"})

大致上配置就这样了。更为详细大家可以通过 [mongoDB权威指南](http://www.amazon.cn/gp/product/B0050CQ4MQ/ref=s9_simh_gw_p14_d0_i1?pf_rd_m=A1AJ19PSB66TGU&pf_rd_s=center-2&pf_rd_r=0ZJ842K8V0Q68AT8CXVD&pf_rd_t=101&pf_rd_p=58223152&pf_rd_i=899254051)这本书深入学习。

