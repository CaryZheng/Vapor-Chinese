# Async

你可能已经注意到一些 Vapor API 期望返回的是 Future 类型。如果你第一次听说 future 的话，可能会有些许困惑。但不要担心，Vapor 使得一切都变得简单了。

本篇指南将会介绍 Async ，详情可见 [Async → Overview](../async/overview.md)

## Futures

因为 ```Future``` 是异步执行的，我们必须使用闭包来进行交互并转换它们的值。就像 Swift 中的 optionals 一样， futures 可以被 mapped 和 flat-mapped 。

## Map

```.map(to:_:)``` 方法允许你将 future 的值转换成另一个值。一旦 ```Future``` 的数据准备就绪后，提供的闭包将会立即被调用。

```
/// Assume we get a future string back from some API
let futureString: Future<String> = ...

/// Map the future string to an integer
let futureInt = futureString.map(to: Int.self) { string in
    print(string) // The actual String
    return Int(string) ?? 0
}

/// We now have a future integer
print(futureInt) // Future<Int>
```

## Flat Map

```.flatMap(to:_:)``` 方法允许你将 future 的值转换成另一个 future 值。因为 flatMap 可以使你避免生成内嵌的 futures（比如：```Future<Future<T>>```），所以以 "flat" 来命名。换言之，这将使得你的 futures 更加扁平化。

```
/// Assume we get a future string back from some API
let futureString: Future<String> = ...

/// Assume we have created an HTTP client
let client: Client = ... 

/// Flat-map the future string to a future response
let futureResponse = futureString.flatMap(to: Response.self) { string in
    return client.get(string) // Future<Response>
}

/// We now have a future response
print(futureResponse) // Future<Response>
```

> 提示
> 
> 上述例子中如果使用 ```.map(to:_:)``` 替代的话，我们将会最终得到 ```Future<Future<Response>>``` 。

## Chaining

Futures 有这么一个特性，转换可以被 chained 。它允许你可以很容易的进行多次转换。

让我们修改上述示例：

```
/// Assume we get a future string back from some API
let futureString: Future<String> = ...

/// Assume we have created an HTTP client
let client: Client = ... 

/// Transform the string to a url, then to a response
let futureResponse = futureString.map(to: URL.self) { string in
    guard let url = URL(string: string) else {
        throw Abort(.badRequest, reason: "Invalid URL string: \(string)")
    }
    return url
}.flatMap(to: Response.self) { url in
    return client.get(url)
}

print(futureResponse) // Future<Response>
```

在初次调用 map 后，生成了个临时的 ```Future<URL>```。然后该 future 立即被 flat-mapped 成 ```Future<Response>``` 。

> 提示
> 
> 你可以在 map 和 flat-map 闭包中 ```throw``` 错误。这将引起 future 失败，并附带抛出来的错误信息。

## Worker

你也许会看到 Vapor 很多方法中都有一个 ```on: Worker``` 参数。这些方法通常是要执行异步任务，而且要访问 ```EventLoop``` 。

Vapor 中常见的 ```Workers``` 有如下几个：

* ```Application```
* ```Request```
* ```Response```

```
/// Assume we have a Request and some ViewRenderer
let req: Request = ...
let view: ViewRenderer = ...

/// Render the view, using the Request as a worker. 
/// This ensures the async work happens on the correct event loop.
///
/// This assumes the signature is:
/// func render(_: String, on: Worker)
view.render("home.html", on: req)
```