#go 

Go语言的 sort.Sort 函数不会对具体的序列和它的元素做任何假设。相反，它使用了一个接口类型 sort.Interface 来指定通用的排序算法和可能被排序到的序列类型之间的约定。这个接口的实现由序列的具体表示和它希望排序的元素决定，序列的表示经常是一个切片。

实现排序有2个方法
# 调用Sort包的Slice方法
该方法是go1.8版本后支持的，调用sort包的Slice函数，该函数的签名如下：
```bash
func Slice(x interface{}, less func(i int, j int) bool
```
x参数是一个自定义对象切片，less参数是一个函数对象，该函数对象需要传入2个参数，返回bool值，该函数就是自定义对象实现排序的依据

```bash
	//sort.Sort(menu)
	sort.Slice(menu, func(i, j int) bool {
		if menu[i].Sort < menu[j].Sort {
			return true
		} else {
			return false
		}
	})
```
以上代码实现了菜单的排序，根据菜单的Sort字段进行升序排序。
# 实现Interface接口
一个内置的排序算法需要知道三个东西：序列的长度，表示两个元素比较的结果，一种交换两个元素的方式；这就是 sort.Interface 的三个方法：
## 需要实现的接口
```bash
type Interface interface {
  Len() int
  Less(i, j int) bool
  Swap(i, j int)
}
```
## 实现接口

为了对序列进行排序，我们需要定义一个实现了这三个方法的类型，然后对这个类型的一个实例应用 sort.Sort 函数。那么就要给该对象起一个别名，例子如下：
```bash
// UsvMenus 起别名，用于实现sort包下的三个方法接口，用于对象的自定义排序
type UsvMenus []UsvMenu

func (p UsvMenus) Len() int { return len(p) }

func (p UsvMenus) Less(i, j int) bool {
	return p[i].Sort < p[j].Sort
}

func (p UsvMenus) Swap(i, j int) { p[i], p[j] = p[j], p[i] }
```
> 注意，这里必须得是起别名，不这样的话该接口的 Swap函数就无法实现元素交换

在Less函数和Swap函数都起到了排序实现，首先会是Less函数，该函数会让切片里的i元素的Sort字段和j元素的Sort字段进行比较并返回比较结果，而Swap函数使用Go语言的多变量赋值特性实现元素交换。

在以上例子中就根据了切片元素中的Sort字段进行比较判断，判断i元素小于j元素则返回true，结果将会是升序排序。

## 使用排序
在给需要排序的对象实现接口之后，就可以在业务代码里面进行调用了。<br />调用很简单，就需要写以下代码：
```bash
sort.Sort(menu)
```
调用sort包的Sort方法，传入实现了 `Interface`接口的结构体。

# 常见类型的便捷排序
## 对字符串切片进行升序排序
使用 sort.Strings 直接对字符串切片进行排序。
```bash
names := []string{
    "3. Triple Kill",
    "5. Penta Kill",
    "2. Double Kill",
    "4. Quadra Kill",
    "1. First Blood",
}
sort.Strings(names)
// 遍历打印结果
for _, v := range names {
    fmt.Printf("%s\n", v)
}
```

## sort包内的内建类型排序
| 类  型 | 实现 sort.lnterface 的类型 | 直接排序方法 | 说  明 |
| --- | --- | --- | --- |
| 字符串（String） | StringSlice | sort.Strings(a [] string) | 字符 ASCII 值升序 |
| 整型（int） | IntSlice | sort.Ints(a []int) | 数值升序 |
| 双精度浮点（float64） | Float64Slice | sort.Float64s(a []float64) | 数值升序 |

[参考文章](http://c.biancheng.net/view/81.html)
