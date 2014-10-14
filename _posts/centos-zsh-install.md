title: "CentOS Install Zsh 5.0.5"
date: 2014-03-11 13:25
tags:
- centos
- linux
- tool
categories: 
- Linux
---

> 前提条件： 
>
> 1. `sudo yum install git`
> 2. `sudo yum install ncurses-devel`

1. 编译安装

	1. `[usr@dev01 zsh-5.0.5]$ wget http://sourceforge.net/projects/zsh/files/zsh/5.0.5/zsh-5.0.5.tar.bz2/download && mv download zsh-5.0.5.tar.bz2 `
	2. `[usr@dev01 zsh-5.0.5]$ tar xvjf zsh-5.0.5.tar.bz2`
	3. `[usr@dev01 zsh-5.0.5]$ cd zsh-5.0.5`
	4. `[usr@dev01 zsh-5.0.5]$ ./configure`
	5. `[usr@dev01 zsh-5.0.5]$ make`
	6. `[usr@dev01 zsh-5.0.5]$ sudo make install`

2， 修改默认Shell

	1. `echo "/usr/local/bin/zsh" | tee -a /etc/shells`
	2. `chsh -s /usr/local/bin/zsh`

3. 安装 `oh-my-zsh`

	1. `wget --no-check-certificate https://github.com/robbyrussell/oh-my-zsh/raw/master/tools/install.sh -O - | sh`
	
一切OK!
	
