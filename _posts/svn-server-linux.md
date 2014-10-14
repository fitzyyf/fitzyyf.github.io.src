title: "Linux SVN服务器安装"
date: 2013-01-27 14:10
tags: svn
categories:  
- Linux
---

# SVN安装
> 说明：
>  
> 这个安装是基于Linux的基础上。
>
> 确保安装了C编译环境，如果没有，则执行如下命令
>  
>   `(sudo) yum (apt-get) install gcc`

## 源码安装
如果依赖都已经安装过的话，可以忽略这节介绍。

1. 下载编译依赖

    * apr
        
            wget http://apache.etoak.com/apr/apr-1.4.6.tar.gz
            
    * apr-util
    
            wget http://apache.etoak.com/apr/apr-util-1.5.1.tar.gz
            
    * sqllite
        
            wget http://www.sqlite.org/sqlite-autoconf-3071502.tar.gz
            
    * zlib
        
            wget http://zlib.net/zlib-1.2.7.tar.gz
            
    如果需要SVN的http查看，需要安装Apache
    
    * Apache
        
        这里使用包安装器来进行安装，如果是ubuntu等服务器，则使用apt也可以。
        
            {% codeblock Check Apache httpd Server. lang:sh %}
            # 检查使用安装httdp
            rpm -qa |grep httpd
            # 如果结果没有httpd-devel的话，还需要安装httpd-devel
            # yum install httpd-devel
            yum -y install httpd
            {% endcodeblock %}
        
2. 编译依赖

    * 编译安装Apr
        
            # 解压缩
            tar xfvz apr-1.4.6.tar.gz 
            cd apr-1.4.6
            # 编译到/ifkytek/svnapp/apr
            ./configure --prefix=/usr/local/apr
            make
            sudo make install
            
    * 编译安装apr-util
            
            # 解压缩
            tar xfvz apr-util-1.5.1.tar.gz
            cd apr-util-1.5.1
            # 编译到/iflytek/svnapp/apr-util
            ./configure     --prefix=/usr/local/apr-util --with-apr=/usr/local/apr
            make
            sudo make install
             
    * 编译 zlib
    
            # 解压缩
            tar xfvz zlib-1.2.7.tar.gz
            # 编译安装
            cd zlib-1.2.7
            # 如果是32位的话 ，不需要CFLAGS="-O3 -fPIC"
            CFLAGS="-O3 -fPIC" ./configure --prefix=/usr/local/zlib
            make
            sudo make install
            
    * 编译安装sqllite
    
            # 解压缩
            tar xfvz sqlite-autoconf-3071502.tar.gz
            cd sqlite-autoconf-3071502
            # 执行编译安装
            ./configure --prefix=/usr/local/sqlite
            make
            sudo make install    

2. 下载SVN源码编译安装

        {% codeblock Download svn source and install - svn_install.sh. lang:sh %}
        wget http://mirrors.ibiblio.org/apache/subversion/subversion-1.7.8.tar.gz
        # 创建svn安装目录
        mkdir /iflytek/svnapp
        # 解压缩
        tar xfvz subversion-1.7.8.tar.gz
        cd subversion-1.7.8
        # 编译安装 /usr/sbin/apxs,各个平台可能不同。
        ./configure -with-apxs=/usr/sbin/apxs --prefix=/iflytek/svnapp/subversion--with-apr=/usr/local/apr --with-apr-util=/usr/local/apr-util --with-sqlite=/usr/local/sqlite–with-zlib=/usr/local/zlib
        make
        sudo make install
        {% endcodeblock %}
        

## SVN建库
以上安装完成后，开始进行建库

    # 建立仓库根目录
    makdir /iflytek/repos
    # 建立仓库
    svnadmin create /iflytek/repos/test
    # 导入项目
    svn import /root/project file:///home/svnroot/repository/test -m "初始化导入"
    # 改变目录权限
    chown -R apache:apache（或svn:svn） /iflytek/repos

## SVN配置
配置Httpd访问即可

1. 修改Apache的配置文件 httpd.conf

        #SVN模块加载
        LoadModule dav_svn_module modules/mod_dav_svn.so
        LoadModule dav_module modules/mod_dav.so  //如果apache已加载， 要去掉本行。
        LoadModule authz_svn_module modules/mod_authz_svn.so#资源配置(如果多个项目， 共用一套验证， 下面配置就可以， 不然需要每个项目组分开建)
        <Location /svn>
           DAV svn
           # svn父目录
           SVNParentPath /iflytek/repos 
           # 启用目录级别授权， 权限配置文件
           AuthzSVNAccessFile /var/repos/config/authz.conf         # 连接类型设置
           AuthType Basic 
           # 连接框提示
           AuthName "Subversion  Repository" 
           # 用户配置文件
           AuthUserFile /iflytek/repos/config/authfile
           # 表示在同时启用了Allow（允许）和Require的情况下，指定相关策略的，一共有两个备选值，
           # All表示用户必须同时满足Allow和 Require的条件，而Any则是满足其中之一即可。
           # Satisfy Any用于允许先用匿名方式尝试访问，并根据svnauthz对匿名用户的控制给予访问权限。
           # 若没有这句话，则无论svnauthz中是否加入了 "*=r"的写法，匿名用户都是无法访问的。
           Satisfy Any 
           # 允许匿名访问，不允许Commit，不能与AuthzSVNAccessFile同时使用， 此选项没试过。
           #<LimitExcept GET PROPFIND OPTIONS REPORT> 
           # 采用何种认证
           Require valid-user 
        </Location>
        
2. 增加用户

        # 此目录指向httpd.conf中的"用户配置文件" -c 为创建， -m 为修改
        htpasswd -m /iflytek/repos/config/authfile usr1 
    
3. 增加用户权限
    
    修改`/iflytek/repos/config/authz.conf`,此目录指向httpd.conf中的"权限配置文件"
    
         vi(m) /iflytek/repos/config/authz.conf
         
         [test:/] #这表示，仓库test的根目录下的访问权限
         admin = rw #test仓库wooin用户具有读和写权限
         @developers = rw #@开头的表示组， 组必须存在， 不然死的会很惨。
         @test = r  
         [/] #这个表示在所有仓库的根目录下  
         * = r #这个表示对所有的用户都具有读权限  
         [groups] #这个表示群组设置  
         developers = user1, user2 #这个表示某群组里的成员
         test = user2  
    
    将这个设置完成后。重启Apache，就可以通过[http://localhost/svn/test](http://localhost/svn/test)
    
> 整个SVN安装过程完毕。
