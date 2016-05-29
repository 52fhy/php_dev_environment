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

如果是全新安装的mysql，
需要在`my.ini`增加：
```
early-plugin-load=""
skip-grant-tables
```

>early-plugin-load=""
skip-grant-tables数据库启动的时候 跳跃权限表的限制，不用验证密码，直接登录

然后初始化数据库：
```
mysqld initialize
```

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