# Getting Started with Command

Command 模块是 Vapor Console package ([vapor/console](https://github.com/vapor/console)) 的一部分。该模块提供了一些 API 用于创建 命令行接口（CLIs）。

> 提示
>
> Command API 详情可见 [Command API docs](https://api.vapor.codes/console/latest/Command/index.html)。

## Usage

该 package 包含在 Vapor 中并且默认是被导出的，同时它也可以用于任何 Swift 项目。

编辑 ```Package.swift``` 文件，添加如下内容。

```swift
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
      .target(name: "Project", dependencies: ["Command", ... ])
    ]
)
```

通过 ```import Command``` 来访问相应的 API 。

## Overview

Command 特性可见 [Command → Overview](overview.md)。