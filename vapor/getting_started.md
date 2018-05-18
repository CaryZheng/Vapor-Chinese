# getting Started with Vapor

查看[Getting Started](../getting_started/)主要的入门指南，其中专门介绍了Vapor。此页面主要是为了与其他软件包保持一致

关于Vapor中包含的API的更深入信息，请参阅左侧的小节

# Package

如果你不想使用Vapor的初始化模板，你也可以手动引入vapor库

~~~Swift
// swift-tools-version:4.0
import PackageDescription

let package = Package(
    name: "Project",
    dependencies: [
        ...
        .package(url: "https://github.com/vapor/vapor.git", from: "3.0.0"),
    ],
    targets: [
      .target(name: "Project", dependencies: ["Vapor", ... ])
    ]
)
~~~

使用 **import Vapor** 来获得API调用权限

# API 文档

这份指南将简单介绍Vapor中可用的功能。和通常一样，通过预览 [API 文档](https://api.vapor.codes/vapor/latest/Vapor/index.html) 来获取更多信息