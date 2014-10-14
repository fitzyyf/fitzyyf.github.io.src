title: "gitignore.io 方便生成.gitignore的工具"
date: 2014-03-08 21:17
tags:
- gitignore
- git
categories:  
- 开发笔记
---

一直对使用git时，每次对生成`.gitignore`都是此次拷贝，比较痛苦。今天偶然间发现了这个站点，很不错！[http://www.gitignore.io](http://www.gitignore.io)。另外也支持命令行形式，其实也是通过网络的方式。大伙一看就明白了。

比如，我这边通过如下命令：

	$ echo "function gi() { curl http://www.gitignore.io/api/\$@ ;}" >> ~/.zshrc && source ~/.zshrc

其他的命令在[http://www.gitignore.io/cli](http://www.gitignore.io/cli)，支持`linux`、`osx`、`Windows`。


具体使用

	$ gi java
	# Created by http://www.gitignore.io
	
	### Java ###
	*.class
	
	# Mobile Tools for Java (J2ME)
	.mtj.tmp/

	# Package Files #
	*.jar
	*.war
	*.ear
	
	# virtual machine crash logs, see http://www.java.com/en/download/help/error_hotspot.xml
	hs_err_pid*
