## 单元测试



在 Go 项目开发中，我们不仅要开发功能，更重要的是确保这些功能稳定可靠，并且拥有一个不错的性能。要确保这些，就要对代码进行测试。开发人员通常会进行单元测试和性能测试，分别用来测试代码的功能是否正常和代码的性能是否满足需求。每种语言通常都有自己的测试包 / 模块，Go 语言也不例外。在 Go 中，我们可以通过testing包对代码进行单元测试和性能测试。


### 如何测试 Go 代码？
Go 语言有自带的测试框架testing，也有 testify ，可以用来实现单元测试（T 类型）和性能测试（B 类型），通过go test命令来执行单元测试和性能测试。

go test 执行测试用例时，是以 go 包为单位进行测试的。执行时需要指定包名，比如go test 包名，如果没有指定包名，默认会选择执行命令时所在的包。go test 在执行时，会遍历以_test.go结尾的源码文件，执行其中以Test、Benchmark、Example开头的测试函数。

#### 测试代码长什么样？

##### 测试文件的命名规范
Go 的测试文件名必须以_test.go结尾。例如，如果我们有一个名为person.go的文件，那它的测试文件必须命名为person_test.go。这样做是因为，Go 需要区分哪些文件是测试文件。这些测试文件可以被 go test 命令行工具加载，用来测试我们编写的代码，但会被 Go 的构建程序忽略掉，因为 Go 程序的运行不需要这些测试代码。

##### 函数的命名规范

测试用例函数必须以Test、Benchmark、Example开头，例如TestXxx、BenchmarkXxx、ExampleXxx，Xxx部分为任意字母数字的组合，首字母大写。这是由 Go 语言和 go test 工具来进行约束的，Xxx一般是需要测试的函数名。

例如，我们有以下函数：
```go
package main
type Person struct { 
	age int64
}

func (p *Person) older(other *Person) bool { 
	return p.age > other.age
}
```

很显然，我们可以把测试函数命名为TestOlder，这个名称可以很清晰地说明它是Older函数的测试用例。但是，如果我们想用多个测试用例来测试TestOlder函数，这些测试用例该如何命名呢？也许你会说，我们命名为TestOlder1、TestOlder2不就行了？

其实，还有其他更好的命名方法。比如，这种情况下，我们可以将函数命名为TestOlderXxx，其中Xxx代表Older函数的某个场景描述。例如，strings.Compare函数有如下测试函数：TestCompare、TestCompareIdenticalString、TestCompareStrings。

##### 变量的命名规范
Go 语言和 go test 没有对变量的命名做任何约束。但是，在编写单元测试用例时，还是有一些规范值得我们去遵守。

单元测试用例通常会有一个实际的输出，在单元测试中，我们会将预期的输出跟实际的输出进行对比，来判断单元测试是否通过。为了清晰地表达函数的实际输出和预期输出，可以将这两类输出命名为expected/actual，或者got/want。例如：
```go
if c.expected != actual {
  t.Fatalf("Expected User-Agent '%s' does not match '%s'", c.expected, actual)
}
```
或者：
```go
if got, want := diags[3].Description().Summary, undeclPlural; got != want {
  t.Errorf("wrong summary for diagnostic 3\ngot:  %s\nwant: %s", got, want)
}
```


#### 怎么执行测试代码？

执行测试代码的命令，是在当前工程的目录下执行以下命令：
```golang
go test [option] [test_file_name]
```

可以灵活的运行指定测试用例；该命令如果不携带参数，则会执行所有测试文件的测试用例；


**选项参数**

- -v，显示所有测试函数的运行细节：
```go
$ go test -v
=== RUN   TestAbs // 测试用例名称
--- PASS: TestAbs (0.00s) // 运行结果
=== RUN   TestMax
--- PASS: TestMax (0.00s)
PASS
ok      github.com/marmotedu/gopractise-demo/31/test    0.002s
```

- -run < regexp >，指定要运行的测试函数：
```go
$ go test -v -run Test
=== RUN   TestAbs
--- PASS: TestAbs (0.00s)
=== RUN   TestBbs
--- PASS: TestBbs (0.00s)
PASS
ok      github.com/marmotedu/gopractise-demo/31/test    0.001s
```

> 示例代码写的是Test，那么 TestAbs 和TestBbs 测试用例都将执行，因为 -run 参数后面跟的是正则表达式，会匹配所有以 Test 开头的测试用例；
> 使用 `-run TestAbs$` 即可执行 TestAbs 测试用例。

- -count N，指定执行测试函数的次数
```go
$ go test -v -run='TestA.*' -count=2
=== RUN   TestAbs
--- PASS: TestAbs (0.00s)
=== RUN   TestAbs
--- PASS: TestAbs (0.00s)
PASS
ok      github.com/marmotedu/gopractise-demo/31/test    0.002s
```


> 这里只介绍单元测试的选项，基准测试后面再提。



#### 单元测试

单元测试用例函数以 Test 开头，例如 TestXxx 或 Test_xxx （ Xxx 部分为任意字母数字组合，首字母大写）。函数参数必须是 *testing.T，可以使用该类型来记录错误或测试状态。

我们可以调用 testing.T 的 Error 、Errorf 、FailNow 、Fatal 、FatalIf 方法，来说明测试不通过；调用 Log 、Logf 方法来记录测试信息。函数列表和相关描述如下表所示：

