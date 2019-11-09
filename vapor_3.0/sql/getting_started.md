# Getting Started with SQL

SQL（[vapor/sql](https://github.com/vapor/sql)）是一个用于在 Swift 中构建和序列化 SQL 查询的模块。 它具有可扩展的基于协议的设计并支持 DQL，DML 和 DDL。

> 提示
> 
> 如果您使用Fluent，通常不需要手动构建SQL查询。

## Package

SQL 是一个轻量级的、纯 Swift 的、没有依赖的模块。 这意味着它可以用作任何 Swift 项目的 SQL 序列化框架，甚至是非 Vapor 的项目。

要将其包含在你的包中，请将以下内容添加到你的 Package.swift 文件中。

```swift
// swift-tools-version:4.0
import PackageDescription

let package = Package(
    name: "Project",
    dependencies: [
        ...
        .package(url: "https://github.com/vapor/sql.git", from: "1.0.0"),
    ],
    targets: [
      .target(name: "Project", dependencies: ["SQL", ... ])
    ]
)
```

使用 `import SQL` 来访问相关API。

本指南的其余部分将概述 SQL 模块中可用的内容。 与往常一样，随时访问 [API docs](http://api.vapor.codes/sql/latest/SQL/index.html) 以获取更多信息。



