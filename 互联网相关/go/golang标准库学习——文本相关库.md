#go

# Golang标准库-文本相关

几乎任何程序都离不开文本（字符串）。Go 中 string 是内置类型，同时它与普通的 slice 类型有着相似的性质，例如，可以进行切片（slice）操作，这使得 Go 中少了一些处理 string 类型的函数，比如没有 substring 这样的函数，然而却能够很方便的进行这样的操作。除此之外，Go 标准库中有几个包专门用于处理文本。

_strings_ 包提供了很多操作字符串的简单函数，通常一般的字符串操作需求都可以在这个包中找到。

_strconv_ 包提供了基本数据类型和字符串之间的转换。在 Go 中，没有隐式类型转换，一般的类型转换可以这么做：int32(i)，将 i （比如为 int 类型）转换为 int32，然而，字符串类型和 int、float、bool 等类型之间的转换却没有这么简单。

进行复杂的文本处理必然离不开正则表达式。_regexp_ 包提供了正则表达式功能，它的语法基于 [RE2](http://code.google.com/p/re2/wiki/Syntax) ，_regexp/syntax_ 子包进行正则表达式解析。

Go 代码使用 UTF-8 编码（且不能带 BOM），同时标识符支持 Unicode 字符。在标准库 _unicode_ 包及其子包 utf8、utf16 中，提供了对 Unicode 相关编码、解码的支持，同时提供了测试 Unicode 码点（Unicode code points）属性的功能。

## strings — 字符串操作

### 是否存在字串
```go
func Contains(s, substr string) bool  // 子串 substr 在 s 中，返回 true

func ContainsAny(s, chars string) bool  // chars 中任何一个 Unicode 代码点在 s 中，返回 true

func ContainsRune(s string, r rune) bool  // Unicode 代码点 r 在 s 中，返回 true
```

示例：
```go
fmt.Println(strings.ContainsAny("team", "i"))  // false
fmt.Println(strings.ContainsAny("failure", "u & i"))  //true，只有有一个存在就true
fmt.Println(strings.ContainsAny("in failure", "s g"))  //true，通商
fmt.Println(strings.ContainsAny("foo", "")) // false
fmt.Println(strings.ContainsAny("", ""))  // false
```

### 字符串分割
_strings_ 包提供了以下分割函数：Fields 、Split 和 SplitAfter、SplitN 和 SplitAfterN。

#### Fields
函数签名：
```go
func Fields(s string) []string
func FieldsFunc(s string, f func(rune) bool) []string
```
Fields 用一个或多个连续的空格分隔字符串 s，返回子字符串的数组（slice）。如果字符串 s 只包含空格，则返回空列表 ([]string 的长度为 0）。其中，空格的定义是 unicode.IsSpace，之前已经介绍过。

常见间隔符包括：'\t', '\n', '\v', '\f', '\r', ' ', U+0085 (NEL), U+00A0 (NBSP)

由于是用空格分隔，因此结果中不会含有空格或空子字符串，例如：
```go
fmt.Printf("Fields are: %q", strings.Fields("  foo bar  baz   "))

// 输出结果
Fields are: ["foo" "bar" "baz"]
```

#### Split
```go
func Split(s, sep string) []string { 
	return genSplit(s, sep, 0, -1) 
}
```

示例：
```go
fmt.Printf("%q\n", strings.Split("a,b,c", ","))
fmt.Printf("%q\n", strings.Split("a man a plan a canal panama", "a "))
fmt.Printf("%q\n", strings.Split(" xyz ", ""))
fmt.Printf("%q\n", strings.Split("", "Bernardo O'Higgins"))
```

输出：
```go
["a" "b" "c"]
["" "man " "plan " "canal panama"]
[" " "x" "y" "z" " "]
[""]
```


###  字符串是否有某个前缀或后缀

#### HasPrefix和HasSuffix
包含2个函数 _HasPrefix_ 函数和 _HasSuffix_ 函数；实现如下：
```go
// s 中是否以 prefix 开始
func HasPrefix(s, prefix string) bool {
  return len(s) >= len(prefix) && s[0:len(prefix)] == prefix
}
// s 中是否以 suffix 结尾
func HasSuffix(s, suffix string) bool {
  return len(s) >= len(suffix) && s[len(s)-len(suffix):] == suffix
}
```

示例示例：

```go
fmt.Println(strings.HasPrefix("Gopher", "Go"))
fmt.Println(strings.HasPrefix("Gopher", "C"))
fmt.Println(strings.HasPrefix("Gopher", ""))
fmt.Println(strings.HasSuffix("Amigo", "go"))
fmt.Println(strings.HasSuffix("Amigo", "Ami"))
fmt.Println(strings.HasSuffix("Amigo", ""))

// 输出
true
false
true
true
false
true
```

### 大小写转换

```go
func ToLower(s string) string
func ToLowerSpecial(c unicode.SpecialCase, s string) string
func ToUpper(s string) string
func ToUpperSpecial(c unicode.SpecialCase, s string) string
```


示例示例
```go
fmt.Println(strings.ToLower("HELLO WORLD"))
fmt.Println(strings.ToLower("Ā Á Ǎ À"))
fmt.Println(strings.ToLowerSpecial(unicode.TurkishCase, "壹"))
fmt.Println(strings.ToLowerSpecial(unicode.TurkishCase, "HELLO WORLD"))
fmt.Println(strings.ToLower("Önnek İş"))
fmt.Println(strings.ToLowerSpecial(unicode.TurkishCase, "Önnek İş"))

fmt.Println(strings.ToUpper("hello world"))
fmt.Println(strings.ToUpper("ā á ǎ à"))
fmt.Println(strings.ToUpperSpecial(unicode.TurkishCase, "一"))
fmt.Println(strings.ToUpperSpecial(unicode.TurkishCase, "hello world"))
fmt.Println(strings.ToUpper("örnek iş"))
fmt.Println(strings.ToUpperSpecial(unicode.TurkishCase, "örnek iş"))
```

输出
```go
hello world
ā á ǎ à
壹
hello world
önnek iş
önnek iş
HELLO WORLD
Ā Á Ǎ À       // 汉字拼音有效
一           //  汉字无效
HELLO WORLD
ÖRNEK IŞ
ÖRNEK İŞ    // 有细微差别
```


## strconv包 
_strconv_ 包主要负责字符串和基本类型之间的转换
这里的基本数据类型包括：布尔、整型（包括有 / 无符号、二进制、八进制、十进制和十六进制）和浮点型等。


### 字符串转为整型
包括三个函数：ParseInt、ParseUint 和 Atoi，函数原型如下：

```go
func ParseInt(s string, base int, bitSize int) (i int64, err error)
func ParseUint(s string, base int, bitSize int) (n uint64, err error)
func Atoi(s string) (i int, err error)
```
ParseInt 转为有符号整型；ParseUint 转为无符号整型；Atoi函数内部调用ParseInt

参数 _base_ 代表字符串按照给定的进制进行解释。一般的，base 的取值为 2~36，如果 base 的值为 0，则会根据字符串的前缀来确定 base 的值：**"0x" 表示 16 进制； "0" 表示 8 进制；否则就是 10 进制。**

参数 _bitSize_ 表示的是整数取值范围，或者说整数的具体类型。取值 0、8、16、32 和 64 分别代表 int、int8、int16、int32 和 int64。


### 整型转为字符串
实际应用中，经常会遇到需要将字符串和整型连接起来，在 Java 中，可以通过操作符 "+" 做到。不过，在 Go 语言中，你需要将整型转为字符串类型，然后才能进行连接。这个时候，_strconv_ 包中的整型转字符串的相关函数就派上用场了。这些函数签名如下：

```go
func FormatUint(i uint64, base int) string	// 无符号整型转字符串
func FormatInt(i int64, base int) string	// 有符号整型转字符串
func Itoa(i int) string
```