
#ubuntu 
#  ubuntu下vscode环境搭建
vscode环境配置步骤如下：
1. 安装Java Extension Pack插件
2. 安装Spring Boot ExtensionPack插件
3. 编辑setting.json文件
## 1）安装Java Extension Pack
在插件市场搜索Java Extension Pack并安装，安装之后将附属包**Language Support for Java(TM) by Red Hat** 安装另外一个版本，安装 **0.64.1**版本

![](https://cdn.jsdelivr.net/gh/chenjianhao66/Myblog_picture-server/20210726232104.png)

![](https://cdn.jsdelivr.net/gh/chenjianhao66/Myblog_picture-server/20210726232338.png)


## 2）安装Spring Boot Extension Pack
-   安装 Spring Boot Extension Pack
-   这里面集合了所有需要的 Spring 开发组件

![](https://cdn.jsdelivr.net/gh/chenjianhao66/Myblog_picture-server/20210726232551.png)

##  3）编辑 setting.json文件
使用快捷键 **ctrl + ，** 弹出设置界面，在搜索框中搜索maven，点击**在setting.json中编辑** 按钮，弹出编辑页面。
![](https://cdn.jsdelivr.net/gh/chenjianhao66/Myblog_picture-server/20210726234031.png)

使用下面的代码替换
```
    "maven.excludedFolders": [
        "**/.*",
        "**/node_modules",
        "**/target",
        "**/bin",
        {
            "workbench.iconTheme": "vscode-icons",
            "workbench.startupEditor": "newUntitledFile",
            "java.errors.incompleteClasspath.severity": "ignore",
            "workbench.colorTheme": "Atom One Dark",
            "java.home":"/Library/Java/JavaVirtualMachines/jdk1.8.0_221.jdk/Contents/Home",
            "java.configuration.maven.userSettings": "/Users/admin/Documents/Maven/conf/settings.xml",
            "maven.executable.path": "/Users/admin/Documents/Maven/bin/mvn",
            "maven.terminal.useJavaHome": true,
            "maven.terminal.customEnv": [
                {
                    "environmentVariable": "JAVA_HOME",
                    "value": "/Library/Java/JavaVirtualMachines/jdk1.8.0_221.jdk/Contents/Home"
                }
            ],
        }
    ],

```

上面需要修改4处代码：
- "java.home"
- "java.configuration.maven.userSettings"
- "environmentVariable"
- "value"
分别修改为所对应的java目录和maven目录。

修改后：
![](https://cdn.jsdelivr.net/gh/chenjianhao66/Myblog_picture-server/20210726235651.png)
以上修改完毕