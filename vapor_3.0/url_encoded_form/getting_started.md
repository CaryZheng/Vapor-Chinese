# Getting Started with URL-Encoded Form

`URL-Encoded Form` ([vapor/url-encoded-form](https://github.com/vapor/url-encoded-form)) 是一个可以帮助你解析和序列化 `application/x-www-form-urlencoded` 数据的小组件。URL-encoded forms 是网络上广泛支持的编码，它常用于序列化通过 POST 请求发送的 Web 表单.

`URL-Encoded Form` 模块通过集成 `Codable` 使你能够容易的使用此编码。

## Vapor

该软件包包含在 Vapor 中并默认导出。 导入 `Vapor` 时，您将可以访问所有`URLEncodedForm` API。

```swift
import Vapor
```

## Standalone

`URL-Encoded Form` 模块是一个轻量级、纯 Swift 、仅有少量依赖的模块。这意味着它可以和 `form-urlencoded` 数据被导入到任何 Swift 工程中，包括那些非 Vapor 工程。
要导入 `URL-Encoded Form` 模块，只需在你的 `Package.swift`文件中添加如下内容：

```swift
// swift-tools-version:4.0
import PackageDescription

let package = Package(
    name: "Project",
    dependencies: [
        ...
        .package(url: "https://github.com/vapor/url-encoded-form.git", from: "1.0.0"),
    ],
    targets: [
        .target(name: "Project", dependencies: ["URLEncodedForm", ...])
    ]
)
```

使用 `import URLEncodedForm` 来访问相关API。

> 警告
> 
> 本章可能含有一些 `Vapor特有` 的API，然而大部分通常适用于 `URL-Endoded Form` 模块。访问 [API Docs](https://api.vapor.codes/url-encoded-form/latest/URLEncodedForm/index.html) 获得更多这些API的信息。

