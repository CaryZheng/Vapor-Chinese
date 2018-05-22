# Getting Started with Multipart

Multipart ([vapor/multipart](https://github.com/vapor/multipart)) 是一个小型包， 可以帮助你解析和序列化 `multipart` 编码了的数据. Multipart 是网络上广泛的支持的编码。 它常用于序列化Web表单，特别是包含图像等富媒体的表单。

Multipart 包通过直接与`Codable`集成可以很容易地使用这种编码。

## Vapor

这个package已经被包含在 Vapor中并默认导出。当你导入`Vapor`时,就可以获得所有 `Multipart` API的调用权限。

```swift
import Vapor
```

## Standalone

Multipart package 是一个轻量级，少依赖的Swift组件库。这意味着你可以在不引入Vapor的情况下，在任何Swift项目中使用multipart-encoded data。

要引入Multipart到你的项目中，可以添加下面代码到你的 `Package.swift` 文件里。

```swift
// swift-tools-version:4.0
import PackageDescription

let package = Package(
    name: "Project",
    dependencies: [
        ...
        .package(url: "https://github.com/vapor/multipart.git", from: "3.0.0"),
    ],
    targets: [
      .target(name: "Project", dependencies: ["Multipart", ... ])
    ]
)
```

使用 `import Multipart` 去使用所有的APIs。

!!! 警告
	指南中的一部分内容会涉及Vapor特有的APIs，但大部分情况下都适用于Multipart package。
	通过访问 [API Docs](https://api.vapor.codes/multipart/latest/Multipart/index.html) 去获得更多关于 Multipart-specific API 的资料。
