最近在codeing代码到了测试阶段，让运维和测试去部署程序的时候发现仅仅通过口口相传是不行的，就算给他讲清楚到了现场之后还是会通过电话来轰炸你；有一些开发人员是写了文档，但是文档层次结构、目录不统一，导致文档不能传达该传达的意思。

最近在浏览论坛的时候找到一片关于语言工程化设计的文章，特此摘抄下来，规范自己的文档编写。



## 工程化规范

首先理解工程化规范包括的两方面：

- 非编码类规范：文档规范，版本规范，Git 规范，发布规范，…
- 编码类规范：目录规范，代码规范，接口规范，日志规范，错误码规范，…



### 非编码类规范

#### 文档规范

##### README文档
README 文档是项目的门面，它是开发者学习项目时第一个阅读的文档，会放在项目的根目录下。因为它主要是用来介绍项目的功能、安装、部署和使用的，所以它是可以规范化的。

下面，我们直接通过一个 README 模板，来看一下 README 规范中的内容：
```

# 项目名称

<!-- 写一段简短的话描述项目 -->

## 功能特性

<!-- 描述该项目的核心功能点 -->

## 软件架构(可选)

<!-- 可以描述下项目的架构 -->

## 快速开始

### 依赖检查

<!-- 描述该项目的依赖，比如依赖的包、工具或者其他任何依赖项 -->

### 构建

<!-- 描述如何构建该项目 -->

### 运行

<!-- 描述如何运行该项目 -->

## 使用指南

<!-- 描述如何使用该项目 -->

## 如何贡献

<!-- 告诉其他开发者如果给该项目贡献源码 -->

## 社区(可选)

<!-- 如果有需要可以介绍一些社区相关的内容 -->

## 关于作者

<!-- 这里写上项目作者 -->

## 谁在用(可选)

<!-- 可以列出使用本项目的其他有影响力的项目，算是给项目打个广告吧 -->

## 许可证

<!-- 这里链接上该项目的开源许可证 -->
```

