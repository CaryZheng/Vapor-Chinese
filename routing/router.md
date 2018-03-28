# Router

Router 是一个协议，你可以让你的自定义路由遵守这个协议

# 注册一个Route

首先创建一个路由，使用HTTP方法、路径和响应

下面是如何通过不变路径创建路由的示例

~~~swift
let responder = BasicAsyncResponder { request in
  return "Hello world"
}

let route = Route(method: .get, path: [.constant("hello"), .constant("world")], responder: responder)
~~~

下面是通过 [parameter](./parameters.md) 创建路由的示例

~~~swift
let responder = BasicSyncResponder { request in
  let name = try request.parameters.next(String.self)
  return "Hello \(name)"
}

let route = Route(method: .get, path: [.constant("greet"), .parameter(String.self)], responder: responder)
~~~

# 通过路由发送请求

假设你有一个request，你可以像下面这样做

~~~Swift
let request = Request(method: .get, URI(path: "/hello/world"))
~~~

让路由可以响应HTTP请求

~~~swift
let responder = router.route(request: request)
~~~
