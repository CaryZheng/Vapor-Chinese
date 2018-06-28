# Fluent SQLite

Fluent SQLite（[vapor/fluent-sqlite](https://github.com/vapor/fluent-sqlite)）是一种安全，快速且易于使用的基于 [Fluent](https://docs.vapor.codes/3.0/fluent/getting-started/) 构建的 SQLite 的 ORM 框架。

> 参考
> 
> Fluent SQLite 是一个纯 Swift、NIO-based [SQLite core](https://docs.vapor.codes/3.0/sqlite/core/) 的基于 [Fluent](https://docs.vapor.codes/3.0/fluent/getting-started/) 的模块。 你应该参考它们的指南了解更多关于这里没有涉及的主题的信息。

## Getting Started

本节将向你展示如何将 Fluent SQLite 添加到你的项目并创建你的第一个 `SQLiteModel`。

### Package

使用 Fluent SQLite 的第一步是将其作为依赖项添加到你的项目中的 SPM 包的清单文件中。

```swift
// swift-tools-version:4.0
import PackageDescription

let package = Package(
    name: "MyApp",
    dependencies: [
        /// 添加其他依赖 ...

        // 🖋🐬 Swift ORM (queries, models, relations, etc) built on SQLite.
        .package(url: "https://github.com/vapor/fluent-sqlite.git", from: "3.0.0-rc"),
    ],
    targets: [
        .target(name: "App", dependencies: ["FluentSQLite", ...]),
        .target(name: "Run", dependencies: ["App"]),
        .testTarget(name: "AppTests", dependencies: ["App"]),
    ]
)
```

不要忘记将模块作为依赖项添加到 `targets` 数组中。一旦添加了依赖项，使用以下命令重新生成Xcode项目：

```
vapor xcode
```

### Model

现在来创建我们的第一个 `SQLiteModel`。 模型表示 SQLite 数据库中的表，它们是与数据交互的主要方法。

```swift
/// A simple user.
final class User: SQLiteModel {
    /// 此 user 的唯一标识符
    var id: Int?

    /// 此 user 的姓名.
    var name: String

    /// 以年份表示此 user 的年龄.
    var age: Int

    /// 创建一个新 user.
    init(id: Int? = nil, name: String, age: Int) {
        self.id = id
        self.name = name
        self.age = age
    }
}
```

上面的例子显示了代表 user 的 `SQLiteModel` 简单模型。`struct` 和 `class` 都可以作为你的模型。你甚至可以实现来自外部模块的类型，唯一的要求是这些类型符合 `Codable`，它必须在合成（自动）一致性的基本类型上声明。

SQLite 数据库的标准做法是使用自动生成的 `INTEGER` 在 id 列中创建和存储唯一标识符。 也可以使用`UUID` 甚至 `String` 作为标识符。下面是相关的协议：

|protocol|type|key|
|--------|----|---|
|`SQLiteModel`|Int|id|
|`SQLiteUUIDModel`|UUID|id|
|`SQLiteStringModel`|String|id|

> 参考
> 
> 查看 [Fluent→Model](https://docs.vapor.codes/3.0/fluent/models/)，了解更多关于使用自定义 ID 类型和键创建模型的信息。


### Migration

你所有的模型（包含一些极少数例外）都应该在数据库中具有相应的表或 scheme。 你可以使用 [Fluent→Migration](https://docs.vapor.codes/3.0/fluent/migrations/) 以可测试，可维护的方式自动生成此 scheme。 Fluent可以轻松为你的模型自动生成迁移。

> 提示
> 
> 如果你要创建表示现有表或数据库的模型，则可以跳过此步骤。

```swift
/// Allows `User` to be used as a migration.
extension User: Migration { }
```

这就是所有需要的。`Fluent` 使用 `Codable` 来分析你的 scheme，并将尝试为其创建最佳 scheme。

如果你有兴趣定制此迁移，请查看 [Fluent→Migration](https://docs.vapor.codes/3.0/fluent/migrations/)。

### Configure

最后一步是配置你的数据库。 至少需要在 [configure.swift](https://docs.vapor.codes/3.0/getting-started/structure/#configureswift) 文件中添加两样东西。

* `FluentSQLiteProvider`
* `MigrationConfig`

让我们来看看。

```swift
import FluentSQLite

/// ...

/// 首先注册 provider
try services.register(FluentSQLiteProvider())

/// 配置 Migration
var migrations = MigrationConfig()
migrations.add(model: User.self, database: .sqlite)
services.register(migrations)

/// 其他 services....
```

注册的 provider 将添加 `Fluent SQLite` 正常工作所需的所有服务。它还包含使用典型开发环境证书的默认数据库配置结构。

如果你拥有非标准证书，你当然可以覆盖此配置结构。

```swift
/// Register custom SQLite Config
let sqliteConfig = SQLiteDatabaseConfig(hostname: "localhost", port: 5432, username: "vapor")
services.register(sqliteConfig)
```

一旦你添加了 `MigrationConfig`，你应该能够运行你的应用程序并看到以下内容： 

```shell
Migrating sqlite DB
Migrations complete
Server starting on http://localhost:8080
```

### Query

现在你已经在数据库中创建了一个模型和一个 scheme，让我们开始你的第一个查询。

```swift
router.get("users") { req in
    return User.query(on: req).all()
}
```

如果运行你的应用程序，并查询该路由，你应该看到一个空数组返回。 现在你只需要添加一些用户！恭喜你获得第一个`Fluent SQLite` 模型和迁移工作。

## Connection

借助 `Fluent`，你始终可以访问底层的数据库驱动程序。使用此底层驱动程序执行查询有时称为 "raw query"。

我们来看看一个 `SQLite` 的 "raw query"。

```swift
router.get("sqlite-version") { req -> Future<String> in
    return req.withPooledConnection(to: .sqlite) { conn in
        return try conn.query("select sqlite_version() as v;").map(to: String.self) { rows in
            return try rows[0].firstValue(forColumn: "v")?.decode(String.self) ?? "n/a"
        }
    }
}
```

在上面的示例中，`withPooledConnection(to:)` 用于创建与由 `.sqlite` 标识的数据库的连接。这是默认的数据库标识符。更多信息请参阅 [Fluent→Database](https://docs.vapor.codes/3.0/fluent/database/#identifier) 。

一旦我们有了SQLiteConnection，我们就可以对它进行查询。 您可以了解更多关于SQLite→Core中可用的方法。

