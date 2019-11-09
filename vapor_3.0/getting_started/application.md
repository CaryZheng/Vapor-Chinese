# Application

每个 Vapor 项目都有个 ```Application``` 。启动时你可以使用 application 来创建任何你需要的服务。

最佳访问 application 的位置是 boot.swift 文件。

```
import Vapor

public func boot(_ app: Application) throws {
    // your code here
}
```

不像其它web框架，Vapor不支持静态访问application。如果你需要从其它类或结构体访问application的话，你应该通过方法或初始构造器传递过去。

> 提示
> 
> 避免静态访问变量可以提高Vapor性能，因为这样的话不需要使用线程安全锁或信号量。

## Services

application 主要功能是启动你的服务。

```
try app.run()
```

然而，application也是一个容器，你可能需要使用它来创建所需要的各种服务。

> 提示
> 
> 不要在路由闭包里直接使用application或任何由它创建的服务。取而代之，应该使用 ```Request``` 来创建服务。

```
let client = try app.make(Client.self)
let res = try client.get("http://vapor.codes").wait()
print(res) // Response
```

> 提示
> 
> 因为不在路由闭包里，所以这里可以使用 ```.wait()``` 来代替 ```.map``` / ```.flatMap```。

想要了解更多关于service的信息，可访问 [Services → Getting Started](services.md) 。