#go 

# Golang的反射
反射是指在程序运行期对程序本身进行访问和修改的能力。程序在编译时，变量被转换为内存地址，变量名不会被编译器写入到可执行部分。在运行程序时，程序无法获取自身的信息。  

支持反射的语言可以在程序编译期将变量的反射信息，如字段名称、类型信息、结构体信息等整合到可执行文件中，并给程序提供接口访问反射信息，这样就可以在程序运行期获取类型的反射信息，并且有能力修改它们。  

Go语言中的反射是由 reflect 包提供支持的，它定义了两个重要的类型 Type 和 Value；任意接口值在反射中都可以理解为由 reflect.Type 和 reflect.Value 两部分组成，并且 reflect 包提供了 reflect.TypeOf 和 reflect.ValueOf 两个函数来获取任意对象的 Value 和 Type。

传入一个空接口值，返回该类型的 Type 类型或者 Value 类型，通过类型对象就可以访问任意值的类型信息，示例：

```go
func TypeOf(i interface{}) Type {}

func ValueOf(i interface{}) Value {}

func main() {
	var a int
	typeOfA := reflect.TypeOf(a)
	fmt.Println(typeOfA.Name(), typeOfA.Kind())
}

//output
int  int
```

其中 Type 类型可以返回该值的结构信息，包括字段信息、方法信息等，Value 类型就可以拿到运行时的值信息；

## 反射三大定律
在 Go 语言里有个反射三定律：

>
反射可以将接口类型变量 转换为“反射类型对象”；
> 
反射可以将 “反射类型对象”转换为 接口类型变量；
>
如果要修改 “反射类型对象” 其类型必须是 可写的；

### 第一定律

> 反射可以将接口类型变量 转换为“反射类型对象”

为了实现从接口变量到反射对象的转换，需要提到 reflect 包里很重要的两个方法：

1.  reflect.TypeOf(i) ：获得接口值的类型
    
2.  reflect.ValueOf(i)：获得接口值的值
    

这两个方法返回的对象，我们称之为反射对象：Type object 和 Value object。示例：
```go
func main() {
    var age interface{} = 25

    fmt.Printf("原始接口变量的类型为 %T，值为 %v \n", age, age)

    t := reflect.TypeOf(age)
    v := reflect.ValueOf(age)

    // 从接口变量到反射对象
    fmt.Printf("从接口变量到反射对象：Type对象的类型为 %T \n", t)
    fmt.Printf("从接口变量到反射对象：Value对象的类型为 %T \n", v)

}

//output
原始接口变量的类型为 int，值为 25
从接口变量到反射对象：Type对象的类型为 *reflect.rtype
从接口变量到反射对象：Value对象的类型为 reflect.Value
```


### 第二定律
> 反射可以将 “反射类型对象”转换为 接口类型变量

和第一定律刚好相反，第二定律描述的是，从反射对象到接口变量的转换。

 reflect.Value 的结构体会接收 `Interface` 方法，返回了一个 `interface{}` 类型的变量（**注意：只有 Value 才能逆向转换，而 Type 则不行，这也很容易理解，如果 Type 能逆向，那么逆向成什么呢？**）示例：
 ```go
 func main() {
    var age interface{} = 25

    fmt.Printf("原始接口变量的类型为 %T，值为 %v \n", age, age)
    
    t := reflect.TypeOf(age)
    v := reflect.ValueOf(age)
    
    // 从接口变量到反射对象
    fmt.Printf("从接口变量到反射对象：Type对象的类型为 %T \n", t)
    fmt.Printf("从接口变量到反射对象：Value对象的类型为 %T \n", v)
    
    // 从反射对象到接口变量
    i := v.Interface()
    fmt.Printf("从反射对象到接口变量：新对象的类型为 %T 值为 %v \n", i, i)

}

//output
原始接口变量的类型为 int，值为 25
从接口变量到反射对象：Type对象的类型为 *reflect.rtype
从接口变量到反射对象：Value对象的类型为 reflect.Value
从反射对象到接口变量：新对象的类型为 int 值为 25
```

当获取到 Interface 对象的时候，就可以使用类型断言转换成最初的原始类型：
​```go
i := v.Interface().(int)
```


### 第三定律
> 如果要修改 “反射类型对象” 其类型必须是 可写的；

Go 语言里的函数都是值传递，只要你传递的不是变量的指针，你在函数内部对变量的修改是不会影响到原始的变量的。

当使用 reflect.Typeof 和 reflect.Valueof 的时候，如果传递的不是接口变量的指针，反射世界里的变量值始终将只是真实世界里的一个拷贝，你对该反射对象进行修改，并不能反映到调用 ValueOf 函数的那个入参对象里。

