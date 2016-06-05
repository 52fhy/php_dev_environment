# 安装PHP+Nginx

@author jiancaigege@163.com
@date 2016-6-5 14:28:02

---

本文以centos6为例。

## 安装PHP
### 下载

http://cn2.php.net/distributions/php-5.6.22.tar.bz2
http://cn2.php.net/distributions/php-7.0.7.tar.bz2

### 更新yum源
这里将Centos的yum源更换为国内的阿里云源。yum安装正常的可以跳过本步骤。

阿里云Linux安装镜像源地址：
http://mirrors.aliyun.com/

1、备份你的原镜像文件，以免出错后可以恢复：
```
mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup
```

2、下载新的CentOS-Base.repo 到`/etc/yum.repos.d/`
```
## CentOS 5
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-5.repo

## CentOS 6
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-6.repo
```

3、生成缓存
```
yum clean all
yum makecache
```

### 安装依赖
```
yum install -y gcc gcc-c++ make cmake bison autoconf 
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
```
wget ftp://mcrypt.hellug.gr/pub/crypto/mcrypt/libmcrypt/libmcrypt-2.5.7.tar.gz
./configure --piefix=/usr/local/libmcrypt   make && make install
```

### 开始安装
```
$ ./configure --prefix=/www/server/php --with-config-file-scan-dir=/www/server/php/etc/ --enable-inline-optimization --enable-opcache --enable-session --enable-fpm --with-fpm-user=www --with-fpm-group=www --with-mysql=mysqlnd --with-mysqli=mysqlnd --with-pdo-mysql=mysqlnd --with-pdo-mysqli=mysqlnd --with-pdo-sqlite --with-sqlite3 --with-gettext --enable-mbstring --enable-xml --with-iconv --with-mcrypt --with-mhash --with-openssl --enable-bcmath --enable-soap --with-xmlrpc --with-libxml-dir --enable-pcntl --enable-shmop --enable-sysvmsg --enable-sysvsem --enable-sysvshm --enable-sockets --with-curl --with-curlwrappers --with-zlib --enable-zip --with-bz2 --with-gd --with-jpeg-dir --with-png-dir --with-freetype-dir --with-iconv-dir --with-readline
 
 $ make
 $ make install 
 ```
 
 ### 配置文件
需要从安装包里复制php.ini到安装目录：
```
$ cp php-5.6.22/php.ini* /www/server/php/etc/
$ cd /www/server/php/etc/
$ ls
```
pear.conf  php-fpm.conf.default  php.ini-development  php.ini-production

```
$ cp php.ini-production php.ini
$ cp php-fpm.conf.default  php-fpm.conf
```

编辑php-fpm.conf，去掉里面那个 `pid = run/php-fpm.pid` 前面的分号。方便以后重启。

保存配置文件后，检验配置是否正确的方法为:
```
/www/server/php/sbin/php-fpm -t
```
如果出现诸如 `test is successful` 字样，说明配置没有问题。

建立软连接：
```
ln -sf /www/server/php/sbin/php-fpm /usr/bin/
ln -sf /www/server/php/bin/php /usr/bin/
ln -sf /www/server/php/bin/phpize /usr/bin/
ln -sf /www/server/php/bin/php-config /usr/bin/
ln -sf /www/server/php/bin/php-cig /usr/bin/
```

### 启动php-fpm
```
 /www/server/php/sbin/php-fpm 
```
如果提示没有www用户，则新增：
```
useradd www
chown -R www:www /www
```

检测是否启动:
```
ps aux |grep php-fpm
netstat -ant |grep 9000
```

查看php-fpm进程数：
```
ps aux | grep -c php-fpm
```

php-fpm操作汇总：
```
/www/server/php/sbin/php-fpm 		# php-fpm启动
kill -INT `cat /www/server/php/var/run/php-fpm/php-fpm.pid` 		# php-fpm关闭
kill -USR2 `cat /www/server/php/var/run/php-fpm/php-fpm.pid` 		#php-fpm重启
```

重启方法二：
```
killall php-fpm
/www/server/php/sbin/php-fpm &
```

如果无法平滑启动，那就终止进程id：
```
ps aux | grep php-fpm
kill -9  1210  #1210指php-fpm进程id
```

## 安装Nginx
http://nginx.org/download/nginx-1.11.1.tar.gz

依赖：
```
yum install pcre-devel
```

配置编译参数
```
$ tar -zxvf nginx-1.11.1.tar.gz
$ cd nginx-1.11.1
$ ./configure --prefix=/www/server/nginx --with-http_realip_module --with-http_sub_module --with-http_gzip_static_module --with-http_stub_status_module  --with-pcre
```

编译安装nginx
```
make
make install
```

设置软连接：
```
ln -sf /www/server/nginx/sbin/nginx /usr/sbin 
```

检测nginx:
```
nginx -t
```
显示：
nginx: configuration file /www/server/nginx/conf/nginx.conf test is successful

成功了。我们重新配置下nginx.conf：
```

```

启动nginx:
```
/www/server/nginx/sbin/nginx

# 或者
nginx
```

重启：
```
/www/server/nginx/sbin/nginx -s reload

# 或者
nginx -s reload
```

如果提示80端口被占用了，可以使用`ps aunx | grep 80`查看。一般是apache占用了。可以使用：
```
chkconfig --list
chkconfig nginx on
service apache off
```
禁止apache启动并关闭apache服务。
 
