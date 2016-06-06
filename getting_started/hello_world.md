# Hello, World
---
本节假设你已经安装了Swift 3和Vapor CLI，并且已经验证过可以正常运行。

## 新建工程
---
让我们通过创建名为 Hello, World 的工程开始吧。

##### Shell
```
vapor new hello-world
```

## 目录结构
---
如果用过其它Web框架的话，你将会觉得Vapor的目录结构看起来很熟悉。

##### Text
```
.
├── App
│   └── Controllers
│   └── Middleware
│   └── Models
│   └── main.swift
├── Public
├── Resources
│   └── Views
└── Package.swift
```

在 Hello, World 工程中，我们将重点看下 ```main.swift``` 文件。

##### Text
```
.
└── App
    └── main.swift
```

使用你最喜欢的文本编辑器来打开 ```main.swift``` 。

> 示例工程
> 
> ```vapor new``` 命令创建了个带有示例和注解的新工程，该注解展示了如何使用该框架。如果愿意的话，你可以删掉这些。



## 应用
---
找到 ```main.swift``` 文件中的这行。

##### Swift
```
let app = Application()
```

示例中 ```Application``` 在这里被唯一创建。 ```Application``` 类有很多的方法，并被广泛使用。

## 路由
---
```app``` 创建后，添加如下代码片段。

##### Swift
```
app.get("hello") { request in
    return "Hello, world"
}
```
```Application``` 上创建的新路由将匹配所有到 ```/hello``` 的 ```GET``` 请求。所有路由闭包通过 ```Request``` 实例来传递，该实例包含了请求的 URI 和数据这些信息。之后我们将学习更多关于 requests 的内容。

这个路由只是简单地返回了字符串。之后我们将学习更多关于通过路由闭包返回的内容。


## 开始
---
在 main 文件底部，确保启动你的应用。

##### Swift
```
app.start()
```

保存文件，再切换到终端。


## 编译
---
Vapor 如此优秀很大一部分是因为Swift的编译器。现在开始吧。确保你在项目根目录，然后执行以下命令。

##### Shell
```
vapor build
```

> <font color="#56C0E0"> CLI 快捷键 </font>
> 
> ```vapor build``` 在后台执行 ```swift build```。


Swift包管理器首先启动，它会从git下载合适的依赖。然后编译链接这些依赖。

完成后，你将会看到类似 ```Linking .build/debug/App``` 的输出。

> <font color="#F2AE43"> 进程被终止 </font>
> 
> 如果看到类似 ```unable to execute command: Killed``` 的消息，你需要增加 交换空间（swap space）。如果你运行在一台有限 RAM 的机器上的话，这可能会发生。


## 运行
---
通过执行以下命令来启动服务。

##### Shell
```
vapor run

# Optionally specify port
vapor run --port=8080
```

你将看到消息： ```Server starting...```。

现在你就可以通过浏览器访问 ```http://localhost/hello``` 了。

> 拒绝访问
> 
> 特定端口号需要超级用户来绑定。运行 ```sudo vapor run``` 来允许访问。如果你决定运行在一个端口（包含 80 端口），确保访问相应的浏览器地址。


## Hello, World
---
浏览器窗口中你将看到如下输出。

##### Text
```
Hello, world
```