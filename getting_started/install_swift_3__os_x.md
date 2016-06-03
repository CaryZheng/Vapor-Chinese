# 安装Swift 3 (OS X)
---
安装Vapor，你需要先安装Swift 3。

## 下载
---
如果你安装了Xcode，表明你已经安装了Swift 2。但是要使用Vapor你需要下载Swift 3。


> <font color="#56C0E0"> Swift Version Manager </font>
> 
> 如果你在多个Swift 3项目中工作，你可能需要看看“安装Swift 3 (Swiftenv)”章节。


##### Shell

```
# Download
curl -O https://swift.org/builds/development/xcode/swift-DEVELOPMENT-SNAPSHOT-2016-05-31-a/swift-DEVELOPMENT-SNAPSHOT-2016-05-31-a-osx.pkg

# Install
open swift-DEVELOPMENT-SNAPSHOT-2016-05-31-a-osx.pkg
```

## 安装
---
##### Shell
```
# Add to path
echo "export PATH=/Library/Developer/Toolchains/swift-latest.xctoolchain/usr/bin:\"\${PATH}\"" >> ~/.bash_profile

# Reload path
source ~/.bash_profile
```

## 验证
---
##### Shell
```
swift --version

# Apple Swift version 3.0-dev (LLVM b010debd0e, Clang 3e4d01d89b, Swift 7182c58cb2)
```