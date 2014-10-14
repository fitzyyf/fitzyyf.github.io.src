title: Centos安装Jenkins 并配置一个Maven项目的自动构建
date: 2014-10-14 21:31:56
tags: 
- jenkin
- maven
- centos
categories:  
- 持续交付

---

>  1.  `CentOS`通过`yum`安装`jenkins`
>  2.  配置`Maven`项目
>  3. `Tomcat`部署

1. `yum`安装`jenkins`
	
	> 参考地址： [https://wiki.jenkins-ci.org/display/JENKINS/Installing+Jenkins+on+RedHat+distributions](https://wiki.jenkins-ci.org/display/JENKINS/Installing+Jenkins+on+RedHat+distributions)
	
	-  创建 `jenkins`的 `yum`的源文件
	
			$ cd  /etc/yum.repos.d/
			$ touch jenkins.repo
			$ vim jenkins.repo
		
		在 `jenkins.repo`中加入以下内容：
			
			[jenkins]
			name=Jenkins
			baseurl=http://pkg.jenkins-ci.org/redhat
			gpgcheck=1
		
	- `rpm`增加`jenkins`源的`key`
		
			$ sudo rpm --import http://pkg.jenkins-ci.org/redhat/jenkins-ci.org.key
	
	- 安装 `jenkins`
	
			$ sudo yum install jenkins
			
	- 配置`jenkins`

		主要可以配置`jenkins`的运行端口和监听端口，以及启动用户，配置文件地址为`/etc/sysconfig/jenkins`，下面为主要修改内容：
		
			$ sudo vim /etc/sysconfig/jenkins
			# 修改启动用户为root,默认为jenkins
			JENKINS_USER="root"
			# 修改运行端口为9999，默认为8080
			JENKINS_PORT="9999"
			# 修改Ajp13端口为8189，默认为8009
			JENKINS_AJP_PORT="8189"

	- 启动`jenkins`

			sudo service jenkins start

2. 配置一个`Maven`项目
	
	- 在浏览器中打开[http://centos_server_ip:9999](http://centos_server_ip:9999),通过菜单`新建`，输入任务名称，并选择一个`Maven项目`即可完成Maven项目的创建。
	
	- 配置截图：
	
		{% img /images/ci/maven_Config_Jenkins.png %}
	
		{% img /images/ci/mavne_build_cli.png %}
	
		这里的Maven构建指令中的`-P`说明，可点击[http://maven.apache.org/guides/introduction/introduction-to-profiles.html](http://maven.apache.org/guides/introduction/introduction-to-profiles.html)进行了解

3. 在`Maven`项目构建任务中增加`Tomcat`配置

	这里主要负责，在`Maven`构建完成后，将`War`包自动发布到`Tomcat`中，当然也有很多类似`Maven Tomcat Deploy`的Maven插件，但是个人感觉都不是很稳定，不如执行调用系统命令的稳定和方便。
	
	{% img /images/ci/tomcat_config.png %}

这样基本的一个工程，自动构建就完成了。
	
