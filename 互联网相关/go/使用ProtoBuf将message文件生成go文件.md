
#go 
因工作需求，需要安装protobuf工具，并且根据proto文件生成go文件，下面对安装以及生成go文件的过程做记录

# 安装ProtoBuf
访问ProtoBuf的[Github](https://github.com/protocolbuffers/protobuf)仓库，使用git将该仓库克隆下来
```bash
git clone https://github.com/protocolbuffers/protobuf.git
```
在执行完以上命令之后进入到该目录，确定好ProtoBuf的版本，可以使用 `git checkout`命令来切换。

确认好版本之后就进行正式编译和安装，执行以下命令并等待结束：

```bash
sudo apt-get install autoconf automake libtool curl make g++ unzip # 安装依赖包
cd protobuf/
git submodule update --init --recursive
sudo ./autogen.sh   #生成配置脚本
sudo ./configure    #生成Makefile文件，为下一步的编译做准备，可以加上安装路径：--prefix=path ，默认路径为/usr/local/
sudo make           #从Makefile读取指令，然后编译
sudo make check     #可能会报错，但是不影响,对于安装流程没有实质性用处，可以跳过该步
sudo make install 
sudo ldconfig       #更新共享库缓存
which protoc        #查看软件的安装位置
protoc --version    #检查是否安装成功
```

# 使用ProtoBuf来生成Go文件
protobuf自带对Java、c++、js、python、ruby、php的文件生成。
## 安装protoc-gen-go插件
生成go的文件需要自己安装 `protoc-gen-go`插件才能生成；安装命令如下：
```bash
go get -u github.com/golang/protobuf/protoc-gen-go
```
保证网络稳定，在安装完之后默认会是在 `$GOPATH/bin`目录下，如果没有配置 `$GOPATH`的话，会出现在 `~/go`目录即home目录。
![2022-03-16_22-20.png](https://cdn.nlark.com/yuque/0/2022/png/25809861/1647440735980-5a05ad30-87f7-4ec6-a9a3-4ef79f33fa2c.png#clientId=u33584b8c-f4d0-4&crop=0&crop=0&crop=1&crop=1&from=ui&id=u5aa53cf3&margin=%5Bobject%20Object%5D&name=2022-03-16_22-20.png&originHeight=64&originWidth=672&originalType=binary&ratio=1&rotation=0&showTitle=false&size=12445&status=done&style=none&taskId=u89be36e9-cc94-4f93-a4ed-446f4e33b3e&title=)
## 导出至环境变量
该可执行程序需要导入到环境变量，在其他文件内使用，使用以下命令
```bash
sudo vim /etc/profile
## 我的GOPATH在home目录
export GOPATH=$HOME/go
PATH=$PATH:$GOPATH/bin

## 退出编辑状态，使用以下命令让环境变量暂时生效，重启之后永久生效
source /etc/profile
```
## 编译生成go文件
进入到proto文件夹中，因为go的包语句原因，每一个单独的文件都会存放在与 `package`语句后包名相同的物理目录名
> 比如生成的 `common.pb.go`go文件里的 `package`语句跟着的包是 `std_msgs`，那么存放 `common.pb.go`文件的物理目录就得叫做 `std_msgs`。

这次我生成的文件因为这个原因，都得单独执行生成语句。
```bash
protoc --proto_path={要到搜索的目录} --go_out={生存go文件的地址}
```
> 注意，如果上一个步骤的插件没有安装成功，那么--go_out该命令是不成功的

而且这种给每个proto文件单独执行转换语句的方法，生成的go文件里面是存在依赖问题的；也就是说A文件要用到B文件里的结构体，但是找不到B的结构体。

那么在这次生成go文件的过程中，需要在go文件中手动的引入包，使文件内的结构体能够正确饮用。

