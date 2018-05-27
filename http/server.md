# Using HTTPServer

HTTP 服务器使用 [`HTTPResponses`](https://api.vapor.codes/http/latest/HTTP/Structs/HTTPResponse.html) 响应 [`HTTPRequests`](https://api.vapor.codes/http/latest/HTTP/Structs/HTTPRequest.html)。 [`HTTPServer`](https://api.vapor.codes/http/latest/HTTP/Classes/HTTPServer.html) 类型为Vapor更高级别的server提供动力。本篇精简指南将向您展示如何手动设置您自己的HTTP服务器。

> 提示
>
>	如果你使用Vapor，你可能不需要直接使用HTTP的API。参考 [Vapor &rarr; Server](../vapor/server.md) 以获得更方便的API。

## Responder

创建一个HTTP服务器很简单，只需要几行代码。第一步是创建一个 [`HTTPServerResponder`](https://api.vapor.codes/http/latest/HTTP/Protocols/HTTPServerResponder.html)。 这将直接负责生成对传入请求的响应。

让我们创建一个响应请求内容的简单响应。

```swift
/// Echoes the request as a response.
struct EchoResponder: HTTPServerResponder {
	/// See `HTTPServerResponder`.
    func respond(to req: HTTPRequest, on worker: Worker) -> Future<HTTPResponse> {
    	// Create an HTTPResponse with the same body as the HTTPRequest
    	let res = HTTPResponse(body: req.body)
    	// We don't need to do any async work here, we can just
    	// se the Worker's event-loop to create a succeeded future.
        return worker.eventLoop.newSucceededFuture(result: res)
    }
}
```

## Start

现在我们有了一个响应，我们可以创建我们的 [`HTTPServer`](https://api.vapor.codes/http/latest/HTTP/Classes/HTTPServer.html). 我们只需要选择要绑定的服务器的主机名和端口。在这个例子中，我们将绑定到 `http://localhost:8123`.

```swift
// Create an EventLoopGroup with an appropriate number
// of threads for the system we are running on.
let group = MultiThreadedEventLoopGroup(numThreads: System.coreCount)
// Make sure to shutdown the group when the application exits.
defer { try! group.syncShutdownGracefully() }

// Start an HTTPServer using our EchoResponder
// We are fine to use `wait()` here since we are on the main thread.
let server = try HTTPServer.start(
	hostname: "localhost", 
	port: 8123, 
	responder: EchoResponder(), 
	on: group
).wait()

// Wait for the server to close (indefinitely).
try server.onClose.wait()
```

静态方法 [`start(...)`](https://api.vapor.codes/http/latest/HTTP/Classes/HTTPServer.html#/s:4HTTP10HTTPServerC5startXeXeFZ) 创建并返回一个新的异步 [`HTTPServer`](https://api.vapor.codes/http/latest/HTTP/Classes/HTTPServer.html) 。Future将在服务器成功引导时完成，否则会在出现问题时生成错误信息。

一旦future启动完成，我们的服务器就在运行。通过等待服务器的 `onClose` future 完成， 我们可以让我们的应用程序保持活跃状态​​，直到服务器关闭。通常情况下，服务器不会自动关闭--它只会无限期地运行。但是，如果调用了 `server.close()` ，应用程序可以正常退出。

## API Docs

就这样！ 恭喜你创建了第一个HTTP服务器和响应。查看 [API docs](https://api.vapor.codes/http/latest/HTTP/index.html) 以获取所有有关可用参数和方法的资料。