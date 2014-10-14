title: "rvm 安装及统一管理相关gem等环境"
date: 2013-01-27 21:58
tags: ruby
categories:  
- 开发笔记

---

# rvm 安装及统一管理相关gem等环境的配置

## 安装RVM

启动`Terminal`执行：

	curl -L https://get.rvm.io | bash -s stable --ruby

等待执行完成成功。

## 安装Ruby&rubygems
现在开始在终端中通过rvm来安装ruby

	rvm install 1.9.3
	rvm use 1.9.3
	rvm rubygems lastst
	
## 设置gem Path

	PATH:$HOME/.rvm/gems/ruby-1.9.3-p374/bin
