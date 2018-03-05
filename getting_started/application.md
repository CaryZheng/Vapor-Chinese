# Application

每个 Vapor 项目都有个 ```Application``` 。开发过程中你可以使用 application 来创建任何你需要的服务。

最佳访问 application 的位置是 boot.swift 文件。

```
import Vapor

public func boot(_ app: Application) throws {
    // your code here
}
```

你也可以在 routes.swift 文件中访问 application 。它被存储在属性中。

```
import Vapor

final class Routes: RouteCollection {
    ...
}
```

不像其它web框架，Vapor不支持静态访问application。如果你需要从其它类或结构体访问application的话，你应该通过方法或初始构造器传递过去。

> 提示
> 
> 避免静态访问变量可以提高Vapor性能，因为这样的话不需要使用线程安全锁或信号量。

## Services

application 主要的用法是构造service。比如，在存储密码前，你可能需要```BCryptHasher```来哈希它们。你可以使用application来创建。

```
import BCrypt

let bcryptHasher = try app.make(BCryptHasher.self)
```

或者你可以通过application来创建HTTP Client。

```
let client = try app.make(Client.self)
let res = client.get("http://vapor.codes")
```

想要了解更多关于service的信息，可访问 Services → Getting Started 。