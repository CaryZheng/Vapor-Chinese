# Getting Started with Service

服务（Service ([vapor/service](https://github.com/vapor/service)) ） 是一个依赖注入（控制翻转）框架。它允许你以可维护的方式去注册、配置和创建应用程序的依赖关系。 

```swift
/// register a service during boot
services.register(PrintLogger.self, as: Logger.self)

/// you can then create that service later
let logger = try someContainer.make(Logger.self)
print(logger is PrintLogger) // true
```

你可以在维基百科上阅读到更多关于 [dependency injection](https://en.wikipedia.org/wiki/Dependency_injection) 的知识。 同时也请参阅 [Getting Started &rarr; Services](../getting-started/services.md) 中的指南。

## Vapor

这个包已包含了Vapor，并且默认导出。当你引入`Vapor`,你可以访问所有`Service`的API 。

```swift
import Vapor
```

## Standalone

这个Servece包轻量化，纯Swift，又少依赖。意思是它可以作为依赖注入框架用在其他Swift项目上，就算不是Vapor的Swift项目。

把它包含到你的包里，只需要添加到你的`Package.swift`文件里。

```swift
// swift-tools-version:4.0
import PackageDescription

let package = Package(
    name: "Project",
    dependencies: [
        ...
        .package(url: "https://github.com/vapor/service.git", from: "1.0.0"),
    ],
    targets: [
      .target(name: "Project", dependencies: ["Service", ... ])
    ]
)
```

用 `import Service` 去访问 APIs.

!!! 警告
    指南中的一部分内容会涉及Vapor特殊的APIs，但大部分都适用于Services包。
	
	访问 [API Docs](https://api.vapor.codes/service/latest/Service/index.html) 去获得 Service-specific API 的资讯.
