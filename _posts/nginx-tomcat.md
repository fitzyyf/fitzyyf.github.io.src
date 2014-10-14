title: nginx、tomcat负载均衡
tags: 
- nginx
- tomcat
categories:  
- 开发笔记
---
## nginx 介绍配置
nginx是一个高性能的Http和反向代理服务，也是一个IMAP/POP3/SMTP代理服务器。是由*lgor sysoev*为俄罗斯访问第二的*Rambler.ru*站点开发的。
## nginx配置

1. 全局配置
		
		#Nginx所用用户和组，window下不指定 
		user www-data;
		#工作的子进程数量（通常等于CPU数量或者2倍于CPU） 
		worker_processes 4;
		#错误日志存放路径 
		#error_log  logs/error.log; 
		#error_log  logs/error.log  notice; 
		error_log  logs/error.log  info; 
		#指定pid存放文件 
		pid /var/run/nginx.pid;
		events {
			# 参考事件模型 使用网络IO模型linux建议epoll，
			#FreeBSD建议采用kqueue，window下不指定。 
			use epoll; 
			#工作进程的最大连接数量，根据硬件调整，和前面工作进程配合起来用，
			#尽量大，但是别把cpu跑到100%就行
			worker_connections 2048;
			# multi_accept on;
		}

		http {

			##
			# Basic Settings
			##
			#开启高效文件传输模式
			sendfile on;
			# 防止网络阻塞
			tcp_nopush on;
			#防止网络阻塞
			tcp_nodelay on;
			# 连接超时时间
			keepalive_timeout 65;
			types_hash_max_size 2048;
			# 服务器版本去除
			server_tokens off;

			# server_names_hash_bucket_size 64;
			# server_name_in_redirect off;
			# 文件扩展名与文件类型映射表
			include /etc/nginx/mime.types;
			# 默认文件类型
			default_type application/octet-stream;


			##
			# Logging Settings
			##
			# 访问日志
			access_log /var/log/nginx/access.log;
			# 错误日志
			error_log /var/log/nginx/error.log;

			##
			# Gzip Settings
			##

			include gzip.conf;

			#设定请求缓冲
		    client_header_buffer_size    1k;
		    large_client_header_buffers  4 4k;

			include upstreams.conf;
			include servers.conf; 
			##
			# nginx-naxsi config
			##
			# Uncomment it if you installed nginx-naxsi
			##

			#include /etc/nginx/naxsi_core.rules;

			##
			# nginx-passenger config
			##
			# Uncomment it if you installed nginx-passenger
			##
			
			#passenger_root /usr;
			#passenger_ruby /usr/bin/ruby;

			##
			# Virtual Host Configs
			##

			include /etc/nginx/conf.d/*.conf;
			include /etc/nginx/sites-enabled/*;
		}


		#mail {
		#	# See sample authentication script at:
		#	# http://wiki.nginx.org/ImapAuthenticateWithApachePhpScript
		# 
		#	# auth_http localhost/auth.php;
		#	# pop3_capabilities "TOP" "USER";
		#	# imap_capabilities "IMAP4rev1" "UIDPLUS";
		# 
		#	server {
		#		listen     localhost:110;
		#		protocol   pop3;
		#		proxy      on;
		#	}
		# 
		#	server {
		#		listen     localhost:143;
		#		protocol   imap;
		#		proxy      on;
		#	}
		#}
2. 集群服务配置，负载均衡
		
		# 集群节点配置
		upstream tomcat_server {
		    # 分配方式：	轮询		默认 每个请求按时间顺序逐一分配到不同的后端服务器
		    # 			weight	指定轮询几率，weight和访问比率成正比，用于后端服务器性能不均的情况。weight 默认为1.weight越大，负载的权重就越大。
		    # 			id_hash 每个请求按访问ip的hash结果分配，这样每个访客固定访问一个后端服务器
		    # 		   	fair 	按后端服务器的响应时间来分配请求，响应时间短的优先分配。
		    # 			url 	按访问url的hash结果来分配请求，使每个url定向到同一个后端服务器，后端服务器为缓存时比较有效。
		    fair;
		    server 192.168.1.102:8888 weight=1; #配置为你的集群节点,每个节点部署的都是Tomcat，当然也可以是其他的Web容器
		    server 192.168.1.103:8888 weight=2;
		    server 192.168.1.105:8080 weight=2;
		}
		
3. 本地代理配置及缓存设置

		# 本地缓存 js.css等资源文件，以及图片等
		location ~ \.(htm|html|gif|jpg|jpeg|png|ico|rar|css|js|zip|txt|flv|swf|doc|ppt|xls|pdf)$ {
		    expires 3d;
		    proxy_store on;
		    proxy_store_access user:rw group:rw all:rw;
		    proxy_temp_path /home/poplar/developer/www/temp;
		    root /home/poplar/developer/www/cems;
		    access_log off;
		    include proxy.conf;
		}
		# 集群转换jsp以及do等请求
		location ~ \.(jsp|jspx|do)$ {
		    # tomcat_server在upstreams.conf中配置
		    proxy_pass http://tomcat_server;  
		    include proxy.conf;
		}  
		location ~ ^/(WEB-INF)/{
		    deny all;
		}   
4. 服务配置

		# 虚拟主机配置
		server {
		    listen       80;  
		    server_name  localhost;    
			#设定首页 
		    index index.html index.htm index.jsp index.do;
			#设定网站的资源存放路径
		    root /home/poplar/developer/www;
		    include locations.conf;
		}
5. 其他的文件
	[代理配置](/postfile/nginx/proxy.conf)
	[gzip配置](/postfile/nginx/gzip.conf)

## tomcat Session
后续：tomcat集群Session采用 memcached-session-manager的方式来进行。
