# SQLite Core

SQLite（[vapor/sqlite](https://github.com/vapor/sqlite)）是 `libsqlite` C库的一个封装。

> 参考
> 
> 更高层的，Fluent SQLite ORM 指南位于 [SQLite→Fluent](https://docs.vapor.codes/3.0/sqlite/fluent/)。

如果符合以下任何一种情况，那么仅使用 `SQLite` 包可能是一个好主意。

* 你有一个非标准结构的现有数据库。
* 你严重依赖于自定义或复杂的SQL查询。
* 你只是不喜欢ORM。

`SQLite` 内核构建在 `DatabaseKit` 之上，它提供了一些便利，如连接池和与Vapor [Services](https://docs.vapor.codes/3.0/getting-started/services/) 集成。

> 提示
> 
> 即使你选择使用 [Fluent SQLite](https://docs.vapor.codes/3.0/sqlite/fluent/)，SQLite Core 的所有功能也可供你使用。


## Getting Started

我们来看看如何开始使用 SQLite Core。

### Package

使用 SQLite Core 的第一步是将其作为依赖添加到你的项目中的 SPM 包清单文件中。

```swift
// swift-tools-version:4.0
import PackageDescription

let package = Package(
    name: "MyApp",
    dependencies: [
        /// Any other dependencies ...

        // 🔵 SQLite 3 wrapper for Swift.
        .package(url: "https://github.com/vapor/sqlite.git", from: "3.0.0-rc"),
    ],
    targets: [
        .target(name: "App", dependencies: ["SQLite", ...]),
        .target(name: "Run", dependencies: ["App"]),
        .testTarget(name: "AppTests", dependencies: ["App"]),
    ]
)
```

不要忘记将模块作为依赖添加到 `targets` 数组中。一旦添加了依赖项，使用以下命令重新生成 Xcode 项目：

```shell
vapor xcode
```

### Config

下一步是在 [configure.swift](https://docs.vapor.codes/3.0/getting-started/structure/#configureswift) 文件中配置数据库。

```swift
import SQLite

/// ...

/// 首先注册 providers
try services.register(SQLiteProvider())
```

注册 provider 将添加 SQLite 正常工作所需的所有服务。它还包括使用内存数据库的默认数据库配置结构。
如果你拥有非标准证书，你当然可以覆盖此配置结构。

```swift
/// 注册自定义 SQLite 配置
let sqliteConfig = SQLiteDatabaseConfig(storage: .memory)
services.register(sqliteConfig)
```

### Query

现在数据库已配置完成，你可以进行第一次查询。

```swift
router.get("sqlite-version") { req -> Future<String> in
    return req.withPooledConnection(to: .sqlite) { conn in
        return try conn.query("select sqlite_version() as v;").map(to: String.self) { rows in
            return try rows[0].firstValue(forColumn: "v")?.decode(String.self) ?? "n/a"
        }
    }
}
```

访问此路由会显示你的 SQLite 版本。

## Connection

`SQLiteConnection` 通常使用 `Request` 容器创建，可以执行两种不同类型的查询。

### Create

有两种创建 `SQLiteConnection` 的方法。

```swift
return req.withPooledConnection(to: .sqlite) { conn in
    /// ...
}
return req.withConnection(to: .sqlite) { conn in
    /// ...
}
```

正如名字所暗示的那样，`withPooledConnection(to:)` 使用连接池。 `withConnection(to:)` 不使用。 即使在高峰负载下，连接池也是确保你的应用程序不会超出数据库限制的好方法。

### Simply Query

使用 `.simpleQuery(_:)` 在不绑定任何参数的 SQLite 数据库上执行查询。你发送给 SQLite 的一些查询可能实际上需要你使用 `simpleQuery(_:)` 方法而不是参数化方法。

> 注意
> 
> 此方法以文本编码方式发送和接收数据，这意味着它不适合传输整数等内容。

```swift
let rows = req.withPooledConnection(to: .sqlite) { conn in
    return conn.simpleQuery("SELECT * FROM users;")
}
print(rows) // Future<[[SQLiteColumn: SQLiteData]]>
```

你也可以选择在回调中接收每一行，这对于节省大型查询的内存非常有用。

```swift
let done = req.withPooledConnection(to: .sqlite) { conn in
    return conn.simpleQuery("SELECT * FROM users;") { row in
        print(row) // [SQLiteColumn: SQLiteData]
    }
}
print(done) // Future<Void>
```

### Parameterized Query

SQLite 还支持发送参数化查询（有时称为预准备语句）。此方法允许你将数据占位符分别插入到 SQL 字符串中并发送。
通过参数化查询发送的数据是二进制编码的，这使得它更有效地发送一些数据类型。一般来说，你应尽可能使用参数化查询。

```swift
let users = req.withPooledConnection(to: .sqlite) { conn in
    return try conn.query("SELECT *  users WHERE name = $1;", ["Vapor"])
}
print(users) // Future<[[SQLiteColumn: SQLiteData]]>
```

你也可以提供一个类似于简单查询的回调函数来分别处理每一行。


