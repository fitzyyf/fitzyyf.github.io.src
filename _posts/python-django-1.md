title: "python Django起步"
date: 2013-02-26 09:34
categories:  
- Web开发
---

1. 配置
---
1. 通过`settings.py`来达到配置的作用，包括数据库等
2. 通过`urls.py`来设置访问路径，同时`rest`方式也在这里设置
3. 可以通过设置环境变量`DJANGO_SETTINGS_MODULE`来使用某个设置文件
4. 可以通过`python manage.py runserver`的方式启动服务器

2. 起步
---
1. `django-admin.py`有如下的相关命令：
    * `startproject` 创建一个新的工程
    * `startapp` 创建一个应用
    * `project`提供配置信息，比如`settings.py`以及`urls.py`等；而`app`是一套Django功能的集合，包含了模型和视图。另外，app的一个约定，如果使用了模型，那么必须创建一个`app`。
2. `django-admin.py startproject studioapp` 创建一个新的名称为`studioapp`的工程，结构如下：
	    
		.
		├── __init__.py  	# 标识这个工程可以作为Python的一个模块使用
		├── manage.py 		# Django 的启动文件
		└── studioapp		# Django 的 配置信息包
		    ├── __init__.py 	# 标识这个文件夹也可以作为Python的一个模块使用	
		    ├── settings.py 	# Django 的配置信息，包含数据库和已安装的app
		    ├── urls.py 		# urls的配置，可以说是路由配置
		    └── wsgi.py

3. `django-admin.py startapp regis` 创建一个新的app，名称为`regis`,结果如下
		
		.
		├── __init__.py
		├── manage.py
		├── regis					# app 
		│   ├── __init__.py 		# app 的模块使用标记
		│   ├── models.py 			# 模型，数据库模型，表示和后台数据库相关的各种操作
		│   ├── serializers.py 		# 序列化
		│   ├── tests.py 			# 测试用例
		│   └── views.py 			# 视图，数据库的结果以不同形式表现出来
		└── studioapp
		    ├── __init__.py
		    ├── settings.py
		    ├── urls.py
		    └── wsgi.py

3. 部署
---

1. 关闭Debug模式，在`setttings.py`中

		DEBUG = False
	
	如果它为True的话，表示以下含义
	
	1. 所有的数据库查询都一直存在于内存中；
	2. 任何的404错误都将出现Django的Debug的特殊界面;
	3. 应用中任何未捕捉的异常，从基本的语法错误到数据库错误等都会返回Django的错误界面。

2. 实现一个404和500的错误页面
3. 设置错误警告

	Django在你的 代码引发未处理的异常时,将会发送一封Email至开发者团队。
	
	设置`settings.py`中的 `ADMINS`设置即可。

4. 可以为不同的环境配置不同的配置文件

### nginx + uWSGI 部署 Django应用

1. 安装 uwsgi
   
	源码安装直接：
	
		wget http://projects.unbit.it/downloads/uwsgi-1.4.6.tar.gz
		tar zxvf uwsgi-1.4.6.tar.gz
		cd uwsgi-1.4.6
		make -f Makefile.Py27
		cp uwsgi /usr/sbin/uwsgi

	mac os x 安装
		
		brew install --universal pcre
		brew install uwsgi

2. Nginx 整合

	1. 配置Django app
		
		在app目录下找到wsgi.py，一般通过`django-admin.py`创建的app都会有这个文件，如果，没有可以手动创建一个
	
			import os

			os.environ.setdefault("DJANGO_SETTINGS_MODULE", "studioapp.settings")
			from django.core.wsgi import get_wsgi_application
			application = get_wsgi_application()		

		然后在配置app目录下新建`django.xml`,名字随意,内容如下

			<uwsgi>
			    <socket>127.0.0.1:8630</socket>
			    <chdir>/Users/yfyang/projs/work/python-rest-sample/Server</chdir>
			    <pythonpath>..</pythonpath>
				<!-- 与 wsgi.py 名字一致 -->
			    <module>wsgi</module>
			</uwsgi>	

	2. 配置nginx
	
		找到nginx的配置文件 nginx.conf,增加如下内容，注意 uwsgi_pass 与上述的socket的配置一致。
			
			location /django {
	            uwsgi_pass 127.0.0.1:8630;
	            include uwsgi_params;
	        }
	
	4. 测试

		启动 uwsgi
			
			uwsgi -x /Users/yfyang/projs/work/python-rest-sample/Server/django.xml
	
		启动 nginx
			
			sudo nginx
		
		访问 [http://127.0.0.1/django](http://127.0.0.1/django).

