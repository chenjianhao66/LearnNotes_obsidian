###  Statistic 代码统计插件
该插件会统计项目中各个文件的数量、大小、行数等信息；

根据扩展名自定义统计详细行数信息 , 包括总行数,代码行数,代码行数占比,注释行数,注释行数占比,空白行数,空白行数占比；

自定义选择多个文件 , 统计各个文件信息；

#### 安装方式
在编辑器中使用快捷键 `ctrl` + `alt` + `s` 来弹出设置面板，点击插件选项，在插件市场中搜索关键字 `Statistic` ，即可安装使用。

#### 插件使用
在 `Statistic` 插件安装完成后，会在编辑器的工具栏窗口多处一个 `Statistic` 图标，如下图所示：
![](https://images-1306554305.cos.ap-guangzhou.myqcloud.com/2022-06-08_13-26.png)

刚刚安装完是没有开始统计代码信息的，需要点击 `Refresh` 按钮来更新项目代码信息；

**配置插件信息**
点击 `Statistic` 面板顶部的第三个按钮 `Settings` ，即可进入设置界面配置插件信息
在配置面板中有以下配置项可以选择：

**Excluded file types**
排除统计的代码文件类型，这里填写后缀名

**Included file types**
引入需要进行代码文件的类型

**Separate TABs file types**
在窗口中展示特定的代码文件类型，勾选之后就会在窗口中展示特定的代码文件类型统计信息；

**Excluded directories**
排除的目录，这些目录下的文件都不会被统计


目前配置是，在**Separate TABs file types** 中添加了 `go`、`conf`、`sh` 这三种类型，也意味着在窗口中出了展示整个项目所有类型的统计信息 `Overview`  窗口外，还将添加 `go`、`conf`、`sh` 这三种类型的窗口，如下图：
![](https://images-1306554305.cos.ap-guangzhou.myqcloud.com/2022-06-08_13-42.png)

#### 统计信息
在代码文件类型窗口中，统计了以下这几种信息：

**源文件**
**总行数**
**源代码行数**
**源代码行数在总行数中的百分比占比**
**注释行数**
**注释行数在总行数中的百分比占比**
**空行行数**
**空行行数在总行数中的百分比占比**