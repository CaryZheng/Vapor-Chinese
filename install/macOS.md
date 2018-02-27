想要在 macOS 上使用 Vapor，你只需要安装 Xcode 9 或更高版本。

> 提示：可以通过Xcode来安装Swift，但是之后你可以使用任意喜欢的文本编辑器来开发Vapor应用。

## 安装Xcode

通过 Mac App Store 安装 [Xcode 9 或更高版本](https://itunes.apple.com/cn/app/xcode/id497799835?mt=12)。

![](../image/install_macos_xcode_snapshot.png)

> 注意：Xcode下载后，你必须打开它来完成安装。这可能需要一段时间。

## 验证安装

通过打开终端运行如下命令来验证是否安装成功：

```
swift --version
```

你将看到如下类似输出：

```
Apple Swift version 4.0.2 (swiftlang-900.0.69.2 clang-900.0.38)
Target: x86_64-apple-macosx10.9
```

Vapor需要Swift 4。

## 安装Vapor

安装好Swift 4后，我们开始安装 [Vapor Toolbox](../getting_started/toolbox.md)

Toolbox 包含所有Vapor依赖，并可以通过便捷的```CLI```工具来创建新项目。

```
brew install vapor/tap/vapor
```

> 注意：如果还没安装Homebrew的话，请先安装它 [brew.sh →](https://brew.sh/)

## 验证安装

通过打开终端运行如下命令来验证是否安装成功：

```
vapor --help
```

你将会看到一长串可用命令集。

## 完成

Vapor安装好后，就可以创建第一个应用了，可参考 [Getting Started → Hello, world](../getting_started/hello_world.md)