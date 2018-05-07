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

首先，创建一个结构体或类，用来表示你期望的数据。

```
import Vapor

struct LoginRequest: Content {
    var email: String
    var password: String
}
```

然后让该结构体或类实现 ```Content``` 。现在我们可以开始解析 HTTP request了。

```
router.post("login") { req -> Future<HTTPStatus> in
    return req.content.decode(LoginRequest.self).map(to: HTTPStatus.self) { loginRequest in
        print(loginRequest.email) // user@vapor.codes
        print(loginRequest.password) // don't look!
        return .ok
    }
}
```

这里我们使用 ```.map(to:)``` 来返回一个 future 。

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

就像 decoding 一样，首先创建一个结构体或类，用来表示你期望的数据。

```
import Vapor

struct User: Content {
    var name: String
    var email: String
}
```

然后只需要让该结构体或类实现 ```Content```。现在我们可以开始 encode 这个 HTTP response了。

```
router.get("user") { req -> User in
    return User(
        name: "Vapor User",
        email: "user@vapor.codes"
    )
}
```

现在你该知道在 Vapor 中如何 encode 和 decode 数据了。

> 提示
> 
> 详情可见 [Vapor → Content](../vapor/content.md)

下一章节是 [Async](async.md) 。