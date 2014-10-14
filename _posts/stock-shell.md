title: "常用Shell命令整理※一"
date: 2014-03-04 14:42
tags: linux
categories:  
- Linux
---

> 大部分经过网络的搜索而来，并经过了验证


## 一、常用命令

1. 文件相关

    命令 | 作用 
    ------------ | -------------
    `find ./ -type d -name ".svn"`|查找所有.svn文件夹
    `find ./ -name ".svn" -exec rm -rf {} \;`|查找并删除.svn文件夹 
    `cat /etc/hosts \| tail -n +2 \| head -n 20`|从第二行开始，显示20行的内容
    `tail -n +2 `|（从第二行开始），如果不带+号，表示显示最后2行的内容
    `sed -n '2,20p' /etc/hosts`|显示文件`etc/hosts`2-20行的内容
    `ln -s 'src_file' 'dest_file'`| 在`dest_file`上建立文件`src_file`的软链接

2. `wget`

    - `wget --header="name:sogyf" http://domain.com/path`
    > 添加额外的HTTP Header来获取指定的站点或者链接
    - `wget  --user-agent="Mozilla/5.0 (Windows; U; Windows NT 5.1; en-US; rv:1.9.2.3) Gecko/20100401 Firefox/3.6.3 (.NET CLR 3.5.30729)"  http://domain.com/path`
    > 伪装浏览器下载
    - `wget --post-data="user=user1&pass=pass1&submit=Login" http://domain.com/login.php`
    > post数据到指定的链接上
    - ` 1. wget --post-data="user=user1&pass=pass1&submit=Login" --save-cookies=cookie.txt --keep-session-cookies http://domain.com/login.php; 2. wget --load-cookies=cookie.txt http://domain.com/path/page_need_login.php` 
    > post用户和密码保存cookie到文件中，然后访问页面时附带这个cookie
    - `wget -r -np -N http://domain.com/`
    > 整站下载，包括css以及js、image等文件
    - [更多示例](https://gist.github.com/lzyy/316719)
    
3. `curl`






## 二、进程相关

命令 | 作用 
------------ | -------------
`ps -aux \| sort -k4nr \| head -n 10` | 查使用内存最多的10个进程
`ps -aux \| sort -k3nr \| head -n 10` | 查使用CPU最多的10个进程 
`netstat -lnp \| grep 9000 `|  通过端口找到进程；**9000**为进程


## 三、操作系统
1. Centos

    命令  | 作用
    ------ | -----
    `rpm -q 'package name'` | 查找包是否安装过
    `rpm -e 'package name'` | 删除一个已经安装过的包
    `gem list \| cut -d" " -f1 \| xargs gem uninstall -aIx` |清理所有已经安装过的gem
    `gem list \| cut -d" " -f1 \| xargs gem uninstall -aIx` | 更新rubygems软件
    `gem cleanup` | 清理无用的gem

2. 查看操作系统信息 | 

    命令  | 作用
    -----  | ----
    `uname -a ` | 查看内核/操作系统/CPU信息 
    `head -n 1 /etc/issue` | 查看操作系统版本 
    `cat /proc/cpuinfo` | 查看CPU信息
    `hostname` | 查看计算机名 
    `lspci -tv `| 列出所有PCI设备
    `lsusb -tv `| 列出所有USB设备 
    `lsmod `| 列出加载的内核模块
    `env `| 查看环境变量 资源
    `free -m` | 查看内存使用量和交换区使用量
    `df -h `| 查看各分区使用情况 
    `du -sh <目录名>` | 查看指定目录的大小
    `grep MemTotal /proc/meminfo `| 查看内存总量
    `grep MemFree /proc/meminfo `| 查看空闲内存量 
    `uptime `| 查看系统运行时间、用户数、负载
    `cat /proc/loadavg` | 查看系统负载 磁盘和分区 
    `mount \| column -t `| 查看挂接的分区状态 
    `fdisk -l` | 查看所有分区 
    `swapon -s `| 查看所有交换分区 
    `hdparm -i /dev/hda` | 查看磁盘参数(仅适用于IDE设备)
    `dmesg \| grep IDE` | 查看启动时IDE设备检测状况

3. 查看操作系统网络信息

    命令  |   作用
    -----| ----
    `ifconfig` | 查看所有网络接口的属性 
    `iptables -L` | 查看防火墙设置 
    `route -n` | 查看路由表 
    `netstat -lntp` | 查看所有监听端口 
    `netstat -antp` | 查看所有已经建立的连接 
    `netstat -s` | 查看网络统计信息
    
4. 查看用户信息

    命令  |   作用
    -----| -----
    `w` | 查看活动用户
    `id <username>` | 查看指定用户信息
    `cut -d: -f1 /etc/passwd` # 查看系统所有用户
    `cut -d: -f1 /etc/group` # 查看系统所有组 
    `crontab -l` # 查看当前用户的计划任务 服务
    `chkconfig --list `# 列出所有系统服务
    `chkconfig --list \| grep on` # 列出所有启动的系统服务

## 编程相关

1. `git`

    - `git archive -o ../updated.zip HEAD $(git diff --name-only HEAD^)`
        > 导出最后一次提交修改过的文件
    - `git archive -o ../latest.zip NEW_COMMIT_ID_HERE $(git diff --name-only OLD_COMMIT_ID_HERE NEW_COMMIT_ID_HERE)`
        > 导出两次提交之间修改过的文件
    - `git name-rev --name-only COMMIT_HASH_HERE`
        > 检查提交的修改是否发布版本的一部分
    - `git update-index --assume-unchanged PATH_TO_FILE_HERE`
        > 忽略跟踪文件的修改
    - `git checkout BRANCH_NAME_HERE -- PATH_TO_FILE_IN_BRANCH_HERE`
        > 在不切换分支的情况下从其它分支检出文件
    - `git checkout --orphan NEW_BRANCH_NAME_HERE`
        > 启动一个无历史的新分支
    - `git --git-dir=PATH_TO_OTHER_REPOSITORY_HERE/.git format-patch -k -1 --stdout COMMIT_HASH_ID_HERE| git am -3 -k`
        > 从无关的本地仓库应用补丁

2
. 


