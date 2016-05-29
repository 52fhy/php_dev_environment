# Server配置实用技巧

[TOC]

---

## rewrite规则写法
阅读原文：
<br/>
rewrite规则写法及nginx配置location总结 - 飞鸿影~ - 博客园
http://www.cnblogs.com/52fhy/p/5055233.html

## ThinkPHP框架里隐藏index.php
### Nginx
```
location / {
   try_files $uri $uri/ /index.php?s=$uri&$args;
}
```

### Apache
在根目录新建.htaccess文件：
```
<IfModule mod_rewrite.c>
  Options +FollowSymlinks
  RewriteEngine On

  RewriteCond %{REQUEST_FILENAME} !-d
  RewriteCond %{REQUEST_FILENAME} !-f
  RewriteRule ^(.*)$ index.php/$1 [QSA,PT,L]
</IfModule>
```

更多阅读原文：
<br/>
ThinkPHP框架里隐藏index.php - 飞鸿影~ - 博客园
http://www.cnblogs.com/52fhy/p/5380185.html

## ci框架里rewrite示例

### Nginx
```
app.52fhy.com.conf

server {
    listen       80;
    server_name  app.52fhy.com;
    index app.php;
    root /www/test/ci/;
    
    location ~ .*\.(php|php5)?$
    {
        #fastcgi_pass  unix:/tmp/php-cgi.sock;
        fastcgi_pass  127.0.0.1:9000;
        fastcgi_index app.php;
        include fastcgi.conf;
    }
    
    location ~ .*\.(gif|jpg|jpeg|png|bmp|swf)$
    {
        expires 30d;
    }
    
    location ~ .*\.(js|css)?$
    {
        expires 1h;
    }

    location / {
      if (!-e $request_filename) {
                rewrite ^/(.*)$ /app.php?/$1 last;
                break;
          }
    }
    
    access_log  /www/log/nginx/access/app.52fhy.com.log;
}
```

## Apache
```
<IfModule mod_rewrite.c>
  Options +FollowSymlinks
  RewriteEngine On

  RewriteCond %{REQUEST_FILENAME} !-d
  RewriteCond %{REQUEST_FILENAME} !-f
  RewriteRule ^(.*)$ /app.php/$1 [QSA,PT,L]
</IfModule>
```

更多阅读原文：
<br/>
ci框架里rewrite示例 - 飞鸿影~ - 博客园
http://www.cnblogs.com/52fhy/p/5061314.html

## Nginx下WordPress的Rewrite
### Nginx
```
location / {
	index index.html index.php;
	if (-f $request_filename) {
	          break;
	}
	if (!-e $request_filename) {
	          rewrite . /index.php  last;
	}
}
```

### Apache
.htaccess的内容如下：

```
# BEGIN WordPress
RewriteEngine On
RewriteBase /
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule . /index.php [L]
# END WordPress
```

更多阅读原文：
<br/>
Nginx下WordPress的Rewrite - 飞鸿影~ - 博客园
http://www.cnblogs.com/52fhy/p/5054507.html

## php-fpm进程数优化
```
pm = dynamic #如何控制子进程，选项有static和dynamic
pm.start_servers = 4 #动态方式下的起始php-fpm进程数量
pm.min_spare_servers = 2 #动态方式下的最小php-fpm进程数
pm.max_spare_servers = 8 #动态方式下的最大php-fpm进程数量
```
`pm.start_servers`缺省值计算公式: `min_spare_servers + (max_spare_servers - min_spare_servers) / 2`。

比如说512M的VPS，加入分配给php-fpm最大250M，建议`pm.max_spare_servers`设置为250/30 ,约为8。至于`pm.min_spare_servers`，则建议根据服务器的负载情况来设置，比如服务器上只是部署php环境的话，比较合适的值在2~5之间。

更多阅读原文：
<br/>
php-fpm进程数优化 - 飞鸿影~ - 博客园
http://www.cnblogs.com/52fhy/p/5051722.html
