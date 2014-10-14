title: Maven 命令笔记
tags: maven
category: 
- 持续交付
---

Maven 命令
---
1.  创建普通的Java项目

		mvn archetype:create 
			-DgroupId=<project's group id> 
			-DartifactId=<project's artifact id>

2. 创建Web项目
		
		mvn archetype:create 
			-DgroupId=<project's group id> 
			-DartifactId=<project's artifact id> 
			-DarchetypeArtifactId=maven-archetype-webapp
	DarchetypeArtifactId，项目骨架标志符号。
			
3. 创建一个支持Flex的maven工程
		
		mvn archetype:generate 
			-DarchetypeRepository=https://repository.sonatype.org/content/groups/public
			-DarchetypeGroupId=org.sonatype.flexmojos
			-DarchetypeArtifactId=flexmojos-archetypes-application
			-DarchetypeVersion=3.8
	描述：
	* archetype:generate:maven标准声明周期中的一个，创建一个新的项目
	* -DarchetypeRepository 告知资源路径
	* DarchetypeGroupId、DarchetypeArtifactId、DarchetypeVersion，定位flexmojos项目，分别表示组名、构件名、版本号
4. 部署远程文件
	
		mvn deploy:deploy-file -DgroupId=<group-id> \
			  -DartifactId=<artifact-id> \
			  -Dversion=<version> \
			  -Dpackaging=<type-of-packaging> \
			  -Dfile=<path-to-file> \
			  -DrepositoryId=<id-to-map-on-server-section-of-settings.xml> \
			  -Durl=<url-of-the-repository-to-deploy>

5. 部署本地包以及源码包
        
     源码包
           
           mvn install:install-file -Dfile=itextpdf-5.3.3-sources.jar -DgroupId=com.itextpdf -DartifactId=itextpdf -Dversion=5.3.3 -Dpackaging=java-source
     
    二进制包
          
            mvn install:install-file  -DgroupId=com.itextpdf -DartifactId=itextpdf -Dversion=5.3.3 -Dpackaging=jar -Dfile=itextpdf-5.3.3.jar

6. 其他常用命令
mvn -v   
mvn help:system  
mvn clean compile  
mvn clean package  
mvn clean test  
mvn clean deploy //部署到版本仓库  
mvn clean install //使其他项目使用这个jar,会安装到maven本地仓库中  
mvn archetype:generate //创建项目架构  
mvn dependency:list //查看已解析依赖  
mvn dependency:tree  
mvn dependency:analyze  
mvn install -Dmaven.test.skip=true// -D 参数的使用，这里是跳过test阶段  
-am, --also-make :同时构建所列模块的依赖模块    
-amd -also-make-dependents 同时构建依赖于所列模块的模块 mvn clean install -pl account-parent -amd -rf account-email  
-pl, --projects <arg> 构建指定的模块，模块间用逗号分隔 mvn clean install -pl account-email,account-persist  
-rf -resume-from <arg> 从指定的模块回复反应堆  mvn clean install -rf account-email  
mvn help:active-profiles :查看当前激活的profiles  
mvn help:all-profiles : 查看所有profiles  
mvn help:effective -pom 查看完整的pom信息  


	
