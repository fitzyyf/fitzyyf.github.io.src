title: maven develop Android Application
date: 2014-10-03 11:58:12
tags: maven
categories: 
- Android

---

> 主要参考 [http://books.sonatype.com/mvnref-book/reference/android-dev.html](http://books.sonatype.com/mvnref-book/reference/android-dev.html)

1. 主要环境配置

		# Android Home
		export ANDROID_HOME=/usr/local/opt/android-sdk
		# Maven Home
		export M2_HOME=/usr/local/opt/maven

2. 在`Maven`仓库中安装`Android`

		# 克隆一份 maven-android-sdk-deployer
		$ git clone git@github.com:mosabua/maven-android-sdk-deployer.git
		$ cd maven-android-sdk-deployer
		# 执行 安装到 本地 Maven 仓库
		$ mvn install -DskipTests
		# 如果只需要安装一个特定的Android版本到本地仓库中，可使用如下命令
		$ mvn install -P 4.4 -DskipTests

	更为详细的 点击 [这里](https://github.com/mosabua/maven-android-sdk-deployer)查看

	我这边主要只安装`Android 4.4W`版本的，需要注意的是，需要保证自己的Android SDK中有对应的Andorid 版本,也就是检查`Andorid_HOME/platforms/`下是否有对应的版本。
		
		 $ mvn clean install -P 4.4W -DskipTests
		 
3. 创建一个支持`Andorid Application`的Maven 工程

	- 用快速命令创建一个`Android`工程（**推荐**）
	
			mvn archetype:generate \
				-DarchetypeArtifactId=android-quickstart \
				-DarchetypeGroupId=de.akquinet.android.archetypes \
				-DarchetypeVersion=1.0.6 \
				-DgroupId=com.company \
				-DartifactId=android-application-name
	
	- 改造现有`Android`工程
		
			$ touch pom.xml
			$ vim pom.xml
		增加如下内容（整个POM文件在这就不拷贝了，主要几点）
		
		1. 依赖，也就是第二步所执行的内容
			
				<dependencies>
					<dependency>
						<groupId>com.google.android</groupId>
						<artifactId>android</artifactId>
						<version>2.1.2</version>
						<scope>provided</scope>
					</dependency>
				 </dependencies>
			
			`scope`为`provided`，表示打包时不进行依赖的处理，只用于编译 
		
		2. 插件配置，也就是 [maven-android-plugin](https://github.com/jayway/maven-android-plugin)
			
                <plugin>
                    <groupId>com.jayway.maven.plugins.android.generation2</groupId>
                    <artifactId>maven-android-plugin</artifactId>
                    <version>2.8.4</version>
                    <configuration>
                        <androidManifestFile>${project.basedir}/AndroidManifest.xml</androidManifestFile>
                        <assetsDirectory>${project.basedir}/assets</assetsDirectory>
                        <resourceDirectory>${project.basedir}/res</resourceDirectory>
                        <nativeLibrariesDirectory>${project.basedir}/src/main/native</nativeLibrariesDirectory>
                        <sdk>
                            <platform>7</platform>
                        </sdk>
                        <deleteConflictingFiles>true</deleteConflictingFiles>
                        <undeployBeforeDeploy>true</undeployBeforeDeploy>
                    </configuration>
                    <extensions>true</extensions>
                </plugin>
	
	 	这样，就改造完成了。			 

4. 之后可以用Maven命令来进行打包处理了

	`mvn clean package -DskipTests android:deploy`

---

这样就OK
