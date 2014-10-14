title: "使用git-svn工作"
date: 2013-02-21 09:16
tags: git-svn
categories:  
- 开发笔记
---

实在不想在自己的机器上弄两套源代码控制软件的客户端，而对git来说，可以很好的和svn一起工作；这里我就说下和git-svn的一些使用过程记录。

# 初始化

支持版本：git是1.5.3后支持git-svn的，如果你的版本比较低的话，还是要升级下。

使用 git-svn 来初始化本地的svn代码库(repository)

	$ git svn clone https://ip:port/svn/iRime	
	Initialized empty Git repository in /ypr/work/GCZX/irime/.git/

# git 本地工作

这里可以在主干上进行直接工作。

	$ cd irime
	# 创建个新文件
	$ echo '#This is Iflytek Rime Java Framework' > Readme.md 
	$ git add Readme.md  
	$ git commit -m '创建基础描述文件'
	# 提交到远程SVN服务器上
	$ git svn dcommit
	# 获取SVN服务器的最新进展
	$ git svn rebase


# 切换分支工作
	
1. 如果直接在主干上进行工作的，不推荐这么做。建议建立一个工作分支，这样本地就可以有个工作中的分支，主干上的保持svn远程仓库一致.

		$ cd irime
		# 建立一个工作分支叫work
		$ git checkout -b work

2. 在本地工作分支工作

		# 创建个新文件
		$ echo '#This is Iflytek Rime Java Framework' > Readme.md 
		$ git add Readme.md
		$ git commit -m '创建基础描述文件'

3. 提交到远程svn仓库
	
		# 暂存工作分支未提交的修改
		$ git stash
		# 回到主干
		$ git checkout master
		# 获取svn仓库最新更新
		$ git svn rebase
		# 合共主干和工作分支
		$ git merge work
		# 提交到远程SVN服务器上
		$ git svn dcommit

4. 回到工作分支继续

		# 从主干取得svn进展
		$ git checkout work
		$ git rebase master
		$ git stash pop

这里的工作分支，其实可以取得有含义些，比如当我修复一个问题(mybatis的分页插件对缓存的支持不友好时)，我可以切换到如下
	
	$ git checkout -b mybatis-pagination
	$ git rebase master
	$ ..do work

# 备注

更多的参考[官方-点击](http://git-scm.com/book/zh/Git-%E4%B8%8E%E5%85%B6%E4%BB%96%E7%B3%BB%E7%BB%9F-Git-%E4%B8%8E-Subversion),以及[点击](http://trac.parrot.org/parrot/wiki/git-svn-tutorial).