建议使用 **[readme.so](https://readme.so/)**  在线网站生成。



##### 项目文档

项目文档包括一切需要文档化的内容，它们通常集中放在 /docs 目录下。当我们在创建团队的项目文档时，通常会预先规划并创建好一些目录，用来存放不同的文档。因此，在开始 Go 项目开发之前，我们也要制定一个软件文档规范。好的文档规范有 2 个优点：易读和可以快速定位文档。

不同项目有不同的文档需求，在制定文档规范时，你可以考虑包含两类文档。

- 开发文档：描述项目开发流程，包括如何搭建开发环境、构建二进制文件、测试、部署等。
- 用户文档：软件的使用文档，对象一般是软件使用者。内容包括 API 文档、SDK 文档、安装文档、功能介绍文档、最佳实践、操作指南、常见问题等。

参考目录结构：

```
docs
├── devel                            # 开发文档，可以提前规划好，英文版文档和中文版文档
│   ├── en-US/                       # 英文版文档，可以根据需要组织文件结构
│   └── zh-CN                        # 中文版文档，可以根据需要组织文件结构
│       └── development.md           # 开发手册，可以说明如何编译、构建、运行项目
├── guide                            # 用户文档
│   ├── en-US/                       # 英文版文档，可以根据需要组织文件结构
│   └── zh-CN                        # 中文版文档，可以根据需要组织文件结构
│       ├── api/                     # API 文档
│       ├── best-practice            # 最佳实践，存放一些比较重要的实践文章
│       │   └── authorization.md
│       ├── faq                      # 常见问题
│       │   ├── iam-apiserver
│       │   └── installation
│       ├── installation             # 安装文档
│       │   └── installation.md
│       ├── introduction/            # 产品介绍文档
│       ├── operation-guide          # 操作指南，可根据 RESTful 资源再划分为更细的子目录，存放系统功能操作手册
│       │   ├── policy.md
│       │   ├── secret.md
│       │   └── user.md
│       ├── quickstart               # 快速入门
│       │   └── quickstart.md
│       ├── README.md                # 用户文档入口文件
│       └── sdk                      # SDK 文档
│           └── golang.md
└── images                           # 图片存放目录
    └── 部署架构v1.png
```



##### API文档

一般由后端开发人员编写，描述组件提供的 API 以及调用方法。

可以编写 Word/Markdown 格式文档、借助工具编写（填充内容）、通过注释生成（如 Swagger）等。

通常需要包含完整的 API 接口介绍文档（**接口描述**、请求方法、请求参数、输出参数和请求示例）、API 接口变更历史文档、通用说明、数据结构说明、错误码描述和 API 接口使用文档。

- README.md ：API 接口介绍文档，会分类介绍 IAM 支持的 API 接口，并会存放相关 API 接口文档的链接，方便开发者查看。
- CHANGELOG.md ：API 接口文档变更历史，方便进行历史回溯，也可以使调用者决定是否进行功能更新和版本更新。
- generic.md ：用来说明通用的请求参数、返回参数、认证方法和请求方法等。
- struct.md ：用来列出接口文档中使用的数据结构。这些数据结构可能被多个 API 接口使用，会在 user.md、secret.md、policy.md 文件中被引用。
- user.md 、 secret.md 、 policy.md ：API 接口文档，相同 REST 资源的接口会存放在一个文件中，以 REST 资源名命名文档名。
- error_code.md ：错误码描述，通过程序自动生成。

其中接口描述：

- 接口描述：描述接口实现的功能。
- 请求方法：接口的请求方法，格式为 HTTP 方法 请求路径，比如 POST /v1/users。在 **通用说明** 中的 **请求方法** 部分，会说明接口的请求协议和请求地址。
- 输入参数：接口的输入字段，分为 Header 参数、Query 参数、Body 参数、Path 参数。每个字段通过：**参数名称**、**必选**、**类型** 和 **描述** 4 个属性来描述。如果参数有限制或者默认值，可在描述部分注明。
- 输出参数：接口返回字段，每个字段通过 **参数名称**、**类型** 和 **描述** 3 个属性来描述。
- 请求示例：真实的 API 接口请求和返回示例。



接口文档示例：

**Get item**

```http
  GET /api/items/${id}
```

| Input | Parameter | Type     | Description                       |
| ----- | :-------- | :------- | :-------------------------------- |
| Query | `id`      | `string` | **Required**. Id of item to fetch |





#### 版本规范

一般使用 **语义化版本规范**（SemVer，Semantic Versioning），即 `主版本号.次版本号.修订号`（X.Y.Z，其中 X、Y 和 Z 为非负的整数，且禁止在数字前方补零）。

也有先行版本号与编译版本号 `v1.2.3-aplha.1+001`：即把先行版本号（Pre-release）和版本编译元数据，作为延伸加到了主版本号.次版本号.修订号的后面：`X.Y.Z[-先行版本号][+版本编译元数据]`。

- 主版本号（MAJOR）：不兼容的 API 修改。

- - 必须在有任何不兼容的修改被加入公共 API 时递增。其中可以包括次版本号及修订级别的改变。每当主版本号递增时，次版本号和修订号必须归零。
  - 主版本号为零（0.y.z）的软件处于开发初始阶段，由于一切都可能随时被改变，不应该被视为稳定版。`1.0.0` 被界定为第一个稳定版本，之后的所有版本号更新都基于该版本修改。

- 次版本号（MINOR）：向下兼容的功能性新增及修改。一般偶数为稳定版本，奇数为开发版本。在任何公共 API 功能被标记为弃用时也必须递增，当有改进时也可以递增。其中可以包括修订级别的改变。每当次版本号递增时，修订号必须归零。

- 修订号（PATCH）：向下兼容的问题修正，即 bug 修复。

- 先行版本号：该版本不稳定，可能存在兼容性问题。

- 编译版本号：编译器在编译过程中自动生成，开发者只定义其格式、不进行人为控制。

使用建议：

- 使用 0.1.0 作为首个开发版本号，在后续每次发行时递增次版本号。

- 稳定版本并第一次对外发布时版本号可定为 1.0.0。

- 严格按照 Angular commit message 规范提交代码时：

- - fix 类型的 commit 可将修订号 +1。
  - feat 类型的 commit 可将次版本号 +1。
  - 带 BREAKING CHANGE 的 commit 可将主版本号 +1。



#### Git规范

常见的开源社区 commit message 规范有： jQuery、Ember、JSHint AngularJS和Conventional commits，这里详细介绍Angular规范。

**Angular**规范包括：

- 语义化：commit message 被归为有意义的类型用来说明本次 commit 的类型。
- 规范化：commit message 遵循预先定义好的规范，比如格式固定、都属于某个类型，可被开发者和工具识别。



##### Commit规范

基本格式：

```
<type>[optional scope]: <description>
// 空行
[optional body]
// 空行
[optional footer(s)]
```

分别对应 Commit message 的三个部分：`Header`，`Body` 和 `Footer`；其中，`Header` 是必需的，`Body` 和 `Footer` 可以省略。

> 建议使用 `git commit` 时不要使用 `-m` 选项，而 `-a` 进入交互界面编辑 commit message。

下面对 **Header**、**Body** 和**Footer**详细介绍



**Header**

```
<type>[optional scope]: <subject:description>
```

只有一行，包括：**type**（必选）、scope（可选）和 **subject**（必选）。



type 分为：

- Development：一般是项目管理类的变更，不会影响最终用户和生产环境的代码，比如 CI 流程、构建方式等的修改。通常可以免测发布。
- Production：会影响最终的用户和生产环境的代码。一定要慎重并在提交前做好充分的测试。

| 类型     | 类别        | 说明                                                         |
| -------- | ----------- | ------------------------------------------------------------ |
| feature  | Production  | 新增功能                                                     |
| fix      | Production  | 修复Bug                                                      |
| perf     | Production  | 提高代码性能的变更                                           |
| refactor | Production  | 其他代码类的变更，这些变更不属于feature、fix、perf和style；例如简化代码、重命名变量、删除冗余代码等 |
| style    | Development | 代码格式类的变更，比如用 gofmt 格式化代码、删除空行等        |
| test     | Development | 新增测试用例或是更新现有测试用例                             |
| ci       | Development | 持续继承和部署相关的改动，比如修改 Jenkins、GitLab CI 等CI配置文件或者更新 systemd unit 文件 |
| docs     | Development | 文档类的更新，包括修改用户文档或者开发文档等                 |
| chore    | Development | 其他类型；比如构建流程、依赖管理或者辅助工具的改动           |



scope 说明 commit 影响范围，必须是名词，因项目而异。

- 初期可设置粒度较大的 scope（如按组件名或功能设置），后续项目有变动或有新功能时可添加新的 scope。
- 不适合设置太具体的值。会导致项目有太多的 scope 难以维护，开发者也难以确定 commit 所属的 scope 导致错放，反而失去分类的意义。



subject 是 commit 的简短描述，明确指出 commit 执行的操作，以动词开头（小写）、使用现在时，不加句号。



**Body**

可分成多行，格式较自由。以动词开头、使用现在时。

必须包括修改的动机、跟上一版本相比的改动点。

示例：

```
The body is mandatory for all commits except for those of scope "docs". When the body is required it must be at least 20 characters long.
```



**Footer**

根据需要选择，一般格式：

```
BREAKING CHANGE: <breaking change summary>
// 空行
<breaking change description + migration instructions>
// 空行
// 空行
Fixes #<issue number>
```

分为两部分，BREAKING CHANGE 和 Fixes

在 **BREAKING CHANGE** 之后跟上简要的描述，空一行写上具体的信息：包括说明变动的描述、变动的理由和迁移方法：

```
BREAKING CHANGE: isolate scope bindings definition has changed and
    the inject option for the directive controller injection was removed.

    To migrate the code follow the example below:

    Before:

    scope: {
      myAttr: 'attribute',
    }

    After:

    scope: {
      myAttr: '@',
    }
    
    The removed `inject` wasn't generaly useful for directives so there should be no code using it.
```

**Fixes**部分跟上该功能需要关闭的Issue列表

关闭的 Bug 需要在 Footer 部分新建一行，并以 Closes 开头列出，比如：`Closes #123`。如果关闭多个 Issue，可以列出：`Closes #123, #432, #886`。

```
Change pause version value to a constant for image
   
   Closes #1137
```



**规范Commit Message的插件**

如果在使用JB系的编辑器，就可以使用 **Git Commit Message Helper** 插件来规范提交信息。