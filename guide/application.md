# Application
---
```Application``` 是提供访问大量Vapor工具的核心类。它负责注册路由，启动服务，添加中间件等等。


## 初始化
---
正如你所见，创建 ```Application``` 实例唯一需要做的就是导入Vapor。

##### Swift
```
import Vapor

let app = Application()
```

不需要传递任何初始化参数即可创建一个application。这将应用默认配置。

如果想自定义 ```Application``` ，有许多可配置选项。

##### Swift
```
let app = Application(
    workDir overrideWorkDir: String? = nil,
    sessionDriver: SessionDriver? = nil,
    config overrideConfig: Config? = nil,
    hash: Hash = Hash(),
    server: Server.Type = HTTPStreamServer<ServerSocket>.self,
    router: RouterDriver = BranchRouter()
)
```

大多数Vapor插件从 ```Provider``` 开始，它们负责配置详情。想要了解更多可阅读下面providers部分。


## Router
路由器负责存储你在 ```Application``` 创建的所有路由信息。你可以使任意类成为应用的路由器，只要该类符合 ```RouterDriver``` 。


##### Swift
```
class CustomRouter: RouterDriver {
    ...
}

let app = Application(router: CustomRouter())
```

可在本指南的路由器区域了解更多详情。


## Server
---
server接收来自客户端的连接，比如web浏览器，并且返回应用响应。

你可以使任意类称为应用的server，只要该类符合 S4 的 ```Server```。想要了解更多关于 S4 和 OpenSwift standards 的内容， 请访问指南的 OpenSwift 区域。

##### Swift
```
import S4

class CustomServer: S4.Server {
    ...
}

let app = Application(server: CustomServer.self)
```

当应用启动的时候，将初始化该server类型。


## Session
---
这部分属性是不可指定的，因为初始化的时候就需要。不过，你可以手动创建 Session 。想要了解更多关于 Session 的内容，请阅读指南的 Session 部分。

##### Swift
```
let session = Session(driver: app.session)
```


## Config
---
通过属性可以访问 Config 中可配置的选项集。

##### Swift
```
let port: Int = app.config["app", "port"] ?? 80
```

上述代码将尝试从 ```Config/app.json``` 获取 ```port``` 选项。如果找不到，它将使用默认端口号 ```80``` 。更多关于Config的内容请阅读指南的Config部分。


## Hash
---
使用该属性可以创建安全的哈希值。

##### Swift
```
let hash = app.hash.make("password123")
```

将会对提供的字符串创建一个哈希值，与默认的哈希算法不同。更多关于Hash的内容请阅读指南的Hash部分。


## Middleware
---
添加自己的 middleware 到应用的 middleware 数组，可以过滤所有接收到的请求和响应。

##### Swift
```
app.middleware.append(AuthMiddleware())
```

你也可以添加 middleware 到个别的路由上。可访问指南的 Routing 和 Middleware 部分以获取更多内容。

> 添加 Middleware
> 
> 除非你想移除 Vapor 里处理 session 和 error handling 的默认中间件，不然请确保你添加了 middleware 。


## Providers
---
Providers 使得给应用添加拓展更容易。

##### Swift
```
import VaporMustache

app.providers.append(VaporMustache.Provider())
```

大多数 Vapor 拓展包需伴随一个 public Provider 类，该类可添加到 providers 数组中。当应用启动的时候， Provider 将有机会应用一些必要的调整。可访问指南的 Providers 部分以获取更多内容。


## Environment
---
包含了当前应用的运行环境。通常分为 development， testing，或者  production。

##### Swift
```
if app.config.environment == .production {
    ...
}
```

可访问指南的 Environment 部分以获取更多关于如何配置和使用 Environment 的内容。


## Working Directory
---
该属性包含应用当前启动时的相对工作路径。默认情况下，是假设从它的根目录启动应用的。

##### Swift
```
app.workDir = "/var/www/my-project/"
```

你可以动态修改工作路径，或者执行的时候覆写默认值。

##### Shell
```
vapor run --workDir="/var/www/my-project"
```


## Resources Directory
---
资源目录是只读属性，它生成了来自工作目录中的资源目录的路径。

##### Swift
```
print(app.resourcesDir)
```