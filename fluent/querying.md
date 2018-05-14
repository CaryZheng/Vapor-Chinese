# Querying Models

一旦你有[model](models.md) (和可选[migration](migrations.md)) 你就可以查询数据库以创建、读取、更新和删除数据。

## Connection

你需要查询数据库的第一件事是连接到它。幸运的是，他们很容易做到。

你可以使用应用程序或传入请求来创建数据库连接。你只需访问数据库标识符。

### Request

访问数据库连接的首选方法是通过传入请求。

```swift
router.get(...) { req in
    return req.withConnection(to: .foo) { db in
        // use the db here
    }
}
```

第一个参数是数据库的标识符。第二个参数是接受连接该数据库的闭包。

!!! 提示
    尽管使用`.withConnection(to: ...)` 接受数据库连接, 我们通常只使用`db`。

闭包期待返回一个`Future<Void>`. 当这个future完成时, 连接将被释放到Fluent的连接池. 这是常见的返回查询的实现，我们很快就可以看见。

### Application

你也可以使用应用程序创建数据库连接。这对于必须从请求/响应事件以外的情况下访问数据库非常有用。

```swift
let res = app.withConnection(to: .foo) { db in
    // use the db here
}
print(res) // Future<T>
```

这通常在你的应用程序的[boot section](../getting-started/structure.md#boot)中完成。

!!! 警告
    不要在路由闭包中使用应用程序创建的数据库连接（响应请求时）。始终使用传入请求创建连接以避免线程问题。

## Create

要创建（保存）Model到数据库，首先初始化model的一个实例，然后调用`.save(on：)`。

```swift
router.post(...) { req in
    return req.withConnection(to: .foo) { db -> Future<User> in
        let user = User(name: "Vapor", age: 3)
        return user.save(on: db).transform(to: user) // Future<User>
    }
}
```

### Response

`.save（on：）`返回一个`Future <Void>`，在当用户保存好时完成。在这个例子中，我们通过调用`.map`并传入最近保存的用户，将该`Future <Void>`映射到`Future <User>`。 

你也可以使用`.map`来返回一个简单的成功响应。

```swift

router.post(...) { req in
    return req.withConnection(to: .foo) { db -> Future<HTTPResponse> in
        let user = User(name: "Vapor", age: 3)
        return user.save(on: db).map(to: HTTPResponse.self) {
            return HTTPResponse(status: .created)
        }
    }
}
```

### Multiple

如果你有多个实例要保存，请使用数组来完成。数组只包含futures。

```swift
router.post(...) { req in
    return req.withConnection(to: .foo) { db -> Future<HTTPResponse> in
        let marie = User(name: "Marie Curie", age: 66)
        let charles = User(name: "Charles Darwin", age: 73)
        return [
            marie.save(on: db),
            charles.save(on: db)
        ].map(to: HTTPResponse.self) {
            return HTTPResponse(status: .created)
        }
    }
}
```

## Read

要从数据库中读取Model，请在数据库连接上使用`.query()`来创建一个[QueryBuilder](../query-builder)。

### All

使用`.all()`从数据库中获取Model的所有实例。

```swift
router.get(...) { req in
    return req.withConnection(to: .foo) { db -> Future<[User]> in
        return db.query(User.self).all()
    }
}
```

### Filter

使用 `.filter(...)`将[filters](../query-builder#filters)应用到查询中。

```swift
router.get(...) { req in
    return req.withConnection(to: .foo) { db -> Future<[User]> in
        return try db.query(User.self).filter(\User.age > 50).all()
    }
}
```

### First

你也可以使用`.first()`来获得第一个结果。

```swift
router.get(...) { req in
    return req.withConnection(to: .foo) { db -> Future<User> in
        return try db.query(User.self).filter(\User.name == "Vapor").first().map(to: User.self) { user in
            guard let user = user else {
                throw Abort(.notFound, reason: "Could not find user.")
            }

            return user
        }
    }
}
```

注意我们在这里使用`.map(to：)`将`.first()`返回的可选用户转换为非可选用户，否则我们会抛出一个错误。

## Update

```swift
router.put(...) { req in
    return req.withConnection(to: .foo) { db -> Future<User> in
        return db.query(User.self).first().map(to: User.self) { user in
            guard let user = $0 else {
                throw Abort(.notFound, reason: "Could not find user.")
            }

            return user
        }.flatMap(to: User.self) { user in
            user.age += 1
            return user.update(on: db).map(to: User.self) { user }
        }
    }
}
```

注意我们在这里使用`.map(to：)`将`.first()`返回的可选用户转换为非可选用户，否则我们会抛出一个错误。

## Delete
```swift
router.delete(...) { req in
    return req.withConnection(to: .foo) { db -> Future<User> in
        return db.query(User.self).first().map(to: User.self) { user in
            guard let user = $0 else {
                throw Abort(.notFound, reason: "Could not find user.")
            }
            return user
        }.flatMap(to: User.self) { user in
            return user.delete(on: db).transfom(to: user)
        }
    }
}
```

注意我们在这里使用`.map(to：)`将`.first()`返回的可选用户转换为非可选用户，否则我们会抛出一个错误。