# Using Providers

[`Provider`](https://api.vapor.codes/service/latest/Service/Protocols/Provider.html) 协议使你更方便的集成外部服务到你的应用中。 所有Vapor的官方包，像[Fluent](../fluent/getting-started.md) 使用provider系统来暴露他们的服务。

Providers 能做:

- 注册服务到你的 [`Services`](https://api.vapor.codes/service/latest/Service/Structs/Services.html) 结构里.
- 连接你的 [`Container`](https://api.vapor.codes/service/latest/Service/Protocols/Container.html) 的生命周期.

## Register

一旦你添加了 Service-exposing [SPM dependency](../getting-started/spm/#dependencies) 到你的项目里, 添加 provider 是很简单的事情了.

```swift
import Foo

try services.register(FooProvider())
```

通常在 [`configure.swift`](../getting-started/structure/#configureswift)完成。

!!! 注意
	你在Github上搜索 [`vapor-service`](https://github.com/topics/vapor-service) 标签就可以搜索到一系列使用此方法来暴露Services的包。 


## Create

创建一个自定义的provider可以更好的管理你的代码。如果你在制作Vapor的第三方包，也可以创建一个provider。

这有一个类似于`Logger`的简单的provider例子。例子来自于 [Services](services.md) 章节。

```swift
public final class LoggerProvider: Provider {
    /// See `Provider`.
    public func register(_ services: inout Services) throws {
		services.register(PrintLogger.self)
		services.register(FileLogger.self)
    }
    
    /// See `Provider`.
    public func didBoot(_ container: Container) throws -> Future<Void> {
    	let logger = try container.make(Logger.self)
    	logger.log("Hello from LoggerProvider!")
    	return .done(on: container)
    }
}
```

现在，当有人注册了`LoggerProvider`到它的`LoggerProvider`结构里，它将会自动注册打印和文件日志功能。当容器启动时，成功的消息将会被打印出来，以验证provider已添加好了。

参阅 [`Provider`](https://api.vapor.codes/service/latest/Service/Protocols/Provider.html) 协议的API文档来获取更多资料。