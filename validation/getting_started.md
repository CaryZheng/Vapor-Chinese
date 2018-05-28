# Getting Started with Validation

Validation([vapor/validation](https://github.com/vapor/validation)) 是用于验证发送至你的应用的数据的模块。它可以帮助验证如names、emails等内容，同时也是易于扩展的，允许你方便的的创建自定义验证。

本章将为你展示如何添加和导入 `Validation` 模块。更多此模块的信息，参考 [Validation->Overview](https://docs.vapor.codes/3.0/validation/overview/)。

## Vapor
该 package 包含在 Vapor 中并且默认是被导出的。导入 `Vapor` 后，你将可以访问所有的 `Validation` API。

```swift
import Vapor
```

## Standalone
`Validation`模块是一个轻量级、纯 Swift 、仅有少量依赖的模块。这意味着它可以作为一个单独的验证模块被导入到任何 Swift 工程中，包括那些非 Vapor 工程。
要导入`Validation`模块，只需在你的 `Package.swift`文件中添加如下内容：

```swift
// swift-tools-version:4.0
import PackageDescription

let package = Package(
    name: "Project",
    dependencies: [
        ...
        .package(url: "https://github.com/vapor/validation.git", from: "2.0.0"),
    ],
    targets: [
        .target(name: "Project", dependencies: ["Validation", ...])
    ]
)
```
使用 `import Validation` 来访问相关API。

> 警告
> 
> 本章可能含有一些 `Vapor特有` 的API，然而大部分通常适用于 `Validation` 模块。访问 [API Docs](https://api.vapor.codes/validation/latest/Validation/index.html) 获得更多 `Vapor特有` API的信息。


