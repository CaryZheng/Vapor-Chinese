# Using WebSockets

不像 HTTP，WebSocket 允许客户端和服务端双向通信。你可以使用文本或二进制格式来发送消息（称为 frames）。客户端和服务端可以随时发送消息，不需要等待请求响应。

尽管 WebSocket 有它自己的协议，但它仍然使用了 HTTP 协议。每个 WebSocket 连接都是从一个 HTTP 请求开始的，response 中的 headers 使用了状态码 ```101 Switching Protocols``` 。在初始化握手后，才是一个 WebSocket 连接。

## WebSocket

[WebSocket](https://api.vapor.codes/websocket/latest/WebSocket/Classes/WebSocket.html) 类表示是一个已连接的 WebSocket 客户端。你可以使用如下方法来监听数据收发的事件。

```swift
let ws: WebSocket = ...
// Send an initial message to this WebSocket
ws.send("Hello!")

// Set a new callback for receiving text formatted data
ws.onText { ws, string in
    // Echo the text back, reversed.
    ws.send(string.reversed())
}
```

> 提示
> 
> 所有回调将接收到一个 WebSocket 引用。可以通过该引用来发送收据，以避免出现循环引用的情况。

```WebSocket``` 连接被关闭后， [onClose](https://api.vapor.codes/websocket/latest/WebSocket/Classes/WebSocket.html#/s:9WebSocketAAC7onCloseXev) future 将会被完成。你可以使用 [close()](https://api.vapor.codes/websocket/latest/WebSocket/Classes/WebSocket.html#/s:9WebSocketAAC5closeyyF) 来关闭自身的连接。

## Server

WebSocket 服务器一次可以连接一个或多个 WebSocket 客户端。正如前面提到的，WebSocket 连接必须通过 HTTP 进行握手连接。因此，WebSocket 服务器是基于 [HTTP servers](../http/server.md) 中的 upgrade 机制实现的。

```swift
// First, create an HTTPProtocolUpgrader
let ws = HTTPServer.webSocketUpgrader(shouldUpgrade: { req in
    // Returning nil in this closure will reject upgrade
    if req.url.path == "/deny" { return nil }
    // Return any additional headers you like, or just empty
    return [:]
}, onUpgrade: { ws, req in
    // This closure will be called with each new WebSocket client
    ws.send("Connected")
    ws.onText { ws, string in
        ws.send(string.reversed())
    }
})

// Next, create your server, adding the WebSocket upgrader
let server = try HTTPServer.start(
    ...
    upgraders: [ws],
    ...
).wait()
// Run the server.
try server.onClose.wait()
```

> 提示
> 
> 访问 [HTTP → Server](../http/server.md) 来了解如何设置 HTTP server。

WebSocket 协议 upgrader 包含两个回调。

第一个回调 ```shouldUpgrade``` 接收需要 upgrade 的 HTTP 请求。该回调基于请求内容决定是否来完成 upgrade 。

第二个回调是 ```onUpgrade```，每当一个新的 WebSocket 客户端连接上后就会触发该回调。在这里你可以配置你的回调然后发送任何初始化数据。

> 警告
> 
> 任意事件循环上都可以调用 upgrade 闭包。如果你一定要访问外部变量的话，请注意避免竞争条件。

## Client

你也可以使用 WebSocket package 来连接 WebSocket 服务器。就像 WebSocket 服务端基于 HTTP 服务端实现一样， WebSocket 客户端也是基于 HTTP 客户端实现的。

```swift
// Create a new WebSocket connected to echo.websocket.org
let ws = try HTTPClient.webSocket(hostname: "echo.websocket.org", on: ...).wait()

// Set a new callback for receiving text formatted data.
ws.onText { ws, text in
    print("Server echo: \(text)")
}

// Send a message.
ws.send("Hello, world!")

// Wait for the Websocket to closre.
try ws.onClose.wait()
```

> 提示
> 
> 访问 [HTTP → Client](../http/client.md) 来了解如何设置 HTTP client。

## API Docs

想要了解更多，可访问 [API docs](https://api.vapor.codes/websocket/latest/WebSocket/index.html) 。