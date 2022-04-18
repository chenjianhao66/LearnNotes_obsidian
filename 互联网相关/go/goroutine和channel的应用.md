#go

# goroutine协程
  
Go 语言支持并发，我们只需要通过 go 关键字来开启 goroutine 即可。

虽然，线程池为逻辑编写者提供了线程分配的抽象机制。但是，如果面对随时随地可能发生的并发和线程处理需求，线程池就不是非常直观和方便了。能否有一种机制：使用者分配足够多的任务，系统能自动帮助使用者把任务分配到 CPU 上，让这些任务尽量并发运作。这种机制在 Go语言中被称为 **goroutine**。

**goroutine** 是轻量级线程，goroutine 的调度是由 Golang 运行时进行管理的；Go 程序会智能地将 goroutine 中的任务合理地分配给每个 CPU。

Go 程序从 main 包的 main() 函数开始，在程序启动时，Go 程序就会为 main() 函数创建一个默认的 goroutine。

## 从函数中创建goroutine
Go 程序中使用 **go** 关键字为一个函数创建一个 goroutine。一个函数可以被创建多个 goroutine，一个 goroutine 必定对应一个函数。

**格式** 
为一个普通函数创建 goroutine 的写法如下：
```go
go 函数名( 参数列表 )
```
- 函数名：要调用的函数名。
- 参数列表：调用函数需要传入的参数。

使用 go 关键字创建 goroutine 时，被调用函数的返回值会被忽略。  
  
如果需要在 goroutine 中返回数据，请使用通道（channel）特性，通过通道把数据从 goroutine 中作为返回值传出。


# channel类型
如果说 goroutine 是 Go语言程序的并发体的话，那么 _channel_ 就是它们之间的通信机制。一个 _channel_ 是一个通信机制，它可以让一个 goroutine 通过它给另一个 goroutine 发送值信息。

每个 _channel_ 都有一个特殊的类型，也就是 _channel_ 可发送数据的类型。一个可以发送 int 类型数据的 _channel_ 一般写为 chan int。

**channel** 是一个数据管道，是 goroutine 之间数据通信桥梁，是线程安全的。_channel_ 分为有缓冲和无缓冲两种类型，其实无缓冲类型可以理解为有缓冲的一种特殊情况。如下图所示：


![](https://images-1306554305.cos.ap-guangzhou.myqcloud.com/1-1PQG035203K.jpg)



## 通道类型
通道（channel）是用来传递数据的一个数据结构。

通道可用于两个 goroutine 之间通过传递一个指定类型的值来同步运行和通讯。操作符 **`<-`** 用于指定通道的方向，发送或接收。**如果未指定方向，则为双向通道。**

这里分2种通道，一种是无缓冲区的通道，一种是有缓冲区的同代

通道是引用类型，需要使用 make 进行创建，格式如下：
```go
通道实例 := make(chan 数据类型)  // 无缓冲区
通道实例 := make(chan 数据类型，buffSize int) // 有缓冲区
```
-   数据类型：通道内传输的元素类型。
-   通道实例：通过make创建的通道句柄。
-   缓冲区大小：指定缓冲区的大小

## 通过通道发送数据

通道创建后，就可以使用通道进行发送和接收操作。

### 1. 通道发送数据的格式
通道的发送使用特殊的操作符`<-`，将数据通过通道发送的格式为：
```go
通道变量 <- 值
```

- 通道变量：通过make创建好的通道实例。
- 值：可以是变量、常量、表达式或者函数返回值等。值的类型必须与ch通道的元素类型一致。


### 2. 通过通道发送数据例子
格式：
```go
1.  // 创建一个空接口通道
2.  ch := make(chan interface{})
3.  // 将0放入通道中
4.  ch <- 0
5.  // 将hello字符串放入通道中
6.  ch <- "hello"
```

把数据往通道中发送时，如果接收方一直都没有接收，那么发送操作将持续阻塞。
_如果该通道是有缓冲区的，那么发送接收操作都不会阻塞，但这个不会阻塞是有条件的；
只有缓冲区满的时候发送操作才会阻塞，接收操作只有缓冲区空着的时候阻塞。_


## 通过通道接收数据
通道的数据接收一共有以下 4 种写法。

**阻塞的接收数据**
阻塞模式接收数据时，将接收变量作为`<-`操作符的左值，格式如下：
```go
data := <-ch
```
执行该语句时将会阻塞，直到接收到数据并赋值给 data 变量。


**非阻塞的接收数据** 
从通道接收数据的时候，多接收一个bool变量，用该变量判断数据是否接收成功，是不会阻塞的
```go
data, ok := <-ch
```
-   data：表示接收到的数据。未接收到数据时，data 为通道类型的零值。
-   ok：表示是否接收到数据。


**接收任意数据，忽略接收的数据**
阻塞接收数据后，忽略从通道返回的数据，格式如下：

```go
<-ch
```

执行该语句时将会发生阻塞，直到接收到数据，但接收到的数据会被忽略。_这个方式实际上只是通过通道在 goroutine 间阻塞收发实现并发同步。_


**循环接收**
通道的数据接收可以借用 for range 语句进行多个元素的接收操作，格式如下：
```go
for data := range ch {
	// do someting
}
```

通道 _ch_ 是可以进行遍历的，遍历的结果就是接收到的数据。数据类型就是通道的数据类型。通过 for 遍历获得的变量只有一个，即上面例子中的 data。

以上就是channel的用法，go程序中使用最多的是通过channel + goroutine + select 组合起来完成并发变成。

# select语句

select 是Go中的一个控制结构，类似于用于通信的switch语句。每个case必须是一个通信操作，必须是一个channel操作，要么是发送要么是接收。 select 随机执行一个可运行的case。如果没有case可运行，它将阻塞，直到有case可运行。

select中的default子句总是可运行的。

如果有多个case都可以运行，select会随机公平地选出一个执行，其他不会执行。

如果没有可运行的case语句，且有default语句，那么就会执行default的动作。

如果没有可运行的case语句，且没有default语句，select将阻塞，直到某个case通信可以运行

语法如下：
```go
select {
    case communication clause  :
       statement(s);      
    case communication clause  :
       statement(s);
    /* 你可以定义任意数量的 case */
    default : /* 可选 */
       statement(s);
}
```

示例示例：
```
package main

import "fmt"

func main() {
   var c1, c2, c3 chan int
   var i1, i2 int
   select {
      case i1 = <-c1:
         fmt.Printf("received ", i1, " from c1\n")
      case c2 <- i2:
         fmt.Printf("sent ", i2, " to c2\n")
      case i3, ok := (<-c3):  // same as: i3, ok := <-c3
         if ok {
            fmt.Printf("received ", i3, " from c3\n")
         } else {
            fmt.Printf("c3 is closed\n")
         }
      default:
         fmt.Printf("no communication\n")
   }    
}
```

意思以上例子只会执行default语句，因为该select语句里所有case的channel都没有执行make初始化，所以无法接收和发送数据，同一时间又有default分支，所以只能执行default语句。