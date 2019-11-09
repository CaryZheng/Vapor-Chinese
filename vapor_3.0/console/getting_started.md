# Getting Started with Console

Console 模块是 Vapor Console package ([vapor/console](https://github.com/vapor/console)) 的一部分。该模块提供了一些 API 用于执行 console I/O，包含输出固定格式的文本，请求用户输入，和显示进度条。

> 提示
> 
> Console API 详情可见 [Console API docs](https://api.vapor.codes/console/latest/Console/index.html)。

## Usage

该 package 包含在 Vapor 中并且默认是被导出的。导入 ```Vapor``` 后，你将可以访问所有的 ```Console``` API。

```
import Vapor // implies import Console
```

## Standalone

Console 模块也可以被用于任何其它 Swift 项目。

想要包含 Console 模块，只需要添加如下内容到 ```Package.swift``` 文件中即可。

```
// swift-tools-version:4.0
import PackageDescription

let package = Package(
    name: "Project",
    dependencies: [
        ...
        /// 💻 APIs for creating interactive CLI tools.
        .package(url: "https://github.com/vapor/console.git", from: "3.0.0"),
    ],
    targets: [
      .target(name: "Project", dependencies: ["Console", ... ])
    ]
)
```

使用 ```import Console``` 来访问相关的 API。

## Overview

Console 特性可见 [Console → Overview](overview.md)。