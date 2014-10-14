title: Python练手，遍历清理mvn clean
tags: python
categories:  
- 编程语言

---

		#!/usr/bin/python
		# -*- coding: utf-8 -*-
		# 用来处理遍历文件夹中的maven项目，寻找到后执行maven的清理操作

		import  os,sys
		import logging

		if len(sys.argv) < 2:  
		    print u'请输入需要清理的文件夹地址\n'

		log = logging.getLogger('pomclear')

		def runMavenClean(filePath):
		    shOutput = os.popen('(cd '+filePath+';mvn clean)').readlines()
		    for shout in shOutput:
		        if shout!='':
		            log.info('maven清理返回：%s;',shout)

		def findPomFile(filePath):
		    if os.path.isdir(filePath):
		        for fileDir in os.listdir(filePath):
		            if fileDir=='pom.xml':
		                runMavenClean(filePath)
		            nextFile = filePath+'/'+fileDir
		            #print nextFile
		            findPomFile(nextFile)

		def cleanFile(pomdir):
		    if os.path.isdir(pomdir):
		        log.info('需要清理的文件：%s',pomdir)
		        os.chdir(pomdir)
		        cwd = os.getcwd()
		        findPomFile(cwd)
		    else:
		        print pomdir

		pomdirs = sys.argv
		for pomdir in pomdirs:
		    cleanFile(pomdir)
		#raw_input("请输入需要清理的文件夹:");
