# PostgreSQL Core

PostgreSQL ([vapor/postgresql](https://github.com/vapor/postgresql)) 是纯Swift实现 (没有 `libpq` 依赖), 事件驱动的, 非阻塞驱动的。 它基于 [SwiftNIO](http://github.com/apple/swift-nio) 网络库来建立。

> 另请参阅
>
>    更高级的Fluent PostgreSQL ORM 指南在 [PostgreSQL &rarr; Fluent](fluent.md)

如果有以下任何一种情况，那么在你的项目中使用PostgreSQL可能是一个好主意。

- 你有一个非标准结构的现有数据库。
- 你严重依赖于自定义或复杂的SQL查询。
- 你只是不喜欢ORM。

PostgreSQL core 建立在 DatabaseKit 之上，它提供了一些便利，如连接池和与Vapor [Services](../getting-started/services.md) 架构的集成

> 提示
>
>    即使您选择使用 [Fluent PostgreSQL](fluent.md), PostgreSQL核心的所有功能也将可供您使用。

## Getting Started

让我们来看看如何开始使用 PostgreSQL core。

### Package

使用PostgreSQL核心的第一步是将其作为依赖项添加到您的项目中的SPM package manifest 文件中。

```swift
// swift-tools-version:4.0
import PackageDescription

let package = Package(
    name: "MyApp",
    dependencies: [
        /// Any other dependencies ...
        
        // 🐘 Non-blocking, event-driven Swift client for PostgreSQL.
        .package(url: "https://github.com/vapor/postgresql.git", from: "1.0.0-rc"),
    ],
    targets: [
        .target(name: "App", dependencies: ["PostgreSQL", ...]),
        .target(name: "Run", dependencies: ["App"]),
        .testTarget(name: "AppTests", dependencies: ["App"]),
    ]
)
```

不要忘记在`targets`数组中添加模块作为依赖项。一旦添加了依赖项，使用以下命令重新生成Xcode项目：

```sh
vapor xcode
```


### Config

下一步是在你的 [`configure.swift`](../getting-started/structure.md#configureswift) 文件里设置数据库。

```swift
import PostgreSQL

/// ...

/// Register providers first
try services.register(PostgreSQLProvider())
```

注册到 provider 将添加PostgreSQL正常工作所需的所有服务。它还包含使用典型开发环境凭证的默认数据库配置结构。 

如果您拥有非标准凭证，您当然可以覆盖此配置结构。

```swift
/// Register custom PostgreSQL Config
let psqlConfig = PostgreSQLDatabaseConfig(hostname: "localhost", port: 5432, username: "vapor")
services.register(psqlConfig)
```

### Query

现在数据库已配置完成，您可以进行第一次查询。

```swift
router.get("psql-version") { req -> Future<String> in
    return req.withPooledConnection(to: .psql) { conn in
        return try conn.query("select version() as v;").map(to: String.self) { rows in
            return try rows[0].firstValue(forColumn: "v")?.decode(String.self) ?? "n/a"
        }
    }
}
```

访问此路由应显示您的PostgreSQL版本。

## Connection

`PostgreSQLConnection`通常使用`Request`容器创建，可以执行两种不同类型的查询。

### Create

有两种创建 `PostgreSQLConnection` 方法。

```swift
return req.withPooledConnection(to: .psql) { conn in
    /// ...
}
return req.withConnection(to: .psql) { conn in
    /// ...
}
```

顾名思义，`withPooledConnection（to：）`利用连接池。 `withConnection（to：）`不使用连接池。即使在高峰负载下，连接池也是确保您的应用程序不会超出数据库限制的好方法。

### Simply Query

使用`.simpleQuery（_ :)`在你的PostgreSQL数据库上执行一个不绑定任何参数的查询。您发送给PostgreSQL的一些查询可能实际上需要您使用`simpleQuery（_ :)`方法而不是参数化方法。

> 注意
>
>    此方法以文本编码方式发送和接收数据，这意味着它不适合传输像整数这类内容。
    
```swift
let rows = req.withPooledConnection(to: .psql) { conn in
    return conn.simpleQuery("SELECT * FROM users;")
}
print(rows) // Future<[[PostgreSQLColumn: PostgreSQLData]]>
```

您也可以选择在回调中接收每一行，这对于保存大型查询的内存非常有用。

```swift
let done = req.withPooledConnection(to: .psql) { conn in
    return conn.simpleQuery("SELECT * FROM users;") { row in
        print(row) // [PostgreSQLColumn: PostgreSQLData]
    }
}
print(done) // Future<Void>
```

### Parameterized Query

PostgreSQL也支持发送参数化查询（有时称为预准备语句）。此方法允许您将数据占位符插入到SQL字符串中并分别发送值。

通过参数化查询发送的数据是二进制编码的，这使得它更有效地发送一些数据类型。一般来说，您应尽可能使用参数化查询。

```swift
let users = req.withPooledConnection(to: .psql) { conn in
    return try conn.query("SELECT *  users WHERE name = $1;", ["Vapor"])
}
print(users) // Future<[[PostgreSQLColumn: PostgreSQLData]]>
```

您也可以提供一个类似于简单查询的回调函数来分别处理每一行。