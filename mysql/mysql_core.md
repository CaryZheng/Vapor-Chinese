# MySQL Core

MySQL([vapor/mysql](https://github.com/vapor/mysql))是一个纯swift语言编写的(不依赖`libmysql`)事件驱动型、非阻塞型MySQL驱动程序。它建立在[SwiftNIO](https://github.com/apple/swift-nio)网络库的基础上。

> 也可以看看
> 
> 更高级的**Fluent MySQL ORM**指南位于[MySQL → Fluent](https://docs.vapor.codes/3.0/mysql/fluent/)

如果以下任何一种情况属实，那么在你的项目中使用MySQL软件包可能是一个好主意。

* 你现有一个非标准结构的数据库
* 你严重依赖自定义或者复杂的SQL语句
* 你不喜欢ORM(Object Relational Mapping - 对象关系映射)

MySQL core建立在提供诸如连接池和[Vapor服务体系集成](https://docs.vapor.codes/3.0/getting-started/services/) 便利的DatabaseKit上。

> 提示
> 
> 即使您使用[Fluent MySQL](https://docs.vapor.codes/3.0/mysql/fluent/)，你任将可以使用所有的MySQL core功能。

# 入门

我们来看看怎么入门MySQL Core

## 包

使用MySQL Core的第一步是将其作为依赖添加到您的项目中的SPM包清单文件中。

```
// swift-tools-version:4.0
import PackageDescription

let package = Package(
    name: "MyApp",
    dependencies: [
        /// 其他依赖 ...

        // 🐬 用于MySQL的非阻塞、事件驱动型的Swift客户端。
        .package(url: "https://github.com/vapor/mysql.git", from: "3.0.0-rc"),
    ],
    targets: [
        .target(name: "App", dependencies: ["MySQL", ...]),
        .target(name: "Run", dependencies: ["App"]),
        .testTarget(name: "AppTests", dependencies: ["App"]),
    ]
)
```



