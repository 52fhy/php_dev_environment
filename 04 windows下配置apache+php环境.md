# windows下配置apache+php环境


[TOC]

---

## PHP安装

由于windows下php扩展5.6的多余7.0，故以php5.6为开发环境。如果对扩展要求不高，可以使用php7，安装过程类似。  

约定：
环境安装目录：
```
D:/phpsetup/
   |——php
   		|——php-5.6.22-Win32-VC11-x86
   |——apache
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

**本文用的apache+php组合，所以选择ts(Thread Safe)版本。根据操作系统选择x64或者x86。**

大多数版本的PHP使用VC9, VC11 or VC14 (Visual Studio 2008, 2012 or 2015分别编译)进行编译的，所以你电脑上需要安装VC运行环境。

电脑需要VC运行环境：
<br/>
VC9 x86 ：http://www.microsoft.com/en-us/download/details.aspx?id=5582
<br/>
VC9 x64 ：http://www.microsoft.com/en-us/download/details.aspx?id=15336
<br/>
VC11 x86 or x64：http://www.microsoft.com/en-us/download/details.aspx?id=30679
<br/>
VC14 x86 or x64 ：http://www.microsoft.com/en-us/download/details.aspx?id=48145

### 配置PHP
本文以[php-5.6.22-Win32-VC11-x86.zip](http://windows.php.net/downloads/releases/php-5.6.22-Win32-VC11-x86.zip)为例。
下载后解压到`D:\phpsetup\php\php-5.6.22-Win32-VC11-x86`目录。

复制一份`php.ini-development`文件为`php.ini`。

需要修改以下地方：

- 更改自定义扩展目录。
找到
```
;extension_dir = "ext"
```
更改为
```
extension_dir = "D:\phpsetup\php\php-5.6.22-Win32-VC11-x86\ext"
```
提示：与apache搭配使用需要写绝对位置。否则扩展加载不了。

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

- 设置ssl(可选)
```
openssl.cafile= cacert.pem
```
注意是绝对路径。

- 添加自定义扩展
```
[memcache]
extension=php_memcache.dll

[redis]
extension=php_redis.dll
```

注意，需要下载对应版本的扩展：
如`php_redis-2.2.7-5.6-ts-vc11-x86`
区分ts,x86。

下载地址
http://pecl.php.net/package/redis/  
http://pecl.php.net/package/memcache  
http://pecl.php.net/package/SeasLog  
http://pecl.php.net/package/xdebug  
http://pecl.php.net/package/yar  


## apache的安装与配置
### 下载apache

PHP官网里说明了，apache.org只提供旧的VC6版本，且不能使用 VC9+版本。apache.org已经不提供适合php的版本了。[apache windows版本](http://httpd.apache.org/docs/current/platform/windows.html#down)下载地址也给了下载建议：
- [ApacheHaus](http://www.apachehaus.com/cgi-bin/download.plx)
- [Apache Lounge](http://www.apachelounge.com/download/)
- [BitNami WAMP Stack](http://bitnami.com/stack/wamp)
- [WampServer](http://www.wampserver.com/)
- [XAMPP](http://www.apachefriends.org/en/xampp.html)

PHP官网也建议到[Apache Lounge](http://www.apachelounge.com/download/)下载。Apache Lounge提供了建立在x86和x64系统的VC9，vc14 VC11版本。PHP官方使用了他们提供的二进制文件构建Apache SAPIs。

Apache Lounge提供的下载地址：
http://www.apachelounge.com/download/VC11/

http://www.apachelounge.com/download/VC11/binaries/httpd-2.4.20-win64-VC11.zip
http://www.apachelounge.com/download/VC11/binaries/httpd-2.4.20-win32-VC11.zip

这里选择了httpd-2.4.20-win32-VC11.zip。

### 安装配置
把下载好的httpd-2.4.20-win32-VC11.zip的包同样解压到`D:/phpsetup/Apache24`目录下。

接下来，我们来配置apache，让它能够和php协同工作。

进入apache的conf目录，打开apache的配置文件httpd.conf。

修改apache软件所在目录：
```
ServerRoot "D:/phpsetup/Apache24"
```

修改主机名：
```
ServerName localhost:80
```

修改www目录：
```
DocumentRoot "D:/phpsetup/www"

