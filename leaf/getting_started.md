# Leaf

**Leaf**是一门模板语言，可以结合 **Futrues**,**Reactive Streams** 和 **Codable**使用。本章会简单介绍如何在在Vapor工程中引用Leaf

# 目录结构示例

~~~
Hello
├── Package.resolved
├── Package.swift
├── Public
├── Resources
│   ├── Views
│   │   └── hello.leaf
├── Public
│   ├── images (images resources)
│   ├── styles (css resources)
├── Sources
│   ├── App
│   │   ├── boot.swift
│   │   ├── configure.swift
│   │   └── routes.swift
│   └── Run
│       └── main.swift
├── Tests
│   ├── AppTests
│   │   └── AppTests.swift
│   └── LinuxMain.swift
└── LICENSE
~~~

# 在工程中添加Leaf

要将Leaf引入Vapor工程中，最简单的办法是在 Package.swift 中添加 Leaf仓库作为依赖

~~~swift
// swift-tools-version:4.0
import PackageDescription

let package = Package(
    name: "project1",
    dependencies: [
        // 💧 A server-side Swift web framework.
        .package(url: "https://github.com/vapor/vapor.git", .branch("beta")),
        .package(url: "https://github.com/vapor/leaf.git", .branch("beta")),
    ],
    targets: [
        .target(
            name: "App",
            dependencies: ["Vapor", "Leaf"]
        ),
        .target(name: "Run", dependencies: ["App"]),
        .testTarget(name: "AppTests", dependencies: ["App"]),
    ]
)

~~~

引入Leaf package 后，你必须通过修改 **configure.swift** 文件来配置Leaf

1. 添加 **import Leaf** 到需要使用的Swift文件的顶部和渲染模板文件中
2. 添加 **try services.register(LeafProvider())**到 **configure()**方法中，使路由可以渲染需要的Leaf模板

# 语法高亮

你或许希望安装一些第三方工具包，来提供 Leaf 模板的语法高亮显示

### Atom

[language-leaf](https://atom.io/packages/language-leaf) by ButkiewiczP

### Xcode

Xcode目前没有支持语法高亮。然而，使用Xcode的 **HTML Syntax Coloring** 会有一些帮助。 选择一个或多个 Leaf 文件，然后选择**Editor > Syntax Coloring > HTML** .之后你选择的文件就可以使用HTML Syntax Coloring，但是这些帮助是有限的，当你运行工程后，会发现这些关联会被移除


这里有一种方法可以[永久关联](https://stackoverflow.com/questions/9050035/how-to-make-xcode-recognize-a-custom-file-extension-as-objective-c-for-syntax-hi)，但这可能会花费你一些时间.

### VS Code

[html-leaf](https://marketplace.visualstudio.com/items?itemName=Francisco.html-leaf) by FranciscoAmado

### CLion & AppCode

CLion & AppCode 的 Leaf 插件已经完成了一些准备工作， Java技能和兴趣的缺失导致了进度非常缓慢，如果你有 Intellij SDK 经验并且想帮助我们的话，在 [Vapor Slack](https://vapor.team)上发送消息给 Tom Holland 