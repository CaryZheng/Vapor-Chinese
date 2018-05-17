# Getting Started with Logging

Logging 模块作为Vapor控制台包 ([vapor/console](https://github.com/vapor/console)) 的一部分提供服务。该模块提供了用于创建日志的便捷API。 

!!! 提示
    为了更深入的了解 Logging 的 APIs, 请查阅 [Logging API docs](https://api.vapor.codes/console/latest/Logging/index.html)。

## Usage

这个包包含了Vapor， 并且默认导出。当你导入`Vapor`时你就可以访问所有的`Logging` APIs了。

```swift
import Vapor // implies import Logging
```

### Standalone

日志模块是 Vapor 控制台包的很大一部分功能，也可以与任何Swift项目一起使用。

要在你的package里引用它，添加一下代码到你的 `Package.swift` 文件里。

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
      .target(name: "Project", dependencies: ["Logging", ... ])
    ]
)
```

使用 `import Logging` 去访问其APIs.

## Overview
继续阅读 [Logging → Overview](overview.md) 去了解 Logging 的功能.