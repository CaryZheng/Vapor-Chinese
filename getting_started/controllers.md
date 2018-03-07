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

控制器方法总是会接收 ```Request``` 并且返回 ```ResponseRepresentable``` 类型。这也包含了 futures ，它也遵循 ```ResponseRepresentable```（比如：```Future<String>```）。

关于如何使用控制器，我们只需要简单初始化控制器，然后传递这个方法给路由器即可。

```
let helloController = HelloController()
router.get("greet", use: helloController.greet)
```

## 使用服务

你可能会想在控制器中访问应用内的服务，很幸运的话这很容易做到。首先，在初始化方法里声明该控制器想要的服务，然后以属性的形式保存它们。

```
final class HelloController {
    let hasher: BCryptHasher

    init(hasher: BCryptHasher) {
        self.hasher = hasher
    }

    ...
}
```

接下来，当初始化控制器的时候，使用 application 来创建这些服务。

```
let helloController = try HelloController(
    hasher: app.make()
)
```