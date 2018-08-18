# 开始使用 Fluent

Fluent ([vapor/fluent](https://github.com/vapor/fluent)) 是为Swift构建的一个类型安全，快速且易于使用的ORM框架。
它利用了Swift强大的类型系统为构建数据库集成提供了一个优雅的基础。

## 选择一个数据库驱动

Fluent 只是构建ORM的框架，而不是ORM本身。要开始使用Fluent，请选择下面的其中一种数据库。

|数据库|仓库|版本h|dbid|备注|
|-|-|-|-|-|
|PostgreSQL|[fluent-postgresql](https://github.com/vapor/fluent-postgresql.git)|1.0.0|`psql`|**推荐**. 开源，符合标准的SQL数据库。大多数云托管服务商有提供。
|
|MySQL|[fluent-mysql](https://github.com/vapor/fluent-mysql)|3.0.0|`mysql`|流行的开源SQL数据库。大多数云托管服务商有提供。该驱动还支持MariaDB。|
|SQLite|[fluent-sqlite](https://github.com/vapor/fluent-sqlite)|3.0.0|`sqlite`|Open source, embedded SQL database. Its simplistic nature makes it a great candiate for prototyping and testing.|
|MongoDB|fluent-mongo|n/a|`mongo`|Coming soon. Popular NoSQL database.|

> 提示
> 
>    使用上表中的信息替换下面Xcode代码段中的所有占位符`<＃...＃>`）。

你也可以在Github上搜索标签 [`fluent-database`](https://github.com/topics/fluent-database) 以获取官方和第三方Fluent数据库驱动的完整列表。

### 包

一旦确定了所需的驱动，下一步就是将其作为依赖添加到项目中SPM包清单文件中。


```swift
// swift-tools-version:4.0
import PackageDescription

let package = Package(
    name: "MyApp",
    dependencies: [
        /// Any other dependencies ...
        .package(url: "https://github.com/vapor/<#repo#>.git", from: "<#version#>"),
    ],
    targets: [
        .target(name: "App", dependencies: ["Fluent<#Database#>", ...]),
        .target(name: "Run", dependencies: ["App"]),
        .testTarget(name: "AppTests", dependencies: ["App"]),
    ]
)
```

不要忘记将模块添加为`targets`数组中的依赖项。添加依赖项后，使用以下命令重新生成Xcode项目：

```sh
vapor xcode
```

## 创建模型

现在让我们创建第一个模型。模型表示数据库中的表，它们是与数据交互的主要方法。

每个驱动都提供了便利模型协议（`PostgreSQLModel`，`SQLiteModel`等），扩展了Fluent的基础[`Model`](https://api.vapor.codes/fluent/latest/Fluent/Protocols/Model.html)协议。这些便利类型通过使用ID键和类型的标准值使声明模型更简洁。

使用所选数据库的名称填写下面的Xcode占位符，即`PostgreSQL`。


```swift
import Fluent<#Database#>
import Vapor

/// A simple user.
final class User: <#Database#>Model {
    /// The unique identifier for this user.
    var id: ID?

    /// The user's full name.
    var name: String

    /// The user's current age in years.
    var age: Int

    /// Creates a new user.
    init(id: ID? = nil, name: String, age: Int) {
        self.id = id
        self.name = name
        self.age = age
    }
}
```

上面的示例显示了表示用户的简单模型。您可以将结构和类都设为模型。您甚至可以使用来自外部模块的类型。唯一的要求是这些类型符合`Codable`，必须在基类型上声明合成（自动）一致性。

看看[Fluent＆rarr; Model]（models.md），了解有关使用自定义ID类型和键创建模型的更多信息。

##配置数据库

现在有了模型，就可以配置数据库。这是在[`configure.swift`](../getting-started/structure.md#confureswift)中完成的。

### Register Provider

第一步是注册数据库驱动provider

```swift
import Fluent<#Database#>
import Vapor

// Register providers first
try services.register(Fluent<#Database#>Provider())

// Other services....
```

注册provider将添加Fluent数据库正常工作所需的所有服务。它还包括一个使用典型开发环境凭据的默认数据库配置结构。

### Custom Credentials

如果使用数据库的默认配置（例如默认凭据或其他配置），那么这可能是需要执行的唯一设置。

有关自定义配置的详细信息，请参阅特定数据库类型的文档。

|数据库|文档|api文档|
|-|-|-|
|PostgreSQL|[PostgreSQL &rarr; Getting Started](../postgresql/getting-started.md)|[`PostgreSQLDatabase`](https://api.vapor.codes/postgresql/latest/PostgreSQL/Classes/PostgreSQLDatabase.html)|
|MySQL|[MySQL &rarr; Getting Started](../mysql/getting-started.md)|[`MySQLDatabase`](https://api.vapor.codes/mysql/latest/MySQL/Classes/MySQLDatabase.html)|
|SQLite|[SQLite &rarr; Getting Started](../sqlite/getting-started.md)|[`SQLiteDatabase`](https://api.vapor.codes/sqlite/latest/SQLite/Classes/SQLiteDatabase.html)|

## Creating a Migration

如果您的数据库驱动程序使用模式（是一个SQL数据库），则需要为您的新数据库创建一个[`Migration`](https://api.vapor.codes/fluent/latest/Fluent/Protocols/Migration.html) 模型。迁移允许Fluent以可靠，可测试的方式为您的模型创建表。您可以稍后创建其他迁移以更新或删除模型的表，甚至可以操作表中的数据。

 要创建迁移，通常首先要创建一个新的结构或类来保存迁移。但是，模型可以利用方便的快捷方式。从现有模型类型创建迁移时，Fluent可以从模型的可编码属性推断出适当的模式。

你可以将迁移一致性添加到模型作为扩展或基本类型声明。

```swift
import Fluent<#Database#>
import Vapor

extension User: <#Database#>Migration { }
``` 
如果您有兴趣了解有关自定义迁移的更多信息，查阅一下 [Fluent &rarr; 迁移](../fluent/migrations.md)。

### Configuring Migrations

创建migrations后，必须使用[`MigrationConfig`](https://api.vapor.codes/fluent/latest/Fluent/Structs/MigrationConfig.html)将其注册到Fluent。这是在[`configure.swift`](../getting-started/structure.md#confureswift)中完成的。

从上表中填写数据库ID（`dbid`），即`psql`。

```swift
import Fluent<#Database#>
import Vapor

// Configure migrations
var migrations = MigrationConfig()
migrations.add(model: User.self, database: .<#dbid#>)
services.register(migrations)

// Other services....
```

> 提示
>
> 如果你要添加的是migration也是model，你可以使用[`add(model:on:)`](https://api.vapor.codes/fluent/latest/Fluent/Structs/MigrationConfig.html#/s:6Fluent15MigrationConfigV3add5model8databaseyxm_11DatabaseKit0G10IdentifierVy0G0AA0B0PQzGtAaKRzAA5ModelRzAjaOPQzAMRSlF) 便利的设定model的 [`defaultDatabase`](https://api.vapor.codes/fluent/latest/Fluent/Protocols/Model.html#/s:6Fluent5ModelPAAE15defaultDatabase0D3Kit0D10IdentifierVy0D0QzGSgvpZ) 属性。否则，使用[`add(migration:on)`](https://api.vapor.codes/fluent/latest/Fluent/Structs/MigrationConfig.html#/s:6Fluent15MigrationConfigV3add9migration8databaseyxm_11DatabaseKit0G10IdentifierVy0G0QzGtAA0B0RzlF) 的方法。

一旦你添加了`MigrationConfig`，你应该能够运行你的应用程序并看到以下内容：

```sh
Migrating <#dbid#> DB
Migrations complete
Server starting on http://localhost:8080
```

## Performing a Query

现在你已在数据库中创建了模型和相应的模式，让我们进行第一次查询。

```swift
router.get("users") { req in
    return User.query(on: req).all()
}
```

如果你运行了app并查询该路由，则应该看到返回一个空数组。现在你只需添加一些用户！祝贺您的第一个Fluent model正常运行。

## Raw Queries

使用Fluent，你始终可以访问底层数据库驱动。使用此底层驱动执行查询有时称为“原始查询”。

要执行原生查询，您需要访问数据库连接。 Vapor的[`Request`](https://api.vapor.codes/vapor/latest/Vapor/Classes/Request.html) 类型具有许多创建新数据库连接的便利。推荐的方法是`withPooledConnection（to：）`。查阅 [DatabaseKit &rarr; Overview &rarr; Connections](../database-kit/overview/#connections) 去了解更多方法。

```swift
router.get("raw") { req -> Future<String> in
    return req.withPooledConnection(to: .<#dbid#>) { conn in
        // perform raw query using conn
    }
}
```

一旦建立了数据库连接，就可以对其进行查询。你可以从数据库文档中了解更多相关方法的信息。