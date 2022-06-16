## 概要
  
Makefile 可以简单的认为是一个工程文件的编译规则，描述了整个工程的编译和链接等规则。其中包含了那些文件需要编译，那些文件不需要编译，那些文件需要先编译，那些文件需要后编译，那些文件需要重建等等。编译整个工程需要涉及到的，在 Makefile 中都可以进行描述。换句话说，Makefile 可以使得我们的项目工程的编译变得自动化，不需要每次都手动输入一堆源文件和参数。

Makefile带来的好处就是——“自动化编译”，一旦写好，只需要一个make命令，整个工程完全自动编译，极大的提高了软件开发的效率。 make是一个命令工具，是一个解释makefile中指令的命令工具，一般来说，大多数的IDE都有这个命令，比如：Delphi的make，Visual C++的nmake，Linux下GNU的make。可见，makefile都成为了一种在工程方面的编译方法。

Makefile 文件由三个部分组成，分别是 Makefile 规则、Makefile 语法和 Makefile 命令（这些命令可以是 Linux 命令，也可以是可执行的脚本文件），先来看下如何使用 Makefile 脚本。

## Makefile 使用方法
在实际使用过程中，我们一般是先编写一个 Makefile 文件，指定整个项目的编译规则，然后通过 Linux **make** 命令来解析该 Makefile 文件，实现项目编译、管理的自动化。

默认情况下，**make** 命令会在当前目录下，按照 GNUmakefile、makefile、Makefile 文件的顺序查找 Makefile 文件，一旦找到，就开始读取这个文件并执行。

大多数的 make 都支持“makefile”和“Makefile”这两种文件名，但**建议使用“Makefile”**。因为这个文件名第一个字符大写，会很明显，容易辨别。make 也支持 -f 和 --file 参数来指定其他文件名，比如 make -f golang.mk 或者 make --file golang.mk 。

## Makefile 规则
规则是 Makefile 中的重要概念，它一般由目标、依赖和命令组成，用来指定源文件编译的先后顺序。Makefile 之所以受欢迎，核心原因就是 Makefile 规则，因为 Makefile 规则可以自动判断是否需要重新编译某个目标，从而确保目标仅在需要时编译。

这里主要来看 Makefile 规则里的规则语法、伪目标和 order-only 依赖。

### 规则语法
Makefile 的规则语法，主要包括 target、prerequisites 和 command，示例如下：

```make
target ...: prerequisites ...
    command
    ...
    ...
```

**target**，可以是一个 object file（目标文件），也可以是一个执行文件，还可以是一个标签（label）。target 可使用通配符，当有多个目标时，目标之间用空格分隔。

**prerequisites**，代表生成该 target 所需要的依赖项。当有多个依赖项时，依赖项之间用空格分隔。

**command**，代表该 target 要执行的命令（可以是任意的 shell 命令）。

- 在执行 command 之前，**默认会先打印出该命令，然后再输出命令的结果；如果不想打印出命令，可在各个 command 前加上@。**
- command 可以为多条，也可以分行写，但每行都要以 tab 键开始。另外，如果后一条命令依赖前一条命令，则这两条命令需要写在同一行，并用分号进行分隔。
- 如果要忽略命令的出错，需要在各个 command 之前加上减号-。


> 只要 targets 不存在，或 prerequisites 中有一个以上的文件比 targets 文件新，那么 command 所定义的命令就会被执行，从而产生我们需要的文件，或执行我们期望的操作。


#### 示例
这次示例是在 ubuntu 20.04 版本的操作系统下执行。

第一步，先创建一个 hello.c 文件

```c
#include <stdio.h>
int main()
{
  printf("Hello World!\n");
  return 0;
}
```

第二步，在当前目录下，编写 Makefile 文件。
```make
hello: hello.o
  gcc -o hello hello.o

hello.o: hello.c
  gcc -c hello.c

clean:
  rm hello.o
```

第三步，执行 make，产生可执行文件。
```bash
$ make
gcc -c hello.c
gcc -o hello hello.o
$ ls
hello  hello.c  hello.o  Makefile
```

上面的示例 Makefile 文件有两个 target，分别是 hello 和 hello.o，每个 target 都指定了构建 command。当执行 make 命令时，发现 hello、hello.o 文件不存在，就会执行 command 命令生成 target。

第四步，不更新任何文件，再次执行 make。
```bash
$ make
make: 'hello' is up to date.
```
当 target 存在，并且 prerequisites 都不比 target 新时，不会执行对应的 command；但是如果文件有更新，则会重新执行对应的 command。


### 伪目标
接下来介绍下 Makefile 中的伪目标。Makefile 的管理能力基本上都是通过伪目标来实现的。在 Makefile 中可以使用.PHONY来标识一个目标为伪目标；

通常情况下，我们需要显式地标识这个目标为伪目标：

```make
.PHONY: clean
clean:
    rm hello.o
```

