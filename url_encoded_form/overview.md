# Using URL-Encoded Form

URL-Encoded Form 是 web 上广泛支持的编码。它通常用于序列化通过 POST 请求发送的 web 表单。此编码还用于在 URL 查询字符串中发送的结构化数据。

它是发送少量数据的相对有效的编码。然而，所有数据必须是百分率编码，使得这种编码对于大量数据是次优的。如果需要上传文件之类的东西，请参阅 [Multipart](https://docs.vapor.codes/3.0/multipart/getting-started/) 编码。

> 提示
> 
> `URL-Ecoded Form` 像 `Vapor` 中的所有其他编码方法一样整合进了 [Content](https://api.vapor.codes/vapor/latest/Vapor/Protocols/Content.html)。有关 [Content](https://api.vapor.codes/vapor/latest/Vapor/Protocols/Content.html) 协议的更多信息，请参见 [Vapor -> Content](https://docs.vapor.codes/3.0/vapor/content/)。

让我们来看看如何解码一个 `application/x-www-form-urlencoded` 请求。

## Decode Body

大多数情况下，你将从 web 表单中解码 `form-urlencoded` 编码的请求。让我们来看看其中一个请求的形式。之后，我们将查看该请求的 HTML 表单将是什么形式。

### Request

这里有一个创建新用户的 `form-urlencoded` 编码的请求。

```http
POST /users HTTP/1.1
Content-Type: application/x-www-form-urlencoded

name=Vapor&age=3&luckyNumbers[]=5&luckNumbers[]=7
```

你可以看到 `[]` 符号用于编码数组。你的 web 表单也需要使用此符号。

### Form

创建 `form-urlencoded` 编码请求的方式有很多种，但最常见的是 HTML web 表单。下面是这个请求的 HTML 表单可能的样子。

```html
<form method="POST" action="/users">
    <input type="text" name="name">
    <input type="text" name="age">
    <input type="text" name="luckyNumbers[]">
    <input type="text" name="luckyNumbers[]">
</form>
```

由于我们没有在 `<form>` 上指定明确的 `enctype` 属性，浏览器将默认进行 URL-encode 编码。我们还提供了两个同名字段，`luckyNumbers[]`，这将表示我们发送一个数组的值。

### Content

现在让我们来看看我们在 `Vapor` 中如何处理这个请求。第一步（一如既往的使用 [Content](https://api.vapor.codes/vapor/latest/Vapor/Protocols/Content.html)）是创建一个实现了 `Codable` 协议的结构体来表示数据结构。

```swift
import Vapor

struct User: Content {
    var name: String
    var age: Int
    var luckyNumbers: [Int]
}
```

现在我们有了 `User` 结构，让我们解码这个请求！我们可以使用 [ContentContainer](https://api.vapor.codes/vapor/latest/Vapor/Structs/ContentContainer.html) 轻松地完成这项工作。

```swift
router.post("users") { req -> Future<HTTPStatus> in
    return try req.content.decode(User.self).map(to: HTTPStatus.self) { user in
        print(user.name) // "Vapor"
        print(user.age) // 3
        print(user.luckyNumbers) // [5, 7]
        return .ok
    }
}
```

现在当你将表单 post 到 `/users` 时，你应该会看到控制台中打印的信息。干得漂亮！

## Encode Body

使用编码 `form-urlencoded` 数据 API 的频率比解码它们的要少得多。然而，使用 Vapor 编码依然很简单。使用前面例子中的同一个 `User` 结构，这里是我们如何编码一个 `form-urlencoded` 编码响应。

```swift
router.get("multipart") { req -> User in
    let res = req.makeResponse()
    let user = User(name: "Vapor", age: 3, luckNumber: [5, 7])
    res.content.encode(user, as: .urlEncodedForm)
    return user
}
```

> 提示
> 
> 如果你在 `Content` 中设置了默认的 `MediaType`，则可以直接在路由闭包中返回它们。

## URL Query

URL-Encoded Forms 对于在 URL 查询字符串中发送结构化数据同样有用。这被广泛用于不允许发送 HTTP 请求体的 GET 请求的数据。

让我们来看看如何从查询字符串中解码一些搜索参数。

```http
GET /users?name=Vapor&age=3 HTTP/1.1
```

第一步（一如既往的使用 [Content](https://api.vapor.codes/vapor/latest/Vapor/Protocols/Content.html)）是创建一个实现了 `Codable` 协议的结构体来表示数据结构。

```swift
import Vapor

struct UsersFilters: Content {
    var name: String?
    var age: Int?
}
```

在这里，我们设置 `name` 和 `age` 均为可选值，因为此路由可以不传任何参数返回所有 user。

现在我们有了一个 `Codable` 结构，我们可以解码 URL 查询字符串。该过程与解码 content 几乎相同，我们只需要使用 `req.query` 代替 `req.content`。

```swift
router.get("users") { req -> Future<[User]> in
    let filters = try req.query.decode(UsersFilters.self)
    print(filters.name) // Vapor
    print(filters.age) // 3
    return // 使用 filters 过滤后的 users
}
```

> 提示
> 
> 解码 URL 查询字符串不是异步的，因为与 HTTP 请求体不同，Vapor 可以确保调用路由闭包时可用。


