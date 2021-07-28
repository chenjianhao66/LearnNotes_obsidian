ubuntu系统下的Maven环境配置有以下几步：
- 去官网下载Maven压缩包
- 解压缩
- 配置环境变量
- 验证是否成功


# 1）到官网下载压缩包
Mavn的安装包需要到[官网](https://maven.apache.org/download.cgi)下载，下载二进制文件，后缀名为 tar.gz的压缩包。
![](https://cdn.jsdelivr.net/gh/chenjianhao66/Myblog_picture-server/2021-07-09_09-22-mavendowload.png)

# 2）下载之后进行解压缩
```
sudo tar -zxvf apache-maven-3.8.1-bin.tar.gz -C /${user_path}
```

${user_path}为你自己想要解压到的目录

# 3）配置环境变量

```
## maven配置 
export MAVEN_HOME=/usr/lib/maven/apache-maven-3.8.1 
export PATH=${PATH}:${MAVEN_HOME}/bin
```

配置完之后使用 `source` 命令使配置生效
```
source /etc/profile
```

# 4）验证配置是否成功
```
mvn -v # 弹出以下信息就代表配置成功 👇 
Apache Maven 3.8.1 (05c21c65bdfed0f71a2f2ada8b84da59348c4c5d) 
Maven home: /usr/lib/maven/apache-maven-3.8.1 
Java version: 1.8.0_292, vendor: Private Build, runtime: /usr/lib/jvm/java-8-openjdk-amd64/jre 
Default locale: zh_CN, platform encoding: UTF-8 OS name: "linux", version: "5.8.0-59-generic", arch: "amd64", family: "unix"
```

![](https://cdn.jsdelivr.net/gh/chenjianhao66/Myblog_picture-server/2021-07-09_09-20-maven.png)