# Async 概览
可能你已经注意到Vapor中的一些API会期望或返回一个通用的`Future`类型。对于第一次听说`futures`的人起初会存在些困惑但别担心，Vapor会让它们变得很易用。

Promises 和 futures是有关联的，但也是截然不同的类型。Promises 用来创建futures。通常，你将使用Vapor的APIs所返回的future来工作从而不必担心创建promises。

| 表格      | 第一列     | 第二列     |
| ---------- | :-----------:  | :-----------: |
| 第一行     | 第一列     | 第二列     |
|type | description| mutability|methods|


