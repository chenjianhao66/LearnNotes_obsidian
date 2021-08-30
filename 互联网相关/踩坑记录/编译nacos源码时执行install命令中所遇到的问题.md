
[[2021-08-02]]
22:23:07
author:陈建浩
#踩坑记录 

--- 

# 问题背景
公司使用阿里巴巴的开源项目nacos作为微服务的服务注册与配置中心，管理界面使用自己开发的一套而不使用nacos提供的管理界面；最快的方法是将nacos的前端管理界面拿过来，修改里面的logo、配色方案等。所以要下载nacos的源码进行修改、编译以及安装打包。
在下载并解压好的 `nacos` 源码目录下，执行 `mvn` 生命周期命令。
# 问题描述
在执行 `mvn install` 命令的时候出现了问题，问题如下：
```
Failed to execute goal org.apache.rat:apache-rat-plugin:0.12:check (default) on project nacos-core:
Too many files with unapproved license: 1 See RAT report in: /var/project/naco-frontend/nacos-2.0.2/core/target/rat.txt
To see the full stack trace of the errors, re-run Maven with the -e switch.
[ERROR] Re-run Maven using the -X switch to enable full debug logging.
[ERROR] 
[ERROR] For more information about the errors and possible solutions, please read the following articles:
[ERROR] [Help 1] http://cwiki.apache.org/confluence/display/MAVEN/MojoFailureException
[ERROR] 
[ERROR] After correcting the problems, you can resume the build with the command
[ERROR]   mvn <args> -rf :nacos-core
```
在项目中未经许可的文件太多了，超过了一个项目能存在未经许可文件数量的阈值，所以报这个错误。
# 问题分析
1. 初步分析是因为 `maven` 的配置出错i，排查过后不是maven的配置问题，排除maven；
2. 第二步查看报错信息中提到的 `模块名/targer/rat.text` 文件；
	- 经过排查，该文件中并没有提示安装失败的提示信息

3. 查看报错信息中的 `org.apache.rat:apache-rat-plugin` 插件是否安装
	- 已经安装；

4. 在搜索引擎搜索关键字 `Failed to execute goal org.apache.rat:apache-rat-plugin:0.12:check (default) on project nacos-core:`


# 解决方法
解决思路，以上报错是因为项目中未经许可的文件太多，已经检查出超过上限的未经许可文件了；
那只需要跳过未经许可的检查即可
执行以下命令：
```
## 下面命令不需要执行
## mvn  clean apache-rat:check -Drat.numUnapprovedLicenses=600 install  -DskipTests

mvn clean install -DskipTests -Drat.skip=true

mvn -Prelease-nacos -Dmaven.test.skip=true -Drat.skip=true clean install -U 

```
执行以下命令之后即可安装打包，问题解决。