# 控制器

控制器可以用来组织你的代码，它们是一系列用于接收 request 和返回 response 的方法集合。

通常情况下可以将控制器存放在 Controllers 目录下。


## 方法

让我们看一个控制器示例：

```
import Vapor

final class HelloController {
    func greet(_ req: Request) throws -> String {
        return "Hello!"
    }
}
```

控制器方法总是会接收 ```Request``` 并且返回 ```ResponseRepresentable``` 类型。

> 提示
> 
> 期望类型为 ```ResponseEncodable``` 的 ```Futures```（比如：```Future<String>```） 也是 ```ResponseEncodable``` 类型。

关于如何使用控制器，我们只需要简单初始化控制器，然后传递这个方法给路由器即可。

```
let helloController = HelloController()
router.get("greet", use: helloController.greet)
```

## 使用服务

如果想在控制器中访问 services ，只需要在路由闭包中以 ```Request``` 作为容器来创建 services 即可。 Vapor 将会负责缓存对应的 services。

```
final class HelloController {
    func greet(_ req: Request) throws -> String {
        return try req.make(BCryptHasher.self).hash("hello")
    }
}
```