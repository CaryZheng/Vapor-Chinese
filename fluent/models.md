# Fluent Model

Model是Fluent的核心。与其他语言的ORM不同，Fluent不会对返回无类型的数组或字典进行查询。而是使用model查询数据库。这使得Swift编译器能够捕获许多让ORM用户困扰多年的错误。

> 资料
>
>    本指南提供了一个`Model`协议及其相关方法和属性的概述。如果你刚开始使用，请在[Fluent &rarr; 开始](getting-started.md)中查看数据库指南。

`Model`是`Fluent`模块中的一个协议. 它扩展了可用于类型擦除的`AnyModel`协议.

## Conformance

`struct` s 和`class` es 都可以符合`Model`, 然后你必须特别注意 如果你使用了一个`struct`作为Fluent的返回值类型。自从Fluent实现异步工作，对值类型(`struct`)任何的突变都必须返回`Model`的新副本作为future的结果。

通常，你将Model符合特定于数据库包（如：PostgreSQLModel）中的其中一种便利Model。 但是，如果你想定制附加属性, 如类型的`idKey`, 将使用`Model`协议自身.

让我们来看看基本的`Model`一致性是怎样的。

```swift
/// A simple user.
final class User: Model {
    /// See `Model.Database`
    typealias Database = FooDatabase

    /// See `Model.ID`
    typealias ID = Int

    /// See `Model.idKey`
    static let idKey: IDKey = \.id

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

> 提示
>
    >使用`final`可以阻止你的类型被子类化。 这可以让你工作起来更轻松。

## Associated Types

`Model`定义了一些关联类型来帮助Fluent创建类型安全的API供你使用。如果你需要不带关联类型的类型擦除版本，请查看`AnyModel`

### Database

此类型向Fluent表明你打算使用此Model的数据库。使用这些信息，Fluent可以动态地将适当的方法和数据类型添加到你使用此Model时创建的任何 `QueryBuilder`中。

```swift
final class User: Model {
    /// See `Model.Database`
    typealias Database = FooDatabase
    /// ...
}
```

可以通过向类或结构体添加泛型来创建这种关联类型（如`User <T>`）。这对于你尝试为Fluent创建通用扩展的情况非常有用，例如可能是附加服务提供者。

```swift
final class User<D>: Model where D: Database {
    /// See `Model.Database`
    typealias Database = D
    /// ...
}
```

你可以向`D`添加更多条件, 例如`QuerySupporting`或者`SchemaSupporting`。你也可以动态扩展和符合你的通用Model来使用`extension User where D: ... { }`。

也就是说，对于大多数情况，你应该尽可能使用具体的类型别名。 Fluent 3旨在让你通过在你的Model和底层驱动程序之间建立强有力的连接来实现数据库的强大功能。

### ID

该属性定义了你的model将用于其唯一标识符的类型。

```swift
final class User: Model {
    /// See `Model.ID`
    typealias ID = UUID
    /// ...
}
```

这通常会像`Int`, `UUID`, 或 `String`，尽管理论上你可以使用任何你喜欢的类型。

## Properties

`Model`上有几个可覆盖的属性，可用于自定义Fluent与数据库的交互方式。

### Name

每当Fluent需要时，该 `String`将用作Model的唯一标识符。

```swift
final class User: Model {
    /// See `Model.name`
    static let name = "user"
    /// ...
}
```

默认情况下，这是你的Model小写的类型名称。

### Entity

实体是用来表示“table”("表")或"collection"(“集合”)的通用词，具体取决于你用于Fluent的后端类型。 

```swift
final class Goose: Model {
    /// See `Model.entity`
    static let entity = "geese"
    /// ...
}
```

默认情况下，该属性将被[name](#name)以复数形式表示. 在文字不能清楚表达和复数形式的单词不规则的情况下，重写此属性非常有用。

### ID Key

ID键是一个可写的[key path](https://github.com/apple/swift-evolution/blob/master/proposals/0161-key-paths.md) 指向你的Model的唯一标识符属性。

通常这将是一个名为`id`的属性 (对于某些数据库来说，它是`_id`). 然而，理论上你可以使用你喜欢的任何键。

```swift
final class User: Model {
    /// See `Model.ID`
    typealias ID = String
  
    /// See `Model.entity`
    static let idKey = \.username
    
    /// The user's unique username
    var username: String?
    
    /// ...
}
```

`idKey` 属性必须指向具有类型匹配[ID](#id)的可选可写（var）属性。

## 生命周期

在`Model`里有几个生命周期的方法,可以通过重写来关联Fluent事件。

|方法      |描述                                                |引发的操作                    |
|------------|-----------------------------------------------------------|----------------------------|
|`willCreate`|Called before Fluent saves your model (for the first time) |Cancels the save.           |
|`didCreate` |Called after Fluent saves your model (for the first time)  |Save completes. Query fails.|
|`willUpdate`|Called before Fluent saves your model (subsequent saves)   |Cancels the save.           |
|`didUpdate` |Called after Fluent saves your model (subsequent saves)    |Save completes. Query fails.|
|`willRead`  |Called before Fluent returns your model from a fetch query.|Cancels the fetch.          |
|`willDelete`|Called before Fluent deletes your model.                   |Cancels the delete.         |

这是`willUpdate(on:)` 方法的例子.

```swift
final class User: Model {
    /// ...
    
    /// See `Model.willUpdate(on:)`
    func willUpdate(on connection: Database.Connection) throws -> Future<Self> {
        /// Throws an error if the username is invalid
        try validateUsername()
       
        /// Return the user. No async work is being done, so we must create a future manually.
        return Future.map(on: connection) { self }
    }
}
```

## CRUD

Model提供了基本的CRUD方法 (创建[create], 读[read], 更新[update], 删除[delete]).

### 创建

此方法为数据库中的model实例创建新的行/数据项

如果你的Model没有一个ID，调用`.save(on:)`将会重定向到这个方法。

```swift
let didCreate = user.create(on: req)
print(didCreate) /// Future<User>
```
> 资料
>
>    如果你使用的是值类型(`struct`), 则由`.create(on:)`返回的model实例将包含Model的新ID。

### Read

两种方法对于从数据库中读取model非常重要，`find(_:on:)` 和 `query(on:)`。

```swift
/// Finds a user with ID == 1
let user = User.find(1, on: req)
print(user) /// Future<User?>
```

```swift
/// Finds all users with name == "Vapor"
let users = User.query(on: req).filter(\.name == "Vapor").all()
print(users) /// Future<[User]>
```

### Update

此方法更新数据库中model实例关联的现有行/数据项。

如果你的Model已经有一个ID，调用`.save(on:)`会重定向到这个方法。 

```swift
/// Updates the user
let didUpdate = user.update(on: req)
print(didUpdate) /// Future<User>
```

### Delete

此方法从数据库中删除与Model实例关联的现有行/数据项。

```swift
/// Deletes the user
let didDelete = user.delete(on: req)
print(didDelete) /// Future<Void>
```

## Method

`Model`提供了一些便利的方法，使其更容易处理。

### Require ID

此方法返回models ID或引发错误。

```swift
let id = try user.requireID()
```