<Directory "D:/phpsetup/www">
```

修改默认索引以支持PHP:
```
DirectoryIndex index.php index.html index.htm 
```

开启rewrite功能：
```
LoadModule rewrite_module modules/mod_rewrite.so
```

自定义404页面(可选)：
```
ErrorDocument 404 /missing.html
```

加载PHP模块,注意绝对路径：
```
#php5.6
LoadModule php5_module D:/phpsetup/php/php-5.6.22-Win32-VC11-x86/php5apache2_4.dll 
<IfModule php5_module> 
    PHPIniDir "D:/phpsetup/php/php-5.6.22-Win32-VC11-x86/" 
    AddType application/x-httpd-php .php
    AddType application/x-httpd-php-source .phps
</IfModule>
```

如果是php7，相应更改即可:
```
#php7
LoadModule php7_module D:/phpsetup/php/php-7.0.13-Win32-VC14-x64/php7apache2_4.dll
<IfModule php7_module> 
    PHPIniDir "D:/phpsetup/php/php-7.0.13-Win32-VC14-x64/" 
    AddType application/x-httpd-php .php
    AddType application/x-httpd-php-source .phps
</IfModule>
```

注意：如果是PHP5.4版本，php目录里只有`php5apache2_2.dll`，需要和Apache2.2搭配。
所以，安装php5.6一定要确认PHP安装包里是否有`php5apache2_4.dll`文件。

可以开启虚拟主机配置文件：
```
Include conf/extra/httpd-vhosts.conf
```
默认httpd-vhosts.conf文件里面写的是供参考的，一但启用该文件，请正确配置，否则无法启用apache服务。

虚拟主机示例：
``` ini
<VirtualHost *:80>
    DocumentRoot "D:/www/app/laravel-5-blog/public/"
    ServerName laravel-5-blog.fhy.com
	DirectoryIndex index.php
	<Directory "D:/www/app/laravel-5-blog/">
		AllowOverride All
	</Directory>
## PHP安装

由于windows下php扩展5.6的多余7.0，故以php5.6为开发环境。如果对扩展要求不高，可以使用php7，安装过程类似。  

约定：
环境安装目录：
```
D:/phpsetup/
   |——php
   		|——php-5.6.22-Win32-VC11-x86
   |——apache
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

**本文用的apache+php组合，所以选择ts(Thread Safe)版本。根据操作系统选择x64或者x86。**

大多数版本的PHP使用VC9, VC11 or VC14 (Visual Studio 2008, 2012 or 2015分别编译)进行编译的，所以你电脑上需要安装VC运行环境。

电脑需要VC运行环境：
<br/>
VC9 x86 ：http://www.microsoft.com/en-us/download/details.aspx?id=5582
<br/>
VC9 x64 ：http://www.microsoft.com/en-us/download/details.aspx?id=15336
<br/>
VC11 x86 or x64：http://www.microsoft.com/en-us/download/details.aspx?id=30679
<br/>
VC14 x86 or x64 ：http://www.microsoft.com/en-us/download/details.aspx?id=48145

### 配置PHP
本文以[php-5.6.22-Win32-VC11-x86.zip](http://windows.php.net/downloads/releases/php-5.6.22-Win32-VC11-x86.zip)为例。
下载后解压到`D:\phpsetup\php\php-5.6.22-Win32-VC11-x86`目录。

复制一份`php.ini-development`文件为`php.ini`。

需要修改以下地方：

- 更改自定义扩展目录。
找到
```
;extension_dir = "ext"
```
更改为
```
extension_dir = "D:\phpsetup\php\php-5.6.22-Win32-VC11-x86\ext"
```
提示：与apache搭配使用需要写绝对位置。否则扩展加载不了。

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

- 设置ssl(可选)
```
openssl.cafile= cacert.pem
```
注意是绝对路径。

- 添加自定义扩展
```
[memcache]
extension=php_memcache.dll

