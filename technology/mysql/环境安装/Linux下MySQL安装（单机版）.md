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
# 修改权限
更改mysql目录下所有的目录及文件夹所属的用户组和用户权限

