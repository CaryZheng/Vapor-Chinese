现在已经安装好了Vapor，可以开始创建第一个Vapor应用了。本篇指南将会一步步指导如何创建项目，编译并且运行起来。



## 新项目

第一步是创建Vapor新项目。这里我们将该项目命名为```Hello```。

打开终端，使用```vapor new```命令。

```
vapor new Hello
```

> 注意：Vapor3目前仍处于Beta阶段，请确保加上```--branch=beta```参数。

一旦完成，进入到新创建的目录里。

```
cd Hello
```

## 生成Xcode项目

使用 Vapor Toolbox 提供的 xcode 命令来生成对应的 Xcode 项目。这将使得我们可以通过Xcode编译运行应用，就像处理 iOS app 一样。

```
vapor xcode
```

Toolbox 将会询问你是否希望自动开启 Xcode，选择 ```yes```将自动打开 Xcode。

## 编译运行

打开 Xcode 后，选择 scheme 菜单中的 Run scheme，然后点击编译运行按钮。

此时，Xcode终端中将会显示如下内容

```
Server starting on localhost:8080
```

## 访问 Localhost

打开浏览器，访问 ```http://localhost:8080/hello```。

将会显示如下内容

```
Hello, world!
```

恭喜，至此已经成功创建、编译并运行了第一个 Vapor 应用。