title: "Hadoop HDFS Java API 开发的一些注意事项"
date: 2013-03-13 13:17
tags:
- hadoop
categories: 
- Hadoop
---

## 环境问题

需要注意如果是利用一台远程的机器来运行伪分布式环境然后通过开发环境连接时，需要将HDFS的协议连接通过hostname指定到当前机器的IP上；否则Hadoop在启动时，会绑定当前机器的本地IP`127.0.0.1`，这样远程开发环境就无法连接了；当然不用hostname用ip也可以。类似这样

1. hosts文件

{% codeblock hosts %}
192.168.11.244  yfyang-PC
{% endcodeblock %}

2. `core-site.xml`文件

{% codeblock core-site.xml lang:xml %}
<property>
  <name>fs.default.name</name>
  <value>hdfs://yfyang-PC:8020</value>
</property>
{% endcodeblock %}

3.  `masters`及`slaves`文件中，将`localhost`修改为`yfyang-PC`

## 版本问题

刚开始，我开发环境使用的客户端版本是apache发布的1.1.1，而远程服务器上用的是 cdh3u5，导致在运行测试代码出现`java.io.EOFException`问题，最后更换为cloudera的客户端后就正常了。

{% codeblock version %}
      compile("org.apache.hadoop:hadoop-client:0.20.2-cdh3u5")
 {% endcodeblock %}
 
## 客户端代码
设置Configuration时指定远程Hadoop的HDFS信息；例如

{% codeblock configurartion lang:java %}
     Configuration conf = new Configuration();
     conf.set("fs.default.name", "hdfs://192.168.11.244:8020");
{% endcodeblock %} 
 
