title: "Maven版本发布"
date: 2012-11-29 11:17
tags: maven
categories: 
- 持续交付
---
#Maven版本管理-记CDMS的1.0发布记录
##Maven配置
1. 首先在Maven的POM中，如果是父子关系中，只需要在父亲的POM文件中增加SCM信息管理，包括开发分支的地址等，如下所示：

        <scm>
		    <connection>scm:svn:https://192.168.75.168:8888/svn/CDMS</connection>
		    <developerConnection>scm:svn:https://192.168.75.168:8888/svn/CDMS</developerConnection>
		    <url>https://192.168.75.168:8888/svn/CDMS</url>
	    </scm>
	
2. 增加release配置，主要是配置想中的tagBase中的tag配置，这样，在后续中发布时，可以通过maven命令直接创建SVN的 Tag

        <build>
		    <plugins>
			    <plugin>
				    <groupId>org.apache.maven.plugins</groupId>
				    <artifactId>maven-release-plugin</artifactId>
				    <version>2.3.2</version>
				    <configuration>
					    <tagBase>https://192.168.75.168:8888/svn/CDMS/Tags/</tagBase>
				    </configuration>
			    </plugin>
		    </plugins>
	    </build>
	
##开始发布
1. 执行命令

        mvn release:prepare

    过程中有如下的提示

        What is the release version for "cdms-parent"? (com.iflytek.ci:parent) 1.0: : 
        你想将1.0-SNAPSHOT发布为什么版本？默认是1.0
        What is the release version for "domain"? (com.iflytek.ci:domain) 1.0: : 
        What is the release version for "persistence"? (com.iflytek.ci:persistence) 1.0: : 
        What is the release version for "service"? (com.iflytek.ci:service) 1.0: : 
        What is the release version for "web"? (com.iflytek.ci:web) 1.0: : 
        What is the release version for "cdms"? (com.iflytek.ci:cdms) 1.0: : 
        What is SCM release tag or label for "cdms"? (com.iflytek.ci:cdms) cdms-1.0: : 
        主干上新的版本是什么？默认为1.1-SNAPSHOT
        What is the new development version for "cdms-parent"? (com.iflytek.ci:parent) 1.1-SNAPSHOT: : 
        What is the new development version for "domain"? (com.iflytek.ci:domain) 1.1-SNAPSHOT: : 
        What is the new development version for "persistence"? (com.iflytek.ci:persistence) 1.1-SNAPSHOT: : 
        What is the new development version for "service"? (com.iflytek.ci:service) 1.1-SNAPSHOT: : 
        What is the new development version for "web"? (com.iflytek.ci:web) 1.1-SNAPSHOT: : 
        What is the new development version for "cdms"? (com.iflytek.ci:cdms) 1.1-SNAPSHOT: : 

2. 执行完成后，我们去查看相关的POM信息，会发现，Maven已经将开发分支中的pom文件中的版本升级为 1.1-SNAPSHOT

        <groupId>com.iflytek.ci</groupId>
        <artifactId>cdms</artifactId>
        <version>1.1-SNAPSHOT</version>
	
    原来的`version` 为 1.0-SNAPSHOT 版本

3. 检查我们配置的release插件中的SVN的Tag地址中，会发现出现一个新的Tag,tag名称为 `cdms-1.0`

    检查 `https://192.168.75.168:8888/svn/CDMS/Tags/` 出现一个新的Tag

## 其他的说明

1. 现在我们想要发布1.1.0，然后将主干升级为1.2.0-SNAPSHOT，同时开启一个1.1.x的分支，用来修复1.1.0中的bug。
首先，在发布1.1.0之前，我们创建1.1.x分支，运行如下命令：

        mvn release:branch -DbranchName=1.1.x -DupdateBranchVersions=true -DupdateWorkingCopyVersions=false
  
2. 如果我们的项目需要在升级后，将最新的Tag（也就是刚刚生成的Tag目录下的源代码）打包分发到远程Maven仓库中，那么可以使用如下命令：   

        mvn release:perform

3. 注意，release是执行过程中依赖命令 SVN，所以需要在终端或者cmd命令行中，确保命令 `svn -version` 能正确运行。
