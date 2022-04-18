2021-12-29
18:20:15
author:陈建浩
#go

--- 

# Golang下载
前往[官网下载界面](https://golang.google.cn/dl/)下载相对应操作系统的文件，这里只演示linux版本的。
![](https://images-1306554305.cos.ap-guangzhou.myqcloud.com/2021-12-29_18-27.png)
下载之后执行以下命令，将该压缩包的内容解压到 `/usr/local` 目录中
```bash
cd ~/下载
tar -C /usr/local -xzf ${下载的go安装包名}
```

**添加环境变量**
编辑 `/etc/profile` 文件

```bash
export GOROOT=/usr/local/go/bin
export PATH=${PATH}:${GOROOT}
```

使环境变量文件永久生效
```bash
source /etc/profile
```

使用命令验证是否添加成功，输出以下文字就输出成功
```bash
$ go version
go version go1.17.5 linux/amd64
```

**添加代理变量**
有一些包和配置文件都是从 `github` 下载下来,因为墙的原因下载起来太慢,执行以下命令将七牛云的代理添加到环境变量中,减少下载时间
```bash
$ go env -w GO111MODULE=on 
$ go env -w GOPROXY=https://goproxy.cn,direct
```
> 使用 `go env` 或者 `go env ${指定变量名}` 来查看环境变量是否设置成功

# Vscode对Go的插件安装以及配置
## 安装go插件
进入到 `vscode` 中，在插件商城搜索关键字 `go`，安装
![](https://images-1306554305.cos.ap-guangzhou.myqcloud.com/2021-12-29_18-33.png)


## 设置vscode的setting文件
使用快捷键 `ctrl + ,` 调出设置窗口，在搜索栏出输入 `setting` 在搜索结果中点击 `编辑settings` 进入编辑窗口;
![](https://images-1306554305.cos.ap-guangzhou.myqcloud.com/2021-12-29_18-35.png)

在弹出的`settings.json` 文件编辑窗口中,在后面添加以下内容:
```xml
"editor.wordWrap": "on",

// 如果useLanguageServer设为true，那么在编写代码时引入本地没有的package时，会自动下载安装

// 就是有时候会非常卡，保存go的编码文件时偶尔会卡死。这点你们自己取舍吧

"go.useLanguageServer": false,

"editor.minimap.renderCharacters": false,

"editor.minimap.enabled": false,

"terminal.external.osxExec": "iTerm.app",

"go.docsTool": "gogetdoc",

"go.testFlags": ["-v","-count=1"],

"go.buildTags": "",

"go.buildFlags": [],

"go.lintFlags": [],

"go.vetFlags": [],

"go.coverOnSave": false,

"go.useCodeSnippetsOnFunctionSuggest": false,

"go.formatTool": "default",

"go.gocodeAutoBuild": false,

//这里一定要配置正确
"go.goroot": "/usr/local/go",

"go.autocompleteUnimportedPackages": true,

"go.formatOnSave": true,

"window.zoomLevel": 0,

"debug.console.fontSize": 16,

"debug.console.lineHeight": 30,
```



## 运行Hello程序
创建一个目录用于放置项目文件
```bash
$ mkdir -p /var/project/go/hello
```

进入该项目目录中,我这里是 `hello`
```bash
$ cd /var/project/go/hello
```

创建 go mod文件,该文件是管理依赖包和模块的
```bash
## 如果是根目录的话,模块名这里就填写项目名
$ go mod init ${模块名}
```

执行以上命令之后目录下就会出现一个 `go.mod` 文件,文件会有以下内容:
![](https://images-1306554305.cos.ap-guangzhou.myqcloud.com/2021-12-29_18-44.png)

> 一开始文件内只有 module 和 go的版本,`require` 字段会在编译执行后追加到文件内.

**编写main.go程序入口文件**
在根目录创建 `main.go` ,将以下内容添加
```go
package main

import (
	"fmt"
)

func main() {
	fmt.Println("Hello,go")
}
```
以上内容导入 `fmt`包和自定义包 `hello/entity`
fmt包是用于控制台输出,entity包只有一个Add函数,用于对2个入参进行累加.


**执行go程序**
在 `main.go` 文件所在目录下,执行以下命令
```bash
$ go run main.go
Hello,go
```

执行成功!