2021-12-30
10:14:29
author:陈建浩
#go  #MongoDB 

--- 
在 `Go` 程序连接到 `MongoDB` 数据库可以分为以下步骤：
- 安装MongoDB Go驱动程序 
- 使用Go驱动程序连接到MongoDB 
- 在Go中使用BSON对象 
- 将CRUD操作发送到MongoDB

## 安装MongoDB Go驱动程序
在Go程序项目路径下的终端输入以下命令即可安装 `MongoDB Go` 驱动程序
```sh
$ go get go.mongodb.org/mongo-driver/mongo@v1.8.1
$ go get go.mongodb.org/mongo-driver/bson
$ go get go.mongodb.org/mongo-driver/mongo/options
```

**在使用mongodb的go文件中正确引入包**
```go
import (
    "go.mongodb.org/mongo-driver/bson"
    "go.mongodb.org/mongo-driver/mongo"
    "go.mongodb.org/mongo-driver/mongo/options"
)
```

## 使用 Go 驱动程序连接到 MongoDB

导入MongoDB Go驱动程序后，可以使用`Connect()`函数连接到MongoDB。`Connect()`函数。必须传递上下文和选项`ClientOptions`和 `context`。连接（）。`ClientOptions`用于设置连接字符串。它还可用于配置驱动程序设置，如写关注点、套接字超时等。选项包文档提供了有关哪些客户端选项可用的更多信息。

```go
package main

import (
	"context"
	"fmt"
	"hello/entity"
	"log"
	"go.mongodb.org/mongo-driver/mongo"
	"go.mongodb.org/mongo-driver/mongo/options"

)

func main() {

	clinetOptions := options.Client().ApplyURI("mongodb://localhost:27017")
	client, err := mongo.Connect(context.TODO(), clinetOptions)
	if err != nil {
		log.Fatal(err)
	}
	// Check the connection
	err = client.Ping(context.TODO(), nil)
	if err != nil {
		log.Fatal(err)
	}
	fmt.Println("Connected to MongoDB!")
}
```


**连接指定的库和集合**
连接后，现在可以通过在返回的`client`对象末尾添加以下代码行，在测试数据库中获得培训师集合的句柄：
```go
client.Database("UsvInfo").Collection("modelUsv")
```


**关闭连接**
```go
err = client.Disconnect(context.TODO())

if err != nil {
    log.Fatal(err)
}
```
# 在Go中使用BSON对象 
在开始向数据库发送查询之前，了解Go驱动程序如何处理BSON对象非常重要。MongoDB中的JSON文档存储在称为BSON（二进制编码JSON）的二进制表示形式中。与其他将JSON数据存储为简单字符串和数字的数据库不同，BSON编码扩展了JSON表示形式，以包括其他类型，如int、long、date、浮点和decimal128。这使得应用程序更容易可靠地处理、排序和比较数据。Go驱动程序有两类类型来表示BSON数据：D类型和原始类型。

D类型族用于使用本机Go类型简洁地构建BSON对象。这对于构造传递给MongoDB的命令特别有用。D族由四种类型组成：
-   `D`: BSON文件。这种类型应该在顺序很重要的情况下使用，例如MongoDB命令。
-   `M`: 无序的地图。它与D相同，只是它不保持顺序。
-   `A`: BSON数组。
-   `E`: D中的单个元素。



# 对MongoDB进行CRUD

连接到数据库后，就可以开始添加和操作一些数据了。集合类型有几个方法，允许向数据库发送查询。

## 插入一条文档
创建一个结构体
```go
package entity
type ModelInspection struct {
	Boatid string
	InspectionItem string
	Description string
	InspectionStatus string	
	CreateTime string	
	LastUpdateTime string
}
```

连接到库，声明结构体对象并且初始化
```go
modelUsv := client.Database("UsvInfo").Collection("modelInspection")
inspection := entity.ModelInspection{
	Boatid: "175200",	
	InspectionItem: "检查项1",	
	Description: "这是描述",	
	InspectionStatus: "NODE",	
	CreateTime: "2021 12-30 10:50",	
	LastUpdateTime: "2021 12-30 10:50",
}
```

使用集合对象调用 `insertOne` 函数进行数据插入

