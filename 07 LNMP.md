# 安装PHP+Nginx

@author jiancaigege@163.com
@date 2016-6-5 14:28:02

[TOC]

---

本文以centos6为例。

## 安装PHP
### 下载

```
http://cn2.php.net/distributions/php-5.6.22.tar.bz2
http://cn2.php.net/distributions/php-7.0.7.tar.bz2
```

### 更新yum源
这里将Centos的yum源更换为国内的阿里云源。yum安装正常的可以跳过本步骤。

阿里云Linux安装镜像源地址：
http://mirrors.aliyun.com/

1、备份你的原镜像文件，以免出错后可以恢复：
``` shell
mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup
```

2、下载新的CentOS-Base.repo 到`/etc/yum.repos.d/`
``` shell
## CentOS 5
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-5.repo

## CentOS 6
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-6.repo
```

3、生成缓存
``` shell
yum clean all
yum makecache
```

### 安装依赖
``` shell
yum install -y wget lrzsz
yum install -y gcc gcc-c++ make cmake bison autoconf wget
yum install -y libtool libtool-ltdl-devel 
yum install -y freetype-devel libjpeg.x86_64 libjpeg-devel libpng-devel gd-devel
yum install -y libmcrypt-devel libmhash-devel
yum install -y python-devel  patch  sudo 
yum install -y openssl* openssl openssl-devel ncurses-devel
yum install -y bzip* bzip2 unzip zlib-devel
yum install -y libevent*
yum install -y libxml* libxml2-devel
yum install -y libcurl* curl-devel 
yum install -y readline-devel
```

libmcrypt库
``` shell
wget ftp://mcrypt.hellug.gr/pub/crypto/mcrypt/libmcrypt/libmcrypt-2.5.7.tar.gz
tar zxvf libmcrypt-2.5.7.tar.gz 
cd libmcrypt-2.5.7
./configure --prefix=/usr/local/libmcrypt
make && make install
```

如果php编译还是提示mcrypt.so出错，重新安装libmcrypt，将`--prefix`改为`--prefix=/usr/local/`试试。

### 开始安装

使用`./configure --help`查看编译支持的选项。如果写了不支持的选项，如php7里不支持`--with-mysql=mysqlnd`会提示：
``` shell
configure: WARNING: unrecognized options: --with-mysql
```

``` shell
wget http://cn2.php.net/distributions/php-7.0.7.tar.bz2
tar jxvf php-7.0.7.tar.bz2 
cd php-7.0.7

$ ./configure --prefix=/www/server/php --with-config-file-scan-dir=/www/server/php/etc/ --enable-inline-optimization --enable-opcache --enable-session --enable-fpm --with-fpm-user=www --with-fpm-group=www --with-mysql=mysqlnd --with-mysqli=mysqlnd --with-pdo-mysql=mysqlnd --with-pdo-sqlite --with-sqlite3 --with-gettext --enable-mbregex --enable-mbstring --enable-xml --with-iconv --with-mcrypt --with-mhash --with-openssl --enable-bcmath --enable-soap --with-xmlrpc --with-libxml-dir --enable-pcntl --enable-shmop --enable-sysvmsg --enable-sysvsem --enable-sysvshm --enable-sockets --with-curl --with-curlwrappers --with-zlib --enable-zip --with-bz2 --with-gd --enable-gd-native-ttf --with-jpeg-dir --with-png-dir --with-freetype-dir --with-iconv-dir --with-readline
 
$ make
$ make install 
```

这里面开启了很多扩展。如果这时候忘了开启，以后还能加上吗？答案是可以的。以后只需要进入源码的ext目录，例如忘了pdo_mysql，进入ext/pdo_mysql，使用phpize工具，像安装普通扩展一样即可生成pdo_mysql.so。
 
### 配置文件
需要从安装包里复制php.ini到安装目录：
``` shell
$ cp php-5.6.22/php.ini* /www/server/php/etc/
$ cd /www/server/php/etc/
$ ls
```
pear.conf  php-fpm.conf.default  php.ini-development  php.ini-production

``` shell
$ cp php.ini-production php.ini
$ cp php-fpm.conf.default  php-fpm.conf
```

编辑php-fpm.conf，去掉里面那个 `pid = run/php-fpm.pid` 前面的分号。方便以后重启。

保存配置文件后，检验配置是否正确的方法为:
``` shell
/www/server/php/sbin/php-fpm -t
```
如果出现诸如 `test is successful` 字样，说明配置没有问题。

建立软连接：
``` shell
ln -sf /www/server/php/sbin/php-fpm /usr/bin/
ln -sf /www/server/php/bin/php /usr/bin/
ln -sf /www/server/php/bin/phpize /usr/bin/
ln -sf /www/server/php/bin/php-config /usr/bin/
ln -sf /www/server/php/bin/php-cig /usr/bin/
```

### 启动php-fpm
``` shell
 /www/server/php/sbin/php-fpm 
```
如果提示没有www用户，则新增：
``` shell
useradd www
chown -R www:www /www
```

检测是否启动:
``` shell
ps aux |grep php-fpm
netstat -ant |grep 9000
```

查看php-fpm进程数：
``` shell
ps aux | grep -c php-fpm
```

php-fpm操作汇总：
``` shell
/www/server/php/sbin/php-fpm 		# php-fpm启动
kill -INT `cat /www/server/php/var/run/php-fpm.pid` 		# php-fpm关闭
kill -USR2 `cat /www/server/php/var/run/php-fpm.pid` 		#php-fpm重启
```