[redis]
extension=php_redis.dll
```

注意，需要下载对应版本的扩展：
如`php_redis-2.2.7-5.6-ts-vc11-x86`
区分ts,x86。

下载地址
http://pecl.php.net/package/redis/  
http://pecl.php.net/package/memcache  
http://pecl.php.net/package/SeasLog  
http://pecl.php.net/package/xdebug  
http://pecl.php.net/package/yar  


## apache的安装与配置
### 下载apache

PHP官网里说明了，apache.org只提供旧的VC6版本，且不能使用 VC9+版本。apache.org已经不提供适合php的版本了。[apache windows版本](http://httpd.apache.org/docs/current/platform/windows.html#down)下载地址也给了下载建议：
- [ApacheHaus](http://www.apachehaus.com/cgi-bin/download.plx)
- [Apache Lounge](http://www.apachelounge.com/download/)
- [BitNami WAMP Stack](http://bitnami.com/stack/wamp)
- [WampServer](http://www.wampserver.com/)
- [XAMPP](http://www.apachefriends.org/en/xampp.html)

PHP官网也建议到[Apache Lounge](http://www.apachelounge.com/download/)下载。Apache Lounge提供了建立在x86和x64系统的VC9，vc14 VC11版本。PHP官方使用了他们提供的二进制文件构建Apache SAPIs。

Apache Lounge提供的下载地址：
http://www.apachelounge.com/download/VC11/

http://www.apachelounge.com/download/VC11/binaries/httpd-2.4.20-win64-VC11.zip
http://www.apachelounge.com/download/VC11/binaries/httpd-2.4.20-win32-VC11.zip

这里选择了httpd-2.4.20-win32-VC11.zip。

### 安装配置
把下载好的httpd-2.4.20-win32-VC11.zip的包同样解压到`D:/phpsetup/Apache24`目录下。

接下来，我们来配置apache，让它能够和php协同工作。

进入apache的conf目录，打开apache的配置文件httpd.conf。

修改apache软件所在目录：
```
ServerRoot "D:/phpsetup/Apache24"
```

修改主机名：
```
ServerName localhost:80
```

修改www目录：
```
DocumentRoot "D:/phpsetup/www"

<Directory "D:/phpsetup/www">
```

修改默认索引以支持PHP:
```
DirectoryIndex index.php index.html index.htm 
```

开启rewrite功能：
```
LoadModule rewrite_module modules/mod_rewrite.so
```

自定义404页面(可选)：
```
ErrorDocument 404 /missing.html
```

加载PHP模块,注意绝对路径：
```
#php5.6
LoadModule php5_module D:/phpsetup/php/php-5.6.22-Win32-VC11-x86/php5apache2_4.dll 
<IfModule php5_module> 
    PHPIniDir "D:/phpsetup/php/php-5.6.22-Win32-VC11-x86/" 
    AddType application/x-httpd-php .php
    AddType application/x-httpd-php-source .phps
</IfModule>
```

如果是php7，相应更改即可:
```
#php7
LoadModule php7_module D:/phpsetup/php/php-7.0.13-Win32-VC14-x64/php7apache2_4.dll
<IfModule php7_module> 
    PHPIniDir "D:/phpsetup/php/php-7.0.13-Win32-VC14-x64/" 
    AddType application/x-httpd-php .php
    AddType application/x-httpd-php-source .phps
</IfModule>
```

注意：如果是PHP5.4版本，php目录里只有`php5apache2_2.dll`，需要和Apache2.2搭配。
所以，安装php5.6一定要确认PHP安装包里是否有`php5apache2_4.dll`文件。

可以开启虚拟主机配置文件：
```
Include conf/extra/httpd-vhosts.conf
```
默认httpd-vhosts.conf文件里面写的是供参考的，一但启用该文件，请正确配置，否则无法启用apache服务。

虚拟主机示例：
``` ini
<VirtualHost *:80>
    DocumentRoot "D:/www/app/laravel-5-blog/public/"
    ServerName laravel-5-blog.fhy.com
	DirectoryIndex index.php
	<Directory "D:/www/app/laravel-5-blog/">
		AllowOverride All
	</Directory>
    ErrorLog "logs/laravel-5-blog.fhy.com-error.log"
    CustomLog "logs/laravel-5-blog.fhy.com-access.log" common
