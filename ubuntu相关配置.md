# ubuntu相关配置
# 1、代码环境相关配置
##  安装 Java

^34a7f2

在ubuntu系统中安装 `Java` 的步骤如下：
- 通过软件源来安装Java，这里使用 `apt-get` 或者下载官网的安装包并解压
- 配置 `/etc/profile` ，增加环境变量 ^d21106
- 最后通过 `java -version`、`javac -version` 来验证Java是否安装成功


安装 Java 有两种方式
### apt-get方式
在保证系统能联网的情况下，使用``apt-get``方式来安装 JDK ：
1. 更新软件源
```
sudo apt-get update
```
2. 安装 open-Java的依赖包
```
sudo apt-get install openjdk-8-jre
sudo apt-get install openjdk-8-jdk-headless

## 最后安装 open-java
sudo apt-get install openjdk-8-jdk

## 该jdk最后会安装到 /usr/lib/jvm 目录下
```
3. 配置环境变量 ^13bd6e

```
## 通过图形化编辑器打开该文件
sudo gedit /etc/profile

## 在文件的末尾添加以下配置
## JAVA_HOME的地址请根据自己的情况配置
export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-amd64
export JRE_HOME=${JAVA_HOME}/jre
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib
export PATH=${JAVA_HOME}/bin:$PATH
```
4. 修改配置文件后，使用 `source` 命令让配置生效
```
source /etc/profile
```
5. 验证 Java 环境是否配置好
```
java -version

javac -version
```

### 下载官网安装包方式
1. 前往官网下载对应版本的 tar.gz安装包
[下载地址](https://www.oracle.com/java/technologies/javase/javase-jdk8-downloads.html)
2. 将下载下来的安装包解压
```
sudo tar -zxvf jdk-8u291-linux-i586.tar.gz -C /usr/lib/jvm/
```
> 注意：-C参数后面跟的时候指定安装的位置

3. 解压完之后配置环境变量，[[#^13bd6e|配置环境变量与上文配置一致]]，修改后让配置文件生效。

5. 验证 Java 环境是否配置好。


## Maven配置
1. 下载安装包


Mavn的安装包需要到[官网](https://maven.apache.org/download.cgi)下载，下载二进制文件，后缀名为 tar.gz的压缩包。

下载完毕之后进行解压缩：
```
sudo tar -zxvf apache-maven-3.8.1-bin.tar.gz -C /${user_path}
```

2. 配置环境变量

命令里的${user_path}根据自己的情况而定。解压缩之后在 `/etc/profile` 配置文件里面添加环境变量。
```
## maven配置
export MAVEN_HOME=/usr/lib/maven/apache-maven-3.8.1
export PATH=${PATH}:${MAVEN_HOME}/bin
```
3. 修改配置文件后，使用`source`命令让配置生效。
```
source /etc/profile
```
4. 验证是否配置成功
```
mvn -v

# 弹出以下信息就代表配置成功 👇

Apache Maven 3.8.1 (05c21c65bdfed0f71a2f2ada8b84da59348c4c5d)
Maven home: /usr/lib/maven/apache-maven-3.8.1
Java version: 1.8.0_292, vendor: Private Build, runtime: /usr/lib/jvm/java-8-openjdk-amd64/jre
Default locale: zh_CN, platform encoding: UTF-8
OS name: "linux", version: "5.8.0-59-generic", arch: "amd64", family: "unix"
```

## MySQL数据库安装
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

