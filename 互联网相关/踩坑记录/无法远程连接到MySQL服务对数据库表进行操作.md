[[2021-08-03]]
22:27:19
author:陈建浩
#踩坑记录 

--- 

# 问题背景
在 `Ubuntu` 系统里安装 `MySQL` 服务，需要使用代码对 `MySQL` 数据库里的表数据进行操作。

# 问题描述
在使用第三方工具对 `Mysql` 服务进行远程访问时显示错误，拒绝访问。

# 问题分析
1. 初步分析是用户密码二者有一样是错的
	在经过排查和验证后，用户密码均为正确。
2. 怀疑是账号没有远程链接的权限
3. 在搜索引擎搜索关键字，`MySQL远程链接失败`

# 解决方法

1. 新建用户，对新用户赋予权限并刷新缓存

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