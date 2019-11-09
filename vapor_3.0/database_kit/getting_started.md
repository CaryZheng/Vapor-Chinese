# Getting Started with Database Kit

Database Kit ([vapor/database-kit](https://github.com/vapor/database-kit)) 是一个用来配置和操作数据库的框架。它包含了一些核心服务，比如：缓存、日志和连接池。

> 提示
> 
> 如果使用 Fluent，你通常不需要手动使用 Database Kit。但是学习这些API可能以后会用得上。

## Package

Database Kit package 是轻量化的、全部由Swift实现，并且关联很少的依赖。这意味着即使不使用 Vapor ，也可以用于任何 Swift 项目中。

引入该 package，只需编辑 ```Package.swift``` 文件。

```swift
// swift-tools-version:4.0
import PackageDescription

let package = Package(
    name: "Project",
    dependencies: [
        ...
        .package(url: "https://github.com/vapor/database-kit.git", from: "1.0.0"),
    ],
    targets: [
      .target(name: "Project", dependencies: ["DatabaseKit", ... ])
    ]
)
```

使用 ```import DatabaseKit``` 来访问相应的 API 。

## API Docs

接下来将讲解 DatabaseKit package 能实现哪些功能。想要了解更多，可访问 [API docs](https://api.vapor.codes/database-kit/latest/DatabaseKit/index.html) 。