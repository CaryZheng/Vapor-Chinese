# Basics

在Vapor中,默认路由是 EngineRouter。通过实现Router协议，可以实现一个自定义路由

~~~
let router = try EngineRouter.default()
~~~

这里有两个可用的API，一个是由Routing库提供的，另一个是由Vapor内部实现的

## 通过Routing注册一个路由

AsyncRouter 的 on 方法可以注册一个地址路由。例如下面是一个 GET /hello/world 的路由，并且会返回一个 "hello world!"

~~~swift
router.on(.get, to: "hello", "world") { request in
  return try Response(body: "Hello world!") 
}
~~~

对于可变路径你可以使用 [parameters](./parameters.md)

尾随闭包中会接收到一个 request。路由可以抛出error异常并且返回一个符合 future response 的响应

## 通过Vapor注册路由

Vapor中我们目前加入了对 .get,.put,.post,.patch 和 .delete 路由的支持

如果是可变路径，你也可以在这里使用[parameters](./parameters.md)

Vapor 有一个额外的好处就是，除了直接返回 Response 外还可以返回Future<ResponseRepresentable> 或者 Future<Response>

~~~swift
router.get("components", "in", "path") { request in
  return Response(status: .ok)
}
~~~

## 路由注册完成

在路由注册之后，你必须将router 添加到你的 services 中

~~~
services.register(router, as: Router.self)
~~~

学习更多有关services的内容，请前往 [Getting Started -> Services](../getting_started/services.md)