# Command Overview

本指南通过创建一个自定义的 CLI 示例来介绍 Command 模块。比如，我们实现 [cowsay](https://en.wikipedia.org/wiki/Cowsay)，它是一个命令，用来输出包含一头牛信息的 ASCII 图片。

> 提示
> 
> 你可以通过 ```brew install cowsay``` 来安装 ```cowsay``` 程序。

```
$ cowsay Hello
  -----
< Hello >
  -----
          \   ^__^
           \  (oo\_______
              (__)\       )\/\
                   ||----w |
                   ||     ||
```

## Command

第一步先创建一个符合 [Coomand](https://api.vapor.codes/console/latest/Command/Protocols/Command.html) 的类型。

现在我们来实现必要的方法。

## Arguments

Command 可以有 0个或多个 [CommandArgument](https://api.vapor.codes/console/latest/Command/Structs/CommandArgument.html)。这些参数在命令被执行的时候会用到。

```swift
/// Generates ASCII picture of a cow with a message.
struct CowsayCommand: Command {
    /// See `Command`
    var arguments: [CommandArgument] {
        return [.argument(name: "message")]
    }

    ...
}
```

这里我们定义了一个参数，cow 将会说出 ```message``` 。这需要运行 ```cowsay``` 命令来实现。

## Options

Command 有 0个或多个 [CommandOption](https://api.vapor.codes/console/latest/Command/Structs/CommandOption.html)。这些选项不是必要的，可以通过 ```--``` 或 ```-``` 语法传递进来。

```swift
/// Generates ASCII picture of a cow with a message.
struct CowsayCommand: Command {
    ...
    /// See `Command`
    var options: [CommandOption] {
        return [
            .value(name: "eyes", short: "e", default: "oo", help: ["Change cow's eyes"]),
            .value(name: "tongue", short: "t", default: " ", help: ["Change cow's tongue"]),
        ]
    }
    ...
}
```

这里我们定义了两个选项， ```eyes``` 和 ```tongue``` 。这些使得用户可以改变 cow 的模样。

## Help

接下来我们定义一个可选的 help 消息。用户可以传递 ```help``` 来触发。

```swift
/// Generates ASCII picture of a cow with a message.
struct CowsayCommand: Command {
    ...
    /// See `Command`
    var help: [String] {
        return ["Generates ASCII picture of a cow with a message."]
    }
    ...
}
```

我们来演示下该命令是如何使用的。

```swift
Usage: <executable> cowsay <message> [--eyes,-e] [--tongue,-t] 

Generates ASCII picture of a cow with a message.

Arguments:
  message n/a

Options:
     eyes Change cow's eyes
   tongue Change cow's tongue
```

## Run

最后，需要我们编码来实现：

```swift
/// Generates ASCII picture of a cow with a message.
struct CowsayCommand: Command {
    ...

    /// See `Command`.
    func run(using context: CommandContext) throws -> Future<Void> {
        let message = try context.argument("message")
        /// We can use requireOption here since both options have default values
        let eyes = try context.requireOption("eyes")
        let tongue = try context.requireOption("tongue")
        let padding = String(repeating: "-", count: message.count)
        let text: String = """
          \(padding)
        < \(message) >
          \(padding)
                  \\   ^__^
                   \\  (\(eyes)\\_______
                      (__)\\       )\\/\\
                        \(tongue)  ||----w |
                           ||     ||
        """
        context.console.print(text)
        return .done(on: context.container)
    }
}
```

```CommandContext``` 使得我们可以访问任何需要的内容，包含```Container```。接下来就是去配置它了。

## Config

使用 ```CommandConfig``` 结构体来向你的容器注册 command。这通常是在 ```configure.swift``` 中完成的。

```swift
/// Create a `CommandConfig` with default commands.
var commandConfig = CommandConfig.default()
/// Add the `CowsayCommand`.
commandConfig.use(CowsayCommand(), as: "cowsay")
/// Register this `CommandConfig` to services.
services.register(commandConfig)
```

通过 ```--help``` 来检查命令是否被正确配置。

```
swift run Run cowsay --help
```

就是这样了。

```
$ swift run Run cowsay 'Good job!' -e ^^ -t U
  ---------
< Good job! >
  ---------
          \   ^__^
           \  (^^\_______
              (__)\       )\/\
                U  ||----w |
                   ||     ||
```