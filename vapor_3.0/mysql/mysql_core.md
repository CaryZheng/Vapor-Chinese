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
> 即使你使用[Fluent MySQL](https://docs.vapor.codes/3.0/mysql/fluent/)，你任将可以使用所有的MySQL core功能。

# 入门

我们来看看怎么入门MySQL Core

## 包

使用MySQL Core的第一步是将其作为依赖添加到你的项目中的SPM包清单文件中。

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

不要忘记将模块作为依赖添加到```targets```数组中。一旦你添加了依赖，使用以下命令重新生成Xcode项目：

```
vapor xcode
```

# 配置

下一步是在[configure.swift](https://docs.vapor.codes/3.0/getting-started/structure/#configureswift)文件中配置数据库。

```
import MySQL

/// ...

/// Register providers first
try services.register(MySQLProvider())
```

注册提供程序将添加Fluent MySQL所需的所有服务以使其正常工作。它还包括一个使用典型开发环境开发的默认数据库配置结构。

如果你没有标准资格你可以重写这个配置结构。

```
/// Register custom MySQL Config
let mysqlConfig = MySQLDatabaseConfig(hostname: "localhost", port: 3306, username: "vapor")
services.register(mysqlConfig)
```

# 查询

既然数据库已经配置好了，你可以进行第一次查询。

```
router.get("mysql-version") { req -> Future<String> in
    return req.withPooledConnection(to: .mysql) { conn in
        return try conn.query("select @@version as v;").map(to: String.self) { rows in
            return try rows[0].firstValue(forColumn: "v")?.decode(String.self) ?? "n/a"
        }
    }
}
```

访问此路由应显示你的MySQL版本。

# 连接

一个```MySQLConnection```通常使用Request容器创建并且可以执行两种不同类型的查询。

## 创建

有一些使用```Container```（典型的是```Request ```）创建```MySQLConnectiona``` 的方法。

```
return req.withPooledConnection(to: .mysql) { conn in
    /// ...
}
return req.withConnection(to: .mysql) { conn in
    /// ...
}
```

顾名思义，```withPooledConnection(to:)```利用连接池，而```withConnection(to:)```不是。连接池是确保即使在峰值负载下也不超过数据库限制并发量的好方法。

你也可以使用```MySQLDatabase.makeConnection(on:)```和传递一个[Worker](https://docs.vapor.codes/3.0/getting-started/async/)来手动创建连接。

## 简单查询

用```.simpleQuery(_:)```在MySQL数据库上执行不绑定任何参数的查询。你发送给MySQL的某些查询实际上可能要求使用该```simpleQuery(_:)```方法而不是参数化方法。

> 注意
>
> 此方法以文本编码方式发送和接收数据，这意味着它不适合传输像整数这样的数据。

```
let rows = req.withPooledConnection(to: .mysql) { conn in
    return conn.simpleQuery("SELECT * FROM users;")
}
print(rows) // Future<[[MySQLColumn: MySQLData]]>

```

你还可以选择接收回调中的每一行，这对于节省大型查询的内存非常有用。

```
let done = req.withPooledConnection(to: .mysql) { conn in
    return conn.simpleQuery("SELECT * FROM users;") { row in
        print(row) // [MySQLColumn: MySQLData]
    }
}
print(done) // Future<Void>
```

## 参数化查询

MySQL还支持发送参数化查询（有时称为预准备语句）。此方法允许你将数据占位符插入SQL语句并单独发送这些数值。

通过参数化查询发送的数据是二进制编码的，这使得它更有效地发送一些数据类型。通常，你应该尽可能使用参数化查询。

```
let users = req.withPooledConnection(to: .mysql) { conn in
    return try conn.query("SELECT *  users WHERE name = $1;", ["Vapor"])
}
print(users) // Future<[[MySQLColumn: MySQLData]]>
```

你还可以提供类似于简单查询的回调，以单独处理每一行。

