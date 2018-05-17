# Logging Overview

当你的app运行时，logging包提供便捷的API去记录日志信息。 [`Logger`](https://api.vapor.codes/console/latest/Logging/Protocols/Logger.html) 协议为记录日志声明了一些常用接口. 一个默认的 [`PrintLogger`](https://api.vapor.codes/console/latest/Logging/Classes/PrintLogger.html) 是可用的, 但是你也可以实现一个自定义的方法去记录日志信息。

## Log

首先，你会想用`Container`去创建一个`Logger`实例。然后你可以使用便捷方法去记录日志信息。

```swift
let logger = try req.make(Logger.self)
logger.info("Logger created!")
```

参阅API文档里关于 [`Logger`](https://api.vapor.codes/console/latest/Logging/Protocols/Logger.html) 的所有可用方法。

参考更多 [Service &rarr; Services](../service/services.md#instance) 的资料来看看怎么注册一个自定义logger。