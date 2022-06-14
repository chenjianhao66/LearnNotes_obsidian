### 背景

自从工作以来，在编码过程中用到的中间件组件比较少，Redis也是一直想学的缓存数据库；但是因为个人原因一直没有学，在2022年初的时候学习go，并发现go语言相对于其他语言更轻量，编写代码也更方便，性能更加强大。

在决定深入学习go的时候，需要一个项目来深入go的各种语法以及代码风格，摒弃掉Java的编码风格。有了2个需求，在逛 `HelloGithub` 网站Go语言项目专题时，看到了这个项目，就决定fork项目下来阅读源码，在学习 `Redis` 相关知识时也深入学习Go的进阶知识。



### Godis简介

```
Godis 是一个用 Go 语言实现的 Redis 服务器。本项目旨在为尝试使用 Go 语言开发高并发中间件的朋友提供一些参考。

关键功能:
- 支持 string, list, hash, set, sorted set, bitmap 数据结构
- 自动过期功能(TTL)
- 发布订阅
- 地理位置
- AOF 持久化及 AOF 重写
- 加载和导出 RDB 文件
- Multi 命令开启的事务具有`原子性`和`隔离性`. 若在执行过程中遇到错误, godis 会回滚已执行的命令
- 内置集群模式. 集群对客户端是透明的, 您可以像使用单机版 redis 一样使用 godis 集群
  - `MSET`, `MSETNX`, `DEL`, `Rename`, `RenameNX`  命令在集群模式下原子性执行, 允许 key 在集群的不同节点上
  - Multi 命令开启的事务在集群模式下支持在同一个 slot 内执行
- 并行引擎, 无需担心您的操作会阻塞整个服务器.
```

> 摘抄自 `Godis` 的 `README_CN.md` 文件

在阅读一个项目源码时（其实不只是源码，可以是现实中任意一个想要学习的目标事物），需要确认自己对于这个事物所了解的点、通过这个事物自己想知道一些什么、通过什么方式去学习这个事物以及最后自己学到了什么；

