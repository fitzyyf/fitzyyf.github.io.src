title: "Sonar 持续代码检查工具"
date: 2012-12-8 11:12
tags: sonar
categories: 
- 持续交付
---
## Sonar概述
Sonar是一个代码质量管理的开放平台，且可以通过插件机制，Sonar可以集成不同的测试工具、代码分析工具以及集成工具。Sonar并非简单的把不同的代码检查工具结果直接显示在Web画面，而是通过不同的插件对这些结果进行再加工处理。通过量化的方式来度量代码质量的变化，从而达到方便地对不同规模和种类的工程进行代码质量管理。

另外，对于IDE的支持，Sonar可以在Eclipse和Intellij IDEA这些工具中联机查看结果；同时也提供了大量的集成工具接口，可以方便的在持续集成中使用Sonar。
## 安装
下载Sonar包 [Sonar3.zip](http://dist.sonar.codehaus.org/sonar-3.1.1.zip)

解压ZIP包后，直接解压到任意目录，Sonar自带了Jetty6的应用服务器环境。所以可以直接使用，当然，Sonar也支持在Apache Tomcat中运行。

启动： 在Sonar的bin目录下，windows-X86-64\StartSonar.bat 即可。然后在浏览器中访问：[http://localhost:9000](http://localhost:9000)

插件安装：首先访问 Sonar 主页中 Dashboard > Sonar > Documentation > Sonar Plugin Library 路径，选择一个插件，点击下载路径，将下载的*.jar放到sonar_home\extensions\plugins路径下。重启sonar，刚下载的插件即可开始运行。

### 数据库设置
Sonar默认使用的是 Derby 数据库。所以当正式使用，建议采用主流的关系型数据库，比如Mysql。 在这里，我就以Mysql为例。

1. 在 Mysql中创建 sonar拥护
	
		CREATE USER sonar IDENTIFIED BY 'sonar';
		GRANT ALL PRIVILEGES ON *.* TO 'sonar'@'localhost' IDENTIFIED BY 'sonar' WITH GRANT OPTION;
	
2. 将Mysql的JDBC驱动，复制到 sonar_home\extensions\jdbc-driver\mysql 目录下；
3. 修改 sonar_home\conf\sonar.propertis文件

		sonar.jdbc.url: jdbc:mysql://localhost:3306/sonar?useUnicode=true&characterEncoding=utf8
		sonar.jdbc.driverClassName: com.mysql.jdbc.Driver
 4. 重启sonar即可
 
## 使用 Sonar
sonar支持 Maven、ant等构建工具，另外还支持一些AIX、HPUX等工具。在这里，只介绍Maven的相关配置。

### 修改Maven的settings.xml
关于sonar的配置信息并不是在每个项目的pom.xml文件中，而是在maven的配置文件 settings.xml文件中。具体配置如下：
	
	<profile>
     <id>sonar</id>
     <activation>
         <activeByDefault>true</activeByDefault>
     </activation>
     <properties>
          <sonar.jdbc.url>
          jdbc:mysql://localhost:3306/sonar?useUnicode=true&characterEncoding=utf8
          </sonar.jdbc.url>
          <sonar.jdbc.driver>com.mysql.jdbc.Driver</sonar.jdbc.driver>
          <sonar.jdbc.username>sonar</sonar.jdbc.username>
          <sonar.jdbc.password>sonar</sonar.jdbc.password>
          <sonar.host.url>http://localhost:9000</sonar.host.url>
     </properties>
  </profile>
  
Maven插件会自动将所需的数据上传到Sonar服务器上。上传成功后，sonar安装的插件会对这些数据进行分析和处理，并以各种方式显示给用户，从而使用户方便的对代码质量的监控和管理。

### 执行Maven命令分析代码
执行命令：
	
	 mvn clean install sonar:sonar