重启方法二：
``` shell
killall php-fpm
/www/server/php/sbin/php-fpm &
```

如果无法平滑启动，那就终止进程id：
``` shell
ps aux | grep php-fpm
kill -9  1210  #1210指php-fpm进程id
```

## 安装Nginx
http://nginx.org/download/nginx-1.11.1.tar.gz

依赖：
``` shell
yum install pcre-devel
```

配置编译参数
``` shell
$ tar -zxvf nginx-1.11.1.tar.gz
$ cd nginx-1.11.1
$ ./configure --prefix=/www/server/nginx --with-http_realip_module --with-http_sub_module --with-http_gzip_static_module --with-http_stub_status_module  --with-pcre
```

编译安装nginx
``` shell
make
make install
```

设置软连接：
``` shell
ln -sf /www/server/nginx/sbin/nginx /usr/sbin 
```

检测nginx:
``` shell
nginx -t
```
显示：
nginx: configuration file /www/server/nginx/conf/nginx.conf test is successful

成功了。我们重新配置下nginx.conf：

``` conf
#user  nobody;
worker_processes  1;

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       mime.types;
    default_type  application/octet-stream;
	
	# 解决虚拟主机名字过长 http://www.jb51.net/article/26412.htm
	server_names_hash_bucket_size 128; 

    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;

    #gzip  on;
	
	autoindex on;# 显示目录
	autoindex_exact_size on;# 显示文件大小
	autoindex_localtime on;# 显示文件时间
	
	include vhosts/*.conf;

}

```

配置localhost:
vhosts/localhost.conf
``` conf
server {
	listen       80;
	server_name  localhost;

	#charset utf-8;

	#access_log  logs/host.access.log  main;

	location / {
		root   /www/www/;
		index  index.php index.html index.htm;
	}

	#error_page  404              /404.html;

	# redirect server error pages to the static page /50x.html
	#
	error_page   500 502 503 504  /50x.html;
	location = /50x.html {
		root   html;
	}

	# proxy the PHP scripts to Apache listening on 127.0.0.1:80
	#
	#location ~ \.php$ {
	#    proxy_pass   http://127.0.0.1;
	#}

	# pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
	#
	location ~ \.php$ {
		root           /www/www/;
		fastcgi_pass   127.0.0.1:9000;
		fastcgi_index  index.php;
		fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
		include        fastcgi_params;
	}
}
```

启动nginx:
``` shell
/www/server/nginx/sbin/nginx

# 或者
nginx
```

重启：
``` shell
/www/server/nginx/sbin/nginx -s reload

# 或者
nginx -s reload
```

如果提示80端口被占用了，可以使用`ps aunx | grep 80`查看。一般是apache占用了。可以使用：
``` shell
chkconfig --list
chkconfig nginx on
service apache off
```
禁止apache启动并关闭apache服务。

## 安装扩展

### 安装swoole
Swoole: PHP的异步、并行、高性能网络通信引擎
http://www.swoole.com/

``` shell
wget https://github.com/swoole/swoole-src/archive/swoole-1.8.5-stable.zip
unzip swoole-1.8.5-stable.zip
cd swoole-1.8.5-stable
phpize
./configure
make && make install
```

### 安装redis
服务器端：
http://download.redis.io/releases/redis-3.2.0.tar.gz
```
$ wget http://download.redis.io/releases/redis-3.2.0.tar.gz
$ tar xzf redis-3.2.0.tar.gz
$ cd redis-3.2.0
$ make
```
默认编译完后在当前目录的src目录下。可以复制可执行文件到其他地方：
```
mkdir /www/server/redis
cd src
cp  redis-benchmark redis-check-aof redis-check-rdb redis-cli redis-sentinel redis-server redis-trib.rb /www/server/redis
```

复制配置文件
```
$ cd redis-3.2.0
$ cp redis.conf /www/server/redis/
```

或者安装的时候指定位置：
```
make PREFIX=/www/server/redis install
```

**将Redis的命令所在目录添加到系统参数PATH中:**
修改profile文件：
```
vi /etc/profile
```
在最后行追加: 
```
export PATH="$PATH:/www/server/redis/bin"
```
然后马上应用这个文件： 
```
. /etc/profile  
```
这样就可以直接调用redis-cli的命令了

客户端：
#### 2.0安装
```
wget https://github.com/nicolasff/phpredis/archive/2.2.4.tar.gz
tar -zxvf 2.2.4
cd phpredis-2.2.4/
phpize
./configure 
make && make install
```

#### 3.0安装
phpredis/phpredis: A PHP extension for Redis
https://github.com/phpredis/phpredis

需要先安装igbinary：

PECL :: Package :: igbinary
http://pecl.php.net/package/igbinary

```
wget http://pecl.php.net/get/igbinary-1.2.1.tgz
tar zxvf igbinary-1.2.1.tgz
cd igbinary-1.2.1
phpize
./configure 
make && make install
```

```
wget https://github.com/phpredis/phpredis/archive/3.0.0-rc1.zip
unzip 3.0.0-rc1
cd phpredis-3.0.0-rc1/

phpize
./configure [--enable-redis-igbinary]
make && make install
```

### 安装memcache


## 更多资料
1、linux下为已经编译好的php环境添加mysql扩展
https://ask.hellobi.com/blog/liangyong/69
 
