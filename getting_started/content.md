# Content

Vapor3 中所有的 content 类型（JSON、protobuf、FormURLEncoded、Multipart等等）都是被同等对待的。你只需要解析和序列化 content 即可，该 content 必须是一个实现了```Codable```的类或结构体。

为了方便介绍，我们将以 JSON 为例。但是注意这个 API 对任何支持 content 类型的都是一样的。

## Request

让我们看下如何解析接下来的这个 HTTP request。

```
POST /login HTTP/1.1
Content-Type: application/json

{
    "email": "user@vapor.codes",
    "password": "don't look!"
}
```

## Decode Request

首先，创建一个结构体或类，用来表示你期望的数据。

```
import Foundation
import Vapor

struct LoginRequest: Content {
    var email: String
    var password: String
}
```

然后让该结构体或类实现 ```Content``` 。现在我们可以开始解析 HTTP request了。

```
router.post("login") { req -> Response in
    let loginRequest = try req.content.decode(LoginRequest.self)

    print(loginRequest.email) // user@vapor.codes
    print(loginRequest.password) // don't look!

    return Response(status: .ok)
}
```

就是这么简单！

## Other Request Types

因为上面的示例中在 request 中声明了 JSON 类型，所以 Vapor 会自动按 JSON 类型进行解析。同样的方法也适用于下面这个 request 。

```
POST /login HTTP/1.1
Content-Type: application/x-www-form-urlencoded

email=user@vapor.codes&don't+look!
```

> 提示
> 
> 你可以配置 Vapor 使用哪个 encoders/decoders 。

## Response

让我们看看如何创建 HTTP response 。

```
HTTP/1.1 200 OK
Content-Type: application/json

{
    "name": "Vapor User",
    "email": "user@vapor.codes"
}
```

### Encode Response

就像 decoding 一样，首先创建一个结构体或类，用来表示你期望的数据。

```
import Foundation
import Vapor

struct User: Content {
    var name: String
    var email: String
}
```

然后只需要让该结构体或类实现 ```Content```。现在我们可以开始 encode 这个 HTTP response了。

```
router.get("user") { req -> Response in
    let user = User(
        name: "Vapor User",
        email: "user@vapor.codes"
    )

    let res = Response(status: .ok)
    try res.content.encode(user, as: .json)
    return res
}
```

### Other Response Types

Content 默认将会被自动 encode 成 JSON 类型。你可以通过```as:```参数来重写 content 类型。

```
try res.content.encode(user, as: .formURLEncoded)
```

你也可以更改任意类或结构体默认的 media 类型。

```
struct User: Content {
    /// See Content.defaultMediaType
    static let defaultMediaType: MediaType = .formURLEncoded

    ...
}
```

### Configuring Content

Coming soon.