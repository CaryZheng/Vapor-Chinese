# Using SQL

SQL 模块可帮助你在 Swift 中构建和序列化 SQL 查询。它具有可扩展的基于协议的设计，并支持 DQL，DML 和 DDL：

* **DQL**：数据查询语言（`SELECT`）
* **DML**：数据处理语言（`INSERT`，`DELETE` 等）
* **DDL**：数据定义语言（`CREATE`，`ALTER` 等）

该模块的目标是帮助你以类型安全，一致的方式构建 SQL 查询。 它可以被 ORM 用来将其数据类型序列化到 SQL 中。 它也可以以更快捷更 Swift (译者注："Swifty"双关语) 的方式生成 SQL。本指南的其余部分将为你提供有关手动使用 SQL 库的概述。

## Data Query

DQL（数据查询语言）用于从一个或多个表中获取数据。 这是通过 `SELECT` 语句完成的。 我们来看看如何序列化下面的SQL查询。

```sql
SELECT * FORM users WHERE name = ?
```

此查询获取了当 name 等于某个参数化值的 `users` 表的所有行。你也可以将一个文本值序列化为数据查询，但强烈建议使用参数化。

让我们用 Swift 构建这个查询。

```swift
// 为 users 表创建新的查询
var users = DataQuery(table: "users")

// 创建 "name = ?" 谓词.
let name = DataPredicate(column: "name", comparison: .equal, value: .placeholder)

// 将谓词作为单独元素（非一组）添加到查询中。
users.predicates.append(.predicate(name))

// 序列化查询语句.
let sql = GeneralSQLSerializer.shared.serialize(query: users)
print(sql) // "SELECT * FROM `users` WHERE (`name` = ?)"
```

这里我们使用 [GeneralSQLSerializer](https://api.vapor.codes/sql/latest/SQL/Classes/GeneralSQLSerializer.html) 单例来序列化查询。你也可以实现自定义序列化 [custom serializers](https://docs.vapor.codes/3.0/sql/overview/#serializer)。

## Data Manipulation

DML（数据操作语言）用于对表中的数据进行转变。 这是通过 `INSERT`，`UPDATE` 和 `DELETE` 等语句完成的。我们来看看如何序列化下面的 SQL 查询。

```sql
INSERT INTO users (name) VALUES (?)
```

此查询将 name 等于参数化值的一行插入 user 表中。你也可以将文本值序列化为数据操作查询，但强烈建议使用参数化。

让我们用 Swift 构建这个查询。

```swift
// 为 users 表创建一个数据操作查询.
var users = DataManipulationQuery(statement: .insert, table: "users")

// 创建 column + value.
let name = DataManipulationColumn(column: "name", value: .placeholder)

// 将 column + value 添加到查询.
users.columns.append(name)

// 序列化查询语句.
let sql = GeneralSQLSerializer.shared.serialize(query: user)
print(sql) // "INSERT INTO `users` (`name`) VALUES (?)"
```

这就是生成 `INSERT` 查询所需的全部内容。让我们来看看如果使用 [.update](https://api.vapor.codes/sql/latest/SQL/Enums/DataManipulationStatement.html) 语句代替这个查询将如何序列化。

```swift
// 修改 statement 类型
users.statement = .update

// 序列化查询语句
let sql = GeneralSQLSerializer.shared.serialize(query: user)
print(sql) // "UPDATE `users` SET `name` = ?"
```

你可以看到 SQL 使用适当的语法生成了等效的 `UPDATE` 查询。

## Data Definition

DDL（数据定义语言）用于创建，更新和删除数据库中的数据结构。 这是通过 `CREATE TABLE`，`DROP TABLE` 等语句完成的。下面我们来看看如何序列化下面的 SQL 查询。

```sql
CREATE TABLE users (id INTEGER PRIMARY KEY, name TEXT)
```

此示例使用类似 SQLite 的语法，但这不是必需的。让我们在 Swift 中生成这个查询。

```swift
// 为 users 表创建一个数据定义查询.
var users = DataDefinitionQuery(statement: .create, table: "users")

// 创建并添加 id 列.
let id = DataDefinitionColumn(name: "id", dataType: "INTEGER", attributes: ["PRIMARY KEY"])
users.addColumns.append(id)

// 创建并添加 name 列.
let name = DataDefinitionColumn(name: "name", dataType: "TEXT")
users.addColumns.append(name)

// 序列化查询语句.
let sql = GeneralSQLSerializer.shared.serialize(query: user)
print(sql) // "CREATE TABLE `users` (`id` INTEGER PRIMARY KEY, `name` TEXT)"
```

这就是生成 `CREATE` 查询所需的全部内容。

## Serializer

SQL 模块附带的默认 `GeneralSQLSerializer` 会生成相当 "标准" 的SQL语法。但是每种SQL（SQLite，MySQL，PostgreSQL 等）都有其语法的特定规则。

为了解决这个问题，所有的序列化器都支持 [SQLSerializer](https://api.vapor.codes/sql/latest/SQL/Protocols/SQLSerializer.html) 协议。所有序列化方法都是在此协议上定义的，并带有一个默认实现。自定义序列化器可以实现此协议并仅实现需要自定义的方法。

我们来看看如何使用 `$x` 占位符实现 `PostgreSQLSerializer` 而不是 `?`。

```swift
/// Postgres-flavor SQL serializer.
final class PostgreSQLSerializer: SQLSerializer {
    /// 跟踪当前的占位符数量。
    var count: Int

    /// 创建新的 `PostgreSQLSerializer`.
    init() {
        self.count = 1
    }

    /// 参考 `SQLSerializer`.
    func makePlaceholder() -> String {
        defer { count += 1 }
        return "$\(count)"
    }
}
```

在这里，我们实现了 `PostgreSQLSerializer` 并重写了 `SQLSerializer`的 `makePlaceholder()` 方法。

现在让我们使用这个序列化器来序列化前面例子中的 [data query](https://docs.vapor.codes/3.0/sql/overview/#data-query)。

```swift
// 前面数据查询的例子
let users: DataQuery = ... 

// 序列化查询语句.
let sql = PostgreSQLSerializer().serialize(query: users)
print(sql) // "SELECT * FROM `users` WHERE (`name` = $1)"
```

就是这样，恭喜你实现了一个自定义序列化器。


## API Docs

查看 [API docs](https://api.vapor.codes/sql/latest/SQL/index.html) 获取有关 SQL API 的更多信息。