伪目标可以有依赖文件，也可以作为“默认目标”，例如：

```make
.PHONY: all
all: lint test build
```

因为伪目标总是会被执行，所以其依赖总是会被决议。通过这种方式，可以达到同时执行所有依赖项的目的。

### 语法概览
因为 Makefile 的语法比较多，这里只介绍 Makefile 的核心语法，以及项目中可能用到的 Makefile 的语法，包括命令、变量、条件语句和函数。因为 Makefile 没有太多复杂的语法，你掌握了这些知识点之后，再在实践中多加运用，融会贯通，就可以写出非常复杂、功能强大的 Makefile 文件了。

#### 命令
Makefile 支持 Linux 命令，调用方式跟在 Linux 系统下调用命令的方式基本一致。默认情况下，make 会把正在执行的命令输出到当前屏幕上。但我们可以通过在命令前加@符号的方式，禁止 make 输出当前正在执行的命令。

下面创建一个示例，创建一个 Makefile：
```make
.PHONY: test
test:
    echo "hello world"
```
> 注意，在编写命令 echo “hello world” 时，需要加 `Tab` 键，在Makefile中的命令，必须要以 `Tab` 键开始。


执行 make 命令
```make
$ make test
echo "hello world"
hello world
```

可以看到，make 输出了执行的命令。很多时候，我们不需要这样的提示，因为我们更想看的是命令产生的日志，而不是执行的命令。这时就可以在命令行前加@，禁止 make 输出所执行的命令：
```make
.PHONY: test
test:
    @echo "hello world"
```

再次执行 make 命令：
```bash
$ make test
hello world
```
可以看到，make 只是执行了命令，而没有打印命令本身。这样 make 输出就清晰了很多。

这里，我建议在命令前都加@符号，禁止打印命令本身，以保证你的 Makefile 输出易于阅读的、有用的信息。

默认情况下，每条命令执行完 make 就会检查其返回码。如果返回成功（返回码为 0），make 就执行下一条指令；如果返回失败（返回码非 0），make 就会终止当前命令。很多时候，命令出错（比如删除了一个不存在的文件）时，我们并不想终止，这时就可以在命令行前加 - 符号，来让 make 忽略命令的出错，以继续执行下一条命令，比如：

```make
clean:
    -rm hello.o
```

#### 变量
变量，可能是 Makefile 中使用最频繁的语法了，Makefile 支持变量赋值、多行变量和环境变量。另外，Makefile 还内置了一些特殊变量和自动化变量。

先来看下最基本的变量赋值功能。Makefile 也可以像其他语言一样支持变量。在使用变量时，会像 shell 变量一样原地展开，然后再执行替换后的内容。

Makefile 可以通过变量声明来声明一个变量，变量在声明时需要赋予一个初值，比如ROOT_PACKAGE=github.com/marmotedu/iam。

引用变量时可以通过 \$() 或者 \${} 方式引用。我的建议是，用 \$() 方式引用变量，例如 $(ROOT_PACKAGE)，也建议整个 makefile 的变量引用方式保持一致。

变量会像 bash 变量一样，在使用它的地方展开。比如：

```make
GO=go
build:
    $(GO) build -v .
```
展开后：

```make
GO=go
build:
    go build -v .
```

Makefile 有 4 种变量赋值的方法。

1. = 最基本的赋值方法。

示例：
```make
BASE_IMAGE = alpine:3.10
```

使用这种方法进行变量赋值时，需要注意下面的情况：
```make
A = a
B = $(A) b
A = c
```

B 最后的值为 c b，而不是 a b。也就是说，在用变量给变量赋值时，右边变量的取值，取的是最终的变量值。怎么避免这在情况呢？下面这种变量赋值方法可以避免。

2. :=直接赋值，赋予当前位置的值。

示例：
```make
A = a
B := $(A) b
A = c
```

B 最后的值为 a b。通过 := 的赋值方式，可以避免 = 赋值带来的潜在的不一致。

3. ?= 表示如果该变量没有被赋值，则赋予等号后的值。

示例：
```make
PLATFORMS ?= linux_amd64 linux_arm64
```

4. +=表示将等号后面的值添加到前面的变量上。

示例：
```make
MAKEFLAGS += --no-print-directory
```


Makefile 还支持多行变量。可以通过 define 关键字设置多行变量，变量中允许换行，定义方式为：

```make
define 变量名
变量内容
...
endef
```

变量的内容可以包含函数、命令、文字或是其他变量。例如，我们可以定义一个 USAGE_OPTIONS 变量：

```make
define USAGE_OPTIONS

Options:
  DEBUG        Whether to generate debug symbols. Default is 0.
  BINS         The binaries to build. Default is all of cmd.
  ...
  V            Set to 1 enable verbose build. Default is 0.
endef
```

Makefile 还支持环境变量。在 Makefile 中，有两种环境变量，分别是 Makefile 预定义的环境变量和自定义的环境变量。

