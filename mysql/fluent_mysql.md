# Fluent MySQL

Fluent MySQL([vapor/fluent-mysql](https://github.com/vapor/fluent-mysql))是建立在[Fluent](https://docs.vapor.codes/3.0/fluent/getting-started/)基础上的好用、快速、类型安全的MySQL ORM。

> 也可以看看
> 
> Fluent MySQL是建立在[Fluent](https://docs.vapor.codes/3.0/fluent/getting-started/)基础上的纯Swift语言的工具包，不基于[MySQL Core](https://docs.vapor.codes/3.0/mysql/core/)。你应该在指南中查阅更多这边不涉及主题的资料。

# 入门

这一章节将告诉你怎么把Fluent MySQL添加到你的项目中并且创建你的第一个```MySQLModel```。

# 包

使用Fluent MySQL的第一步是将其作为依赖添加到您的项目中的SPM包清单文件中。

```
// swift-tools-version:4.0
import PackageDescription

let package = Package(
    name: "MyApp",
    dependencies: [
        /// 其他依赖 ...

        // 🖋🐬 Swift ORM (queries, models, relations, etc) built on MySQL.
        .package(url: "https://github.com/vapor/fluent-mysql.git", from: "3.0.0-rc"),
    ],
    targets: [
        .target(name: "App", dependencies: ["FluentMySQL", ...]),
        .target(name: "Run", dependencies: ["App"]),
        .testTarget(name: "AppTests", dependencies: ["App"]),
    ]
)
```

不要忘记将模块作为依赖添加到```targets```数组中。一旦你添加了依赖，使用以下命令重新生成Xcode项目：

```
vapor xcode
```
# 模型

现在，我们来创建我们的第一个```MySQLModel```。模型表示MySQL数据库中的表并且它们是与数据交互的主要方法。

```
/// 一个简单的User例子.
final class User: MySQLModel {
    /// The unique identifier for this user.
    var id: Int?

    /// The user's full name.
    var name: String

    /// The user's current age in years.
    var age: Int

    /// Creates a new user.
    init(id: Int? = nil, name: String, age: Int) {
        self.id = id
        self.name = name
        self.age = age
    }
}
```

上面的示例展示了```MySQLModel```表示一张简单的用户表。您可以将```structs```和```classes``` 都设为模型。您甚至可以把来自外部模块的类型设置为模型。唯一的要求是这些类型遵循```Codable```，必须在基类上声明来自动生成。

MySQL数据库的标准做法是使用自动生成的```INTEGER```在```id```列中创建和存储唯一标识符。也可以通过协议使用```UUID```或甚至```String```作为标识符。

| 协议     |      类型     |键      |
|:----------|:-------------|:------|
|```MySQLModel``` |Int | id       |
|```MySQLUUIDModel``` |UUID   | id|
|```MySQLStringModel``` |String| id|

> 也可以看看
> 
> 在[Fluent → Model](https://docs.vapor.codes/3.0/fluent/models/)中查阅如何使用自定义ID类型和键来创建模型。

# 迁移

所有你要创建的模型（一些罕见的除外）都应该在你的数据库中有对应的表或者纲要。你可以利用[Fluent → Migration](https://docs.vapor.codes/3.0/fluent/migrations/)以一种可测试、维护的方式来生成大纲。Fluent使得替你的模型生成一份迁移变得很简单。

> 提示
> 
> 如果你创建一个表示现存的数据表或者数据库的模型，你可以跳过此步骤。

```
/// 允许`User`用作迁移。
extension User: Migration { }
```

这就是全部。Fluent使用Codable来分析您的模型，并尝试为其创建最佳模式。

如果您有兴趣自定义此迁移，请查看[Fluent → Migration](https://docs.vapor.codes/3.0/fluent/migrations/)。

# 配置

最后一步是配置你的数据库。至少，这需要在[configure.swift](https://docs.vapor.codes/3.0/getting-started/structure/#configureswift)文件中添加两项配置。

 * ```FluentMySQLProvider ```
 * ```MigrationConfig ```

我们来看下

```
import FluentMySQL

/// ...

/// Register providers first
try services.register(FluentMySQLProvider())

/// Configure migrations
var migrations = MigrationConfig()
migrations.add(model: User.self, database: .mysql)
services.register(migrations)

/// Other services....
```
注册提供程序将添加Fluent MySQL所需的所有服务以使其正常工作。它还包括一个使用典型开发环境开发的默认数据库配置结构。

如果你没有标准资格你可以重写这个配置结构。

```
/// Register custom MySQL Config
let mysqlConfig = MySQLDatabaseConfig(hostname: "localhost", port: 3306, username: "vapor", password: "mypassword", database: "mydatabase")
services.register(mysqlConfig)
```

一旦你添加了```MigrationConfig```，你应该能够运行你的应用程序并且会看到如下输出：

```
Migrating mysql DB
Migrations complete
Server starting on http://localhost:8080
```

# 查询

既然你已在数据库中创建了模型和相应的模式，让我们进行第一次查询。

```
router.get("users") { req in
    return User.query(on: req).all()
}
```

如果你运行你的应用程序并且在该路径上查询，你应该会看见返回一个空数组。现在你仅仅需要添加一些使用者！恭喜，你的第一个Fluent MySQL模型和迁移工作了。

# 连接

使用Fluent，您始终可以访问底层数据库驱动程序。使用此底层驱动程序执行查询有时称为“原始查询”。

我们看一下原始查询。

```
router.get("mysql-version") { req -> Future<String> in
    return req.withPooledConnection(to: .mysql) { conn in
        return try conn.query("select @@version as v;").map(to: String.self) { rows in
            return try rows[0].firstValue(forColumn: "v")?.decode(String.self) ?? "n/a"
        }
    }
}
```

在上述例子中，```withPooledConnection(to:)```被用来创建一个同有着```.mysql```标志的数据库连接。这是默认的数据库标识符。查阅[Fluent → Database](https://docs.vapor.codes/3.0/fluent/database/#identifier)来了解更多。

一旦我们建立了```MySQLConnection```连接，我们可以在上面进行查询。你可以在[MySQL → Core](https://docs.vapor.codes/3.0/mysql/core/)中获取更多可使用的相关方法。