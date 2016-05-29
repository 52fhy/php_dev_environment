# windows下配置nginx+php环境


[TOC]

---

## PHP安装

约定：
环境安装目录：
```
D:/phpsetup/
   |——php
   		|——php-5.6.22-nts-Win32-VC11-x86
   |——nginx
   |——mysql
   |——www
```

### 下载
下载地址：
http://php.net/

windows版下载地址：
http://windows.php.net/download

官网里Windows的版本有很多。选择哪个版本呢？

如果你的PHP应用程序以FastCGI方式运行，请选择Non-Thread Safe (NTS) 版本；
如果你的PHP应用程序和Apache一起，请选择 Thread Safe (TS) 版本。

**本文用的nginx+php组合，所以选择nts(Non Thread Safe)版本。根据操作系统选择x64或者x86。**

大多数版本的PHP使用VC9, VC11 or VC14 (Visual Studio 2008, 2012 or 2015分别编译)进行编译的，所以你电脑上需要安装VC运行环境。

电脑需要VC运行环境：
VC9 x86 ：http://www.microsoft.com/en-us/download/details.aspx?id=5582
VC9 x64 ：http://www.microsoft.com/en-us/download/details.aspx?id=15336
VC11 x86 or x64：http://www.microsoft.com/en-us/download/details.aspx?id=30679
VC14 x86 or x64 ：http://www.microsoft.com/en-us/download/details.aspx?id=48145

