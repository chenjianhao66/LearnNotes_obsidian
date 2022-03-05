2022-01-07
17:44:18
author:陈建浩
#Go 

--- 
## 使用Golang读取配置文件

使用Golang读取配置文件内容，需要使用 `io` 包读取配置文件，第三方包 `gopkg.in/yaml.v2` 解析 `yaml` 配置文件内容，然后将读取到的配置文件映射给相对应的结构体，结构体的字段设置需要和配置文件保持一致；
如果存在多个层级的配置文件，比如：
```yaml
a:
   b:
      c: xxx
````
那么相对应的结构体的结构就需要以下：
```go
type a struct {
	b b `yaml:"b"`
}

type b struct {
	c string `yaml: "c"`
}


```

### 环境准备
**示例结构目录**
```
├── configs
│   ├── application.yml
│   └── config.go
├── go.mod
├── go.sum
├── main.go
├── pkg
└── vendor
```

**配置文件**
```yaml
server:
	port: 10070
mongodb:
	database: UsvInfo	
	host: 127.0.0.1	
	port: 7010
mqtt:
	pubAddr: tcp://10.2.103.199:5055	
	subAddr: tcp://10.2.103.199:5054	
	domainID: 175200
```

**安装 gopkg.in/yaml.v2 包**
```bash
// 初始化mod文件，用于管理依赖
$ go mod init ${这里替换成项目名}

// 安装 yaml.v2 包
$ go get gopkg.in/yaml.v2

// 将依赖添加到 vendor文件中
// 如果没有使用 vendor来保存第三库包的这部可以省略
$ go mod vendor

//如果使用了 go mod vendor 命令之后，vendor目录会出现以下第三包文件
└── vendor
    ├── gopkg.in
    │   └── yaml.v2
    │       ├── apic.go
    │       ├── decode.go
    │       ├── emitterc.go
    │       ├── encode.go
    │       ├── LICENSE
    │       ├── LICENSE.libyaml
    │       ├── NOTICE
    │       ├── parserc.go
    │       ├── readerc.go
    │       ├── README.md
    │       ├── resolve.go
    │       ├── scannerc.go
    │       ├── sorter.go
    │       ├── writerc.go
    │       ├── yaml.go
    │       ├── yamlh.go
    │       └── yamlprivateh.go
```

### 创建结构体

这里的结构体需要与配置文件结构保持一致
```go
// config.go文件
// 配置文件结构体
type config struct {
	Server server `yaml:"server"`	
	Mongodb mongodb `yaml:"mongodb"`	
	Mqtt mqtt `yaml:"mqtt"`
}

// 服务配置结构体
type server struct {
	Port int `yaml:"port"`
}

// 数据库连接结构体
type mongodb struct {
	Database string `yaml:"database"`
	Host string `yaml:"host"`
	Port string `yaml:"port"`
}
```

### 读取文件
```go
// config.go文件
import (
	"fmt"
	"io/ioutil"
	"gopkg.in/yaml.v2"
)  

// 声明变量，以供外部引用
var Srver *server
var Mongodb *mongodb
var Mqtt *mqtt

// 读取配置文件的信息
func InitConf() {
	/*
		读取配置文件		
		当前的路径是项目的根路径		
		下面的代码表示从项目的根路径去找configs目录的application.yml文件
	*/	
	yamlFile, err := ioutil.ReadFile("./configs/application.yml")
	if err != nil {
		fmt.Println("读取配置文件失败, -> ", err)
	}
	
	//构建一个配置文件对象
	c := new(config)  
	
	//使用yaml的Unmarshal函数，将文件对象和相对应的配置文件结构体指针传进去
	err = yaml.Unmarshal(yamlFile, &c)
	if err != nil {
		fmt.Println("获取配置文件信息失败 -> ", err)
	}  	
	// 将读取到的文件赋值给全局变量，便于外部引用
	Srver = &c.Server
	Mongodb = &c.Mongodb
	Mqtt = &c.Mqtt
}

//然后将以上函数放到 init 函数中，这样就会自动的获取到配置文件的值
//自动执行init函数
func init() {
	InitConf()
}
```

**main函数**
```go
func main() {
	fmt.Println("读取配置信息文件 mqtt -> \n", configs.Mqtt)	
	fmt.Println("读取配置信息文件 mongodb -> \n", configs.Mongodb)	
	fmt.Println("读取配置信息文件 server -> \n", configs.Srver)
}

// 输出
读取配置信息文件 mqtt - &{tcp://10.2.103.199:5055 tcp://10.2.103.199:5054 175200}
读取配置信息文件 mongodb - &{UsvInfo 127.0.0.1 7010}
读取配置信息文件 server - &{10070}
```

输出不美观，可以在 `config.go` 文件中重写结构体的String函数，可以美化输出
```go
// 重写Server String函数
func (srv server) String() string {
	return fmt.Sprintf("%v", srv.Port)
}
  
// 重写Mongodb String函数
func (mongo mongodb) String() string {	
	return fmt.Sprintf("%v \n %v \n %v", mongo.Database, mongo.Host, mongo.Port)
} 

// 重写Mqtt String函数
func (mqtt mqtt) String() string {
	return fmt.Sprintf("%v \n %v \n %v ", mqtt.PubAddr, mqtt.SubAddr, mqtt.DomainId)
}
```

重写之后的输出
```go
[Running] go run "/var/project/go/usv_platform_go/main.go"
读取配置信息文件 mqtt ->
tcp://10.2.103.199:5055
tcp://10.2.103.199:5054
175200
读取配置信息文件 mongodb ->
UsvInfo
127.0.0.1
7010
读取配置信息文件 server ->
10070 
[Done] exited with code=0 in 0.139 seconds
```

over