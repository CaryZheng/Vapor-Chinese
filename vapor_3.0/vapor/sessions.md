# Using Session

本指南将向您展示如何使用Vapor中的会话来维护已连接客户端的状态。

会话的工作方式是为每个新客户端创建唯一标识符，并要求客户端为每个请求提供此标识符。 当收到下一个请求时，Vapor使用此唯一标识符来恢复会话数据。 这个标识符可以以任何格式传输，但它几乎总是用cookies来完成，这就是Vapor会话的工作方式。

当新客户端连接并设置会话数据时，Vapor将返回一个Set-Cookie头。 然后客户端在Cookie头中为每个请求设置新的值。 所有浏览器自动执行此操作。 如果您决定使会话失效，Vapor将删除任何相关数据并通知客户他们的Cookie不再有效。

# Middleware

使用会话的第一步是开启 [SessionsMiddleware](https://api.vapor.codes/vapor/latest/Vapor/Classes/SessionsMiddleware.html) ,这可以在整个应用的全局进行设置，也可以针对单独的路由进行设置


## Globally

全局开启会话需要添加中间件到你的 [MiddlewareConfig](https://api.vapor.codes/vapor/latest/Vapor/Structs/MiddlewareConfig.html) 中。

```swift
var middlewares = MiddlewareConfig.default()
middlewares.use(SessionsMiddleware.self)
services.register(middlewares)
```

这通常在 [configure.swift](../getting_started/folder_structure.md) 中完成

## Per Route

为路由中的某一组路由开启会话，使用 ```Router``` 中的 [grouped(...)](https://api.vapor.codes/vapor/latest/Vapor/Protocols/Router.html) 方法

```swift
// create a grouped router at /sessions w/ sessions enabled
let sessions = router.grouped("sessions").grouped(SessionsMiddleware.self)

// create a route at GET /sessions/foo
sessions.get("foo") { req in
    // use sessions
}
```

# Sessions

当 ```SessionsMiddleware``` 启动时，它会尝试创建 [Sesssions](https://api.vapor.codes/vapor/latest/Vapor/Protocols/Sessions.html) 和 [SessionsConfig](https://api.vapor.codes/vapor/latest/Vapor/Structs/SessionsConfig.html)。 Vapor将默认使用内存中会话。 您可以通过在 ```configure.swift``` 中注册这些服务来覆盖这两个服务。

您可以使用Fluent数据库（如MySQL，PostgreSQL等）或Redis等缓存来存储会话。 有关更多信息，请参阅相应的指南。

# Session

一旦启用了 ```SessionsMiddleware```，就可以使用[req.session()](https://api.vapor.codes/vapor/latest/Vapor/Classes/Request.html#/s:5Vapor7RequestC7sessionAA7SessionCyKF)来访问会话。 下面是一个简单的例子，它对会话中的 ```“name”``` 值进行简单的CRUD操作。

```swift
// create a route at GET /sessions/get
sessions.get("get") { req -> String in
    // access "name" from session or return n/a
    return try req.session()["name"] ?? "n/a"
}

// create a route at GET /sessions/set/:name
sessions.get("set", String.parameter) { req -> String in
    // get router param
    let name = try req.parameters.next(String.self)

    // set name to session at key "name"
    try req.session()["name"] = name

    // return the newly set name
    return name
}

// create a route at GET /sessions/del
sessions.get("del") { req -> String in
    // destroy the session
    try req.destroySession()

    // signal success
    return "done"
}
```

就是这样，恭喜你完成了Sesssion的使用