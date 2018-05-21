# Async 概览
可能你已经注意到Vapor中的一些API会期望或返回一个通用的`Future`类型。对于第一次听说`futures`的人起初会存在些困惑但别担心，Vapor会让它们变得很易用。

Promises 和 futures是有关联的，但也是截然不同的类型。Promises 用来创建futures。通常，你将使用Vapor的APIs所返回的future来工作从而不必担心创建promises。

| type      | description     | mutability     |methods|
| ---------- | :-----------:  | :-----------: |:-----------: |
| `Future`   |引用一个可能还是不可用的对象  |只读    |  `.map(to:_:)` `.flatMap(to:_:)` `do(_:) catch(_:)` |
| `Promise` | 提供一些异步的对象|可读写|`succeed(_:)` `fail(_:)`|
`Future`是基于回调的异步API的替代品,Futures可以链式以及转化来使用，这是简单闭包无法企及的方式，这使得Futures非常强大。

## Transforming
就像Swift中的可选项，期货可以被`mapped`以及` flat-mapped`。这些是你在futures上执行的最常见操作


| method      | signature     | description   |
| ---------- | :-----------:  | :-----------: |
|map|`to: U.Type, _: (T) -> U`| 把一个future Map成一个不同的值|
|flatMap|`to: U.Type, _: (T) -> Future<U>`|把一个future Map成一个不同的future值|
|transform|`to: U`|将一个future Map成一个已经可用值|

如果你查看`Optional<T>`和`Array <T>`上的`map`和`flatMap`的方法签名，您将看到它们与`Future <T>`上的方法非常相似

## Map
`.map（to：_ :)`方法允许你将future's value 转换为另一个value。由于future's value 可能还不可用（这个值可能是一个异步任务的结果），所以我们必须提供一个闭包来接受值。
```
/// 设想我们从某个API获取了一个future string
let futureString: Future<String> = ...

/// 将这个 future string转为 integer
let futureInt = futureString.map(to: Int.self) { string in
    print(string) // 实际的string
    return Int(string) ?? 0
}

/// 现在我们有了一个 future integer
print(futureInt) // Future<Int>
```

## Flat Map
`.flatMap(to:_:)`方法允许你将future's value 转化为另一个future's value。它以"flat"命名是因为他可以让你避免创建嵌套的futures(比如`FUture<Future<T>>`).换句话说，它可以帮助你保持展平的通用futures.
```
/// 假设我们从某个API获取一个 Future<String> 对象
let futureString: Future<String> = ...

/// 假设我们创建了一个http客户端
let client: Client = ... 

/// Flat-map 操作将Future<String>转为Future<Response>
let futureResponse = futureString.flatMap(to: Response.self) { string in
    return da
尽管我们已经提供了一个可用于转换的value，但这仍然是一种转变。在所有之前的futures完成（或失败）之前，future将不会完成

## Chanining
关于futures转化的重要部分是它们可以通过链式调用。这使你可以轻松表达许多转换和子任务。
让我们修改上面的例子，看看我们如何利用链式。

```
/// 结社我们获取一个future返回值从某个api
let futureString: Future<String> = ...

/// 假设我们已经初始化了一个客户端
let client: Client = ... 

/// 先转化为一个URL再转化为一个Response类型。
let futureResponse = futureString.map(to: URL.self) { string in
    guard let url = URL(string: string) else {
        throw Abort(.badRequest, reason: "Invalid URL string: \(string)")
    }
    return url
}.flatMap(to: Response.self) { url in
    return client.get(url)
}

print(futureResponse) //Future<Response>
```
初次调用`Map`之后，会创建一个临时`Future <URL>`类型，这个future对象会被立刻通过flat-map操作转为一个`Future<Response>`类型。
> 提示
> 
> 你可以在一个flat-map或者一个map的闭包内跑出一个错误，这将会使这个future失败并且抛出这个错误。

## Future
让我们来看看Future <T>上一些不常用的方法
 
### Do / Catch
与Swift的do / catch语法类似，futures也有一种do和catch方法来等待未来的结果。
```
/// Assume we get a future string back from some API
let futureString: Future<String> = ...

futureString.do { string in
    print(string) // 实际值
}.catch { error in
    print(error) // 一个Swift类型的错误
}
```
> 提示
>
> `.do`和`.catch`一起工作。如果你忘记`.catch`，编译器会警告你一个未使用的结果。不要忘记处理错误的情况！

### Always
你可以使用`always`来添加一个无论是失败还是成功都会被调用的回调。
```
/// Assume we get a future string back from some API
let futureString: Future<String> = ...

