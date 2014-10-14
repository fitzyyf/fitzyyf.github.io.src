title: Mac osx&Linux Shell问题汇总
description: Shell碰到的相关问题汇总，包括Linux和Mac Osx相关的记录
tags: linux
categories:  
- Linux
---

设置终端的主提示符信息。

1. 可以通过 `echo $PS1`来查看当前的设置
2. Bash的主提示符为用户名（\u）、@符号、主机名（\h），当前工作目录的基本名和美元符号。这个提示符将出现在所有的交互式shell中。其他的信息配置如下

参数选项| 说明
-----	| -----------------------------------------
\d		| 日期是“星期 月 日”的格式（如，Wed May 26）
\h  	| 主机名
\n		| 换行符
\nnn	| 对应8进制数nnn的字符
\s		| shell的名称，$0的基名（最后一个斜线后面的部分）
\t		| 当前时间是HH-MM-SS的格式
\u		| 当前用户的用户名
\w		| 当前的工作目录
\W		| 当前工作目录的基本名
\#		| 命令的编号
\!		| 命令的历史编号
\$		| 如果有效的UID是0，是一个#号，否则是$
\\		| 反斜杠
\[		| 开始一个非打印字符序列。可以用来在提示符中嵌入一个终端控制序列
\]		| 非打印字符序列的结尾
\a		| ASCII报警字符
\e		| ASCII擦除符(033)
\H		| 主机名
\T		| 以12小时格式表示的当前时间：HH:MM:SS
\v		| bash的版本，如，2.03
\V		| bash的发行号和路径级别，如，2.03.0
\@		| 以12小时制AM/PM格式表示的当前时间

3. 查找进程的明令 ps 和 grep,例如 我要查看正在运行的`idea`的相关进程

    	ps -ef|grep idea

命令说明：1. ps:显示当前状态处于running的进程
        2. ef 分别为 e 显示所有进程 f 全格式；以全格式显示当前所有的进程
        3. grep:从ps的范围中查找


## 从本地通过ssh拷贝文件到远程服务器上

	scp fs.war root@192.168.83.196:/iflytek/gddev/apache-tomcat-6.0.35/webapps

命令描述：将本地的文件fs.war拷贝到远程服务器192.168.83.196的/iflytek/gddev/apache-tomcat-6.0.35/webapps文件夹中。

## Mac Osx 命令
1. 让隐藏的软件图标在Dock中呈半透明

	启用命令

		defaults write com.apple.Dock showhidden -bool YES && killall Dock
	
	恢复命令

		defaults delete com.apple.dock expose-animation-duration; killall Dock
		
2. 从Mail中拷贝邮件地址的时候只要纯净的email

		defaults write com.apple.mail AddressesIncludeNameOnPasteboard -bool false
3. 在快速查看中直接选中文本

		defaults write com.apple.finder QLEnableTextSelection -bool TRUE;killall Finder
4. 	多版本JDK的设置
	
		export JAVA_HOME=`/usr/libexec/java_home -v '1.6*'`

5. 清理Mac下的无效菜单的相关

	如果需要清理菜单重复项和无效的关联，可以在终端运行下面命令，在本地、系统和用户空间上，重建LS数据库：

		/System/Library/Frameworks/CoreServices.framework/Frameworks/LaunchServices.framework/Support/lsregister 
		-kill -r -domain local -domain system -domain user

	lsregister 命令参数如下：

	* `-kill`：重置全局LS数据库(最先执行)
	* `-lint`：打印详细应用程序文件关联注册中的错误信息
	* `-convert`：将老数据库中的信息注册到新的LS数据库
	* `-load`：加载LS插件
	* `-lazy n`：指定一个注册等待时间
	* `-r`：递归的查找文件夹内容以做关联之用（不包括pkg类型文件和隐藏文件夹下的内容）
	* `-R`：递归的查找文件夹内容以做关联之用（包括pkg类型文件和隐藏文件夹下的内容）
	* `-f`：强制更新所有对应注册信息
	* `-v`：输出lsregister运行详细信息
	* `-dump`：在注册完成后显示数据库内容
	* `-h`：显示此帮助

6. mac禁用spotlight（来源[如何禁用Mac系统的Spotlight](http://www.isofts.org/shutdown-spotlight/))
    
    禁用代码
    
        sudo launchctl unload -w /System/Library/LaunchDaemons/com.apple.metadata.mds.plist
	
    恢复代码
        
        sudo launchctl load -w /System/Library/LaunchDaemons/com.apple.metadata.mds.plist
    
    删除 spotlight 图标
    
        cd /System/Library/CoreServices/
        sudo mv Search.bundle/ Search2.bundle/
        # 接着在 活动管理器中 退出 SystemUIServer 进程.
    
    恢复spotlight图标
    
        cd /System/Library/CoreServices/ 
        sudo mv Search2.bundle/ Search.bundle/
        # 接着在 活动管理器中 退出 SystemUIServer 进程.
 
7. OS X Mountain Lion 修改Doc的背景色

	* 无玻璃效果
		
			defaults write com.apple.dock no-glass -boolean YES; killall Dock
		
	* 恢复为默认
	
			defaults write com.apple.dock no-glass -boolean NO; killall Dock

8. 
