# Using Multipart

Multipart是网络上广泛支持的编码。它最常用于序列化Web表单，特别是包含像图片这类富媒体的表单。它允许在每个部分编码成任意数据，这要归功于单独定义的独特分隔符 _boundary_ 。这个边界由client保证不出现在数据的任何地方。

Multipart是一种功能强大的编码，但很少使用它的基本格式。最常用的是`multipart/form-data`。该编码为多部分数据的每个部分添加了一个`"name"` 属性。这是序列化Web表单所必需的。对于本指南的其余部分，都是在假设我们正在讨论`multipart/form-data`，除非另有说明。

> 提示
>
>    Multipart 融入在 [`Content`](https://api.vapor.codes/vapor/latest/Vapor/Protocols/Content.html) 里 像在Vapor里所有其他编码的方法。 参阅 [Vapor &rarr; Content](../vapor/content.md) 以获得更多关于 [`Content`](https://api.vapor.codes/vapor/latest/Vapor/Protocols/Content.html) 协议的资料。 

让我们来看看如何解码一个 `multipart/form-data`-encoded 的请求。

## Decode

大多数情况下，您将解码来自Web表单的`multipart/form-data`编码请求。让我们来看看这些请求中的哪一个可能看起来像。之后，我们将看看该请求的HTML表单是什么样的。

### Request

以下是通过`multipart/form-data`编码请求创建新用户的示例。

```http
POST /users HTTP/1.1
Content-Type: multipart/form-data; boundary=123

--123
Content-Disposition: form-data; name="name"

Vapor
--123
Content-Disposition: form-data; name="age"

3
--123
Content-Disposition: form-data; name="image"; filename="droplet.png"

<contents of image>
--123--
```

您可以看到多部分数据使用 _boundary_（在本例中为`“123”`）来分隔数据。这通常会是一个更长的字符串。发送多部分编码请求的客户端必须确保其提供的边界不会出现在发送给您的内容中的任何位置。这就是允许这种编码用于发送文件等的原因。

### Form

有很多方法可以创建多部分编码的请求，但最常见的是HTML Web表单。以下是这个请求的HTML表单可能看起来像什么。

```html
<form method="POST" action="/users" enctype="multipart/form-data">
    <input type="text" name="name">
    <input type="text" name="age">
    <input type="file" name="image">
</form>
```

记下`<form>`上的`enctype`属性以及`file`类型的输入。这允许我们通过网络表单发送文件。

### Content

现在我们来看看如何在Vapor中处理这个请求。第一步（与[`Content`]（https://api.vapor.codes/vapor/latest/Vapor/Protocols/Content.html））一样）是创建一个代表数据结构的`Codable`结构。

```swift
import Vapor

struct User: Content {
    var name: String
    var age: Int
    var image: Data
}
```

!!! 提示
    如果您想要访问文件名，则可以使用 [`File`](https://api.vapor.codes/core/latest/Core/Structs/File.html) 而不是`Data`。

现在我们有了 `User` 结构， 让我们解码这个请求！我们可以使用 [`ContentContainer`](https://api.vapor.codes/vapor/latest/Vapor/Structs/ContentContainer.html) 轻松完成此操作。

```swift
router.post("users") { req -> Future<HTTPStatus> in
    return try req.content.decode(User.self).map(to: HTTPStatus.self) { user in
        print(user.name) // "Vapor"
        print(user.age) // 3
        print(user.image) // Raw image data
        return .ok
    }
}
```

现在，当您将表单post到`/users`时，您应该会看到控制台中打印的信息。干得不错！

## Encode

API对多部分数据进行编码的次数比对其进行解码的次数少得多。但是，Vapor的编码也很容易。使用我们前面例子中的`User`结构，这里是我们如何编码一个多部分编码的响应。

```swift
router.get("multipart") { req -> User in
    let res = req.makeResponse()
    let user = User(name: "Vapor", age: 3, image: Data(...))
    res.content.encode(user, as: .formData)
    return user
}
```

> 提示
>
>    如果你在 `Content` 类型上设置了一个默认的`MediaType`，那么你可以直接在路由关闭中返回它们。

## Parsing & Serializing

Multipart包还提供了用于解析和序列化 `multipart/form-data` 数据而不使用`Codable`的API。查看 [API Docs](https://api.vapor.codes/multipart/latest/Multipart/index.html)了解更多关于使用这些API的信息。
