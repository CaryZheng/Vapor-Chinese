# 安装Swift 3: Swiftenv
---
安装Vapor，你需要先安装Swift 3。


## Swiftenv
---
Swiftenv 能让你方便地维护计算机上的多个Swift 3版本。详情请见 [Swiftenv's GitHub](https://github.com/kylef/swiftenv) 。


## 通过 Homebrew 安装
---
我们将讲述如何通过 Homebrew 来安装 Swiftenv ，因为这是最简便的方法。如果你想用另一种方式来安装，请参考 [Swiftenv README](https://github.com/kylef/swiftenv) 。

##### Shell
```
brew install kylef/formulae/swiftenv
```

现在把它添加到你的 PATH 。

##### Text
```
# OS X
echo 'if which swiftenv > /dev/null; then eval "$(swiftenv init -)"; fi' >> ~/.bash_profile
source ~/.bash_profile
 
# Ubuntu
echo 'if which swiftenv > /dev/null; then eval "$(swiftenv init -)"; fi' >> ~/.bashrc
source ~/.bashrc
```

这将确保任何时候启动一个新终端 Swiftenv 都是有效的。当你输入一个命令，如 ```swift build``` ，它将自动为当前项目选用合适的 Swift 版本。

>  <font color="#56C0E0"> Swift Version Files </font>
>  
> 名为 ```.swift-version``` 的隐藏文件存在于很多 Swift 3 项目中，这意味着 Swiftenv 将使用对应的 Swift 版本用于编译。


## 安装 Swift
---

安装了```swiftenv``` 后，就可以为 Vapor 安装合适的 Swift 版本。

##### <font color="#91A7D3"> Text </font>
```
swiftenv install DEVELOPMENT-SNAPSHOT-2016-06-06-a
swiftenv global DEVELOPMENT-SNAPSHOT-2016-06-06-a
```

如上命令将安装该快照版本，若当前项目中 ```.swift-version``` 文件不存在，该版本将作为默认的 Swift 版本。