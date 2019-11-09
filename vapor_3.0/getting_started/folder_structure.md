# 结构

本章节将解释一个典型的 Vapor 应用的目录结构，来帮助你熟悉 Vapor 项目。

## 目录结构

Vapor 的目录结构是基于 [SPM 目录结构](spm.md#目录结构) 构建的。

```
.
├── Public
├── Sources
│   ├── App
│   │   ├── Controllers
│   │   ├── Models
│   │   ├── boot.swift
│   │   ├── configure.swift
│   │   └── routes.swift
│   └── Run
│       └── main.swift
├── Tests
│   └── AppTests
└── Package.swift
```

接下来让我们看下每个目录和文件都是做什么的。

## Public

此目录包含了任何可公开访问的文件。通常是一些图片、style sheets和脚本。

当Vapor响应request请求的时候，会首先检查被请求的文件是否在该目录下。如果存在，它将忽略应用逻辑并且立即返回该文件。

比如，```localhost:8080/favicon.ico``` 请求将会检查 ```Public/favicon.ico``` 是否存在。如果存在，Vapor将直接返回该文件。

## Sources

此目录包含了所有的Swift源码文件。最顶级目录（```App``` 和 ```Run```）对应 package's modules，正如 [package manifest](spm.md) 中描述的。

## App

这是最重要的目录，因为所有应用逻辑都将存放于这里。

### Controllers

Controllers 能很好地将应用逻辑进行分类。大多数 controller 有很多方法用来接收 request 并返回对应的 response 。

> 提示
> 
> Vapor 支持 MVC 模式，但是不强制使用它。

### Models

```Models``` 目录用来存放 [Content](content.md) 结构或 Fluent Models 。

### boot.swift

此文件包含了一个方法，该方法在应用被启动后，但在开始运行之前将会被调用。应用启动的时候你可以在这里做些处理。

你可以访问 [Application](application.md) ，在这里你可以创建任何想要的 [services](application.md) 。

### configure.swift

此文件包含了一个方法，该方法接受 config、environment 和 services 作为输入参数。在这里可以更改 config 或者 注册 services 。

### routes.swift

此文件包含一个方法，该方法用于添加各种路由。

你将会注意到有一个路由示例，正如我们之前所见，它返回了 "hello, world" response。

你可以创建任意多的路由集合，只需要确认在主路由集合中注册它们即可。

## Tests

```Sources``` 目录中每个非可执行的 module 都在 ```...Tests``` 目录下有对应的测试文件。

## AppTests

此目录包含了 ```App``` module 的测试用例。想要了解更多，可参考 [Testing → Getting Started](../testing/getting_started.md) 。

## Package.swift

最后是 SPM [package manifest](spm.md) 。