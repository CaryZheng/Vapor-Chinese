# 管理项目

Swift Package Manager（简称：SPM）用于构建项目源码以及各种依赖。它类似于 Cocoapods、Ruby gems、和 NPM 。大多数情况下，直接使用 [Vapor Toolbox](toolbox.md) 即可，toolbox 内部将会与 SPM 进行交互。然而，理解基础概念还是很重要的。

> 提示
> 
> 想了解更多关于 SPM ，可访问 [Swift.org →](https://swift.org/package-manager/)

## Package Manifest

关于 SPM ，首先要看的是 package manfiest 。它位于项目根目录中，并被命名为 ```Package.swift``` 。

## Dependencies

Dependencies 代表你的 package 所需要依赖的其它的 SPM package。所有 Vapor 应用都依赖于 Vapor package ，但是你也可以添加其它想要的 dependency 。

```
// swift-tools-version:4.0
import PackageDescription

let package = Package(
    name: "VaporApp",
    dependencies: [
        // 💧 A server-side Swift web framework. 
        .package(url: "https://github.com/vapor/vapor.git", from: "3.0.0"),
    ],
    targets: [ ... ]
)
```

上面这个示例，你可以看到 [vapor/vapor →](https://github.com/vapor/vapor) 3.0或以上版本是这个 package 的 dependency 。当你在 package 中添加了 dependency ，接下来你必须设置 targets 。

> 提示
> 
> 只要修改了 package manifest，都需要调用 ```vapor update``` 来使得变更生效。

## Targets

Targets 包含了所有的 modules、executables 以及 tests 。

```
// swift-tools-version:4.0
import PackageDescription

let package = Package(
    name: "VaporApp",
    dependencies: [ ... ],
    targets: [
        .target(name: "App", dependencies: ["Vapor"]),
        .target(name: "Run", dependencies: ["App"]),
        .testTarget(name: "AppTests", dependencies: ["App"]),
    ]
)
```

虽然可以添加很多 targets 来组织你的代码，但大部分 Vapor 应用有3个 targets 就足够了。每个 target 声明了它依赖的 modules 。为了在代码中可以 ```import``` 这些 modules ，你必须添加 module 名字。一个 target 可以依赖于工程中其它的 target 或者暴露出来的 modules 。

> 提示
> 
> Executable targets (包含```main.swift```文件的targets) 不能被其它 modules 导入。这就是为什么 Vapor 会有 ```App``` 和 ```Run``` target。任何包含在```App```中的代码都可以在```AppTests```中被测试验证。

## 目录结构

以下是典型的 SPM package 目录结构。

```
.
├── Sources
│   ├── App
│   │   └── (Source code)
│   └── Run
│       └── main.swift
├── Tests
│   └── AppTests
└── Package.swift
```

每个 ```.target``` 对应 ```Sources``` 中的一个文件夹。每个 ```.testTarget``` 对应 ```Tests``` 中的一个文件夹。

## Troubleshooting

如果你遇到 SPM 相关的问题，可以尝试 clean 下工程项目试试。

```
vapor clean
```
