# Getting Started with HTTP

HTTP ([vapor/http](https://github.com/vapor/http)) 是一个基于SwiftNIO构建的非阻塞事件驱动的HTTP库。它使SwiftNIO的HTTP处理程序轻松工作，并提供更高级别的功能，如媒体类型，客户端升级，流式体等等。创建HTTP回显服务器只需要几行代码。

> 提示
>
>    如果你使用Vapor，大部分的HTTP API都已被囊括在它的便捷方法里。通常，与您交互的唯一HTTP类型是Request或Response的http属性。

## Vapor

Vapor包含了此包并默认导出， 当你导入 `Vapor` ，你可以使用所有的 `HTTP` APIs。

```swift
import Vapor
```

## Standalone

HTTP包是轻量级的，纯Swift实现，并且只依赖于 SwiftNIO 。这意味着它可以在任何Swift项目中使用HTTP框架 - 甚至是没使用Vapor的项目。

要将其包含到你的包中, 添加以下代码到 `Package.swift` 文件里。

```swift
// swift-tools-version:4.0
import PackageDescription

let package = Package(
    name: "Project",
    dependencies: [
        ...
        .package(url: "https://github.com/vapor/http.git", from: "3.0.0"),
    ],
    targets: [
      .target(name: "Project", dependencies: ["HTTP", ... ])
    ]
)
```

使用 `import HTTP` 去使用它的 APIs.

指南的其余部分将概要介绍HTTP包中的可用内容。与往常一样，请随时访问 [API docs](https://api.vapor.codes/http/latest/HTTP/index.html)以获取更深入的信息。