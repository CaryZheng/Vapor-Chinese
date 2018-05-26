# Async 入门
Async是作为Vapor Core([vapor/core](https://github.com/vapor/core))的一部分来提供使用。它是基于SwiftNIO([SwiftNIO](https://github.com/apple/swift-nio))构建的便利的API（主要是扩展）集合。
> 提示 
>
>你可以阅读更多关于SwiftNIO的异步类型 (Future, Promise, EventLoop, 以及更多)in its GitHub ([README](https://github.com/apple/swift-nio/blob/master/README.md)) or its ([API Docs](https://apple.github.io/swift-nio/docs/current/NIO/index.html))。 


## 用法
该模块包包含在`Vapor`中并默认导出。导入`Vapor`时，你将可以访问所有的`Async` API。
```
import Vport //这同时意味着 `import Async`
```
## 独立性
`Async`模块是Vapor Core包的一部分，但是你也可以与单独导入`Async`与任何Swift项目一起使用。
要将其包含在你的包中，请将以下内容添加到你的`Package.swift`文件中
```
// swift-tools-version:4.0
import PackageDescription

let package = Package(
    name: "Project",
    dependencies: [
        ...
        .package(url: "https://github.com/vapor/core.git", from: "3.0.0"),
    ],
    targets: [
      .target(name: "Project", dependencies: ["Async", ... ])
    ]
)
```
使用`import Async`来访问APIs。
## 概览
了解异步功能的概述可见[ Async → Overview](overview.md)。