</VirtualHost>
```
其中DocumentRoot设置项目所在路径，ServerName设置主机名，DirectoryIndex设置入口文件；Directory里AllowOverride设置开启.htaccess功能。

可以开启主机别名配置文件：
```
Include conf/extra/httpd-alias.conf
```

如果安装的PHP x64位版本，Apache也需要是x64位版本的。然后还要将php目录下的`libeay32.dll`、`ssleay32.dll`、`libssh2.dll`以及ext目录下的`php_curl.dll`等四个文件，都复制放到System32目录下。否则curl扩展无法使用。(http://my.oschina.net/lsfop/blog/496181)

### 运行apache+php
运行方式一：
手动运行bin目录下的ApacheMonitor.exe

运行方式二：
将apache安装为系统服务，可以开机自动启动。

以管理员权限运行cmd。

进入apache24的bin目录，安装Apache 服务：
```
httpd -k install
```

停止Apache 
```
httpd -k stop
```

重启Apache
``` 
httpd -k restart
```

卸载Apache服务
```
httpd -k uninstall
```

测试Apache配置文件httd.conf
```
httpd -t
```

查看Apache版本
```
httpd -V
```

Apache命令行帮助
```
httpd -h
```

删除服务：
```
sc delete Apache2.4
```

## 测试Apache和PHP
成功启动Apache后，在www目录编写phpinfo.php：
```
<?php
echo phpinfo();
```
浏览器地址栏输入localhost/phpinfo.php，显示PHP相关信息即表明成功了。

## 版本选择总结

Linux下安装推荐编译安装，不用考虑TS、NTS区别。版本建议64位（看机器是否支持）。  
PHP7: http://php.net/get/php-7.0.13.tar.bz2/from/a/mirror  
php5: http://php.net/get/php-5.6.28.tar.bz2/from/a/mirror  
Nginx: http://nginx.org/download/nginx-1.10.2.tar.gz  


Windows下安装时注意：  

需要先安装VC11或VC14:  
	1)VC11: https://www.microsoft.com/en-us/download/details.aspx?id=30679  
	2)VC14: https://www.microsoft.com/en-us/download/details.aspx?id=48145  
	
1、如果使用Apache，请使用TS版本PHP：  
PHP7:   
	1)VC14_x86: http://windows.php.net/downloads/releases/php-7.0.13-Win32-VC14-x86.zip  
	2)VC14_x64：http://windows.php.net/downloads/releases/php-7.0.13-Win32-VC14-x86.zip  
	
PHP5:  
	1)VC11_x86: http://windows.php.net/downloads/releases/php-5.6.28-Win32-VC11-x86.zip  
	1)VC11_x64: http://windows.php.net/downloads/releases/php-5.6.28-Win32-VC11-x64.zip  

Apache:  
	1、VC14_x64: https://www.apachelounge.com/download/VC14/binaries/httpd-2.4.23-win64-VC14.zip  
	2、VC14_x86: https://www.apachelounge.com/download/VC14/binaries/httpd-2.4.23-win32-VC14.zip  
	3、VC11_x64:https://www.apachelounge.com/download/VC11/binaries/httpd-2.4.23-win64-VC11.zip  
	4、VC11_x86:https://www.apachelounge.com/download/VC11/binaries/httpd-2.4.23-win32-VC11.zip  
	
搭配原则是：VC14+PHP7_TS+Apache_VC14 、VC11+PHP5_TS+Apache_VC11。  

2、如果使用Nginx，请使用NTS版本PHP：  
	
PHP7:   
	1)VC14_x86: http://windows.php.net/downloads/releases/php-7.0.13-nts-Win32-VC14-x86.zip  
	2)VC14_x64：http://windows.php.net/downloads/releases/php-7.0.13-nts-Win32-VC14-x64.zip  
	
PHP5:  
	1)VC11_x86: http://windows.php.net/downloads/releases/php-5.6.28-nts-Win32-VC11-x86.zip  
	1)VC11_x64: http://windows.php.net/downloads/releases/php-5.6.28-nts-Win32-VC11-x64.zip  
	
Nginx: http://nginx.org/download/nginx-1.10.2.zip  
