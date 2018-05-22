
# 使用 Client

[Client](https://api.vapor.codes/vapor/latest/Vapor/Protocols/Client.html) 是一个便捷的 [HTTP -> Client](../http/client.md) 的封装层,它会自动解析URL的主机名和端口信息，并帮助您对[内容](./content.md)进行编解码

~~~swift
let res = try req.client().get("http://vapor.codes")
print(res) // Future<Response>
~~~

# Container 容器

你需要做的第一件事情是创建客户端的服务容器 [Container](../getting_started/services.md)

如果你正在使用外部请求的结果来作为服务端内部请求的输入，你需要使用 ```Request``` 容器来创建客户端，这是很常见的案例

如果你在启动的时候需要使用 client ，请使用 ```Application```,或者你在 ```command``` 下使用 command 上下文容器

当你已经获得 ```Container``` ，使用 [client()](https://api.vapor.codes/vapor/latest/Vapor/Extensions/Container.html#/s:5Vapor6clientXeXeF)方法来创建 ```client```


```swfit
// Creates a generic Client
let client = try container.client()
```

# send 发送

一旦您有了 ```Client``` ,你可以使用 [send()]() 方法来发送```请求```,需要注意的是请求URI必须包含请求方式和主机名

```swift
let req: Request ...
let res = try client.send(req)
print(res) // Future<Response>
```

你也可以使用像 [get(...)](https://api.vapor.codes/vapor/latest/Vapor/Protocols/Client.html#/s:5Vapor6ClientPAAE3getXeXeF),[post(...)](https://api.vapor.codes/vapor/latest/Vapor/Protocols/Client.html#/s:5Vapor6ClientPAAE4postXeXeF) 这样的便捷方法

```swift
let user: User ...
let res = try client.post("http://api.vapor.codes/users") { post in
    try post.content.encode(user)
}
print(res) // Future<Response>
```

有关编解码消息内容的信息，请参阅 [Content](./content.md)