其中，自定义的环境变量可以覆盖 Makefile 预定义的环境变量。

默认情况下，Makefile 中定义的环境变量只在当前 Makefile 有效，如果想向下层传递（Makefile 中调用另一个 Makefile），需要使用 export 关键字来声明。下面的例子声明了一个环境变量，并可以在下层 Makefile 中使用：

```make
...
export USAGE_OPTIONS
...
```


Makefile 还支持**自动化变量**。自动化变量可以提高我们编写 Makefile 的效率和质量。

在 Makefile 的模式规则中，目标和依赖文件都是一系列的文件，那么我们如何书写一个命令，来完成从不同的依赖文件生成相对应的目标呢？这时就可以用到自动化变量。

所谓自动化变量，就是这种变量会把模式中所定义的一系列的文件自动地挨个取出，一直到所有符合模式的文件都取完为止。这种自动化变量只应出现在规则的命令中。

Makefile 中支持的自动化变量见下表。

![](https://images-1306554305.cos.ap-guangzhou.myqcloud.com/makefile的自动变量命令.png)

> 这部分我没搞清楚，后续在更新。



#### 条件语法
Makefile 也支持条件语句。

条件语法有 4 个关键字：ifeq、ifneq、ifdef、ifndef。但无论是哪个关键字，条件语法的流程都是一样的：

```make
# if 语法
<conditional-directive> // 这里就条件语法关键字中 4 选 1
<text-if-true> // 当条件为 true 时则执行这里
endif


# if else 的语法
<conditional-directive>
<text-if-true>
else
<text-if-false>
endif
```

ifeq 表示条件语句的开始，并指定一个条件表达式。表达式包含两个参数，参数之间用逗号分隔，并且表达式用圆括号括起来。

else 表示条件表达式为假的情况。

endif 表示一个条件语句的结束，任何一个条件表达式都应该以 endif 结束。

为了加深理解，看看这 4 个关键字的例子：

1. ifeq：条件判断，判断是否相等。

```make
ifeq (<arg1>, <arg2>)
ifeq '<arg1>' '<arg2>'
ifeq "<arg1>" "<arg2>"
ifeq "<arg1>" '<arg2>'
ifeq '<arg1>' "<arg2>"
```

可以使用函数或者变量去替代里面的 arg1 或者 arg2。

2. ifneq：条件判断，判断是否不相等。

```make
ifneq (<arg1>, <arg2>)
ifneq '<arg1>' '<arg2>'
ifneq "<arg1>" "<arg2>"
ifneq "<arg1>" '<arg2>'
ifneq '<arg1>' "<arg2>"
```

比较 arg1 和 arg2 的值是否不同，如果不同则为真。


3. ifdef：条件判断，判断变量是否已定义。

```make
ifdef <variable-name>
```

如果值非空，则表达式为真，否则为假。也可以是函数的返回值。


4. ifndef：条件判断，判断变量是否未定义。

```make
ifndef <variable-name>
```

如果值为空，则表达式为真，否则为假。也可以是函数的返回值。


#### 函数

Makefile 同样也支持**函数**，函数语法包括定义语法和调用语法。

我们先来看下**自定义函数**。 make 解释器提供了一系列的函数供 Makefile 调用，这些函数是 Makefile 的预定义函数。我们可以通过 define 关键字来自定义一个函数。自定义函数的语法为：


```make
define 函数名
函数体
endef
```


示例：

```make
define Foo
    @echo "my name is $(0)"
    @echo "param is $(1)"
endef
```

define 本质上是定义一个多行变量，可以在 call 的作用下当作函数来使用，在其他位置使用只能作为多行变量来使用，例如：

```make
var := $(call Foo)
new := $(Foo)
```

> 注意，要想调用语法需要使用 $(call 函数名)语法。


示例：
```make
.PHONY: test
test:
	@echo "hello world"

.PHONY: fun
fun:
	@$(call echo)

define echo
	@echo "test function"
endef
```

使用命令测试：

```make
$ make fun

输出：
test function
```

自定义函数是一种过程调用，没有任何的返回值。可以使用自定义函数来定义命令的集合，并应用在规则中。


再来看看**预定义函数**，make 编译器也定义了很多函数，这些函数叫作预定义函数，调用语法和变量类似，语法为：

```make
$(<function> <arguments>)
```

或者

```make
${<function> <arguments>}
```

\<function> 是函数名，\<arguments> 是函数参数，参数间用逗号分割。函数的参数也可以是变量。

Makefile 预定义函数能够帮助我们实现很多强大的功能，在编写 Makefile 的过程中，如果有功能需求，可以优先使用这些函数。如果你想使用这些函数，那就需要知道有哪些函数，以及它们实现的功能。

常用的函数包括下面这些：
![](https://images-1306554305.cos.ap-guangzhou.myqcloud.com/2022-06-16_17-18.png)