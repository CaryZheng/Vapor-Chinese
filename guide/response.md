# Response
---
有时候自定义 ```Response``` 对象需要被创建来替代 ```ResponseRepresentable``` 。例如，你需要手动控制 cookies 。

## 初始化
---
you很多方便的 Response 初始化方法。

## Empty
---
当 Status 是重要内容的时候， Empty responses 是有用的。

##### Swift
```
Response(status: .created)
```

## Text
---
当在路由闭包中返回 ```String``` 时候 Text responses 被创建，但它们也能被手动创建。

##### Swift
```
Response(status: .enhanceYourCalm, text: "Too many requests")
```

## JSON
---
当在路由闭包中返回 ```Json()``` 时候 JSON responses 被创建，但它们也能被手动创建。

##### Swift
```
Response(status: .internalServerError, text: JSON([
    "message": "Uh oh"
]))
```

## Redirect
---
重定向客户端到不同的URL，同时返回301状态码。

##### Swift
```
Response(redirect: "http://google.com")
```

## Cookies
---
想把 cookies 添加到 response ，请确保它是可变的，然后使用 ```cookies``` 属性来设置。

##### Swift
```
app.get("cookie") { request in
    var response = Response(status: .ok, text: "Cookie set")
    response.cookies["id"] = "123"

    return response
}
```

## Async
---
如果发送 response 前你需要等待一个回调来完成，你需要使用 async response 来初始化。它提供了一个 ```Stream``` 对象，当完成后可以使用它来发送 ```Data``` 并且关闭它。

##### Swift
```
Response(async: { stream in
    try stream.send("hi".data, timingOut: 20)
    try stream.close()
})
```