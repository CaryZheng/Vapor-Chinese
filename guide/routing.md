# Routing
---
Vapor 中的路由可以被定义在任意文件中，只要能访问到 ```Application``` 即可。通常总是在 ```main.swift``` 文件中。


## Basic
---
最基本的路由包含了一个方法，路径和闭包。

##### Swift
```
app.get("welcome") { request in 
    return "Hello"
}
```

可用的标准 HTTP 方法包含 ```get``` 、 ```post``` 、 ```put``` 、 ```patch``` 、 ```delete``` 和 ```options``` 。

你也可以使用 ```any``` 匹配所有方法。


## Request
---
传递到路由闭包中的第一个参数是 Request 的一个实例。它包含了 method 、 URI 、 body 等等。可访问指南的 Request 部分以获取更多内容。

##### Swift
```
let method = request.method
```


## JSON
---
以 JSON 返回，仅仅只需要使用 ```Json``` 包装你的数据结构。

##### Swift
```
app.get("version") { request in
    return Json(["version": "1.0"])
}
```


## Response Representable
---
所有的路由闭包都返回 ```ResponseRepresentable``` 数据结构。默认情况下， Strings 和 JSON 都遵循该 protocol ，但是你可以添加你自己的。

##### Swift
```
public protocol ResponseRepresentable {
    func makeResponse() -> Response
}
```

可访问 Response 部分以获取更多内容。


## Parameters
---
可以通过传递你想要接收的数据类型来描述参数。

##### Swift
```
app.get("hello", String) { request, name in 
    return "Hello \(name)"
}
```

当你添加一个参数类型，比如 ```String``` ，闭包将会需要包含另一个输入变量。该变量类型与上述类型一致，即 ```String``` 。


## String Initializable
---
任意符合 ```StringInitializable``` 的类型都可以作为参数。默认情况下， ```String``` 和 ```Int``` 符合该 protocol ，但是你可以添加你自己的。

##### Text
```
class User: StringInitializable {
    ...
}

app.get("users", User) { request, user in 
    return "Hello \(user.name)"
}
```

利用 Swift 拓展性，你可以拓展已存在的类型来支持该行为。

##### Swift
```
extension User: StringInitializable {
    init?(from string: String) throws {
        guard let int = Int(string) else {
            return nil //Will Abort.InvalidRequest
        }

        guard let user = User.find(int) else {
            throw UserError.NotFound
        }

        self = user
    }
}
```

你可以抛出自定义错误或者返回 ```nil``` 来抛出默认错误。可访问 Error  部分以获取更多内容。


## Groups
---
使用 ```grouped(_: String)``` 来给路由组加前缀。

##### Swift
```
app.grouped("v1") { v1 in
    v1.get("users") { request in
        return User.all
    }
}
```

## Grouped Middleware
---
除了给端点分组外，也可以给 middleware 分组，让它只应用于特定范围内的路由。

##### Swift
```
app.grouped(AuthMiddleware(), UserMiddleware()) { group in 
    group.get("token") { req in
      return getToken(req)
    }
}
```

## Caveats
---
类型安全的路由最多支持 5 个参数。

##### Swift

```
app.get("users", User, "posts", Post, "comments") { request, user, post in 
    //this is the maximum
}
```

如果你使用 ```app.group``` 和 ```app.host``` 的话，超过 5 个参数是不太可能的。你也可以在固定的路由区域包含斜杠。

##### Swift 

```
app.get("v1/users", User, "resources/written/posts", Post) { request, user, post in 
    //only four parameters used here
}
```

此外，由于编译器的一个 [bug](https://bugs.swift.org/browse/SR-899) ， ```.self``` 有时候在类型名后面是必需的。为了避免这种情况的发生，你可以通过以下方式来解决：

##### Swift

```
let Int = Swift.Int.self
let String = Swift.String.self
```

支持 Swift evolution [pull request](https://github.com/apple/swift-evolution/pull/197) 来修复该问题。
更新：已被 [approved](https://github.com/apple/swift-evolution/blob/master/proposals/0090-remove-dot-self.md) 

## Custom Routing 
---
Vapor 仍然支持用户自定义示例或长 URLs 的传统路由方式。

##### Swift
```
app.get("users/:user_id") { request in
    request.parameters["user_id"] // Optional<String>
}
```