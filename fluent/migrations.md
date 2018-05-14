# Getting Started with Migrations

Migrations 对数据库结构进行有组织，可测试和可靠变更的一种方式 - 即使在生产中！

Migrations 通常用于为model准备database schema。但是，它们也可以用于对数据库进行常规查询。

在本指南中，我们将介绍创建两种类型的migrations。 

## Model Schema

让我们来看看如何准备一个支持数据库的Schema来接受 
`User`（上一节中[previous section](models.md)的Model）。

就像我们使用`Model`协议一样，我们会将`User`符合`Migration`协议。

```swift
import Fluent

extension User: Migration {

}
```

Swift会告知我们`User`还没遵循协议，让我们添加所需的方法！

### Prepare

第一种实施方法是“prepare”。此方法是你可以对数据库进行你想要的修改。

 对于我们的`User` model，我们只是想创建一个可以存储一个或多个用户的表。为此，我们将在提供的数据库连接上使用`.create（...）`函数。

```swift
extension User: Migration {
    /// See Migration.prepare
    static func prepare(on connection: MySQLConnection) -> Future<Void> {
        return connection.create(self) { builder in
            try builder.field(for: \.id)
            try builder.field(for: \.name)
            try builder.field(for: \.age)
        }
    }
}
```

我们通过`self`（简写为`User.self`，因为这是一个静态方法）作为`.create`方法的第一个参数。这向Fluent表明我们想要为`User`模型创建一个schema。

接下来，我们传递一个闭包，它接受一个`SchemaBuilder`作为我们的`User`模型。 然后，我们可以在这个构建器中调用`.field`来描述我们希望的哪些字段。

于我们将关键路径传递给我们的`用户`模型（用`\ .`表示），Fluent可以查看这些属性的类型。
对于大多数常见类型（`String`，`Int`，`Double`等），Fluent将自动确定要使用的最佳数据库字段类型。

你也可以手动选择要用于给定字段的数据库字段类型。

```swift
try builder.field(type: .text, for: \.name)
```

要了解更多关于创建，更新和删除schemas，请查阅 [Fluent &rarr; Schema Builder](../schema-builder).

### Revert

Revert 和 prepare相反. 它的工作是撤销在prepare中完成的工作. 当你使用`--revert`选项启动你的应用程序时会激活它的用法。 

要为我们的model实现`revert`，我们只需使用`.delete`来表示我们想要删除为`User`创建的schema。

```swift
extension User: Migration {
    /// See Migration.revert
    static func revert(on connection: MySQLConnection) -> Future<Void> {
        return connection.delete(self)
    }
}
```

## Example 例子

我们现在有一个完整的migration功能的model！

```swift
extension TestUser: Migration {
    /// See Migration.prepare
    static func prepare(on connection: SQLiteConnection) -> Future<Void> {
        return connection.create(self) { builder in
            try builder.field(for: \.id)
            try builder.field(for: \.name)
            try builder.field(for: \.age)
        }
    }

    /// See Migration.revert
    static func revert(on connection: SQLiteConnection) -> Future<Void> {
        return connection.delete(self)
    }
}
```

## Done 完成

你有一个可靠的Fluent的model和migration了，可以继续学习 [querying](querying.md)你的model. 