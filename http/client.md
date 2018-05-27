# Using HTTPClient

HTTP客户端向远程HTTP服务器发送请求，然后生成并返回响应。 HTTP客户端通常只在几秒钟到几分钟内有效，并可能发送一个或多个请求。 [`HTTPClient`](https://api.vapor.codes/http/latest/HTTP/Classes/HTTPClient.html) 类型为Vapor高级客户端提供动力。本篇精简指南将向您展示如何手动向服务器发送HTTP请求。


> 提示
>
>	如果你使用Vapor，你可能不需要直接使用HTTP的API。参考 [Vapor &rarr; Client](../vapor/client.md) 以获得更便捷的API。

在这个例子中，我们将获取Vapor的主页。第一步是创建一个连接的HTTP客户端。 使用静态方法 [`connect(...)`](https://api.vapor.codes/http/latest/HTTP/Classes/HTTPClient.html#/s:4HTTP10HTTPClientC7connectXeXeFZ) 执行此操作。

```swift
// Connect a new client to the supplied hostname.
let client = try HTTPClient.connect(hostname: "vapor.codes", on: ...).wait()
print(client) // HTTPClient
// Create an HTTP request: GET /
let httpReq = HTTPRequest(method: .GET, url: "/")
// Send the HTTP request, fetching a response
let httpRes = try client.send(httpReq).wait()
print(httpRes) // HTTPResponse
```

请注意，我们正在传递 _hostname_ 。 这与完整的URL不同。 您可以使用Foundation的 `URL` 和 `URLComponents` 解析出主机名。  Vapor的便捷API自动执行此操作。

> 警告
>
>    指南假定你位于主线程中。如果你在route（路由）内部，不要使用 `wait()`。参见 [Async &rarr; Overview](../async/overview/#blocking) 以获取更多资料。

在我们连接HTTP客户端后，我们可以使用 [`HTTPRequest`](https://api.vapor.codes/http/latest/HTTP/Structs/HTTPRequest.html) 的 [`send(...)`](https://api.vapor.codes/http/latest/HTTP/Classes/HTTPClient.html#/s:4HTTP10HTTPClientC4sendXeXeF) 方法。 这将返回一个  [`HTTPResponse`](https://api.vapor.codes/http/latest/HTTP/Structs/HTTPResponse.html) 其中包含从服务器发回的标头和主体。参见 [HTTP &rarr; Message](message.md) 以获得更多关于HTTP消息的信息。

## API Docs

就这样！ 恭喜你做了第一个HTTP请求。 查看 [API docs](https://api.vapor.codes/http/latest/HTTP/index.html) 以获取所有有关可用参数和方法的资料。