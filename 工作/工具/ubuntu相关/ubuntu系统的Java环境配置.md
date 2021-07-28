# ubuntu系统下Java环境配置
安装Java有两种方式
-	通过apt软件源在线安装openJDK或者orlcaeJDK；
-	通过orlcae官网下载Linux版本的jdk安装包并解压。

## 1）通过apt软件源安装OpenJDK
1. 更新软件源

```
sudo apt-get update
```

2. 安装open-Java

```
sudo apt-get install openjdk-8-jre
sudo apt-get install openjdk-8-jdk-headless

## 最后安装 open-java
sudo apt-get install openjdk-8-jdk

## 该jdk最后会安装到 /usr/lib/jvm 目录下
```

3. 配置环境变量
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

^eba47f

4. 修改配置文件后，使用 `source` 命令让配置生效
```
source /etc/profile
```
5. 验证 Java 环境是否配置好
```
java -version

javac -version
```

效果如下图所示：

![](https://cdn.jsdelivr.net/gh/chenjianhao66/Myblog_picture-server/2021-07-09_09-16-Java.png)

## 2）下载官网安装包方式
1. 前往官网下载对应版本的 tar.gz安装包
[下载地址](https://www.oracle.com/java/technologies/javase/javase-jdk8-downloads.html)
![](https://cdn.jsdelivr.net/gh/chenjianhao66/Myblog_picture-server/2021-07-09_09-23-javaDownload.png)

2. 将下载下来的安装包解压
```
sudo tar -zxvf jdk-8u291-linux-i586.tar.gz -C /usr/lib/jvm/
```
> 注意：-C参数后面跟的时候指定安装的位置

3. 解压完之后配置环境变量，[[#^eba47f|配置环境变量与上文配置一致]]，修改后让配置文件生效。

5. 验证 Java 环境是否配置好。
---