```go
insertResult, err := modelUsv.InsertOne(context.TODO(), inspection)
if err != nil {
	log.Fatal(err)
}
fmt.Println("Inserted a single document: ", insertResult.InsertedID)
```
在调用 `insertOne` 函数进行数据插入的时候，要传入上下文对象以及插入对象，这个函数返回2个参数：插入成功后返回的对象环境以及插入失败后的异常信息；
插入成功后的数据对象中 `InsertedID` 字段是代表着插入主键。

## 插入多条文档
插入多条文档与上个方法用法一样，只不过第二个参数使用数组进行传参；
```go
modelUsv := client.Database("UsvInfo").Collection("modelInspection")
inspection := entity.ModelInspection{
	Boatid: "测试船2",
	InspectionItem: "检查项2",	
	Description: "这是描述2",	
	InspectionStatus: "NODE2",	
	CreateTime: "2021 12-30 13:50",	
	LastUpdateTime: "2021 12-30 13:50",
}

inspection2 := entity.ModelInspection{
	Boatid: "测试船3",
	InspectionItem: "检查项3",
	Description: "这是描述3",
	InspectionStatus: "NODE3",
	CreateTime: "2021 12-30 13:51",
	LastUpdateTime: "2021 12-30 13:51",
}

objectArray := []interface{}{inspection, inspection2}  
insertResult, err := modelUsv.InsertMany(context.TODO(), objectArray)
if err != nil {
	log.Fatal(err)
}
  
fmt.Println("Inserted a single document: ", insertResult.InsertedIDs)
```

## 更新文档
UpdateOne（）方法允许您新单个文档。它需要一个筛选文档来匹配数据库中的文档，并需要一个更新文档来描述更新操作。可以使用bson构建匹配数据的文档。




## 查询文档
要查找文档，需要一个筛选文档以及一个指向可将结果解码的指针。要查找单个文档，请使用FindOne()方法。此方法返回可解码为值的单个结果。使用在更新查询中使用的相同筛选器变量来匹配文档。


```go
// 查询数据

func SelectOne() {
	// 声明一个筛选文档条件，这个条件使用 bson.D来实现
	filter := bson.D{{"boatid", "测试船5"}}
	
	// 获取客户端实例
	connect := dataConnect("UsvInfo", "modelInspection")  
	
	// 声明结果对象
	var result entity.ModelInspection
	
	// 查询，参数分别是 go上下文对象、筛选文档条件
	// FindOne函数会返回*SingleResult指针，该指针需要调用Decode函数进行解码，Decode函数需要传入一个结果指针
	// 如果查询成功就会将结果赋值给结果指针
	err := connect.FindOne(context.TODO(), filter).Decode(&result)  
	
	if err != nil {
		log.Fatal("发生错误，抛出异常", err.Error())
	}  
	fmt.Printf("查询出的单个文档结果: %+v\n", result)

}
```


如果是查询多个文档，那就使用 `Find()` 函数，将第二个参数（过滤器）使用一个空的bson文档替代即可：
```go
// 查询多个数据
func SelectMary() { 

	// 获取客户端实例
	connect := dataConnect("UsvInfo", "modelInspection") 
	
	// 设置查询选项，这里选择对boatid进行降序排序（1代表升序排序），并且只显示2条数据
	// 还可以设置跳过数据 SetSkip(int)，表示跳过多少条数据显示
	option := options.Find().SetSort(bson.D{{"boatid", -1}}).SetLimit(2)  
	
	// 第一个参数表示go的上下文对象；
	// 第二个参数表示过滤器，使用bson文档表示，这里为空表示查询所有数据，如果精确查询或者模糊查询都在这个参数里面指定；
	// 第三个数据表示查询选项。
	coll, err := connect.Find(context.TODO(), bson.D{}, option)  
	
	if err != nil {
		log.Fatalln("发生异常", err.Error())
	
	}	  
	
	// 定义一个接受数据的数组
	var results []entity.ModelInspection  
	
	// All函数将数据赋值给 第二个参数数组里
	if err = coll.All(context.TODO(), &results); err != nil {
		log.Fatal(err)
	}	  
	
	// 遍历这个数组拿到数据
	for _, result := range results {
		fmt.Println(result)
	}
}
```


# 参考资料
[go语言操作mongodb](https://juejin.cn/post/6908063164726771719#heading-26)
[Go中的MongoDB](https://juejin.cn/post/7000157993971122189#heading-13)
[GO——mongo实现模糊查询](https://blog.csdn.net/WU2629409421perfect/article/details/109127291)
[go操作mongo官网](https://docs.mongodb.com/drivers/go/current/usage-examples/find/)