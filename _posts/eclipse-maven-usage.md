title: "Eclipse 使用 Maven 来进行开发"
date: 2013-08-24 16:18
tags:
- maven
categories: 
- 开发笔记
---

个人开发一直使用的 Intellij IDEA，去年春节的时候，正好[OSChina](http://www.oschina.net)做活动，就购买了IDEA的授权。只是在现在的环境中，还是很多人用的是Eclipse，而Eclipse如果要搞好熟悉的话，好多插件要弄，要折腾。这篇文章也就是为了使用Eclipse而编写。不过我选择的Eclipse是 Spring 发行的 STS。非常不错，感谢Spring。

1. 下载安装 STS

	到[sts](http://www.springsource.org/sts)下载自己操作系统对应的Eclipse STS，也可以选择下载其STS的插件，我无法忍受Eclipse的插件安装的过程和时间，所以我这里选择了直接下载 STS Eclipse发行版本。

	下载后，进行安装即可，这都是 一路点下 既可以了。

2. 配置Maven

	打开STS后，Maven默认情况下使用的是STS自带的Maven，建议更改为 自己安装的Maven, 方式如下：
	
	{% img /images/blog/sts-maven-settings.png %}

3. 导入Maven项目

	选择导入Maven项目：
	
	{% img /images/blog/sts-import-maven-1.png %}
	
	{% img /images/blog/sts-import-maven-2.png %}
	
	{% img /images/blog/sts-import-maven-3.png %}

4. 运行项目

	然后运行项目即可：
	
	{% img /images/blog/sts-run-project-1.png %}

	{% img /images/blog/sts-run-project-2.png %}
	
	{% img /images/blog/sts-run-project-3.png %}

还是感觉IDEA方便些。它可以直接打开pom文件就可以了。