### 配置PHP
本文以[php-5.6.22-nts-Win32-VC11-x86.zip](http://windows.php.net/downloads/releases/php-5.6.22-nts-Win32-VC11-x86.zip)为例。
下载后解压到`D:\phpsetup\php\php-5.6.22-nts-Win32-VC11-x86`目录。

复制一份`php.ini-development`文件为`php.ini`。

需要修改以下地方：

- 更改自定义扩展目录。
找到
```
;extension_dir = "ext"
```
更改为
```
extension_dir = "ext"
```
提示：与nginx搭配使用不需要写绝对位置。

- 让php能够与nginx结合。
找到
```
;cgi.fix_pathinfo=1
```
我们去掉这里的分号:
```
cgi.fix_pathinfo=1
```
**这一步非常重要，这里是php的CGI的设置。**


- 开启扩展
往下看，再找到：
```
;extension=php_curl.dll
```

去掉部分注释：
```
extension=php_bz2.dll
extension=php_curl.dll
extension=php_fileinfo.dll
extension=php_gd2.dll
extension=php_gettext.dll
;extension=php_gmp.dll
;extension=php_intl.dll
;extension=php_imap.dll
;extension=php_interbase.dll
;extension=php_ldap.dll
extension=php_mbstring.dll
extension=php_exif.dll      ; Must be after mbstring as it depends on it
extension=php_mysql.dll
extension=php_mysqli.dll
;extension=php_oci8_12c.dll  ; Use with Oracle Database 12c Instant Client
extension=php_openssl.dll
;extension=php_pdo_firebird.dll
extension=php_pdo_mysql.dll
;extension=php_pdo_oci.dll
extension=php_pdo_odbc.dll
extension=php_pdo_pgsql.dll
extension=php_pdo_sqlite.dll
extension=php_pgsql.dll
;extension=php_shmop.dll

; The MIBS data available in the PHP distribution must be installed. 
; See http://www.php.net/manual/en/snmp.installation.php 
;extension=php_snmp.dll

extension=php_soap.dll
extension=php_sockets.dll
extension=php_sqlite3.dll
;extension=php_sybase_ct.dll
extension=php_tidy.dll
extension=php_xmlrpc.dll
extension=php_xsl.dll
```

- 设置默认时区
```
date.timezone=PRC
```

- 开启自定义扩展
```
[memcache]
extension=php_memcache.dll

[redis]
extension=php_redis.dll
```

注意，需要下载对应版本的扩展：
如`php_redis-2.2.7-5.6-nts-vc11-x86`
区分ts,x86。

下载地址
http://pecl.php.net/package/redis/2.2.7/windows
http://pecl.php.net/package/memcache


## nginx的安装与配置
### 下载nginx
http://nginx.org/en/download.html
http://nginx.org/download/nginx-1.10.0.zip

### 安装配置
把下载好的nginx-1.10.0的包同样解压到`D:/phpsetup/nginx-1.10.0`目录下，并重命名为nginx。

接下来，我们来配置nginx，让它能够和php协同工作。

进入nginx的conf目录，打开nginx的配置文件nginx.conf，找到：
```
location / { 
	root html;　#这里是站点的根目录 
	index index.html index.htm;
}
```

修改www目录及index：
```
location / { 
	root D:/phpsetup/www;　　　　#这里是站点的根目录 
	index index.php index.html index.htm;

	# 解决虚拟主机名字过长 http://www.jb51.net/article/26412.htm
	server_names_hash_bucket_size 128; 

	# 显示目录
	autoindex on;
	autoindex_exact_size on;# 显示文件大小
	autoindex_localtime on;# 显示文件时间

	include vhosts/*.conf;
}
```
这里需要注意，路径分隔符请使用`/`而不要使用Windows中的`\`以防歧义。

将下面的server部分全部用#注释掉。我们新建子目录vhosts，把server部分写到这里面，方便管理。
新建localhost.conf：
内容：

```
server {
    listen       80;
    server_name  localhost;
    index index.php index.html index.htm;
    root D:/phpsetup/www/;

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
    access_log  logs/localhost.log;
}
```

保存配置文件，就可以了。

### 运行nginx+php
nginx+php的环境就初步配置好了，来跑跑看。我们可以输入命令：
```
php-cgi.exe -b 127.0.0.1:9000 -c D:/phpsetup/php/php-5.6.22-nts-Win32-VC11-x86/php.ini
```

来启动php，并手动启动nginx(可能不可行)。

下面利用脚本来实现。
首先下载`RunHiddenConsole.zip`包并解压到nginx目录内。
`RunHiddenConsole.exe`的作用是在执行完命令行脚本后可以自动关闭脚本，而从脚本中开启的进程不被关闭。

然后来创建脚本，命名为`start_nginx.bat`，我们在Notepad++里来编辑它:
```
@echo off
REM Windows 下无效
REM set PHP_FCGI_CHILDREN=5

REM 每个进程处理的最大请求数，或设置为 Windows 环境变量
set PHP_FCGI_MAX_REQUESTS=1000

REM NGINX只能使用非线程安全(nts)的PHP版本
REM 使用绝对位置
REM set PHP_PATH= D:/phpsetup/php/php-5.6.22-nts-Win32-VC11-x86
REM set NGINX_PATH=D:/phpsetup/nginx

REM 使用相对位置也是可以的
set PHP_PATH= ../php/php-5.6.22-nts-Win32-VC11-x86
set NGINX_PATH=./
 
echo Starting PHP FastCGI...
RunHiddenConsole %PHP_PATH%/php-cgi.exe -b 127.0.0.1:9000 -c %PHP_PATH%/php.ini
 
echo Starting nginx...
RunHiddenConsole %NGINX_PATH%/nginx.exe -p %NGINX_PATH%
```

再另外创建一个名为`stop_nginx.bat`的脚本用来关闭nginx:
```
@echo off
echo Stopping nginx...  
taskkill /F /IM nginx.exe > nul
echo Stopping PHP FastCGI...
taskkill /F /IM php-cgi.exe > nul
exit

```

这样，我们的服务脚本也都创建完毕了。

双击`start_nginx.bat`看看进程管理器是不是有两个`nginx.exe`的进程和一个`php-cgi.exe`的进程呢？

这样nginx服务就启动了，而且php也以fastCGI的方式运行了。