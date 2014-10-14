title: "Alfred Search Maven Repository workflow"
date: 2013-04-09 19:51
tags: 
- alfred
categories:  
- 开发笔记
---

一直使用Alfred，最近它发布了2.0版本，其中，包含了一个很有用的功能：workflow，支持流程方式，并支持`shell`、`ruby`、`python`等脚本来编写流程节点需要做的使用，非常方便。

另外，经常在使用`Maven`或者`Gradle`在找相关架包的`groupId`和`ArfactID`时，经常登录[search.maven.org](http://search.maven.org)，然后键入关键字去检索。看到Alfred的这个功能后，就萌生了一个写个去检索[search.maven.org](http://search.maven.org)的workflow。

快下班花了20分钟搞定，真的很容易啊。

效果如下：

{% img /images/blog/Alfred-maven-searc-wf.png %}

选择后，会自动将组件的相关坐标拷贝到剪切板中。比如`org.jboss.netty:netty:3.2.9.Final`

下载：

[Downloads](https://raw.github.com/yfyang/alfred-mavenRepository-workflow/master/Maven%20Repository%20Search.alfredworkflow)

源码：

[Sources](https://github.com/yfyang/alfred-mavenRepository-workflow)