| 函数                       | 描述                                                         |
| -------------------------- | ------------------------------------------------------------ |
| t.Log，t.Logf              | 正常信息                                                     |
| t.Error，t.Errorf          | 测试失败信息                                                 |
| t.Fatal，t.Fatalf          | 致命错误，测试程序退出的信息                                 |
| t.Fail                     | 当前测试标记为失败                                           |
| t.Failed                   | 查看失败标记                                                 |
| t.FailNow                  | 标记失败，并且终止当前测试函数的执行。                       |
| t.Skip，t.Skipf，t.Skipped | 调用 t.Skip 方法，相当于先后对 t.Log 和 t.SkipNow 方法进行调用；<br />调用 t.Skipf 方法，相当于先后对 t.Logf 和 t.SkipNow 方法进行调用；<br />方法 t.Skipped 的结果值会告知我们当前的测试是否已被忽略 |
| t.Parallel                 | 标记为可并行运算                                             |

现在对下面的函数进行测试：
```go
package test

import (
  "fmt"
  "math"
  "math/rand"
)

// Abs returns the absolute value of x.
func Abs(x float64) float64 {
  return math.Abs(x)
}

// Max returns the larger of x or y.
func Max(x, y float64) float64 {
  return math.Max(x, y)
}

```

编写测试文件：
```go
func TestAbs(t *testing.T) {
    got := Abs(-1)
    if got != 1 {
        t.Errorf("Abs(-1) = %f; want 1", got)
    }
}

func TestMax(t *testing.T) {
    got := Max(1, 2)
    if got != 2 {
        t.Errorf("Max(1, 2) = %f; want 2", got)
    }
}
```

执行 go test 命令来执行单元测试用例：
```go
$ go test
PASS
ok      github.com/marmotedu/gopractise-demo/31/test    0.002s
```
go test命令自动搜集所有的测试文件，也就是格式为 `*_test.go` 的文件，从中提取全部测试函数并执行。

**多个输入的测试用例**
前面介绍的单元测试用例只有一个输入，但是很多时候，我们需要测试一个函数在多种不同输入下是否能正常返回。
这时候，我们可以编写一个稍微复杂点的测试用例，用来支持多输入下的用例测试。例如，我们可以将TestAbs改造成如下函数：

```go
func TestAbs_2(t *testing.T) {
    tests := []struct {
        x    float64
        want float64
    }{
        {-0.3, 0.3},
        {-2, 2},
        {-3.1, 3.1},
        {5, 5},
    }

    for _, tt := range tests {
        if got := Abs(tt.x); got != tt.want {
            t.Errorf("Abs() = %f, want %v", got, tt.want)
        }
    }
}
```

上述测试用例函数中，可以分为两部分，一部分是声明测试数据，另一部分则是将测试数据传入测试函数内进行测试，让我们分别来看这 2 段代码；

我们定义了一个结构体数组，数组中的每一个元素代表一次测试用例。数组元素的的值包含输入和预期的返回值：
```go
tests := []struct {
    x    float64
    want float64
}{
    {-0.3, 0.3},
    {-2, 2},
    {-3.1, 3.1},
    {5, 5},
}
```

上述测试用例，将被测函数放在 for 循环中执行：
```go
for _, tt := range tests {
	if got := Abs(tt.x); got != tt.want {
		t.Errorf("Abs() = %f, want %v", got, tt.want)
	}
}
```

上面的代码将输入传递给被测函数，并将被测函数的返回值跟预期的返回值进行比较。

如果相等，则说明此次测试通过，如果不相等则说明此次测试不通过。

通过这种方式，我们就可以在一个测试用例中，测试不同的输入和输出，也就是不同的测试用例。如果要新增一个测试用例，根据需要添加输入和预期的返回值就可以了，这些测试用例都共享其余的测试代码。

上面的测试用例中，我们通过got != tt.want来对比实际返回结果和预期返回结果。我们也可以使用github.com/stretchr/testify/assert包中提供的函数来做结果对比，主要是 for 循环这里，下面是使用了和没使用的区别：

```go
	// 未使用 testify 包
    for _, tt := range tests {
        if got := Abs(tt.x); got != tt.want {
            t.Errorf("Abs() = %f, want %v", got, tt.want)
        }
    }

	// 使用了 testify 包
    for _, tt := range tests {
        got := Abs(tt.x)
        assert.Equal(t, got, tt.want)
    }
}
```

使用assert来对比结果，有下面这些好处：
- 友好的输出结果，易于阅读。
- 因为少了if got := Xxx(); got != tt.wang {}的判断，代码变得更加简洁。
- 可以针对每次断言，添加额外的消息说明，例如assert.Equal(t, got, tt.want, "Abs test")。

assert 包还提供了很多其他函数，供开发者进行结果对比，例如Zero、NotZero、Equal、NotEqual、Less、True、Nil、NotNil等。


#### 特殊的测试函数
有一些函数比较特殊，这些函数需要依赖第三方服务，比如数据库、网络等等；在这些场景中可以用以下第三包来实现：

- sqlmock：可以用来模拟数据库连接。数据库是项目中比较常见的依赖，在遇到数据库依赖时都可以用它。
- httpmock：可以用来 Mock HTTP 请求。
- bouk/monkey：猴子补丁，能够通过替换函数指针的方式来修改任意函数的实现。如果 golang/mock、sqlmock 和 httpmock 这几种方法都不能满足我们的需求，我们可以尝试用猴子补丁的方式来 Mock 依赖。可以这么说，猴子补丁提供了单元测试 Mock 依赖的最终解决方案。