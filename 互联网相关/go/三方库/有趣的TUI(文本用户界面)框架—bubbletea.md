## 简介
一个强大的 TUI（文本用户界面）框架。Bubble Tea 非常适合构建复杂交互的终端应用程序，同时还能让命令行程序变得多彩和炫酷
bubble tea非常适合简单和复杂的终端应用，无论是在线、全窗口还是两者的混合。

## 快速使用
安装 `bubbletea` 库
```bash
go get -u github.com/charmbracelet/bubbletea
```

使用 `bubbletea` 的流程如下：

创建一个结构体类型（也可以是基础类型的类型别名），实现 `Model` 接口，接口定义如下：
```go
type Model interface {  
    Init() Cmd  
  
	Update(Msg) (Model, Cmd)  
  
	View() string  
}
```


编写代码，倒数5秒的例子
```go
package main  

import (  
   "fmt"  
   "log"   "os"   "time"  
   tea "github.com/charmbracelet/bubbletea"  
)

// 定义一个类型
// 该类型实现 Model接口
type model int

// 该模型会返回一个 tick函数，tick函数会返回一个Msg类型变量
// Msg类型变量是interface{}的类型别名，所以返回什么数据都可以
func (m model) Init() tea.Cmd {  
   return tick  
}

func (m model) Update(msg tea.Msg) (tea.Model, tea.Cmd) {  
  
   switch msg.(type) {  
   case tea.KeyMsg:  
      return m, tea.Quit  
   case tickMsg:  
      m -= 1  
      if m <= 0 {  
         return m, tea.Quit  
      }  
      return m, tick  
   }  
   return m, nil  
}

func (m model) View() string {  
   fmt.Println("1. action View func")  
   return fmt.Sprintf("Hi. This program will exit in %d seconds. To quit sooner press any key.\n", m)  
}

type tickMsg time.Time

// 这里返回 time包的Time变量
func tick() tea.Msg {  
   fmt.Println("2. 执行tick函数~")  
   time.Sleep(time.Second)  
   return  tickMsg{}  
}


// main函数
// 只需要调用NewProgram函数，调用其返回的变量方法Start即可完成
func main() {  
   // Initialize our program  
   p := tea.NewProgram(model(5))  
   if err := p.Start(); err != nil {  
      log.Fatal(err)  
   }  
}
```


运行代码
```exec
go build main.go
main.exc

//输出
Hi. This program will exit in 5 seconds. To quit sooner press any key.
Hi. This program will exit in 4 seconds. To quit sooner press any key.
Hi. This program will exit in 3 seconds. To quit sooner press any key.
Hi. This program will exit in 2 seconds. To quit sooner press any key.
Hi. This program will exit in 1 seconds. To quit sooner press any key.
```
以上的输出只会出现一行数据，其中的变化也就是倒数5秒，为了展示将所有数据都展示出来。


##  程序解读

**Init() Cmd**
是将调用的第一个函数。它返回可选的初始命令；返回值 `Cmd` 是一个函数变量，该函数会返回一个 `Msg`，`Msg` 是一个空接口，所以这里返回任意类型数据都可以，也可以直接返回 `nil`；
如果返回数据，那么在执行完 `view` 方法后就会立即执行该数据，转到 `Update` 方法；
如果返回 `nil` ，那么则会直接执行 `Update` 方法。


**Update**
执行完 `View` 方法后则会执行该方法，使用该方法检查消息（msg），并作为响应更新模型和/或发送命令。
这里的消息就是 `Init` 方法返回的数据，如果 `Init()` 方法返回的是nil，则会阻塞，等待键盘映射；

**View**
该方法会第一时间调用，返回一个字符串。