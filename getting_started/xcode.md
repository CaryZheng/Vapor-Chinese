# Xcode
---
Xcode使得Swift编程更快速更智能。你可以通过Xcode编译运行你的Vapor工程，也可以使用代码自动补全和option点击查看文档功能。


## 生成Xcode工程
---
生成并打开Xcode工程，执行如下命令：

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
> Apple在不同快照之间存在破坏性的改变。如果你使用错误的工具链，Vapor很有可能会编译不通过。


## 编译运行
---
从schemes列表中选择App，并且按下 ```Command+R``` 或者点击播放按钮去编译运行你的工程。

![](https://github.com/CaryZheng/Vapor-Chinese/blob/master/image/xcode_snapshot_2.png?raw=true)


如果一切正常，你将在Xcode终端看到如下输出：