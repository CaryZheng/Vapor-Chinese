# Using Crypto 使用Crypto

Crypto ([vapor/crypto](https://github.com/vapor/crypto)) 是一个包含与密码和数据生成相关的公共API的库。 这个包包含两个模块：

- `Crypto`
- `Random`

## With Vapor

此包默认包含在Vapor中，只需添加即可：

```swift
import Crypto
import Random
```

## Without Vapor

要使用它, 添加以下代码到 `Package.swift` 文件里.

```swift
// swift-tools-version:4.0
import PackageDescription

let package = Package(
    name: "Project",
    dependencies: [
        ...
        .package(url: "https://github.com/vapor/crypto.git", .upToNextMajor(from: "x.0.0")),
    ],
    targets: [
      .target(name: "Project", dependencies: ["Crypto", "Random", ... ])
    ]
)
```

使用 `import Crypto` 去访问Crypto的APIs，同样使用 `import Random` 去访问Random的APIs.