# Using HTTP Message

有两种类型的HTTP消息， [`HTTPRequest`](https://api.vapor.codes/http/latest/HTTP/Structs/HTTPRequest.html) 和 [`HTTPResponse`](https://api.vapor.codes/http/latest/HTTP/Structs/HTTPResponse.html)。 大多数情况下，它们非常相似，但有一些差异。

## Request

HTTP请求由客户端发送到服务器，并且它们应始终只收到一个HTTP响应。 HTTP请求通过标准HTTP消息包含两个唯一字段：

- [`method`](https://api.vapor.codes/http/latest/HTTP/Structs/HTTPRequest.html#/s:4HTTP11HTTPRequestV6methodXev)
- [`url`](https://api.vapor.codes/http/latest/HTTP/Structs/HTTPRequest.html#/s:4HTTP11HTTPRequestV3urlXev)

该方法和URL定义了服务器上正在请求的内容。

```swift
/// GET /hello
let httpReq = HTTPRequest(method: .GET, url: "/hello")
```

您可以在初始化HTTP请求时定义这些参数，或者如果请求可变，则可以稍后进行设置。

```swift
var httpReq: HTTPRequest = ...
httpReq.method = .POST
httpReq.url = URL(...)
```

您可以使用Foundation的`URLComponents`从其基本组件创建URL。 HTTP请求也有一个属性 [`urlString`](https://api.vapor.codes/http/latest/HTTP/Structs/HTTPRequest.html#/s:4HTTP11HTTPRequestV9urlStringSSv) 您可以使用它来手动设置一个自定义URL`String `，而不需要通过`URL`。

以下是一个序列化的HTTP请求的样子。这个是查询`/hello`。

```http
GET /hello HTTP/1.1
Content-Length: 0
```

## Response

HTTP响应由服务器响应HTTP请求而生成。 HTTP响应在通用HTTP消息中只有一个唯一字段：

- [`status`](https://api.vapor.codes/http/latest/HTTP/Structs/HTTPResponse.html#/s:4HTTP12HTTPResponseV6statusXev)

HTTP状态用于通知客户端有什么错误。状态由状态码和错误原因组成。代码总是三位数字，原因是解释代码的短字符串。您可以在 [httpstatuses.com](https://httpstatuses.com) 上查看所有状态码。

```swift
let httpRes = HTTPResponse(status: .ok, body: "hello")
```

所有常用的HTTP状态都会有预定义的值，比如`.ok`代表`200 OK`。您还可以定义自己的自定义状态代码。

您可以在初始化HTTP响应时定义状态，或者如果响应是可变的，则可以稍后进行设置

```swift
var httpRes: HTTPResponse = ...
httpRes.status = .notFound
```

这是一个序列化的HTTP响应的例子。

```http
HTTP/1.1 200 OK
Content-Length: 5
Content-Type: text/plain

hello
```

## Headers

每个HTTP消息都有一个header集。header包含关于消息的元数据并帮助解释消息正文中的内容。 

```http
Content-Length: 5
Content-Type: text/plain
```

必须至少有一个 `"Content-Length"` 或者 `"Transfer-Encoding"` headers来定义消息正文的长度。几乎总是有一个 `"Content-Type"` headers，它解释了正文包含的数据的 _type_ 。还有许多其他常见的headers，如指定消息何时被创建的 `"Date"` 等等。

您可以使用“headers”属性访问HTTP消息的headers。

```swift
var message: HTTPMessage ...
message.headers.firstValue(for: .contentLength) // 5
```

如果您正在使用常见的HTTP header进行交互，则可以使用便捷的HTTP名称而不是原始的`String`。

## Body

HTTP消息可以有一个包含任意数据的 [`HTTPBody`](https://api.vapor.codes/http/latest/HTTP/Structs/HTTPBody.html) 。 这些数据可以是静态的也可以是流式的，并且可以是任何你想要的格式。使用 [`contentType`](https://api.vapor.codes/http/latest/HTTP/Protocols/HTTPMessage.html#/s:4HTTP11HTTPMessagePAAE11contentTypeXev) header来描述数据的类型。

```swift
var message: HTTPMessage = ...
message.body = HTTPBody(string: "Hello, world!")
message.contentType = .plainText
```

> 提示
>
>    如果需要，设置 `body` 属性将会自动更新 `"Content-Length"` or `"Transfer-Encoding"` 的headers.
 
```swift
var message: HTTPMessage = ...
message.body = HTTPBody(string: """
{"message": "Hello, world!"}
""")
message.contentType = .json
```

## Codable

定义了两个协议，以便使用HTTP协议的 `Codable` :

- [`HTTPMessageEncoder`](https://api.vapor.codes/http/latest/HTTP/Protocols/HTTPMessageEncoder.html)
- [`HTTPMessageDecoder`](https://api.vapor.codes/http/latest/HTTP/Protocols/HTTPMessageDecoder.html)

这两个编码器允许您将自定义的`Codable`类型编码和解码到HTTP正文中，并设置适当的内容类型headers。

默认情况下，HTTP为`JSONEncoder`和`JSONDecoder`提供了一致性，但Vapor包含更多类型的编码器。

下面是一个将`Codable`结构编码为HTTP响应的示例。

```swift
struct Greeting: Codable {
    var message: String
}
// Create an instance of Greeting
let greeting = Greeting(message: "Hello, world!")
// Create a 200 OK response
var httpRes = HTTPResponse(status: .ok)
// Encode the greeting to the response
try JSONEncoder().encode(greeting, to: &httpRes, on: ...)
```

## API Docs

查看 [API docs](https://api.vapor.codes/http/latest/HTTP/index.html) 以获取所有相关方法的更深入的资料。
