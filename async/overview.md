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

