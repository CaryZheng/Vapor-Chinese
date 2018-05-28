# Validation Overview
`Validation` 是用于验证发送至你的应用的数据的模块。它可以帮助验证如names、emails等内容，同时也是易于扩展的，允许你方便的的创建自定义验证。

## Swift & Codable
`Swift` 的强类型系统和 `Codable` 可以方便的帮助 web 应用完成大部分基础验证工作。

```Swift
struct User: Codable {
    var id: UUID?
    var name: String
    var age: Int
    var email: String?
}
```
例如：当你解码上面的 `User` 模型时，`Swift` 会自动帮你确认以下内容：

* `id` 是一个有效的 `UUID` 或者为 `nil`。
* `name` 是一个有效的 `String` 且不为 `nil`。
* `age` 是一个有效的 `Int` 且不为 `nil`。
* `email` 是一个有效的 `String` 或者为 `nil`。

这是很好的第一步，但还有改进的空间。这里还有一些 `Swift` 和 `Codable` 不关心但不够理想的地方：

* `name` 是空字符串 `""`
* `name` 包含非字母数字字符
* `age` 是负数 `-42`
* `email` 不是正确的格式 `test@@vapor.codes`

幸运的是 `Validation` 可以提供帮助。

## Validatable
我们来看看 `Validation` 如何帮助你验证传入的数据。首先我们从为前面的 `User` 模型实现 **[Validatable](https://api.vapor.codes/validation/latest/Validation/Protocols/Validatable.html)** 协议开始。

> 注意
> 
> 这里假设 `User` 已经符合 `Reflectable` 协议（使用 `Fluent` 的 `Model` 协议时默认添加）。如果没有，您需要手动实现符合 `Reflectable` 协议。

```swift
extension User: Validatable {
    /// See `Validatable`
    static func validations() -> Validations<User> {
        // define validations
    }
}

let user = User(...)
// 由于 `User` 符合 `Validatable` 协议，我们得到了一个新的 `validate()` 方法
// 一旦验证失败，将会抛出异常。
try user.validate()
```

这是 `Validatable` 协议的基本结构。我们再来看看我们如何实现 `validations()` 方法。

## Validations
首先我们验证 name 至少有3个字符长。

```swift
extension User: Validatable {
    /// see `Validatable`
    static func validations() throws -> Validations<User> {
        var validations = Validations(User.self)
        try validations.add(\.name, .count(3...))
        return validations
    }
}
```

`count(...)` 验证接受 `Swift` [Range](https://developer.apple.com/documentation/swift/range) 参数，并验证集合的数量是否在此范围内。通过仅在 `...` 的左侧设置一个值，我们设置了一个有下界的范围。

[这里](https://api.vapor.codes/validation/latest/Validation/Structs/Validator.html)可以看到所有可用的验证器。

## Operators
验证 name 有三个字符或以上固然很好，但我们还希望确保 name 只含有字母数字字符。我们可以通过使用 `&&` 组合多个验证器。

```swift
try validations.add(\.name, .count(3...) && .alphanumeric)
```

现在，只有三个或更多个字符和字母数字时，我们的 name 才会被视为有效。  
[这里](https://api.vapor.codes/validation/latest/Validation/Functions.html)可以看到所有可用的操作符。

## Nil
只有当值存在时，你才可能要对可选项进行验证。`&&` 和 `||` 操作符有特殊的重载来帮助你做到这一点。

```swift
try validations.add(\.email, .email || .nil)
```

[nil](https://api.vapor.codes/validation/latest/Validation/Structs/Validator.html#/s:10Validation9ValidatorV3nilXevZ) 验证器用于检查一个 `T?` 可选值是否为 `nil`。

[email](https://api.vapor.codes/validation/latest/Validation/Structs/Validator.html#/s:10Validation9ValidatorVAASSRszlE5emailACySSGvZ) 验证器用于检查一个 `String` 是否符合邮箱地址格式。但是，在 `User` 中这个属性是一个 `String?`，意味着 email 属性不能直接使用这个验证器。
我们可以使用 `||` 组合两个操作来表达我们想要的验证：如果 email 不为 `nil`，则验证它是否符合邮箱格式。

## Validate
让我们用的新知识完成剩下的验证。

```swift
extension User: Validatable {
    /// See `Validatable`
    static func validations() throws -> Validations<User> {
        var validations = Validations(User.self)
        try validations.add(\.name, .alphanumeric && .count(3...))
        try validations.add(\.age, .range(18...))
        try validations.add(\.email, .email || .nil)
        return validations
    }
}
```

现在让我们来尝试验证我们的模型。

```swift
router.post(User.self, at: "users") { req, user -> User in
    try user.validate()
    return user
}
```

当你请求该路由时，如果数据不符合你的验证，你应该会看到抛出的异常错误。如果数据正确，则您的 user 模型将成功返回。

恭喜你已经设置了你的第一个 `Validatable` 模型。查看 [API docs](https://api.vapor.codes/validation/latest/Validation/index.html) 以获取更多的信息和示例代码。

