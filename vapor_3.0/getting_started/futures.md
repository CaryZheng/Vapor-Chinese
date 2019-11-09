# Futures

你可能已经注意到一些 Vapor API 返回的是 ```Future``` 类型。如果你第一次听说 futures，你可能会有些许困惑。但无需担心，Vapor 使用起来很便捷。

> 提示
> 
> Vapor 使用的 promises, futures 和 event loops均来自于 [Swift NIO](https://github.com/apple/swift-nio) 库。这篇文章涉及到一些基础，也有Vapor添加的类型别名和扩展，目的是为了使用起来更方便。

promises 和 futures 是相互关联的，但是是不同的类型。promises 用于创建 futures。大多数时候，你将会使用 Vapor API 返回的 futures，而不需要自己去创建 promises。

type | description | mutability | methods
----|----|----|----
```Future``` | 引用一个还未就绪的对象 | 只读 | ```.map(to:_:)``` ```.flatMap(to:_:)``` ```do(_:)``` ```catch(_:)```
```Promise``` | 提供一些异步对象 | 可读可写 | ```succeed(_:)``` ```fail(_:)```

futures 基于异步回调API。futures 很强大，可以很方便的被 chained 和 transformed，而闭包却无法实现。

## Transforming

就像 Swift 中的可选类型，futures可以被 mapped 和 flat-mapped。以下是 futures 最常用的操作符。

method | signature | description
----|----|----
```map``` | ```to: U.Type, _: (T) -> U``` | Maps a future value to a different value.
```flatMap``` | ```to: U.Type, _: (T) -> Future<U>``` | Maps a future value to different future value.
```transform``` | ```to: U``` | Maps a future to an already available value.

如果你查看 ```Optional<T>``` 和 ```Array<T>``` 上的 ```map``` 和 ```flatMap``` 语法，你将会发现与 ```Future<T>``` 很相似。

### Map

```.map(to:_:)``` 方法允许你将 future 的 value 转换为另一个 value。因为 future 的值可能还未就绪（它可能是一个异步任务的结果），所以我们必须提供一个闭包来接收这个值。

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

### Flat Map

```.flatMap(to:_:)``` 方法允许你将 future 的 value 转换为另一个 future value。它叫做 ```flat``` 因为这样可以避免创建嵌套的 futures （比如：```Future<Future<T>>```）。换言之，它能帮助你保持通用的 futures flat。

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
> 如果在上述例子中使用 ```.map(to:_:)```，我们将会返回 ```Future<Future<Response>>``` 类型。

### Transform

```.transform(_:)``` 方法允许你修改 future 的 value，来替代已存在的 value。这对于 ```Future<Void>``` 的转换是很有帮助的。

> 提示
> 
> ```Future<Void>``` 有时候也被叫做信号，被用于通知某些异步操作的完成或失败。

```
/// Assume we get a void future back from some API
let userDidSave: Future<Void> = ...

/// Transform the void future to an HTTP status
let futureStatus = userDidSave.transform(to: HTTPStatus.ok)
print(futureStatus) // Future<HTTPStatus>
```

尽管我们提供了已经就绪的value给 ```transform```，但这仍然是一个转化，所以之前所有的 future 都完成（或失败）后该 future 才会结束。

## Chaining

future 上的转换是可以被 chained，这使得你可以表达多种转化和子任务。

让我们修改上面的例子，来看看如何利用 chaining。

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

在 map 初始被调用后，会生成一个临时的 ```Future<URL>```。该 future 会被立即 flat-mapped 成 ```Future<Response>```。

> 提示
> 
> 你可以在 map 和 flat-map 闭包内 ```throw``` 错误。这将导致 future 触发失败，并且携带着错误信息。

### Future

让我们看看其它的，通常很少被使用到的 ```Future<T>``` 方法。

#### Do / Catch

与 Swift ```do``` / ```catch``` 语法类似，future有个 ```do``` 和 ```catch``` 方法用于等待 future 的结果。

```
/// Assume we get a future string back from some API
let futureString: Future<String> = ...

futureString.do { string in
    print(string) // The actual String
}.catch { error in
    print(error) // A Swift Error
}
```	

> 提示
> 
> ```.do``` 和 ```.catch``` 一起协作。如果你忘记调用 ```.catch```，编译器将会警告你未使用返回结果。不要忘记处理错误情况！

#### Always

你可以使用 ```always```  来增加一个回调，无论 future 成功还是失败，该方法都会被执行。

```
/// Assume we get a future string back from some API
let futureString: Future<String> = ...

futureString.always {
    print("The future is complete!")
}
```

> 提示
> 
> 你可以给 future 增加很多回调。

#### Wait

你可以使用 ```.use``` 来同步等待 future 完成。因为 future 是有可能失败的，所以该调用会抛出异常。

```
/// Assume we get a future string back from some API
let futureString: Future<String> = ...

/// Block until the string is ready
let string = try futureString.wait()
print(string) /// String
```

> 提示
> 
> 不要在路由闭包或控制器中使用该方法。详情可见 Blocking 章节。

## Promise

大多数时候，你只需要转换 Vapor API 返回的 futures即可。然而，有时候你可能需要创建自己的 promise 。

创建 promise ，你将需要访问 ```EventLoop```。所有 Vapor 的容器中都有 ```eventLoop``` 属性可用。通常，这将会是当前的 ```Request``` 。

```
/// Create a new promise for some string
let promiseString = req.eventLoop.newPromise(String.self)
print(promiseString) // Promise<String>
print(promiseString.futureResult) // Future<String>

/// Completes the associated future
promiseString.succeed(result: "Hello")

/// Fails the associated future
promiseString.fail(error: ...)
```

> 提示
> 
> 一个 promise 只能被完成一次。任何后来的完成都将会被忽略。

### Thread Safety

Promises 可以在任意线程中被完成（```succeed(result:)``` / ```fail(error:)```）。这就是为什么 promises 需要一个 event-loop 来初始化。Promises确保了完成和执行都是在同一个 event-loop 。

## Event Loop

当你的应用启动时，它将会为每个 CPU 核都创建一个 event loop 。每个 event loop 有一个线程。Vapor 中的 event loop 与 Node.js 中的 event loop 很类似。唯一不同点就是 Vapor 可以在一个进程中运行多个 event loop，因为 Swift 支持多线程。

每次一个客户端连接上服务端，都将分配一个 event loop 。所以服务端和客户端所有的交互都是发生在同一个 event loop 中（其实也就是 event loop 线程）。

event loop 负责追踪每个被连接的客户端状态。如果有个来自客户端的请求等待被读取，这个 event loop 将会触发一个读取通知，使得该数据可以被读取。一旦完整的请求被读取，任何等待该请求数据的 futures 将会被完成。

### Worker

我们把可以访问 event loop 的事物称作 ```Workers```。Vapor 中的每个容器都是一个 worker。

Vapor 中接触最多的容器是：

* Application
* Request
* Response

你可以通过这些容器的 ```.eventLoop``` 属性来访问 event loop。

```
print(app.eventLoop) // EventLoop
```

Vapor 中有许多方法需要传递当前的 worker。经常是以这样的形式 ```on: Worker``` 出现。如果你在一个路由闭包或者控制器中，可以传递当前的 ```Request``` 或 ```Response```。启动 app 时如果需要一个 worker，可以使用 ```Application``` 。

### Blocking

有个很重要的规则一定要遵守：

> 提示
> 一定不要在一个 event loop 中直接阻塞调用。

以下是一个阻塞调用的示例，使用的是 ```libc.sleep(_:)``` 。

```
router.get("hello") { req in
    /// Puts the event loop's thread to sleep.
    sleep(5)

    /// Returns a simple string once the thread re-awakens.
    return "Hello, world!"
}
```

```sleep(_:)``` 是一个用来阻塞当前线程的命令，根据传入的时间参数来决定阻塞几秒。如果你直接在一个 event loop 中阻塞任务，这个 event loop 将没有能力再去响应任何其它分配过来的客户端。换言之，如果你在一个 event loop 中 ```sleep(5)```（可能成百上千） ，所有处在这个 event loop 上已连接的客户端都将会被推迟至少 5 秒。

确保运行所有阻塞式任务都在后台运行。当该任务以非阻塞方式完成后，使用 promises 去通知 event loop。

```
router.get("hello") { req in
    /// Create a new void promise
    let promise = req.eventLoop.newPromise(Void.self)

    /// Dispatch some work to happen on a background thread
    DispatchQueue.global() {
        /// Puts the background thread to sleep
        /// This will not affect any of the event loops
        sleep(5)

        /// When the "blocking work" has completed,
        /// complete the promise and its associated future.
        promise.succeed()
    }

    /// Wait for the future to be completed, 
    /// then transform the result to a simple String
    return promise.futureResult.transform(to: "Hello, world!")
}
```

不是所有的阻塞调用都像 ```sleep(_:)``` 那样明显。如果你有疑虑一个调用是否是阻塞的，可以查阅该方法本身或询问他人。如果这个方法是处理磁盘或网络 IO 操作，这时候使用同步 API（没有回调或 futures），那它就是阻塞的。

> 提示
> 
> 如果你的应用中，需要大量阻塞操作的话，你需要考虑使用 ```BlockingIOThreadPool``` 来控制线程数量，并用来处理阻塞任务。这将避免当阻塞任务完成后，你的 event loop 获取不到 CPU 时间。