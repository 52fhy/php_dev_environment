# 理解nginx的配置

---

Nginx配置文件主要分成四部分：`main（全局设置）`、`server（主机设置）`、`upstream（上游服务器设置，主要为反向代理、负载均衡相关配置）`和 `location（URL匹配特定位置后的设置）`，每部分包含若干个指令。

`main部分`设置的指令将影响其它所有部分的设置；`server部分`的指令主要用于指定虚拟主机域名、IP和端口；`upstream`的指令用于设置一系列的后端服务器，设置反向代理及后端服务器的负载均衡；`location部分`用于匹配网页位置（比如，根目录"/","/images",等等）。

他们之间的关系式：server继承main，location继承server；upstream既不会继承指令也不会被继承。它有自己的特殊指令，不需要在其他地方的应用。

可以有多个server，location属于server子集。

一个完整的`nginx.conf`：
```

user  www www;
worker_processes  1; # worker角色的工作进程个数，简单可设置CPU核数

error_log  /www/log/nginx/error.log crit;
pid        /www/server/nginx/logs/nginx.pid;

#Specifies the value for maximum file descriptors that can be opened by this process. 
worker_rlimit_nofile 65535;

events 
{
  use epoll;
  worker_connections 65535;
}

# http设置
http {
	include       mime.types;
	default_type  application/octet-stream;

	autoindex on;  #自动显示目录
    autoindex_exact_size off; #人性化方式显示文件大小否则以byte显示
    autoindex_localtime on; #按服务器时间显示，否则以gmt时间显示

    # log格式设置
    # log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

       # 解决虚拟主机名字过长 http://www.jb51.net/article/26412.htm
	server_names_hash_bucket_size 128;

	client_header_buffer_size 32k;
	large_client_header_buffers 4 32k;

	sendfile on; # 开启高效文件传输模式；下载服务器设置为off
	
	tcp_nopush on; # 默认值即可
	tcp_nodelay on; # 默认值即可

	keepalive_timeout 60; # 长连接超时时间，单位是秒
	send_timeout 20; # 指定响应客户端的超时时间
	
	client_max_body_size 10m;# 允许客户端请求的最大单文件字节数
    client_body_buffer_size 128k; # 缓冲区代理缓冲用户端请求的最大字节数

    # FastCGI设置
	fastcgi_connect_timeout 300;
	fastcgi_send_timeout 300;
	fastcgi_read_timeout 300;
	fastcgi_buffer_size 64k;
	fastcgi_buffers 4 64k;
	fastcgi_busy_buffers_size 128k;
	fastcgi_temp_file_write_size 128k;

    # gzip压缩功能设置
	gzip on; # 开启gzip压缩输出，减少网络传输
	gzip_min_length  1k;
	gzip_buffers     4 16k;
	gzip_http_version 1.0;
	gzip_comp_level 2;
	gzip_types       text/plain application/x-javascript text/css application/xml;
	gzip_vary on;
	
	# Nginx带宽限制
	#limit_zone  crawler  $binary_remote_addr  10m;

	# http_proxy设置，按需设置
    proxy_connect_timeout   75; # 代理连接超时
    proxy_send_timeout   75; # 代理发送超时
    proxy_read_timeout   75; # 代理接收超时
    proxy_buffer_size   4k;
    proxy_buffers   4 32k;
    proxy_busy_buffers_size   64k;
    proxy_temp_file_write_size  64k;
    proxy_temp_path   /www/server/nginx/proxy_temp 1 2;
    
    #利用proxy_cache来缓存文件
    #levels设置目录层次 
    #keys_zone设置缓存名字和共享内存大小 
    #inactive在指定时间内没人访问则被删除在这里是1天 
    #max_size最大缓存空间
    proxy_cache_path /www/server/nginx/proxy_cache levels=1:2 keys_zone=content:20m inactive=1d max_size=100m;  
    #等号中间要加的，关键只要加上proxy_cache_path  
    
    # 设定负载均衡后台服务器列表，按需设置，一般不需要
    upstream  backend  { 
      #ip_hash; 
      server   192.168.10.100:8080 max_fails=2 fail_timeout=30s ;  
      server   192.168.10.101:8080 max_fails=2 fail_timeout=30s ;  
    }
	
	# 虚拟主机配置，server 指令开始
	include /www/server/nginx/conf/vhosts/*.conf;
}

```

## 完整的vhost的配置文件：
`/www/server/nginx/conf/vhosts/me.52fhy.com.conf`
```
# 虚拟主机配置
server {
    listen       80;
    server_name  me.52fhy.com;
    index index.php index.html index.htm;
    root /52fhy.com/wordpress/;
    
    # WordPress Rewrite
    location / {
        if (-f $request_filename/index.html){
            rewrite (.*) $1/index.html break;
        }
        if (-f $request_filename/index.php){
            rewrite (.*) $1/index.php;
        }
        if (!-f $request_filename){
            rewrite (.*) /index.php;
        }
    }
    
    # 解析php|php5后缀
    location ~ .*\.(php|php5)?$
    {
        #fastcgi_pass  unix:/tmp/php-cgi.sock;
        fastcgi_pass  127.0.0.1:9000;
        fastcgi_index index.php;
        include fastcgi.conf;
    }
    
    # 设置gif|jpg|jpeg|png|bmp|swf文件缓存时间
    location ~ .*\.(gif|jpg|jpeg|png|bmp|swf)$
    {
        expires 30d; # 缓存30天
    }
    
    # 设置js|css文件缓存时间
    location ~ .*\.(js|css)?$
    {
        expires 1h; # 缓存1小时
    }

    # 访问日志
    access_log  /www/log/me.52fhy.com.log;
}
```

## 负载均衡示例
```
upstream site {
server 192.168.3.82:8040;
server 192.168.3.82:8041;
}
 
server {
listen 8080;
server_name 192.168.3.82;
 
#charset koi8-r;
 
#access_log logs/host.access.log main;
 
location / {
root html;
index index.html index.htm;
proxy_pass http://site;
 
}
```
访问192.168.3.82：8080的时候，会来回访问192.168.3.82:8040和192.168.3.82:8041。

>参考：
<br/>
1、Nginx 服务器安装及配置文件详解
http://www.runoob.com/w3cnote/nginx-install-and-config.html#rd
<br/>
2、nginx配置location总结及rewrite规则写法 | Sean's Notes
http://seanlook.com/2015/05/17/nginx-location-rewrite/
<br/>
3、Nginx 图片 js文件缓存配置方法-nginx-操作系统-壹聚教程网
http://www.111cn.net/sys/nginx/64074.htm
<br/>
4、nginx利用proxy_cache来缓存文件
http://blog.51yip.com/apachenginx/1018.html
<br/>
5、Nginx负载均衡 - 小刚qq - 博客园
http://www.cnblogs.com/xiaogangqq123/archive/2011/03/04/1971002.html
<br/>

本文发表在博客园，原文请看：
http://www.cnblogs.com/52fhy/p/5054516.html

