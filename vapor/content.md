# Content

在Vapor3 中，所含有内容类型(JSON, protobuf, [URLEncodedForm](../url_encoded_form/), [Multipart](../multipart/),等)使用方法都是一样的。你所有需要解析和序列化的内容必须是一个遵守了 ```Codable``` 协议的类或结构体

在这里我们主要使用JSON来作为例子，但请记住，API的使用方法对于任何支持的类型都是相同的

# Server

这个部分介绍一下在客户端和服务端连接中消息发送的编解码。查看 [client](#client) 来了解外部API消息发送的编解码方式

## Request

我们来看看你将如何解析并发送如下请求到服务器

```swift
POST /login HTTP/1.1
Content-Type: application/json

{
    "email": "user@vapor.codes",
    "password": "don't look!"
}
```

首先创建一个类或者结构体来表示你期望的数据


```swift
import Vapor

struct LoginRequest: Content {
    var email: String
    var password: String
}
```

注意key与请求数据中的key名需要相同，以及数据类型也需要相同。接下来让类或结构体遵守 Content 协议

### Decode

现在我们可以来解码上面的HTTP请求了。每一个[request](https://api.vapor.codes/vapor/latest/Vapor/Classes/Request.html)都有一个[ContentContainer](https://api.vapor.codes/vapor/latest/Vapor/Structs/ContentContainer.html)，我们可以使用它来解码消息体中的内容

```swift
router.post("login") { req -> Future<HTTPStatus> in
    return req.content.decode(LoginRequest.self).map { loginRequest in
        print(loginRequest.email) // user@vapor.codes
        print(loginRequest.password) // don't look!
        return HTTPStatus.ok
    }
}
```
我们在```decode(...)```过程中使用 ```.map(to:)```方法来返回一个 [future](../async/getting_started.md)

> Note
> 
> 从请求解码这个过程是异步的，因为HTTP允许使用分块传输编码方式将主体分为多个部分

### Router

为了使传入请求的内容解码更加容易，Vapor在 [Router](https://api.vapor.codes/vapor/latest/Vapor/Protocols/Router.html) 中提供了一些自动完成此功能的扩展。

```swift
router.post(LoginRequest.self, at: "login") { req, loginRequest in
    print(loginRequest.email) // user@vapor.codes
    print(loginRequest.password) // don't look!
    return HTTPStatus.ok
}
```

### Detect Type

由于本例中的HTTP请求中我们将JSON声明为其内容类型，因此Vapor会自动使用JSON解码器。 对于以下请求，同样的方法也可以正常工作

```swift
POST /login HTTP/1.1
Content-Type: application/x-www-form-urlencoded

email=user@vapor.codes&don't+look!
```

所有HTTP请求都必须包含一个内容类型才能生效。 因此，如果遇到未知类型，Vapor将自动选择适当的解码器或者抛出错误。

> Tip
> 
> 你可以 [配置](#configure) Vapor使用的默认编码器和解码器


### Custom

如果需要你可以重写Vapor的编解码器，并传入

```swift
let user = try req.content.decode(User.self, using: JSONDecoder())
print(user) // Future<User>
```

## Response

我们来看一看如何为你的服务创建如下的响应

```swift
HTTP/1.1 200 OK
Content-Type: application/json

{
    "name": "Vapor User",
    "email": "user@vapor.codes"
}
```

和解码一样，先创建一个结构体或者类来实现你期望的数据类型

```swift
import Vapor

struct User: Content {
    var name: String
    var email: String
}
```

然后让类或结构体遵守 ```content```协议

### Encode

现在我们来编码上面的HTTP响应

```swift
router.get("user") { req -> User in
    return User(name: "Vapor User", email: "user@vapor.codes")
}
```

这会创建一个默认的 带有```200 OK``` 状态码和最小标题的 ```Response``` 。你可以通过便利方法 ```encode(...)``` 来自定义响应体

```swift
router.get("user") { req -> Future<Response> in
    return User(name: "Vapor User", email: "user@vapor.codes")
        .encode(status: .created)
}
```

### Override Type

Response的内容默认会编码成JSON，你可以通过 ```as:``` 并传入参数来改变编码方式

```swift
try res.content.encode(user, as: .urlEncodedForm)
```

你也可以修改类或者结构体使用的媒体类型

```swift
struct User: Content {
    /// See `Content`.
    static let defaultContentType: MediaType = .urlEncodedForm

    ...
}
```

# <a name="client"></a> Client

编码一个有Client 发送的请求内容和编码服务器HTTP响应的方式类似

## Request

我们来看看如何编码下面请求

```swift
POST /login HTTP/1.1
Host: api.vapor.codes
Content-Type: application/json

{
    "email": "user@vapor.codes",
    "password": "don't look!"
}
```

### Encode

首先创建一个结构体或者类来实现你期望的数据类型

```swift
import Vapor

struct LoginRequest: Content {
    var email: String
    var password: String
}
```

现在我们已经准备好发起我们的请求了，假设我们在一个路由闭包内发起这个请求，我们将使用传入的请求作为一个容器

```swift
let loginRequest = LoginRequest(email: "user@vapor.codes", password: "don't look!")
let res = try req.client().post("https://api.vapor.codes/login") { loginReq in
    // encode the loginRequest before sending
    try loginReq.content.encode(loginRequest)
}
print(res) // Future<Response>
```

## Response

继续我们在 encode 部分的例子，让我们看看如何解码来自client的Response

```swift
HTTP/1.1 200 OK
Content-Type: application/json

{
    "name": "Vapor User",
    "email": "user@vapor.codes"
}
```

当然我们首先要创建一个结构体或类来表示期望的数据类型

```swift
import Vapor

struct User: Content {
    var name: String
    var email: String
}
```

### Decode

现在我们准备好解码 Client 响应了

```swift
let res: Future<Response> // from the Client

let user = res.flatMap { try $0.content.decode(User.self) }
print(user) // Future<User>
```

## Example

现在让我们看看完整的 [Client](https://api.vapor.codes/vapor/latest/Vapor/Protocols/Client.html) 请求，它们都会对内容进行编解码

```swift
// 创建 LoginRequest 数据
let loginRequest = LoginRequest(email: "user@vapor.codes", password: "don't look!")
// POST /login
let user = try req.client().post("https://api.vapor.codes/login") { loginReq in 
    // 请求发送前编码
    return try loginReq.content.encode(loginRequest) 
}.flatMap { loginRes in
    // 接收到响应后解码
    return try loginRes.content.decode(User.self) 
}
print(user) // Future<User>
```

# Query String

URL-Encode的表单数据也可以像Content那样通过HTTP请求的URI query string来编解码. 所有你需要做的就是创建一个类或者结构体并且遵守 [Content](https://api.vapor.codes/vapor/latest/Vapor/Protocols/Content.html) 协议。在这个例子里我们将使用下面的结构体

```swift
struct Flags: Content {
     var search: String?
     var isAdmin: Bool?
}
```

## Decode

所有的 [Request]() 都有一个 [QueryContainer]() 你可以解码 query string

```swift
let flags = try req.query.decode(Flags.self)
print(flags) // Flags
```
## Encode

你也可以编码内容。这在使用 [Client](https://api.vapor.codes/vapor/latest/Vapor/Protocols/Client.html) 来编码查询字符串时很有用

```swift
let flags: Flags ...
try req.query.encode(flags)
```

# Dynamic Properties

关于Content的最多的问题是：

> 如何添加属性到单独的一个Response

Vapor3处理 ```Content``` 的方式完全基于 ```Codable```。在任何时候(可公开访问的)，你的数据都应该在一个类似 ```[String:Any]```的数据结构中，你可以按你期望的方式修改它。因此，你的App接收和返回的数据结构都必须是静态定义的。

让我们来看看一个常见的场景，以便更好的理解这一点。通常你创建一个用户时，需要几种不同的数据格式:

* create: 提供两次密码以便检查是否匹配
* internal: 你应该存储一个散列函数而不是明文密码
* public: 列出用户时不应该包含密码的哈希值

为了实现这个，你需要创建三种类型的数据.

```swift
// Data required to create a user
struct UserCreate: Content {
    var email: String
    var password: String
    var passwordCheck: String
}

// Our internal User representation
struct User: Model {
    var id: Int?
    var email: String
    var passwordHash: Data
}

// Public user representation
struct PublicUser: Content {
    var id: Int
    var email: String
}

// Create a router for POST /users
router.post(UserCreate.self, at: "users") { req, userCreate -> PublicUser in
    guard userCreate.password == passwordCheck else { /* some error */ }
    let hasher = try req.make(/* some hasher */)
    let user = try User(
        email: userCreate.email, 
        passwordHash: hasher.hash(userCreate.password)
    )
    // save user
    return try PublicUser(id: user.requireID(), email: user.email)
}
```

对于其它方法，例如 ```PATCH``` 和 ```PUT``` ，你或许需要创建更多的类型来支持唯一语义

## 好处 Benefits

与动态解决方案相比，这种方法看起来可能有点冗余，但它拥有很多关键优势:

* **静态类型** : 基于Swift和Codable，
* **可读性** : 使用Swift类型时，无需使用字符串和可选链
* **可维护性** : 在大型项目中，信息的分离使项目非常干净
* **可共享** ： 定义路由所接受，返回的类型，可用于OpenAPI规范，甚至可以直接和客户端共享
* **性能** : 使用Swift的类型，比使用[Sting:Any]字典性能更好

# JSON

JSON是一种非常流行的API编码格式，日期，数据，浮点型等编码方式是非标准的。 正因为如此，使用自定义的JSONDecoder可以让Vapor在与其他API进行交互时更很容易。

```swift
// Conforms to Encodable
let user: User ... 
// Encode JSON using custom date encoding strategy
try req.content.encode(json: user, using: .custom(dates: .millisecondsSince1970))
```

也可以用下面方法解码

```swift
// Decode JSON using custom date encoding strategy
let user = try req.content.decode(json: User.self, using: .custom(dates: .millisecondsSince1970))
```

如果你想在全局使用自定义JSON编解码器，你可以在 [configuration](#configure) 中进行

# <a name="configure"></a> Configure

使用 [ ContentConfig ](https://api.vapor.codes/vapor/latest/Vapor/Structs/ContentConfig.html) 来注册应用程序的编解码器,这些编解码器将用于任何您使用 ```content.encode/content.decode``` 的地方

```swift
/// Create default content config
var contentConfig = ContentConfig.default()

/// Create custom JSON encoder
var jsonEncoder = JSONEncoder()
jsonEncoder.dateEncodingStrategy = .millisecondsSince1970

/// Register JSON encoder and content config
contentConfig.use(encoder: jsonEncoder, for: .json)
services.register(contentConfig)
```
