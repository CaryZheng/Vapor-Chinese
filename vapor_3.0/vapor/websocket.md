
# Using WebSockets

Vapor 包含一些处理简单的 Websocket [客户端](../websocket/overview.md) 和 [服务器](../websocket/overview.md) 的便利方法

## Server

Vapor的WebSocket服务器可以像HTTP服务器路由的方式处理请求。

如果注册了WebSocket服务，当Vapor的主HTTP [Server](https://api.vapor.codes/vapor/latest/Vapor/Protocols/Server.html) 启动时，它将尝试创建一个 [WebSocketServer](https://api.vapor.codes/vapor/latest/Vapor/Protocols/WebSocketServer.html)。 它将作为HTTP升级处理程序添加到服务中。

所以要创建一个WebSocket服务器，你需要做的就是在 [configure.swift](../getting_started/folder_structure.md) 中注册一个。

```swift
// Create a new NIO websocket server
let wss = NIOWebSocketServer.default()

// Add WebSocket upgrade support to GET /echo
wss.get("echo") { ws, req in
    // Add a new on text callback
    ws.onText { ws, text in
        // Simply echo any received text
        ws.send(text)
    }
}

// Register our server
services.register(wss, as: WebSocketServer.self)
```

就是这样。下次启动服务器时，您将能够在 ```GET/echo``` 处执行WebSocket升级。 您可以使用名为wsta的macOS和Linux简单命令行工具对它进行测试。

```
$ wsta ws://localhost:8080/echo
Connected to ws://localhost:8080/echo
hello, world!
hello, world!

```

### Parameters

像Vapor的HTTP的路由一样，您也可以使用WebSocket服务中使用类似路由的参数。

```swift
// Add WebSocket upgrade support to GET /chat/:name
wss.get("chat", String.parameter) { ws, req in
    let name = try req.parameters.next(String.self)
    ws.send("Welcome, \(name)!")

    // ...
}
```
现在我们来测试一下

```
$ wsta ws://localhost:8080/chat/Vapor
Connected to ws://localhost:8080/chat/Vapor
Welcome, Vapor!
```


## Client

Vapor还支持作为客户端连接到WebSocket服务器。 连接到WebSocket服务器的最简单方法是通过 [Client](https://api.vapor.codes/vapor/latest/Vapor/Protocols/Client.html) 上的 [webSocket(...)](https://api.vapor.codes/vapor/latest/Vapor/Protocols/Client.html#/s:5Vapor6ClientPAAE9webSocketXeXeF)方法。

在这个例子中，我们将假设我们的应用程序连接到boot.swift中的WebSocket服务器

```swift
// connect to echo.websocket.org
let done = try app.client().webSocket("ws://echo.websocket.org").flatMap { ws -> Future<Void> in
    // setup an on text callback that will print the echo
    ws.onText { ws, text in
        print("rec: \(text)")
        // close the websocket connection after we recv the echo
        ws.close()
    }

    // when the websocket first connects, send message
    ws.send("hello, world!")

    // return a future that will complete when the websocket closes
    return ws.onClose
}

print(done) // Future<Void>

// wait for the websocket to close
try done.wait()
```



