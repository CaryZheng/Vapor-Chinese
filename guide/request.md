# Request
---
每个路由闭包可获得一个 ```Request``` 实例。这允许你访问 application 创建的 request ，比如 URI 、输入数据和 session 。

## Data
---
通过 request 任何发送过来的输入内容，比如 query 、 form-encoded 或 JSON-encoded 数据，均可以使用 ```data``` 属性来访问。

##### Swift
```
request.data["name"]
```

> 顺序的重要性
> 
> 因为 query data 和 body data 都可以通过 ```data``` 属性来访问，可能两者都包含一个拥有相同 key 的 value。在这种情况下， query data 优先级比 body data 高。

## Polymorphic
---
Request data 下标访问总是返回一个符合 ```Polymorphic``` 的对象。

```Polymorphic``` 是贯穿框架的通用 protocol 。它允许未知类型的对象可以更容易地转换成类型值。

比如，如果期望发送过来的key ```age``` 是一个 ```Int``` 类型，你可以使用 ```.Int``` 属性。

##### Swift
```
guard let age = request.data["age"].int else {
    throw Abort.badRequest
}
```

## Path Indexable
---
通过使用逗号语法，嵌套的值可以很容易地被访问到。

##### Swift
```
let firstName = request.data["name", "first"].string
```

你可以使用 ```String``` 和 ```Int``` 混合方式来形成路径。

##### Swift
```
let firstUserEmail = request.data["users", 0, "email"].string
```

## Specific Access
---
如果需要明确访问 query 、 form-encoded 或 JSON-encoded data ，你也可以做到

##### Swift
```
request.data.query // StructuredData
request.data.json // JSON?
request.data.formEncoded // StructuredData?
request.data.mulipart // [String: MultiPart]?
```

## S4
---
```Request``` 类型是基于 Open Swift's [S4](https://github.com/open-swift/S4) standards 的 request 的。这意味着你可以从 Vapor 直接传递 requests 到任意其它兼容 S4 的项目。

##### Swift
```
public var uri: C7.URI
public var version: S4.Version
public var headers: S4.Headers
public var body: S4.Body
public var storage: [String : Swift.Any]

public init(
    method: S4.Method, 
    uri: C7.URI, 
    version: S4.Version, 
    headers: S4.Headers, 
    body: S4.Body
)
```

## Method
---
```method``` 属性包含了 request 使用的 HTTP 方法。

##### Swift
```
enum Method {
    case delete
    case get
    case head
    case post
    case put
    case connect
    case options
    case trace
    case patch
    case other(method: String)
}
```

## URI
---
```uri``` 属性包含了关于统一资源标识符的信息，它被用于确定 application 中指定的路由或资源。

##### Swift
```
struct URI {
    var scheme: String?
    var host: String?
    var port: Int?
    var path: String?
    var query: [String: [String?]]
    var fragment: String?
}
```

## Headers
---
```headers``` 属性包含了一个字典，该字典拥有 request 所有的头信息。 key 是不区分大小写的，并且可以包含一个或多个值（根据 HTTP 规范）。

##### Swift
```
let contentType = request.headers["content-type"]
```

## Body
---
使用 ```body``` 属性，可以访问到 request body 里面的原生数据。

##### Swift
```
let data = request.body
```