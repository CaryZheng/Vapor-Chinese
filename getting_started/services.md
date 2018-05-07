# Services

Services 是一个用于 Vapor 的依赖注入（也叫做反向控制）框架。这个 services 框架允许你去注册、配置和初始化任何应用中可能需要的服务。

## Container

大多数是通过容器来进行服务之间的交互的。一个容器是由以下构成的：

* Services: 一系列已注册的服务
* Config: 特定服务的配置
* Environment: 应用当前环境（测试、生产环境等等）
* Worker: 关联容器的 event loop

Vapor使用的大多数容器是：

* ```Application```
* ```Request```
* ```Response```

启动应用时你应该使用 ```Application``` 来作为容器去创建必要的服务。使用 ```Request``` 和 ```Response``` 容器来创建响应 request 的服务（在路由闭包和控制器中）。

### Make

创建服务很简单，只需要在容器上调用 ```.make(_:)``` 并传递你想要的类型，该类型通常是一个协议，比如：```Client```。

```
let client = try req.make(Client.self)
```

如果你明确知道想要哪个服务，也可以直接指定具体的类型。

```
let leaf = try req.make(LeafRenderer.self)
print(leaf) /// Definitely a LeafRenderer

let view = try req.make(ViewRenderer.self)
print(view) /// ViewRenderer, might be a LeafRenderer
```

> 提示
> 
> 尽可能的依赖于协议而不是具体类型。这将会更容易测试你的代码（可以很容易的切换具体实现），可以对代码进行解耦。

## Services

```Services``` 结构体包含了所有已注册过的服务（包括你自己或service提供者的），你将经常需要在 ```configure.swift``` 文件中注册配置你的服务。

### Instance

你可以使用 ```.register(_:)``` 来注册初始化服务实例。

```
/// Create an in-memory SQLite database
let sqlite = SQLiteDatabase(storage: .memory)

/// Register to sevices.
services.register(sqlite)
```

注册完一个服务后，可以通过 ```Container``` 来创建。

```
let db = app.make(SQLiteDatabase.self)
print(db) // SQLiteDatabase (the one we registered earlier)
```

### Protocol

注册服务时，你也可以约束一个具体协议。你可能已经注意到 Vapor 是如何注册它的路由器的。

```
/// Register routes to the router
let router = EngineRouter.default()
try routes(router)
services.register(router, as: Router.self)
```

因为使用了 ```as: Router.self``` 来注册 ```router```，所以我们可以使用具体类型或协议来创建它。

```
let router = app.make(Router.self)
let engineRouter = app.make(EngineRouter.self)
print(router) // Router (actually EngineRouter)
print(engineRouter) // EngineRouter
print(router === engineRouter) // true
```

## Environment

Environment 用来动态改变 Vapor app 在特定场景下的行为。比如，应用部署后你可能想使用不同的用户名和密码用于数据库。使用 ```Environment``` 就可以很容易地管理起来。

当从命令行运行 Vapor app 时，你可以传递 ```--env``` 参数来指定环境。默认情况下当前环境会是```.development```。

```
swift run Run --env prod
```

上述例子中，我们以 ```.production``` 环境来运行 Vapor。该环境指定了 ```isRelease = true``` 。

你可以使用传递到 ```configure.swift``` 中的 environment 来动态注册服务。

```
let sqlite: SQLiteDatabase
if env.isRelease {
    /// Create file-based SQLite db using $SQLITE_PATH from process env
    sqlite = try SQLiteDatabase(storage: .file(path: Environment.get("SQLITE_PATH")!))
} else {
    /// Create an in-memory SQLite database
    sqlite = try SQLiteDatabase(storage: .memory)
}
services.register(sqlite)
```

> 提示
> 
> 使用静态方法 ```Environment.get(_:)``` 来获取环境的字符串值。

你也可以基于 environment 使用 ```.register(_:)``` 方法来动态注册服务。

```
services.register { container -> BCryptConfig in
  let cost: Int

  switch container.environment {
  case .production: cost = 12
  default: cost = 4
  }

  return BCryptConfig(cost: cost)
}
```

## Config

如果多个服务都适用于给定的协议，你需要使用 ```Config``` 来声明你更想用哪个。

```
ServiceError.ambiguity: Please choose which KeyedCache you prefer, multiple are available: MemoryKeyedCache, FluentCache<SQLiteDatabase>.
```

这也可以在 ```configure.swift``` 中完成，只需要使用 ```config.prefer(_:for:)``` 方法。

```
/// Declare preference for MemoryKeyedCache anytime a container is asked to create a KeyedCache
config.prefer(MemoryKeyedCache.self, for: KeyedCache.self)

/// ...

/// Create a KeyedCache using the Request container
let cache = req.make(KeyedCache.self)
print(cache is MemoryKeyedCache) // true
```