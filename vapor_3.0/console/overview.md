# Console Overview

本篇指南将简单介绍 Console 模块，讲解如何输出固定格式的文本和请求用户输入。

## Terminal

[Terminal](https://api.vapor.codes/console/latest/Console/Classes/Terminal.html) 是 [Console](https://api.vapor.codes/console/latest/Console/Protocols/Console.html) 协议的默认实现。

```
let terminal = Terminal()
print(terminal is Console) // true
terminal.print("Hello")
```

本篇指南将使用一个通用类型 ```Console```，但是直接使用 ```Terminal``` 也是可以的。你可以使用任意 [Container](https://api.vapor.codes/service/latest/Service/Protocols/Container.html) 来创建 console 。

```
let console = try req.make(Console.self)
console.print("Hello")
```

## Output

[Console](https://api.vapor.codes/console/latest/Console/Protocols/Console.html) 提供了一系列方法用于请求用户输入，最基本的方法是 ```input(isSecure:)``` 。

```
/// Accepts input from the terminal until the first newline.
let input = console.input()
console.print("You wrote: \(input)")
```

## Ask

使用 ```ask(_:)``` 来提示用户输入内容。

```
/// Outputs the prompt then requests input.
let name = console.ask("What is your name?")
console.print("You said: \(name)")
```

将会输出如下内容

```
What is your name?
> Vapor
You said: Vapor
```

## Confirm

使用 ```confirm(_:)``` 来提示用户输入 yes / no 。

```
/// Prompts the user for yes / no input.
if console.confirm("Are you sure?") {
    // they are sure
} else {
    // don't do it!
}
```

将会输出如下内容

```
Are you sure?
y/n> yes
```

> 提示
> 
> ```confirm(_:)``` 将持续提示用户，直到成功响应了 yes 或 no 为止。