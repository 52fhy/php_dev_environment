# 理解apache配置

[TOC]

---

## http.conf配置详解

## VirtualHost配置
相关的配置有：`Listen`,  `NameVirtualHost`,  `<VirtualHost>`。


1、Listen要监听的端口，多个端口，要写多个Listen；否则Apache启动的时候，不会启动相应的套接字。例如： 
```
Listen 80
Listen 8080
```
2、设置NameVirtualHost
（注意：2.4版本不需要再设置NameVirtualHost）
```
NameVirutalHost *:80
```

3、VirtualHost
重要：Apache 在接受到请求时，首先会默认第一个VirtualHost，然后再找匹配的，如果没有匹配的，就是第一个VirtualHost起作用。
```
<VirtualHost *:80>
	ServerName *
	DocumentRoot </opt/wamp/www/>
	<Directory /opt/wamp/www/>
		Order deny,allow
		Allow from all
	</Direcotry>
<VirtualHost/>

<VirtualHost *:80>
	ServerName test.com
	DocumentRoot </opt/wamp/www/test/>
	<Directory /opt/wamp/www/test/>
		Order deny,allow
		Allow from all
	</Direcotry>
<VirtualHost/>
```
需要在hosts文件添加
```
127.0.0.1  test.com
```
然后重启apache。

## Apache2.2和Apache2.4中httpd.conf配置文件的异同
Windows环境从Apache2.2改成Apache2.4后httpd.conf中的设置异同。

1、权限设定方式变更

2.2使用`Order Deny / Allow`的方式，2.4改用`Require`：

apache2.2：
```
Order deny,allow
Deny from all
```

apache2.4：
```
Require all denied
```
此处比较常用的有如下几种：
```
Require all denied

Require all granted

Require host xxx.com

Require ip 192.168.1 192.168.2

Require local
```

注意：若有设定在`htaccess`文件中的也要修改

2、设定日志纪录方式变更

`RewriteLogLevel` 指令改为 `logLevel`

LOGLEVEL设置第一个值是针对整个Apache的预设等级，后方可以对指定的模块修改此模块的日志记录等级

比如：
```
LogLevel warn rewrite: warn
```
3、Namevirtualhost 被移除
2.2版本在没有NameVirtualHost的时候，后续VirtualHost中ServerName就失效了。会以第一个VirtualHost的DocumentRoot以及其他相关属性为准。

新版的Apache已经去除了NameVirtualHost 这个配置，因为确实没什么用,参数在VirtualHost中都已经指明了。

4、需载入更多的模块
开启Gzip在apache2.2中需载入`mod_deflate`，apache2.4中需载入`mod_filter`和`mod_deflate`

开启SSL在apache2.2中需载入`mod_ssl`，apache2.4中需载入`mod_socache_shmcb`和`mod_ssl`

5、在windows环境建议的设置

```
EnableSendfile Off
EnableMMAP Off
```
当Log日志出现AcceptEx failed等错误时建议设置
```
AcceptFilter http none
AcceptFilter https none
```
说明：Win32DisableAcceptEx在apache2.4中被AcceptFilter None取代

6、Listen设定的调整

以443为例，不可以只设定Listen 443

会出现以下错误：

(OS 10048)一次只能用一个通讯端地址（通讯协定/网路位址/连接) : 
```
AH00072: make_sock: could not bind to address [::]:443
```

(OS 10048)一次只能用一个通讯端地址（通讯协定/网路位址/连接) : 
```
AH00072: make_sock: could not bind to address 0.0.0.0:443
AH00451: no listening sockets available, shutting down
AH00015: Unable to open logs
```

因此需指定监听的IP，可设定多个

例如：
```
Listen 192.168.2.1:443
Listen 127.0.0.1:443
```

更多查看：
<br/>
http://www.apache.org/dist/httpd/CHANGES_2.4
<br/>
http://httpd.apache.org/docs/2.4/upgrading.html


>参考：
<br/>
1、Apache VirtualHost配置 - wpjsolo - 博客园
http://www.cnblogs.com/wpjsolo/archive/2012/01/19/2327457.html
<br/>
2、Apache2.2和Apache2.4中httpd.conf配置文件的异同 - 服务器技术 - UPUPW绿色服务器平台
http://www.upupw.net/server/n75.html
<br/>