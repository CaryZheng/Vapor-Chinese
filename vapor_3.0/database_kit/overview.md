# Using Database Kit

Database Kit 是一个用来配置和操作数据库的框架。它可以管理 pool connections、创建缓存和日志查询。

许多 Vapor packages，比如 Fluent 驱动、Redis 和 Vapor core，都是基于 Database Kit 实现的。本篇指南将介绍一些你可能会用到的通用API。

## Config

首先你接触到的通常会是 [DatabasesConfig](https://api.vapor.codes/database-kit/latest/DatabaseKit/Structs/DatabasesConfig.html) 结构。该类型可以配置一个或多个数据库，最后会产生 [Database](https://api.vapor.codes/database-kit/latest/DatabaseKit/Structs/Databases.html) 结构。这通常发生在 ```configure.swift``` 中。

```swift
// Create a SQLite database.
let sqliteDB = SQLiteDatabase(...)

// Create a new, empty DatabasesConfig.
var dbsConfig = DatabasesConfig()

// Register the SQLite database using '.sqlite' as an identifier.
dbsConfig.add(sqliteDB, as: .sqlite)

// Register more DBs here if you want

// Register the DatabaseConfig to services.
services.register(dbsConfig)
```

使用 [add(...)](https://api.vapor.codes/database-kit/latest/DatabaseKit/Structs/DatabasesConfig.html) 方法，你可以注册 [Database](https://api.vapor.codes/database-kit/latest/DatabaseKit/Protocols/Database.html) 到 config 中。你可以注册 database 实例，database 类型，或者一个创建 datadase 的闭包。当容器启动时，后面两个方法将可用。

你也可以配置 database 选项，比如开启日志。

```swift
// Enable logging on the SQLite database
dbsConfig.enableLogging(for: .sqlite)
```

详情可见 ```logging``` 小节。

## Identifier

大多数 database 的整合都提供了一个默认的 [DatabaseIdentifier](https://api.vapor.codes/database-kit/latest/DatabaseKit/Structs/DatabaseIdentifier.html) 来使用。然而，你也可以创建自己的。通常是创建一个 static extension 来实现。

```swift
extension DatabaseIdentifier {
    /// Test database.
    static var testing: DatabaseIdentifier<MySQLDatabase> {
        return "testing"
    }
}
```

```DatabaseIdentifier``` 是 ```ExpressibleByStringLiteral``` 类型，它允许你创建一个像 ```String``` 的类型。

## Databases

一旦你注册了 ```DatabasesConfig``` 到你的 services 中，并启动了一个容器，你就可以利用 [Container](https://api.vapor.codes/database-kit/latest/DatabaseKit/Extensions/Container.html) 上的 extension 来创建连接。

```swift
// Creates a new connection to `.sqlite` db
app.withNewConnection(to: .sqlite) { conn in
    return conn.query(...) // do some db query
}
```

下一节将介绍如何创建和管理 connections 。

## Connections

Database Kit 主要包含创建、管理和 pooling connections 这几块。创建新的 connect 会占用应用的时间开销，许多云服务限制了一个服务的连接总数。因此，对于高并发的 web 应用，谨慎管理连接池是很重要的。

## Pools

Connection 管理的常见解决方案是使用连接池。这些 pool 通常有连接数上限。每次需要一个 connection 的时候，连接池首先检查是否有可用的 connection。如果没有可用的，才会创建新的 connection 。如果没有可用的 connection 并且连接池已经达到上限，新的 connection 请求将需要等到一个可用的 connection 返回才行。

最简单的请求和释放 pooled connection 方法是 [withPooledConnection(...)](https://api.vapor.codes/database-kit/latest/DatabaseKit/Extensions/Container.html#/s:11DatabaseKit20withPooledConnectionXeXeF) 。

```swift
// Requests a pooled connection to `.psql` db
req.withPooledConnection(to: .psql) { conn in
    return conn.query(...) // do some db query
}
```

该方法将请求一个 connection 到指定的 database 中，并且当 connection 可用时会调用提供的闭包。当闭包中的 ```Future``` 完成后，该 connection 将自动回到连接池中。

如果你需要在闭包外访问 connection ，你可以使用相关的 request / release 方法。

```swift
// Request a connection from the pool and wait for it to be ready.
let conn = try app.requestPooledConnection(to: .psql).wait()

// Ensure the connection is released when we exit this scope.
defer { app.releasePooledConnection(conn, to: .psql) }
```

你可以通过使用 [DatabaseConnectionPoolConfig](https://api.vapor.codes/database-kit/latest/DatabaseKit/Structs/DatabaseConnectionPoolConfig.html) 结构来配置连接池。

```swift
// Create a new, empty pool config.
var poolConfig = DatabaseConnectionPoolConfig()

// Set max connections per pool to 8.
poolConfig.maxConnections = 8

// Register the pool config.
services.register(poolConfig)
```

为了避免竞争条件（race conditions），pools 从不会在事件循环中共享，通常每个事件循环每个 database 都对应一个 pool 。这意味着应用中的 connection 数量等同于 ```numThreads * maxConns``` 。

## New

如果有需要，你可以创建一个新的 connection 。这不会影响到你的 pooled connections 。当测试运行 app 时，创建新的 connection 是很有用的。但不要在路由闭包中尝试使用，因为这将创建很多 connection ，并会阻塞你的应用。

与 pooled connection 类似，打开和关闭新的 connection 可以使用 [withNewConnection(...)](https://api.vapor.codes/database-kit/latest/DatabaseKit/Extensions/Container.html#/s:11DatabaseKit17withNewConnectionXeXeF) 来完成。

```swift
// Creates a new connection to `.sqlite` db
app.withNewConnection(to: .sqlite) { conn in
    return conn.query(...) // do some db query
}
```

该方法创建了一个新的 connection ，当 connection 被打开时，将会调用该闭包。当 ```Future``` 返回了结果，这个 connection 将会自动被关闭。

你也可以通过 [newConnection(...)](https://api.vapor.codes/database-kit/latest/DatabaseKit/Extensions/Container.html#/s:11DatabaseKit13newConnectionXeXeF) 来简化打开新 connection 的方式。

```swift
// Creates a new connection to `.sqlite` db
let conn = try app.newConnection(to: .sqlite).wait()

// Ensure the connection is closed when we exit this scope.
defer { conn.close() }
```

## Logging

Database 通过 [LogSupporting](https://api.vapor.codes/database-kit/latest/DatabaseKit/Protocols/LogSupporting.html) 协议可以支持日志的查询。可以通过 ```DatabasesConfig``` 来进行配置。

```swift
// Enable logging on the SQLite database
dbsConfig.enableLogging(for: .sqlite)
```

默认情况下，使用使用简单的 print logger，但你也可以使用一个自定义的 ```DatabaseLogHandler``` 。

```swift
// Create a custom log handler.
let myLogger: DatabaseLogHandler = ...

// Enable logging on SQLite w/ custom logger.
dbsConfig.enableLogging(for: .sqlite, logger: myLogger)
```

## Keyed Cache

Database 通过 [KeyedCacheSupporting](https://api.vapor.codes/database-kit/latest/DatabaseKit/Protocols/KeyedCacheSupporting.html) 协议来实现带 key 的缓存。符合该协议的 Database可以创建 [DatabaseKeyedCache](https://api.vapor.codes/database-kit/latest/DatabaseKit/Classes/DatabaseKeyedCache.html) 实例。

Keyed caches 可以获取、设置、删除 ```Codable``` 类型值。它们通常被称为 “键值存储”。

你可以使用 [Container](https://api.vapor.codes/database-kit/latest/DatabaseKit/Extensions/Container.html#/s:11DatabaseKit10keyedCacheXeXeF) 上的 extension 来创建一个 keyed cache 。

```swift
// Creates a DatabaseKeyedCache with .redis connection pool
let cache = try app.keyedCache(for: .redis)

// Sets "hello" = "world"
try cache.set("hello", to: "world").wait()

// Gets "hello"
let world = try cache.get("hello", as: String.self).wait()
print(world) // "world"

// Removes "hello"
try cache.remove("hello").wait()
```

详情可见 [KeyedCache](https://api.vapor.codes/database-kit/latest/DatabaseKit/Protocols/KeyedCache.html) 协议。

## API Docs

详情可见 [API docs](https://api.vapor.codes/database-kit/latest/DatabaseKit/index.html) 。