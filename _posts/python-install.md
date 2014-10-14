title: "Python安装过程"
date: 2012-11-02 11:24
tags: python
categories:  
- 编程语言
---
# Python安装过程
Python在Linux下的安装步骤，包括对easy_install以及pip工具的安装.

1. python安装
    
        [root@localhost ~]# wget http://www.python.org/ftp/python/2.7.3/Python-2.7.3.tar.bz2
        [root@localhost ~]# tar -xvf Python-2.7.3.tar.bz2
        [root@localhost ~]# cd Python-2.7.3
        [root@localhost Python-2.7.3]# ./configure 
        [root@localhost Python-2.7.3]# make
        [root@localhost Python-2.7.3]# make install
        [root@localhost Python-2.7.3]# python -V
        Python 2.7.3
   
   python安装成功
   
2. easy_install工具安装

        [root@localhost ~]# wget -q http://peak.telecommunity.com/dist/ez_setup.py
        [root@localhost ~]# python ez_setup.py
        [root@localhost ~]# easy_install --help
        Global options:
        --verbose (-v)  run verbosely (default)
        --quiet (-q)    run quietly (turns verbosity off)
        --dry-run (-n)  don't actually do anything
        --help (-h)     show detailed help message
        --no-user-cfg   ignore pydistutils.cfg in your home directory
    
    easy_install安装成功
    
3. pip工具安装

        [root@localhost ~]# easy_install pip
        [root@localhost ~]# pip help
      
   执行成功表示安装完成
  

