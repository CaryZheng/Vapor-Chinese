# Fluent Transactions Fluent事务

事务允许你在将数据保存到数据库之前确保多个操作成功完成。启动事务后，你可以正常运行Fluent查询。但是，在事务完成之前，不会将数据保存到数据库。如果在事务操作期间（由你或数据库）抛出错误，则所有更改都不会生效。

要执行事务，您需要访问可以连接到数据库的内容。这通常是传入的HTTP请求。使用[`transaction(on:_:)`](https://api.vapor.codes/fluent/latest/Fluent/Extensions/DatabaseConnectable.html#/s:11DatabaseKit0A11ConnectableP6FluentE11transaction2on_3NIO15EventLoopFutureCyqd_0_GAA0A10IdentifierVyqd__G_AJ10ConnectionQyd__KctAD21TransactionSupportingRd__r0_lF)方法。

使用[Getting Started &rarr; Choosing a Driver](getting-started/#choosing-a-driver) 中的数据库名称填充下面的Xcode代码中占位符。

```swift
req.transaction(on: .<#dbid#>) { conn in
    // use conn as your connection
}
```

进入事务闭包后，必须使用提供的连接（在示例中名为`conn`）来执行查询。

闭包期望通用的future返回值。一旦这个future成功完成，事务将被提交。

```swift
var userA: User = ...
var userB: User = ...

return req.transaction(on: .<#dbid#>) { conn in
    return userA.save(on: conn).flatMap { _ in
        return userB.save(on: conn)
    }.transform(to: HTTPStatus.ok)
}
```

上面的事例将在事务之前保存用户A  _then_ 用户B.如果任一用户未能保存，则两者都不会保存。事务完成后，结果将转换为指示完成的简单HTTP状态响应。