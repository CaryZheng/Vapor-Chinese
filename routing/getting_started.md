# Getting Started with Routing

Routing([vapor/routing](https://github.com/vapor/routing))是像HTTP请求路由那样的一个小巧的框架。它允许你注册或者查看一些嵌套或者动态路径的组件。

例如，路由组件可以帮助你完成一个像下面这样的请求，并获得一些动态该组件的值

~~~
/users/:user_id/comments/:comment_id
~~~

# Vapor

Routing package已经被包含在 Vapor中并默认到处，当你导入Vapor时,就可以获得所欲Routing API的调用权限

> Tip：
> 如果你使用Vapor，大多数路由API会被包装一层更方便的方法。你可以通过 [Vapor -> Routing](../)来了解更多信息

~~~swift
import Vapor
~~~

# 独立功能

Routing 是一个轻量级，少依赖的Swift组件库。这意味着你可以在不引入Vapor的情况下，在任何Swift项目中使用这个库。

引入Routing到你的项目中，可以添加下面代码到你的 **Package.swift**文件中

~~~swift
// swift-tools-version:4.0
import PackageDescription

let package = Package(
    name: "Project",
    dependencies: [
        ...
        .package(url: "https://github.com/vapor/routing.git", from: "3.0.0"),
    ],
    targets: [
      .target(name: "Project", dependencies: ["Routing", ... ])
    ]
)
~~~

通过 **import Routing** 来获得API调用权限

> 警告：
> 这份文档会包含一些vapor特有API，但是大部分情况下适用于大多数路由，你可以通过[API文档](https://api.vapor.codes/routing/latest/Routing/index.html)来查看Routing的API信息

