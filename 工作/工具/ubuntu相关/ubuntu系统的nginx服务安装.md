#  nginx安装
通过 `apt-get` 方式进行安装
```
sudo apt-get install nginx
```
查看 `nginx` 是否安装成功
```
nginx -v
```
![](https://cdn.jsdelivr.net/gh/chenjianhao66/Myblog_picture-server/2021-07-14_10-09-nginx.png)

nginx文件安装完成之后的文件位置：

-   /usr/sbin/nginx：主程序
-   /etc/nginx：存放配置文件
-   /usr/share/nginx：存放静态文件
-   /var/log/nginx：存放日志