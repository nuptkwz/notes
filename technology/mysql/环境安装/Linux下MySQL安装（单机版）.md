> 本文主要介绍了Linux下MySQL的安装。
# 下载
到官网下载自己需要的mysql版本，地址：https://downloads.mysql.com/archives/community/
```
wget https://downloads.mysql.com/archives/get/p/23/file/mysql-5.7.17-linux-glibc2.5-x86_64.tar.gz
```

# 解压
```
tar -zxvf mysql-5.7.17-linux-glibc2.5-x86_64.tar.gz
```

# 解压完成后，移动该文件到新文件夹
- 新建文件夹/usr/local/mysql，将解压后的文件移动到此
```
mv mysql-5.7.17-linux-glibc2.5-x86_64 /usr/local/mysql
```
- 创建data目录，存放mysql的数据文件
```
mkdir /usr/local/mysql/data
```

# 添加执行用户名和组
```
groupadd mysql
useradd -g mysql mysql
```

# 修改权限
更改mysql目录下所有的目录及文件夹所属的用户组和用户权限
```
chown -R mysql:mysql /usr/local/mysql
chown -R 755 /usr/local/mysql
```

# 初始化安装 
务必记住初始化输出末尾的密码（数据库管理员临时密码）
```
./mysqld --user=mysql --basedir=/usr/local/mysql --datadir=/usr/local/mysql/data --initialize
```

# 编辑配置文件my.cnf，添加配置如下
```
datadir=/usr/local/mysql/data
basedir=/usr/local/mysql
port=3306
socket=/var/lib/mysql/mysql.sock
# Disabling symbolic-links is recommended to prevent assorted security risks
symbolic-links=0
```

# 启动、停止mysql服务器
到/usr/local/mysql/support-files目录下
- 启动
```
启动
./mysql.server start
或者
service mysql start
```

- 停止
```
service mysqld stop
```

- 重启
```
service mysql restart
```

# 登陆mysql
- 登陆
```
mysql -uroot -h localhost -p
```

# 遇到的巨坑
用root用户执行mysqld 跟mysqld_safe 不加--user=root参数 指定用户时会报错的
[启动mysql服务时一直提示ERROR! The server quit without updating PID file
](https://blog.csdn.net/zqin0/article/details/106444580/)
因此root账号登陆需要：
```
service mysql start --user=root
```

# 参考
1. [Linux下安装mysql-5.7.24](https://www.jianshu.com/p/276d59cbc529)
2. [Linux启动/停止/重启Mysql数据库的方法](https://www.cnblogs.com/linjiqin/p/3544472.html)
3. [Can't connect to local MySQL server through socket [Solved]](https://tableplus.com/blog/2018/08/solved-cant-connect-to-local-mysql-server-through-socket.html)