futureString.always {
    print("The future is complete!")
}
```
> 笔记
> 
> 你可以根据需求为future添加尽可能多的回调

### Wait
你可以使用.wait（）同步等待future完成。由于future可能会失败，所以这个调用会抛出异常。
 ```
 /// Assume we get a future string back from some API
let futureString: Future<String> = ...

/// 一直阻塞到string操作完成
let string = try futureString.wait()
print(string) /// String
 ```
> 警告
>
> 不要在一个vapor 的路由或者控制器中使用这个方法，阅读`Blocking`来获取更多信息。

## Promise
大多数情况下，你是通过调用vapor的API's来获取一个需要操作的future。但是，在某些时候，你可能需要创建自己的promise。

要创建promise，你需要访问`EventLoop`。Vapor中的所有容器都有一个可以使用的`eventLoop`属性。最常见的是，就是当前的`Request`。

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

> 信息
>
> 一个Promise对象只能够被完成一次，所有的后续完成操作都会被忽略。

### 线程安全

Promises可以从任何线程完成` (succeed(result:) / fail(error:)) `。这就是为什么promise需要一个事件循环来初始化。Promise确保完成操作在它的事件循环执行。

## Event Loop （事件循环）

它通常会为其运行的CPU中的每个内核创建一个事件循环在你的应用程序启动时。每个事件循环只有一个线程。如果你熟悉Node.js的事件循环，就会发现Vapor中的事件循环与Node.js中的非常相似。唯一的区别是Vapor可以在一个进程中运行多个事件循环，因为Swift支持多线程。

每次客户端连接到你的服务器时，它都将被分配给其中一个事件循环。从这一点开始，服务器和该客户端之间的所有通信都将发生在同一个事件循环（并关联到这个事件循环的线程）上。

事件循环负责跟踪每个连接的客户端的状态。如果有来自客户端的请求等待读取，则事件循环触发读取通知，导致数据被读取。一旦读取完整个请求，任何等待该请求数据的futures都将完成。

### Worker
有权访问事件循环的事物称为`Worker`。Vapor的每个容器都是一个worker。Vapor中最常用的容器是:
* `Application`
* `Request`
* `Response`

你可以在这些容器上使用`.eventLoo`p属性来访问事件循环。
```
print(app.eventLoop) // EventLoop
```
Vapor中有许多方法需要传递当前worker。它通常会有像`on: Worker`这样的标签。如果你处于路由封或控制器的闭包中，请传递当前`Request`或`Response`。如果你在启动应用程序时需要一个worker，请使用`Application`.

### Blocking（阻塞）
绝对关键的规则如下
> 危险
>
> 切勿直接在事件循环中进行阻塞调用。

一个阻塞调用的例子,类似于libc.sleep（_ :).
```
router.get("hello") { req in
    /// 使当前事件循环阻塞5s
    sleep(5)

    ///当线程重新工作室返回一个简单字符串
    return "Hello, world!"
}
```
`sleep（_ :)`是一个用于在提供的秒数内阻止当前线程的命令。如果你直接在事件循环中阻塞工作，则在阻塞工作期间，事件循环将无法响应分配给它的任何其他客户端。换句话说，如果你在事件循环中进行睡眠（5），则连接到该事件循环的所有其他客户端（可能为数百或数千个）将被延迟至少5秒

确保只有在后台才运行任何阻塞工作。当这项工作以非阻塞方式完成时，使用Promise通知事件循环。

```
router.get("hello") { req in
    /// 创建一个空的promise
    let promise = req.eventLoop.newPromise(Void.self)

    /// 在后台线程执行
    DispatchQueue.global() {
        /// 使后台线程睡眠
        /// 这不会影响到当前的事件循环
        sleep(5)

        /// 当阻塞工作完成时
        /// promise和他关联的future会被完成
        promise.succeed()
    }

    /// 等待 future完成
    /// 将结果转为一个简单的字符
    return promise.futureResult.transform(to: "Hello, world!")
   
}
```
并非所有的阻塞调用会像`sleep(:)`如此明显，当你怀疑你调用的方法可能被阻塞时，研究下这个方法本身或者询问相关的人。如果该函数正在执行磁盘或者网络IO并且使用的是一个同步API(无回调或者futures)，那么就会发生阻塞的情况。

> 信息
> 
> 如果阻塞工作是应用程序的核心部分，则应该考虑使用BlockingIOThreadPool来控制你创建的阻塞工作的线程数。这可以避免你的阻塞工作正在完成时从饿死CPU时间开始的事件循环。

