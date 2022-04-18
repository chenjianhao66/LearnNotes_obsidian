#ubuntu 
#  1）MySQL数据库环境配置
MySQL数据库最快捷的安装方法是通过 `apt-get` 方式安装
```
sudo apt-get update
sudo apt-get install mysql-server
```
注意：在执行以上命令的过程中，会提示让你配置root账号的密码，要记住这个密码；

在安装成功之后，通过以下命令来查看 MySQL 服务器是否安装成功
```
systemctl status mysql

执行上面命令见到以下字样，就代表安装成功 👇
● mysql.service - MySQL Community Server
     Loaded: loaded (/lib/systemd/system/mysql.service; enabled; vendor preset: enabled)
     Active: active (running) since Thu 2021-07-08 09:38:03 CST; 4h 11min ago
    Process: 868 ExecStartPre=/usr/share/mysql/mysql-systemd-start pre (code=exited, status=0/SUCCESS)
   Main PID: 931 (mysqld)
     Status: "Server is operational"
      Tasks: 38 (limit: 18784)
     Memory: 400.5M
     CGroup: /system.slice/mysql.service
             └─931 /usr/sbin/mysqld

7月 08 09:38:02 yz-ThinkPad-E15-Gen-2 systemd[1]: Starting MySQL Community Server...
7月 08 09:38:03 yz-ThinkPad-E15-Gen-2 systemd[1]: Started MySQL Community Server.
```

![](https://cdn.jsdelivr.net/gh/chenjianhao66/Myblog_picture-server/2021-07-09_09-19-mysql.png)

## 

# 2）数据库踩坑记录
## 1. mysql用户拒绝远程访问

解决方法:新建一个用户,赋予权限,然后刷新缓存.
```
## 进入mysql管理界面
sudo mysql -yroot -p

## 创建用户
mysql> create user 'test'@'%' identified by 'Yz123456.';

## 赋予权限,赋予test用户对所有数据库有所有权限
mysql> grant all privileges on *.* to 'test'@'%' with grant option;

## 刷新权限
mysql> flush privileges;

## 查看用户权限

mysql> select user,host from user;
+------------------+-----------+
| user             | host      |
+------------------+-----------+
| root             | %         |
| test             | %         |
| debian-sys-maint | localhost |
| mysql.infoschema | localhost |
| mysql.session    | localhost |
| mysql.sys        | localhost |
+------------------+-----------+
6 rows in set (0.00 sec)

## 如果user对应的host列中是 % 的话就代表可以远程连接,localhost代表仅能本地连接.

至此,所有设置完毕. mysql图形化界面可以通过test用户来进行远程访问.

## 如果想要root设置允许远程连接的呢?

## 解决方案:

### 进入mysql管理界面并且选择mysql数据库

mysql -uroot -p
mysql> use mysql;

## 设置root账户的访问权限

update user set host='%' where user='root';

## 执行上面sql语句后,查询用户的权限

mysql> select user,host from user;

解决完毕.

### 可以对一个用户进行精细化权限赋予,下面是例子:
### 赋予test用户对testdb数据库有update的权限.
mysql> grant all privileges on testdb.update to 'test'@'%' with grant option;

```