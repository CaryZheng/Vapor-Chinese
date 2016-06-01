# Validation
---
Vapor 提供了不同方式来验证进入到 application 中的数据。让我们看看通常情况。

## Common Usage
---
一些有用方便的 validators 被默认包含。你可以使用这些来验证进入到 application 中的数据，或者整合它们创建出你自己的 validators。

让我们看看验证数据最常用的方式。

##### Swift
```
class Employee {
    var email: Valid<Email>
    var name: Valid<Name>

    init(request: Request) throws {
        name = try request.data["name"].validated()
        email = try request.data["email"].validated()
    }
}
```

此处我们有一个典型的 ```Employee``` 模型，它拥有 ```email``` 和 ```name``` 属性。通过声明这些属性为 ```Valid<>``` ，你将确保这些属性只能包含有效的数据。Swift 类型检查系统将阻止任何没通过验证的内容。

从 ```Valid<>``` 属性中取数据，你必须使用 ```.validated()``` 方法。对于由 ```request.data``` 返回的任意数据都是有效的。

```Email``` 是 Vapor 中一个实际的 validator ，但是 ```Name``` 不是。让我们看看如何创建一个 Validator 。

##### Swift
```
Valid<OnlyAlphanumeric>
Valid<Email>
Valid<Unique<T>>
Valid<Matches<T>>
Valid<In<T>>
Valid<Contains<T>>
Valid<Count<T>>
```

## Validators vs. ValidationSuites
---
Validators ，如 ```Count``` 和 ```Contains``` 可以有许多种配置。例如：

##### Text
```
let name: Valid<Count<String>> = try "Vapor".validated(by: Count.max(5))
```

这里我们验证 ```String``` 至多5个字符长度。 ```Valid<Count>``` 类型表明字符串已经被验证是一个特定长度，但它不能准确表示 count 是多少。字符串可以被验证小于3个字符或者大于一百万。

因此，类型安全方面 ```Validator``` 可能不是一些应用所期望的那样。

```ValidationSuite``` 修复了该问题。联合 ```Validator``` 和 ```ValidationSuite``` 可准确描述什么数据类型可以被验证为有效的。

## Custom Validator
---
这里讲讲如何创建一个自定义的 ```ValidationSuite``` 。

##### Swift
```
class Name: ValidationSuite {
    static func validate(input value: String) throws {
        let evaluation = OnlyAlphanumeric.self
            && Count.min(5)
            && Count.max(20)

        try evaluation.validate(input: value)
    }
}
```

你必须实现一个方法。 在该方法中，使用任何其它的 validator 或 逻辑去创建自定义 validator 。这里我们定义了 ```Name``` ，只接受纯字母的 ```String``` ，并且长度介于5到20之间。

现在我们可以确保任何 ```Valid<Name>``` 类型的内容都符合这些规则。

## Combining Validators
---
```Name``` validator 中，你可以看到使用 ```&&``` 来联合 validator 。你可以使用 ```&&``` 或者 ```||``` 来联合任何 validator ，就好像你使用 ```if``` 语句似的。

你也可以使用 ```!``` 来反转 validator 的结果。

##### Swift
```
let symbols = input.validated(by: !OnlyAlphanumeric.self)
```

## Testing Validity
---
```validated() throw``` 是最常用的方法用于验证，还有两种其它方法。

##### Swift
```
let passed = input.passes(Count.min(5))
let valid = try input.tested(Count.min(5))
```

```passes()``` 返回一个 ```boolean``` ，该值表明是否通过测试验证。如果没有通过验证 ```tested()``` 将会抛出异常。但不像 ```validated``` 返回 ```Valid<>``` 类型， ```tested()``` 返回原始类型。

## Validation Failures
---
Vapor 将会在 ```ValidationMiddleware``` 中自动捕获 validation failures 。但是你也可以在自己或自定义 response 中捕获特定的失败类型。

##### Swift
```
do {
    //validation here
} catch let error as ValidationErrorProtocol {
    print(error.message)
}
```