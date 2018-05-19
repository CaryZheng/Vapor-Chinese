# Using Services

本指南将向您展示如何注册，配置和创建您自己的service。在这个例子中，我们将假设两个不同的`Logger`实现。

- `PrintLogger`: 打印日志。
- `FileLogger`: 将日志保存到文件。 已经符合 `ServiceType`。

## Register

我们来看看我们如何注册我们的`PrintLogger`。首先，您必须将您的类型符合`Service`。最简单的方法是简单地在添加扩展中。

```swift
extension PrintLogger: Service { }
```

这是一个空的协议，所以应该没有什么方法需要实现。

### Factory

现在service可以注册到 [`Services`](https://api.vapor.codes/service/latest/Service/Structs/Services.html) 的结构里. 通常在[`configure.swift`](../getting-started/structure/#configureswift)完成。

```swift
services.register(Logger.self) { container in
	return PrintLogger()
}
```

通过使用工厂方法（闭包形式）来注册 `PrintLogger` , 我们允许 [`Container`](https://api.vapor.codes/service/latest/Service/Protocols/Container.html) 在需要时动态创建服务。 随后创建的任何 [`SubContainer`](https://api.vapor.codes/service/latest/Service/Protocols/SubContainer.html)s 都可以再次调用此方法来创建自己的 `PrintLogger`s.


### Service Type

为了更容易地注册服务，您可以将符合 [`ServiceType`](https://api.vapor.codes/service/latest/Service/Protocols/ServiceType.html)。

```swift
extension PrintLogger: ServiceType {
	/// See `ServiceType`.
    static var serviceSupports: [Any.Type] {
    	return [Logger.self]
    }

	/// See `ServiceType`.
    static func makeService(for worker: Container) throws -> PrintLogger {
    	return PrintLogger()
    }
}
```

符合 [`ServiceType`](https://api.vapor.codes/service/latest/Service/Protocols/ServiceType.html) 的服务只能使用类型名称进行注册。这也将自动符合 `Service` 。

```swift
services.register(PrintLogger.self)
```
### Instance

您还可以将预初始化（pre-initialized）的实例注册到 `Services`.

```swift
services.register(PrintLogger(), as: Logger.self)
```

> 警告
>
>	如果使用引用类型 (`class`) 则此方法将在  between all [`Container`](https://api.vapor.codes/service/latest/Service/Protocols/Container.html)s and [`SubContainer`](https://api.vapor.codes/service/latest/Service/Protocols/SubContainer.html)s 之间分享 _同一个_ 对象。 小心保护免受竞争条件。

## Configure

如果给定接口注册了两个或以上个数量的service，我们将需要选择使用哪种service。

```swift
services.register(PrintLogger.self)
services.register(FileLogger.self)
```

假设上述服务已注册，我们可以使用service [`Config`](https://api.vapor.codes/service/latest/Service/Structs/Config.html)。

```swift
switch env {
case .production: config.prefer(FileLogger.self, for: Logger.self)
default: config.prefer(PrintLogger.self, for: Logger.self)
}
```

这里我们使用 [`Environment`](https://api.vapor.codes/service/latest/Service/Structs/Environment.html) 动态的选择service. 通常在 [`configure.swift`](../getting-started/structure/#configureswift) 里完成。

> 注意
>
>	您还可以基于环境而不是使用服务配置（service config）动态 _register_ services。 
>	但是，选择service时是来自框架亦或是provider时必须采用service config方式。

## Create

当你已经注册了services 后，你可以使用一个 [`Container`](https://api.vapor.codes/service/latest/Service/Protocols/Container.html) 去创建它们。

```swift
let logger = try someContainer.make(Logger.self)
logger.log("Hello, world!")

// PrintLogger or FileLogger depending on the container's environment
print(type(of: logger)) 
```

> tip
>
>	通常框架将为您创建任何必需的容器。如果您想创建一个用于测试，您可以使用 [`BasicContainer`](https://api.vapor.codes/service/latest/Service/Classes/BasicContainer.html) 。
