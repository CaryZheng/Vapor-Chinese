# Getting Started with Template Kit

Template Kit ([vapor/template-kit](https://github.com/vapor/template-kit)) 是一个框架，它实现了 Swift 中的模版语言。当前被用于强化 Leaf ([vapor/leaf](https://github.com/vapor/leaf))，并且希望未来能支持更多的语言。

> 注意
> 
> 本篇文档主要是针对想通过 Template Kit 来实现模版语言的开发者的。关于如何使用 Leaf 可参考 [Leaf → Getting Started](../leaf/getting_started.md)。

## Vapor

该 package 包含在 Vapor 中并且默认是被导出的。当导入 ```Vapor``` 后，你可以访问所有 ```TemplateKit``` API。

```swift
import Vapor
```

## Standalone

Template Kit package 是轻量化的、全部由Swift实现，并且关联很少的依赖。这意味着即使不使用 Vapor ，也可以用于任何 Swift 项目中。

引入该 package，只需编辑 Package.swift 文件。

```swift
// swift-tools-version:4.0
import PackageDescription

let package = Package(
    name: "Project",
    dependencies: [
        ...
        .package(url: "https://github.com/vapor/template-kit.git", from: "1.0.0"),
    ],
    targets: [
      .target(name: "Project", dependencies: ["TemplateKit", ... ])
    ]
)
```

使用 ```import TemplateKit``` 来访问相关的 API。

## Overview

让我们看下 Leaf 是如何使用 Template Kit 来渲染视图的。

假设我们有个 ```greeting.leaf``` 模版，包含如下内容：

```
Hello, #capitalize(name)!
```

渲染视图的第一步是将语法解析成抽象语法树(AST)。这是 Leaf 视图渲染的一部分，因为 Leaf 有特定的语法。

Leaf 通过创建符合 [TemplateParser](https://api.vapor.codes/template-kit/latest/TemplateKit/Protocols/TemplateParser.html) 的 ```LeafParser``` 来实现这一点。

```
greeting.leaf -> LeafParser -> AST
```

代码中是这样的：

```swift
func parse(scanner: TemplateByteScanner) throws -> [TemplateSyntax]
```

```greeting.leaf``` 示例中的 AST 看起来是这样的：

```
[
    .raw(data: "Hello. "), 
    .tag(
        name: "capitalize", 
        parameters: [.identifier("name")]
    ),
    .raw(data: "!"), 
]
```

现在 Leaf 已经创建了 AST 。Template Kit 将会负责将 AST 转换成一个已渲染的视图。所有要做的就是使用 ```TemplateData``` 来填充变量。

```swift
let data = TemplateData.dictionary(["name": "vapor"])
```

上述数据将会与 AST 一起，通过 [TemplateSerializer](https://api.vapor.codes/template-kit/latest/TemplateKit/Classes/TemplateSerializer.html) 来创建渲染好的视图。

```
AST + Data -> TemplateSerializer -> View
```

渲染出来的视图看起来像这样：

```
Hello, Vapor!
```

所有这些步骤都是被符合 [TemplateRenderer](https://api.vapor.codes/template-kit/latest/TemplateKit/Protocols/TemplateRenderer.html) 协议的 ```LeafRenderer``` 处理的。一个模版渲染器只是一个包含了 parser 和 serializer 的对象。当你实现它后，你将会获得 Template Kit 中一系列有用的 extension ，它们将帮助导入文件并且缓存解析后的 AST 。最后得到的就是已经渲染好的视图了。

完整的流程是这样的：

```
                            LeafRenderer
                                 |
|----------------------------------------------------------------|
 greeting.leaf -> LeafParser -> AST -> TemplateSerializer -> View
                                 ^
                                /
                   TemplateData
```

代码中该 [method](https://api.vapor.codes/template-kit/latest/TemplateKit/Protocols/TemplateRenderer.html#/s:11TemplateKit0A8RendererPAAE6renderXeXeF) 看起来是这样的：

```swift
public func render(_ path: String, _ context: TemplateData) -> Future<View>
```

Template Kit API详情可见 [API docs](https://api.vapor.codes/template-kit/latest/TemplateKit/index.html) 。