#windows下安装mysql

---

约定：
环境安装目录：
```
D:/phpsetup/
   |——php
   |——nginx
   |——mysql
   |——www
```

mysql下载地址：<br/>
MySQL :: Download MySQL Community Server
http://dev.mysql.com/downloads/mysql/

mysql-5.7.12-win32.zip
<br/>
http://120.52.73.11/cdn.mysql.com//Downloads/MySQL-5.7/mysql-5.7.12-win32.zip

mysql-5.7.12-winx64.zip
<br/>
http://120.52.73.13/cdn.mysql.com//Downloads/MySQL-5.7/mysql-5.7.12-winx64.zip

1、解压
解压到`D:/phpsetup/mysql/`。
如果在安装根目录没有data文件夹，请创建data文件夹，以存放数据。

2、设置环境变量
在PATH中增加：
```
D:\phpsetup\mysql\bin;
```

或者cmd里运行：
```
path=%PATH%;D:\phpsetup\mysql\bin;
```

3、配置mysql
复制my-default.ini为my.ini：
```
[mysqld]
basedir = D:\phpsetup\mysql
datadir = D:\phpsetup\mysql\data
port = 3306
```

4、安装MySQL服务

使用管理员权限在bin目录执行，
```
mysqld -install
Service successfully installed.
```

启动mysql
```
net start mysql
```
发现启动失败。之前用的5.6没有问题，然后查了资料：

MySQL数据库在升级到5.7版本后，和之前的版本有些不一样，没有data文件夹，导致无法启动mysql。

>我们都知道MySQL数据库文件是保存在data文件夹中的，网上有人说把5.6版本的data文件夹拷贝一个，试了确实可以启动服务了，但是无法修改管理员密码，下面还是给个标准的解决方法。

只需输入如下命令回车即可：
```
mysqld --initialize-insecure --user=mysql
```
执行完上面命令后，MySQL会自建一个data文件夹，并且建好默认数据库，登录的用户名为root，密码为空，后面的操作就跟之前版本一样了。

然后启动mysql
```
net start mysql

MySQL 服务正在启动 .
MySQL 服务已经启动成功。
```

停止mysql服务：
```
net stop mysql
```

移除mysql服务：
```
mysqld --remove
```

5、登陆MySQL
初次安装直接敲MySQL就可以进去，默认使用空用户登陆。