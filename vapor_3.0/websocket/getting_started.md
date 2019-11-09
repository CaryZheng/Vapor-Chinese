# Getting Started with WebSocket

WebSocket ([vapor/websocket](https://github.com/vapor/websocket)) 是基于 SwiftNIO 的非阻塞、事件驱动型的 WebSocket 库。该库使得很容易使用基于 SwiftNIO 中的 WebSocket ，并能与 [HTTP](../http/getting_started.md) 客户端和服务端进行整合。仅仅只需很少代码就能创建一个 WebSocket echo server 。

> 提示
> 
> 如果使用 Vapor，大部分 WebSocket API 将会以更便捷的方式显示出来。

## Vapor

该 package 包含在 Vapor 中并默认被导出。导入 ```Vapor``` 后你可以访问所有 ```WebSocket``` API。

```swift
import Vapor
```

## Standalone

WebSocket package 是轻量的，它由纯Swift实现，并且只依赖于 SwiftNIO 。这意味着即使不使用 Vapor ，也可以用于任何 Swift 项目中。

引入该 package，只需编辑 Package.swift 文件。

```swift
// swift-tools-version:4.0
import PackageDescription

let package = Package(
    name: "Project",
    dependencies: [
        ...
        .package(url: "https://github.com/vapor/websocket.git", from: "1.0.0"),
    ],
    targets: [
      .target(name: "Project", dependencies: ["WebSocket", ... ])
    ]
)
```

使用 ```import WebSocket``` 来访问相关的 API。

接下来将讲解 WebSocket package 能实现哪些功能。想要了解更多，可访问 [API docs](https://api.vapor.codes/websocket/latest/WebSocket/index.html) 。