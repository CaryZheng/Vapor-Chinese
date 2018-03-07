# 路由

路由是用来查找 request 对应的 response。

## 创建路由器

Vapor中默认路由是 ```EngineRouter```。你可以通过实现```Router```协议来实现自定义路由。

```
let router = try EngineRouter.default()
```

有两个API可用，一个是由```Routing```库提供的，另一个是Vapor内置的。

我们推荐使用Vapor内置的API，下面将会继续给予说明。

## 注册路由

假设当访问```GET /users```时，你想要返回user列表。不考虑授权问题的话，代码将类似这样的。

```
router.get("users") { req in
    return // fetch the users
}
```

Vapor中路由通常以这些形式存在 ```.get```、```.put```、```.post```、```.patch```和```.delete```。你可以通过```/```来表示路径，也可以使用逗号还分割。我们推荐后者，因为它可读性更好。

```
router.get("path", "to", "something") { ... }
```

## 路由

最佳存放路由的位置是 routes.swift 文件。你将会在那找到一个可直接使用的路由器。

```
import Vapor

final class Routes: RouteCollection {
    ...

    func boot(router: Router) throws {
        router.get("hello") { req in
            return "Hello, world!"
        }
    }
}
```

你必须返回一个包含```ResponseEncodable```的 Future。通常```ResponseEncodable```类型是 Content、Response和View。

## 参数

有时候你可能想要路由中某个路径是动态的。这通常是用于要获取一个元素的id，比如```GET /users/:id```。

```
router.get("users", Int.parameter) { req -> Future<String> in
    let id = try req.parameter(Int.self)
    return // fetch the user with id
}
```

这里传的不是字符串，而是你期望的参数类型。在这里```User```有个```Int```类型的 ID 。

> 提示
> 
> 你也可以使用自定义参数类型。

## 注册路由之后

注册完路由后你必须注册这个路由器作为一个服务。