针对以上的点，可以使用 `KWHL` 图表来辅助学习
![](https://images-1306554305.cos.ap-guangzhou.myqcloud.com/202206132057355.png)

在思维导图的 `W(我想知道些什么)` 一栏中的知识点是我在看到 `Godis` 这个项目后所得的疑问以及想要的知识点中，可以根据知识点的特性分类，分别是 **TCP服务**、**数据结构以及命令实现**、**集群实现以及AOF持久化**；

下面我就从main函数开始阅读源代码。


### 运行代码
将  [`Godis`](https://github.com/HDT3213/godis) 项目代码通过 `git` 克隆到本地

**下载依赖**

进入到克隆的代码目录中执行以下命令：
```bash
go mod tidy
```


**运行main函数**

在下载完依赖之后直接运行main函数即可：
```bash
go run main.go
```
运行之后即可通过 `redis-cli` 工具连接到 `godis`

### 阅读源码

#### main函数
无论什么项目都是从 main 函数开始，所以先来看 godis 项目的 main 函数：
```go
// 默认配置  
var defaultProperties = &config.ServerProperties{  
   Bind:           "0.0.0.0",  
   Port:           6399,  
   AppendOnly:     false,  
   AppendFilename: "",  
   MaxClients:     1000,  
}  
  
// 检查文件是否存在并且该文件是否是目录  
func fileExists(filename string) bool {  
   info, err := os.Stat(filename)  
   return err == nil && !info.IsDir()  
}
func main() {  
   print(banner)  
   logger.Setup(&logger.Settings{  
      Path:       "logs",  
      Name:       "godis",  
      Ext:        "log",  
      TimeFormat: "2006-01-02",  
   })  
   configFilename := os.Getenv("CONFIG")  
   // 查看环境变量 CONFIG 是否存在，存在的直接使用环境变量所指向的配置文件地址  
   //  
   // 如果不存在，先检查根目录下是否存在 redis.conf 配置文件，存在则使用，不存在则使用默认的配置文件，默认的配置文件见 21行  
   if configFilename == "" {  
      if fileExists("redis.conf") {  
         config.SetupConfig("redis.conf")  
      } else {  
         config.Properties = defaultProperties  
      }  
   } else {  
      config.SetupConfig(configFilename)  
   }  
  
   // 构建tcp包的配置文件对象，地址是配置文件的地址和端口的字符串拼接，传入Handler接口实例  
   err := tcp.ListenAndServeWithSignal(&tcp.Config{  
      Address: fmt.Sprintf("%s:%d", config.Properties.Bind, config.Properties.Port),  
   }, RedisServer.MakeHandler())  
   if err != nil {  
      logger.Error(err)  
   }  
}
```

首先在控制台打印 banner 图，该图是 godis 的logo；紧接着是对 logger 的初始化配置，调用 logger 包的 Setup 函数，将一个 Settings 结构体作为入参传进去，字段属性；

##### 日志设置

Setup函数所在位置是 `godis/lib/logger/logger.go` 文件， Setup 函数总体作用如下：

1. 拼接 log 文件名（ gods-2022-06-4.log ）
2. 根据该 log 文件名检查该文件是否存在，不存在的话就创建；存在的话判断有没有权限，没有权限就返回错误；
3. 然后是对 logger 对象进行设置，包括了日志输出流、log 信息前缀以及日志分割条件

##### 加载配置文件

先通过 os 库的 Getenv 函数获取指定 key 的环境变量并赋值给 configFilename ，判断 configFielname 对象是否为空字符串

- 如果是空字符串，判断当前当前目录是否存在 redis.conf 配置文件
  - 如果存在，则调用 config 包的 SetupConfig 函数去读取配置文件，根据读取文件以及反射去获取配置文件的内容并赋值给 config 包的包级变量 Properties；具体逻辑是在 `godis/config/config.go`
  - 如果不存在则是将 defaultProperties 变量赋值给 config 包的包级变量 Properties
- 如果不为空字符串，则代表这着环境变量中获取到了值，该值作为 SetupConfig 函数的入参传入函数。

至此，main 函数的预先加载部分就结束了，下面是 TCP 服务部分。

#### TCP服务端部分

##### 服务优雅关闭实现以及TCP服务实现
来看看 godis 是如何实现 TCP 服务的优雅关闭以及连接监听功能的。
**godis** 项目是通过标准库 **net/http** 库来实现 TCP 服务器的，具体代码：

```go
   // 构建tcp包的配置文件对象，地址是配置文件的地址和端口的字符串拼接，传入Handler接口实例  
   err := tcp.ListenAndServeWithSignal(&tcp.Config{  
      Address: fmt.Sprintf("%s:%d", config.Properties.Bind, config.Properties.Port),  
   }, RedisServer.MakeHandler())  
   if err != nil {  
      logger.Error(err)  
   }  
```

调用 tcp 包的ListenAndServeWithSignal 函数，该函数会需要传入一个 Config 对象结构以及一个 Handelr 接口实例对象；Config 对象的 Address 字段使用了 字符串拼接，将配置文件里的地址以及端口以 `bind:port` 的格式赋值
然后 main 函数就结束了，具体逻辑以及阻塞操作都在该函数里面，进这个函数看具体逻辑。


ListenAndServeWithSignal 函数位于 `godis/tco/server.go` 文件内，具体逻辑如下：
```go 
// ListenAndServeWithSignal binds port and handle requests, blocking until receive stop signal
func ListenAndServeWithSignal(cfg *Config, handler tcp.Handler) error {
   // 声明关闭通知通道、系统信号量的通道  
   closeChan := make(chan struct{})  
   sigCh := make(chan os.Signal)  
   // 监听并捕获 sighup（挂起）、sigquit(退出)、sigterm（终止）、sigint（终端）这些信号量，并将这些信号量写入到sigCh管道中  
   signal.Notify(sigCh, syscall.SIGHUP, syscall.SIGQUIT, syscall.SIGTERM, syscall.SIGINT)  
  
   // 起goroutine来监听 sigCh管道的数据，有数据的就代表要退出程序了，往 closeChan 管道内发送数据  
   go func() {  
      sig := <-sigCh  
      switch sig {  
      case syscall.SIGHUP, syscall.SIGQUIT, syscall.SIGTERM, syscall.SIGINT:  
         closeChan <- struct{}{}  
      }  
   }()  
   listener, err := net.Listen("tcp", cfg.Address)  
   if err != nil {  
      return err  
   }  
   //cfg.Address = listener.Addr().String()  
   logger.Info(fmt.Sprintf("bind: %s, start listening...", cfg.Address))  
   ListenAndServe(listener, handler, closeChan)  
   return nil  
}
```

该函数主要做了以下事情：
创建了 2 个无缓冲管道，管道内传递的数据类型分别是 struct{} 和 os.Signal ，分别是关闭通道以及监听系统信号量的通道；调用 os/signal 包的 Notify 函数，该函数需要传入一个类型为 os.Signal 的管道以及 []os.Signal切片，该函数会监听 Signal切片里的信号量事件，如果有切片内的信号量事件发生，那么就会将这些信号量时间发送到管道内。

在代码中，传入了 syscall.SIGHUP, syscall.SIGQUIT, syscall.SIGTERM, syscall.SIGINT 信号量，这些信号量分别代表着 挂起、退出、终止、中断事件，在系统监听到有这些事件发生，就发送到管道内。

之后就另起一个协程，监听信号量管道，因为是无缓冲管道，所以会阻塞监听，一有数据之后就通过 switch 语句来判断其类型，因为调用了 Notify 函数，所以传入到该管道的数据类型必定是这几种，所以会命中这个 case ，将一个空结构体对象发送到 closeChan 管道中。

后面通过 net 包的 Listen 函数返回一个 listener 对象，然后将该对象通过入参的方式调用 ListenAndServe 函数。


来看一下 ListenAndServe 函数

```go
  
// ListenAndServe binds port and handle requests, blocking until close//  
// 绑定端口并且处理请求，阻塞处理请求  
func ListenAndServe(listener net.Listener, handler tcp.Handler, closeChan <-chan struct{}) {  
   // listen signal  
   go func() {  
      <-closeChan  
      logger.Info("shutting down...")  
      _ = listener.Close() // listener.Accept() will return err immediately  
      _ = handler.Close()  // close connections  
   }()  
  
   // listen port  
   defer func() {  
      // close during unexpected error  
      _ = listener.Close()  
      _ = handler.Close()  
   }()  
   ctx := context.Background()  
   var waitDone sync.WaitGroup  
  
   // 起死循环来处理tcp的连接，每来一个连接就起一个goroutine来处理  
   // 具体的处理逻辑则是 handler接口实例的 Handle方法  
   for {  
      conn, err := listener.Accept()  
      if err != nil {  
         break  
      }  
      // handle  
      logger.Info("accept link")  
      waitDone.Add(1)  
      go func() {  
         defer func() {  
            waitDone.Done()  
         }()  
         handler.Handle(ctx, conn)  
      }()  
   }  
   waitDone.Wait()  
}
```

ListenAndServe 函数与 ListenAndServeWithSignal 函数同在一个文件内，该函数一上来就起了个协程来监听 closeChan 管道，只有这个管道有数据，那么就代表着触发了中断、挂起等信号量，该协程就继续往下执行代码，下面的代码则是对 listener 和 handler 对象执行 close 方法，即关闭 tcp 服务。
> 注意，该关闭只是代表着不再接收新的 tcp 连接，当前正在处理的连接会继续执行逻辑，原因请看最后一行代码的 waitDone.wait() 方法，这样就是叫做  ***服务的优雅关闭***

listener.Accept()  会阻塞等待 tcp 客户端连接，有客户端连接之后则返回 Conn 接口实例对象，调用 waitDown 对象的 Add 方法，起协程将 Conn 实例传入到 handler 接口实例的 Handle 方法。
> 那这个 handler 是个接口，那最终执行该方法的是哪个实例呢？

其实这个在 main 函数的时候就已经看出来了，在 main 函数调用 ListenAndServerWithSignal 函数时就传入了具体值，代码：

```go
// 构建tcp包的配置文件对象，地址是配置文件的地址和端口的字符串拼接，传入Handler接口实例  
err := tcp.ListenAndServeWithSignal(&tcp.Config{  
   Address: fmt.Sprintf("%s:%d", config.Properties.Bind, config.Properties.Port),  
}, RedisServer.MakeHandler())
```


##### 协议解析
RedisServer 包的 MakeHandler 函数就返回一个实现了 Handler 接口的结构体，该结构体以及函数所在位置是：`godis/reids/server/server.go`；通过阅读代码，该函数大体上干了以下这几件事：
1. 通过 net.Conn 对象创建一个结构体对象 Connection
2. 调用 parser 包的 ParseStream函数，将 Connection 作为入参传入，该函数会根据协议去解析数据，解析成功后会将数据发送到一个管道内，向外返回这个管道，该管道会返回客户端通过 TCP 连接所传回的数据（命令）；
3. 通过管道接收客户端所输入的命令，将命令传入 Handler 对象的 DB字段 的 Exec 方法执行，具体的命令执行逻辑就在该方法内。

> 那客户端和服务端根据根据什么协议去通信呢？
> godis 采用的是 redis 的协议，也称作  **RESP** (REdis Serialization Protocol) 协议 ，具体协议格式请看[官网](https://redis.com.cn/topics/protocol.html)
> 也正是因为通信协议用的是 RESP 协议，所以通过 **redis-cli** 命令行工具也能连接到 godis

具体代码如下：
```go
// Handle receives and executes redis commands
func (h *Handler) Handle(ctx context.Context, conn net.Conn) {  
   // 如果当前的Handler对象已经close的话，则直接关闭掉该连接并退出  
   // 那什么情况会出现closing字段会被设置呢？  
   // TODO 需要找出closing字段被设置的代码  
   if h.closing.Get() {  
      // closing handler refuse new connection  
      _ = conn.Close()  
      return  
   }  
  
   // 传入net包的连接对象，返回connection包的连接对象  
   client := connection.NewConn(conn)  
   // 连接成功之后对 activeConn 字段更新，表示目前激活的连接数量
   h.activeConn.Store(client, 1)  
  
   // 根据conn连接对象获取一个只读消息的通道，该通道会返回 Payload 类型的数据  
   ch := parser.ParseStream(conn)  
   for payload := range ch {  
      if payload.Err != nil {
      // 从管道中读取数据的错误判断  
         if payload.Err == io.EOF ||  
            payload.Err == io.ErrUnexpectedEOF ||  
            strings.Contains(payload.Err.Error(), "use of closed network connection") {  
            // connection closed  
            h.closeClient(client)  
            logger.Info("connection closed: " + client.RemoteAddr().String())  
            return  
         }  
         // protocol err  
         errReply := protocol.MakeErrReply(payload.Err.Error())  
         err := client.Write(errReply.ToBytes())  
         if err != nil {  
            h.closeClient(client)  
            logger.Info("connection closed: " + client.RemoteAddr().String())  
            return  
         }  
         continue  
      }  
      // 通过数据校验，走到这里的都是正常可执行的命令
      fmt.Printf("从管道中读取到数据 --> \n%s \n", payload.Data.ToBytes())  
      if payload.Data == nil {  
         logger.Error("empty payload")  
         continue  
      }  
      // 类型断言，将Data字段数据进行类型转换判断，看是否能转换成 protocol 包的 MultiBulkReply 类型
      r, ok := payload.Data.(*protocol.MultiBulkReply)  
  
      if !ok {  
         logger.Error("require multi bulk protocol")  
         continue  
      }  
      // 类型转换成功
      // 解析协议，并把命令赋值给Args，最后执行Args的命令  
      result := h.db.Exec(client, r.Args)  
      if result != nil {  
         _ = client.Write(result.ToBytes())  
      } else {  
         _ = client.Write(unknownErrReplyBytes)  
      }  
   }  
}

```

在协议解析这一部分，最重要的代码就是 parser 包的 ParseStream 方法，该函数所在位置是 `godis/redis/parser/parser.go` ,具体代码如下：
```go
// ParseStream reads data from io.Reader and send payloads through channel
func ParseStream(reader io.Reader) <-chan *Payload {  
   ch := make(chan *Payload)  
   go parse0(reader, ch)  
   return ch  
}
```

代码也很简单，创建一个数据类型为 Payload 指针类型的管道，起一个协程执行 parse0 函数，将 conn 连接对象和该管道作为入参，然后返回该管道。从这一部分就可以看出，既然外面是从该管道读取数据，那么往该管道写入数据的就是 parse0 函数。该函数与 ParseStream 函数同属一个文件

> 在进行协议解析时，得先确认 RESP 协议会通过 TCP 协议发送什么数据过来
> 举例， 在 redis-cli 执行命令： get name
> 在通过 RESP 协议解析后发送给服务端，服务端在接收到该消息后的格式如下：
> *2
$3
get
$4
name
>>
> 符号解析
>  \* 之后跟的数据代表着参数的数量
>  % 之后跟的下一行命令的字符数量
> 

那么在解析数据时，我们需要处理的数据有 3 种，分别是 `*` 、`$` 以及 `命令字符串`；总体流程如下：客户端在输入一条命令时会解析成上面的数据格式，那么我们就可以每次读一行数据，使用 readingMultiLine 字段来表明当前行数据是 `*`、`$` 或者是普通字符串命令，readingMultiLine 为 false 时代表是 `*`、`$`，为 true 时代表是普通字符串命令。根据 readingMultiLine 的取值去做不同的逻辑处理。

当 readingMultiLine 为 true 时，还要判断是 `*` 还是 `$` ，这两个字符都要有不同的逻辑。


在解析数据时，使用 readState 结构体来控制解析流程，该结构体的定义如下：
```go
type readState struct {  
   readingMultiLine  bool  // 判断当前命令是 * $ 还是命令
   expectedArgsCount int   // 命令的行数
   msgType           byte  // 消息类型
   args              [][]byte  // 存放命令的比特数组
   bulkLen           int64     // 取值为 0 和 不为0 ，为 0 时代表着当前读的数据是 $ 和 *； 不为 0 时则代表着当前命令需要读的字符长度
}
```

parse0 函数具体代码如下：
```go
  
func parse0(reader io.Reader, ch chan<- *Payload) {  

   bufReader := bufio.NewReader(reader)  
   var state readState  
   var err error  
   var msg []byte  
   for {  
      // read line  
      var ioErr bool  
      // 根据bufReader读数据，根据bulkLen的值来确认是读一行数据还是读 (bulkLen值+2) 长度的数据  
      msg, ioErr, err = readLine(bufReader, &state)  
      // 从reader里读取数据失败  
      if err != nil {  
         // 判断是否是读取失败，是读取失败则关闭管道  
         if ioErr { // encounter io err, stop read  
            ch <- &Payload{  
               Err: err,  
            }  
            close(ch)  
            return  
         }  
         // protocol err, reset read state  
         // 是读取的msg切片最后1个或者1个2个字符不是 \r和\n  
         ch <- &Payload{  
            Err: err,  
         }  
         state = readState{}  
         continue  
      }  
      fmt.Printf("从conn连接中获取到的数据 -> %v \n", string(msg))  
  
      // parse line  
      // 初始值是 false      // 判断是否要读取多行，这里是读单行的逻辑  
      if !state.readingMultiLine {  
         // receive new response  
         if msg[0] == '*' {  
            // multi bulk protocol  
            err = parseMultiBulkHeader(msg, &state)  
            if err != nil {  
               ch <- &Payload{  
                  Err: errors.New("protocol error: " + string(msg)),  
               }  
               state = readState{} // reset state  
               continue  
            }  
            if state.expectedArgsCount == 0 {  
               ch <- &Payload{  
                  Data: &protocol.EmptyMultiBulkReply{},  
               }  
               state = readState{} // reset state  
               continue  
            }  
         } else if msg[0] == '$' { // bulk protocol  
            err = parseBulkHeader(msg, &state)  
            if err != nil {  
               ch <- &Payload{  
                  Err: errors.New("protocol error: " + string(msg)),  
               }  
               state = readState{} // reset state  
               continue  
            }  
            if state.bulkLen == -1 { // null bulk protocol  
               ch <- &Payload{  
                  Data: &protocol.NullBulkReply{},  
               }  
               state = readState{} // reset state  
               continue  
            }  
         } else {  
            // single line protocol  
            result, err := parseSingleLineReply(msg)  
            ch <- &Payload{  
               Data: result,  
               Err:  err,  
            }  
            state = readState{} // reset state  
            continue  
         }  
      } else {  
         // 读取多行  
         // receive following bulk protocol  
         err = readBody(msg, &state)  
         if err != nil {  
            ch <- &Payload{  
               Err: errors.New("protocol error: " + string(msg)),  
            }  
            state = readState{} // reset state  
            continue  
         }  
         // if sending finished  
         if state.finished() {  
            var result redis.Reply  
            if state.msgType == '*' {  
               result = protocol.MakeMultiBulkReply(state.args)  
            } else if state.msgType == '$' {  
               result = protocol.MakeBulkReply(state.args[0])  
            }  
            ch <- &Payload{  
               Data: result,  
               Err:  err,  
            }  
            state = readState{}  
         }  
      }  
   }  
}
```

这个函数的逻辑如下：

通过调用 bufio 包的 NewReader 函数，将 conn 对象作为入参，返会一个  Reader 类型对象 bufReader ，然后调用 readLine 函数；

readLine 函数会根据 state 对象的 bulken 字段是否等于 0 去判断：
- 如果为 0 则代表当前读取的数据是 `*` 和 `$` 开头的；需要从 bufReader 中一直读取数据，直至读取到 `\n` 字符，连同 `\n` 字符一同读取并赋值给 msg 比特切片返回；
- 如果不为 0 则代表着当前命令需要读的字符长度，算上 `\r\n` ，在 bulken 字段加上 2 ，则是当前命令的长度，读取并赋值给 msg 切片返回。

下面是 readLine 函数的具体代码：
```go
func readLine(bufReader *bufio.Reader, state *readState) ([]byte, bool, error) {  
   var msg []byte  
   var err error  
   if state.bulkLen == 0 { // read normal line  
      msg, err = bufReader.ReadBytes('\n')  
      if err != nil {  
         return nil, true, err  
      }  
      // 如果读取到的消息长度为0，或者msg切片的倒数第二个位置不是 \r 时则进入if语句  
      if len(msg) == 0 || msg[len(msg)-2] != '\r' {  
         return nil, false, errors.New("protocol error: " + string(msg))  
      }  
   } else { // read bulk line (binary safe)  
      msg = make([]byte, state.bulkLen+2)  
      _, err = io.ReadFull(bufReader, msg)  
      if err != nil {  
         return nil, true, err  
      }  
      if len(msg) == 0 ||  
         msg[len(msg)-2] != '\r' ||  
         msg[len(msg)-1] != '\n' {  
         return nil, false, errors.New("protocol error: " + string(msg))  
      }  
      state.bulkLen = 0  
   }  
   return msg, false, nil  
}
```

在执行完 readLine 函数之后就要根据 state 对象的 **readingMultiLine** 字段判断：
- 如果该字段为 false ，则代表当前命令是 `*` 或者 `$`，然后在根据 msg[0] 的字符类型判断，`*` 执行 parseMultiBulkHeader 函数，`$` 执行 parseBulkHeader 函数；
- 如果该字段为 true ，代表着当前命令是普通命令字符串，执行 readBody 函数，该函数会将命令行参数添加到 state 对象的 args字段。

下面是 parseMultiBulkHeader 函数、 parseBulkHeader函数以及 readBody 函数的具体代码：
```go
// 当读取到的数据是 * 时执行的函数
func parseMultiBulkHeader(msg []byte, state *readState) error {  
   var err error  
   var expectedLine uint64  // 预期的行数  
   expectedLine, err = strconv.ParseUint(string(msg[1:len(msg)-2]), 10, 32)   // 将msg切片去头和倒数2位byte（\r\n），并转成int32赋值给expectedLine  
   if err != nil {  
      return errors.New("protocol error: " + string(msg))  
   }  
   if expectedLine == 0 {  
      state.expectedArgsCount = 0  
      return nil  
   } else if expectedLine > 0 {  
      // first line of multi bulk protocol  
      state.msgType = msg[0]  
      state.readingMultiLine = true  // 将 readingMultLine 字段置true，代表下次读取的命令是普通字符串
      state.expectedArgsCount = int(expectedLine)  // 代表着客户端输入的命令 key 的长度，比如 set name ，长度则为2
      state.args = make([][]byte, 0, expectedLine) 
      return nil  
   } else {  
      return errors.New("protocol error: " + string(msg))  
   }  
}  
// 当读取到的数据是 $ 时执行的函数
func parseBulkHeader(msg []byte, state *readState) error {  
   var err error  
   // 将切片切割，去头和倒数2位,返回int64类型的数值并赋值给state的bulkLen字段  
   // 代表着下次读取命令的字符串长度
   state.bulkLen, err = strconv.ParseInt(string(msg[1:len(msg)-2]), 10, 64)  
   if err != nil {  
      return errors.New("protocol error: " + string(msg))  
   }  
   if state.bulkLen == -1 { // null bulk  
      return nil  
   } else if state.bulkLen > 0 {  
      state.msgType = msg[0]  
      state.readingMultiLine = true  // 将 readingMultLine 字段置true，代表下次读取的命令是普通字符串
      state.expectedArgsCount = 1  
      state.args = make([][]byte, 0, 1)  
      return nil  
   } else {  
      return errors.New("protocol error: " + string(msg))  
   }  
}

// 当读取到的数据是普通字符串时
func readBody(msg []byte, state *readState) error {  
   line := msg[0 : len(msg)-2]  
   var err error  
   if line[0] == '$' {  
      // bulk protocol  
      state.bulkLen, err = strconv.ParseInt(string(line[1:]), 10, 64)  
      if err != nil {  
         return errors.New("protocol error: " + string(msg))  
      }  
      if state.bulkLen <= 0 { // null bulk in multi bulks  
         state.args = append(state.args, []byte{})  
         state.bulkLen = 0  
      }  
   } else {  
      state.args = append(state.args, line)  // 将命令字符串添加到 state 对象的 args 字段中
   }  
   return nil  
}
```

在读取命令字符串时都要进行调用 finished 函数判断，判断当前的命令是否已经是最后一个命令了，下面是 finished 函数的具体代码：
```go
func (s *readState) finished() bool {  
   return s.expectedArgsCount > 0 && len(s.args) == s.expectedArgsCount  
}
```
expectedArgsCount 字段会在 parseMultiBulkHeader 函数进行设置，代表着客户端所输入命令的个数；如果 expectedArgsCount >0 并且 args 切片长度等于 expectedArgsCount 字段的话，则代表命令以及全部读取完成。
> args 字段会在 readBody 函数中进行追加

如果 finished 函数返回的值是 true ，则代表着已经是最后一个命令了