因此在反射的规则里

-   不是接收变量指针创建的反射对象，是不具备『**可写性**』的
    
-   是否具备『**可写性**』，可使用 `CanSet()` 来获取得知
    
-   对不具备『**可写性**』的对象进行修改，是没有意义的，也认为是不合法的，因此会报错。

```go
func main() {
    var name string = "Go编程时光"

    v := reflect.ValueOf(name)
    fmt.Println("可写性为:", v.CanSet())
}
//output
可写性为: false
```


要让反射对象具备可写性，需要注意两点

1.  创建反射对象时传入变量的指针
    
2.  使用 `Elem()`函数返回指针指向的数据


完整示例：
```go
func main() {
    var name string = "Go编程时光"
    v1 := reflect.ValueOf(&name)
    fmt.Println("v1 可写性为:", v1.CanSet())

    v2 := v1.Elem()
    fmt.Println("v2 可写性为:", v2.CanSet())
}

//output
v1 可写性为: false
v2 可写性为: true
```

知道了如何使反射的世界里的对象具有可写性后，接下来是时候了解一下如何对修改更新它，根据数据类型调用以下方法去修改值：
![11](https://images-1306554305.cos.ap-guangzhou.myqcloud.com/2022-08-15_17-41.png)

上图的方法就是我们修改值的入口，示例：
```go
func main() {
    var name string = "Go编程时光"
    fmt.Println("name 的原始值为：", name)

    v1 := reflect.ValueOf(&name)
    v2 := v1.Elem()

    v2.SetString("Python编程时光")
    fmt.Println("通过反射对象进行更新后，name 变为：", name)
}

//output
name 的原始值为： Go编程时光
通过反射对象进行更新后，name 变为： Python编程时光
```

## Type 类型

### Kind 方法
Type 类型可以获取该值的结构信息，比如说该对象是一个结构体，那么就可以根据 Type 类型拿到该结构体的字段信息以及所属的方法；如果是个基本类型，就可以拿到所属类型；
示例：
```go
// 定义一个Enum类型
type Enum int

const (
	Zero Enum = 0
)

func main() {

	// 声明一个空结构体
	type cat struct {
	}

	// 获取结构体实例的反射类型对象
	typeOfCat := reflect.TypeOf(cat{})
	
	// 显示反射类型对象的名称和种类
	fmt.Println(typeOfCat.Name(), typeOfCat.Kind())
	
	// 获取Zero常量的反射类型对象
	typeOfA := reflect.TypeOf(Zero)
	
	// 显示反射类型对象的名称和种类
	fmt.Println(typeOfA.Name(), typeOfA.Kind())
}

// output
cat struct  
Enum int
```

其中 Kind 方法会返回该值的类型信息，而 reflect 包中所支持的类型信息就有：
```go
type Kind uint  
const (  
	Invalid Kind = iota // 非法类型  
	ool // 布尔型  
	nt // 有符号整型  
	nt8 // 有符号8位整型  
	nt16 // 有符号16位整型  
	nt32 // 有符号32位整型  
	Int64 // 有符号64位整型  
	Uint // 无符号整型  
	Uint8 // 无符号8位整型  
	Uint16 // 无符号16位整型  
	Uint32 // 无符号32位整型  
	Uint64 // 无符号64位整型  
	Uintptr // 指针  
	Float32 // 单精度浮点数  
	Float64 // 双精度浮点数  
	Complex64 // 64位复数类型  
	Complex128 // 128位复数类型  
	Array // 数组  
	Chan // 通道  
	Func // 函数  
	Interface // 接口  
	Map // 映射  
	Ptr // 指针  
	Slice // 切片  
	String // 字符串  
	Struct // 结构体  
	UnsafePointer // 底层指针
)
```

Map、Slice、Chan 属于引用类型，使用起来类似于指针，但是在种类常量定义中仍然属于独立的种类，不属于 Ptr。



## 对属性的操作


任意值通过 reflect.TypeOf() 获得反射对象信息后，如果它的类型是结构体，可以做一些操作拿到该结构体所拥有的字段以及方法。与成员获取相关的方法如下：

| 方法                                                         | 说明                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| Field(i int) StructField                                     | 根据索引，返回索引对应的结构体字段的信息。当值不是结构体或索引超界时引发 panic |
| NumField() int                                               | 返回结构体成员字段数量。当类型不是结构体或索引超界时引发 panic |
| FieldByName(name string) (StructField, bool)                 | 根据给定字符串返回字符串对应的结构体字段的信息。没有找到时 bool 返回 false，当类型不是结构体或索引超界时引发 panic |
| FieldByIndex(index []int) StructField                        | 多层成员访问时，根据 []int 提供的每个结构体的字段索引，返回字段的信息。没有找到时返回零值。当类型不是结构体或索引超界时 引发 panic |
| FieldByNameFunc( match func(string) bool) (StructField,bool) | 根据匹配函数匹配需要的字段。当值不是结构体或索引超界时引发 panic |

在 Field 方法返回的 StructFiled 结构体，该结构体会描述结构体的成员信息，通过这个信息可以获取成员与结构体的关系，如偏移、索引、是否为匿名字段、结构体标签（Struct Tag）等，而且还可以通过 StructField 的 Type 字段进一步获取结构体成员的类型信息。StructField 的结构如下：

```go
type StructField struct {  
    Name string          // 字段名  
    PkgPath string       // 字段路径  
    Type      Type       // 字段反射类型对象  
    Tag       StructTag  // 字段的结构体标签  
    Offset    uintptr    // 字段在结构体中的相对偏移  
    Index     []int      // Type.FieldByIndex中的返回的索引值  
    Anonymous bool       // 是否为匿名字段  
}
```

下面通过一个示例去了解该结构体的用法：
```go
  
type Person struct {
    name string
    age int
    gender string
}

func (p Person)SayBye()  {
    fmt.Println("Bye")
}

func (p Person)SayHello()  {
    fmt.Println("Hello")
}

func main() {
    p := Person{"666", 27, "male"}

    v := reflect.ValueOf(p)

    fmt.Println("字段数:", v.NumField())
    fmt.Println("第 1 个字段：", v.Field(0))
    fmt.Println("第 2 个字段：", v.Field(1))
    fmt.Println("第 3 个字段：", v.Field(2))

    fmt.Println("==========================")
    // 也可以这样来遍历
    for i:=0;i<v.NumField();i++{
        fmt.Printf("第 %d 个字段：%v \n", i+1, v.Field(i))
    }
}
//output
字段数: 3
第 1 个字段： 666
第 2 个字段： 27
第 3 个字段： male
==========================
第 1 个字段：666
第 2 个字段：27
第 3 个字段：male
```


## 对方法的操作
对结构体的方法需要调用 NumMethod 方法和 Method 方法，示例：
```go
type Person struct {
    name string
    age int
    gender string
}

func (p Person)SayBye()  {
    fmt.Println("Bye")
}

func (p Person)SayHello()  {
    fmt.Println("Hello")
}

func main() {
    p := &Person{"写代码的明哥", 27, "male"}

    t := reflect.TypeOf(p)

    fmt.Println("方法数（可导出的）:", t.NumMethod())
    fmt.Println("第 1 个方法：", t.Method(0).Name)
    fmt.Println("第 2 个方法：", t.Method(1).Name)

    fmt.Println("==========================")
    // 也可以这样来遍历
    for i:=0;i<t.NumMethod();i++{
       fmt.Printf("第 %d 个方法：%v \n", i+1, t.Method(i).Name)
    }
}

//output
方法数（可导出的）: 2
第 1 个方法： SayBye
第 2 个方法： SayHello
==========================
第 1 个方法：SayBye
第 2 个方法：SayHello
```


### 动态调用方法(使用索引且无参数)
注意，如果调用方法的话就不能使用 Type 类型了，需要用 Value 类型。因为 Type 类型只是一个结构体，并不承载实际的数据；调用方法的话是需要到实际字段以及值的，所以这里如果使用 Type 类型去调用方法的话，会引发 panic
```go
type Person struct {
    name string
    age int
}

func (p Person)SayBye() string {
    return "Bye"
}

func (p Person)SayHello() string {
    return "Hello"
}

func main() {
    p := &Person{"wangbm", 27}

    t := reflect.TypeOf(p)
    v := reflect.ValueOf(p)

    for i:=0;i<v.NumMethod();i++{
       fmt.Printf("调用第 %d 个方法：%v ，调用结果：%v\n",
           i+1,
           t.Method(i).Name,
           v.Elem().Method(i).Call(nil))
    }
}

//output
调用第 1 个方法：SayBye ，调用结果：[Bye]
调用第 2 个方法：SayHello ，调用结果：[Hello]
```

### 动态调用方法（使用函数名且无参数）
```go
type Person struct {
    name string
    age int
    gender string
}

func (p Person)SayBye()  {
    fmt.Print("Bye")
}

func (p Person)SayHello()  {
    fmt.Println("Hello")
}

func main() {
    p := &Person{"写代码的明哥", 27, "male"}

    v := reflect.ValueOf(p)

    v.MethodByName("SayHello").Call(nil)
    v.MethodByName("SayBye").Call(nil)
}
//output
Hello, my name is wangbm and i'm 27 years old.
```