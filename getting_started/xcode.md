# Xcode
---
Xcode 使得 Swift 编程更快速更智能。你可以通过 Xcode 编译运行你的 Vapor 工程，也可以使用代码自动补全和 option 点击查看文档功能。


## 生成 Xcode 工程
---
生成并打开 Xcode 工程，执行如下命令：

##### Text
```
vapor xcode
```


## 选择工具链
---
通过 Xcode > Toolchains ，确保你选择了正确的工具链。

![](https://raw.githubusercontent.com/CaryZheng/Vapor-Chinese/master/image/xcode_snapshot_1.png)


Vapor ```0.11``` 使用 ```06-06 (a)``` 工具链。


> 错误的工具链
> 
> Apple 在不同快照之间存在破坏性的改变。如果你使用错误的工具链，Vapor 很有可能会编译不通过。


## 编译运行
---
从 schemes 列表中选择 App ，并且按下 ```Command+R``` 或者点击播放按钮去编译运行你的工程。

![](https://github.com/CaryZheng/Vapor-Chinese/blob/master/image/xcode_snapshot_2.png?raw=true)


如果一切正常，你将在 Xcode 终端看到如下输出：

![](https://raw.githubusercontent.com/CaryZheng/Vapor-Chinese/133c03a9faace40a98153cd23431856f52931ba2/image/xcode_snapshot_3.png)


## 参数
---
你可以通过 Xcode 传递参数到 executable ，就如同使用命令行。

当 ```App``` 被选中的时候，从 scheme 列表选择 Edit Scheme... 。

![](https://github.com/CaryZheng/Vapor-Chinese/blob/master/image/xcode_snapshot_4.png?raw=true)


在那，你可以控制启动的时候哪些参数被传递进去。

![](https://github.com/CaryZheng/Vapor-Chinese/blob/master/image/xcode_snapshot_5.png?raw=true)


## 重要参数
---
Xcode 会编译运行文件系统临时目录下的可执行文件。如果你执行 ```vapor run serve``` ，将得到非预期的结果。

想要避免它，你应该在 Xcode scheme 中包含如下参数。

![](https://github.com/CaryZheng/Vapor-Chinese/blob/master/image/xcode_snapshot_6.png?raw=true)


##### Text
```
serve
--workdir=$(SRCROOT)
```

第一个参数显示执行 ```serve``` 命令。 如果没有提供， ```serve``` 将默认被执行。但以后这可能会发生变化，具体可参考指南中的命令章节。

第二个参数传递了一个 Xcode 变量作为 Vapor 根目录，这将使得所有的视图和其它静态资源能够被找到和渲染。