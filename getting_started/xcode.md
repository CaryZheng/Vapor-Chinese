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


Vapor ```0.9``` 使用 ```05-09 (a)``` 工具链。


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