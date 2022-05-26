#Go 


# 简介
[cobra](http://github.com/spf13/cobra)是一个命令行程序库，可以用来编写命令行程序。同时，它也提供了一个脚手架， 用于生成基于 cobra 的应用程序框架。非常多知名的开源项目使用了 cobra 库构建命令行，如[Kubernetes](http://kubernetes.io/)、[Hugo](http://gohugo.io/)、[etcd](https://github.com/coreos/etcd)等等等等。 本文介绍 cobra 库的基本使用和一些有趣的特性。


cobra 提供非常丰富的功能：

-   轻松支持子命令，如`app server`，`app fetch`等；
-   完全兼容 POSIX 选项（包括短、长选项）；
-   嵌套子命令；

首先需要明确 3 个基本概念：

-   命令（Command）：就是需要执行的操作；
-   参数（Arg）：命令的参数，即要操作的对象；
-   选项（Flag）：命令选项可以调整命令的行为。

下面示例中，`server`是一个（子）命令，`--port`是选项：

`hugo server --port=1313`

下面示例中，`clone`是一个（子）命令，`URL`是参数，`--bare`是选项：

`git clone URL --bare`


## 简单示例

现在我有一个需求，本地的markdown编辑器我使用的是 `obsidian`，线上博客使用的是 `hexo`静态博客，静态博客需要将markdown文件上传到博客目录，使用命令生成静态网页文件供别人浏览；

在进行文章分类的时候需要添加一些 `hexo` 独有的标签字段来表示该文章的分类，但这些标签字段在本地markdown编辑器 `obsidian` 中特别不友好，所以我现在一个程序来帮我完成以下事情：

- 将本地编辑好的markdown文件添加 `hexo` 博客独有的字段信息
- 添加字段信息后添加到博客文章文件夹单独管理
- 将生成好字段的文件上传到博客服务器，编译生成静态文件

在经过分析之后第一点和第二点可以实现，第三点中需要ssh到远端服务器并上传，这一步还不知道怎么弄；


第一步和第二步可以通过cobra库来实现自动化，项目结构如下：
```cmd
├─.idea
├─cmd
│  ├─blog.go
│  └─root.go
├─internal
│  ├─blog
│  │  └──blog.go
│  └─model
│     └──head.go
├─main.go
```


main文件
```go
package main  
  
import "github.com/chenjianhao66/sunshine_commandLine/cmd"  
  
func main() {  
   cmd.Execute()  
}
```

该文件只调用cmd包下的Execute函数

cmd包的文件
```go
package cmd  
  
import (  
   "github.com/spf13/cobra"  
)  
  
var rootCmd = &cobra.Command{  
   Use:   "sunshine",  
   Short: "阳光工具集",  
   Long:  `这是一个个人命令行工具集，目前打算将博客的文章自动添加hexo的markdown头部信息，并且发布到个人博客服务器中，编译文件并运行`,  
   Run: func(cmd *cobra.Command, args []string) {  
  
   },  
}  
  
func Execute() {  
   rootCmd.Execute()  
}
```

以下是上面程序运行之后的状态截图
![](https://images-1306554305.cos.ap-guangzhou.myqcloud.com/202204182220911.png)

cobra.Command对象有一个数组，代表着子命令